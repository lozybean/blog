Title: 标记VCF文件FILTER
Date: 2016-10-27
Category: bioinformatics
Tags: bioinformatics, vcf, filter, pyvcf, safety eval

## 前文

本文将会涉及到的内容有:

1. 使用gatk对VCF文件进行FILTER标记;
2. 使用PyVCF处理VCF文件的基本操作; 
3. 使用eval执行表达式参数;
4. 根据filterName自动识别record类型, 决定是否标记FILTER
5. 如何安全地使用eval;

## 使用gatk标记VCF FILTER以及存在的问题

使用gatk工具集中的`VariantFiltration`, 可以利用表达式参数来进行FILTER的标记, 如:

```bash
java -jar gatk.jar -T VariantFiltration \
-R genome.fasta -V calling.vcf -o maked.vcf \
-filter "QD<2.0||FS>60.0||SOR>4.0||MQ<40.0||MQRankSum<-12.5||ReadPosRankSum<-8.0" \
-filterName "SNP_filter" \
-filter "QD<2.0||FS>200.0||SOR>10.0||InbreedingCoeff<-0.8||ReadPosRankSum<-20.0" \
-filterName "INDEL_filter"
```

通过传入表达式语句(简单的java代码), 对VCF文件中的每一个通过判断的record, 将FILTER标记为对应的"filterName". 具体参考[gatk官方文档](https://software.broadinstitute.org/gatk/gatkdocs/org_broadinstitute_gatk_tools_walkers_filters_VariantFiltration.php).

这个工具非常方便, 但是存在一些问题:

1. 程序内部问题: 判断的值类型要和VCF文件描述中的相同, 比如, 若`##INFO=<ID=QD,Type=Float`将QD描述为浮点型, 则必须要使用`QD<2.0`这样的判断, 若使用`QD<2`则会出错.
2. 使用过程问题: 当一个VCF文件中, 同时存在SNP和INDEL时, 这两个"filter"是一起执行, 并且不会根据"filterName"来自动区分SNP和INDEL的. 意识到这一点非常重要, 因为SNP和INDEL的过滤标准不一样, 如一个INDEL的record, 其`ReadPosRankSum=-10.0`时, 该工具会将这条record标记为"SNP_filter", 而根据INDEL的标准, 该record其实应该为PASS, 虽然后期可以根据record的类型具体判断, 但是这样会容易引起混淆.

## 使用PyVCF进行FILTER的标记

为了上述两个问题, 可以尝试自己编写python程序来进行FILTER标记, 其中会用到PyVCF这个优秀的python库来处理VCF格式的文件.

### 在VCF文件中添加FILTER描述

在VCF文件的头部, 有许多`##`开头的头信息, 这些信息并不是简单的注释, 而是对VCF文件中每一项的具体描述, 指导对该文件的解析.

当我们添加FILTER标记时, 也应该在VCF的文件头中添加相应的FILTER描述.

PyVCF的VCFReader类中, 其`filter`属性(一个OrderedDict)指定了目前的filter, 每一个filter都是`_Filter`的`namedtuple`, 需要在其中添加进我们需要添加的filter:

```python
from vcf import VCFReader, VCFWriter
from vcf.parser import _Filter

def filter_marker(vcf_in, vcf_out, filters):
    """
    filters: a dict like {filterName: filterString}
    """
    with open(vcf_in) as fp_in, open(vcf_out, 'w') as fp_out:
        vcf_reader = VCFReader(fp_in)
        for k, v in filters.items():
            vcf_reader.filters.update({
                k: _Filter(id=k, desc=v)
            })
        vcf_writer = VCFWriter(fp_out, vcf_reader)
```

在上述代码最后, VCFWriter类传入的第二个参数指定了输出VCF文件中的头信息使用更新后的头信息.

### 执行语句字符串

在python中, 可以利用`eval`来对一个语句进行直接执行表达式:

```python
eval(code, globals=None, locals=None)
```

注意上述的globals和locals并不是key-value的参数, 而是必须通过位置参数传入.

由于eval只能执行表达式(expression), 但是并不能执行赋值(assignment)语句(如: `a=1`)以及声明(statement)语句(如: if语句).

当然, **eval并不是安全的, 很容易带来代码注入攻击, 所以不应该暴露给外部人员使用**.

了解了上述问题之后, 可以使用以下语句, 来直接执行表达式的判断:

```
eval("QD<2.0 or FS>200.0 or SOR>10.0 or InbreedingCoeff<-0.8 or ReadPosRankSum<-20.0", {}, record.INFO)
```

其中, record.INFO指定了局部作用域, 可以在其中获得需要比较的量;

### 添加默认值

只使用record.INFO作为局部变量是不够的, 当比较的值不出现在record.INFO中时, 可以在全局作用域中添加一些常用的指标.

首先, 需要定义一个无穷大的对象, 以便作为默认值来比较:

```python
class Inf(object):
    def __lt__(self, other):
        return False

    def __le__(self, other):
        return False

    def __gt__(self, other):
        return True

    def __ge__(self, other):
        return True

    def __eq__(self, other):
        if isinstance(other, Inf):
            return True
        else:
            return False
            
    def __neg__(self):
        return NInf()

class NInf(object):
    def __lt__(self, other):
        return True

    def __le__(self, other):
        return True

    def __gt__(self, other):
        return False

    def __ge__(self, other):
        return False

    def __eq__(self, other):
        if isinstance(other, Inf):
            return True
        else:
            return False
    
    def __neg__(self):
        return Inf()
```

然后添加一些常用指标:

```python
inf = Inf()
default_values = {
    'QUAL': record.QUAL,
    'DP': inf,
    'MQ': inf,
    'QD': inf,
    'FS': -inf,
    'SOR': -inf,
    'MQRankSum': inf,
    'ReadPosRankSum': inf,
    'InbreedingCoeff': inf,
}

eval("QD<2.0 or FS>200.0 or SOR>10.0 or InbreedingCoeff<-0.8 or ReadPosRankSum<-20.0", default_values, record.INFO)
```

当局部作用域(record.INFO)中找不到相应的属性时, 会自动去全局作用域(default_values)中找到默认的属性;

### 自动判断record类型

通过给filterName命名时, 使用"SNP"或者"INDEL"开头, 以便实现仅仅当record是其中某种类型时, 才执行FILTER的标记.

在PyVCF中, 可以直接通过record的`is_snp`和`is_indel`等属性判断record的类型:

```python
from vcf import VCFReader, VCFWriter
from vcf.parser import _Filter

def filter_marker(vcf_in, vcf_out, filters):
    """
    filters: a dict like {filterName: filterString}
    """
    with open(vcf_in) as fp_in, open(vcf_out, 'w') as fp_out:
        vcf_reader = VCFReader(fp_in)
        for k, v in filters.items():
            vcf_reader.filters.update({
                k: _Filter(id=k, desc=v)
            })
        vcf_writer = VCFWriter(fp_out, vcf_reader)
        for record in vcf_reader:
            default_values = {
                'QUAL': record.QUAL,
                'DP': inf,
			    'MQ': inf,
			    'QD': inf,
			    'FS': -inf,
			    'SOR': -inf,
			    'MQRankSum': inf,
			    'ReadPosRankSum': inf,
			    'InbreedingCoeff': inf,
			}
            if record.FILTER is None:
                record.FILTER = []
            for k, v in filters.items():
                if k.startswith('SNP') and not record.is_snp:
                    continue
                if k.startswith('INDEL') and not record.is_indel:
                    continue
                judgement = eval(v, default_values, record.INFO)
                if judgement:
                    record.FILTER.append(k)
            vcf_writer.write_record(record)
```

通过filterName和record实际类型的判断, 可以避免gatk的VariantFiltration对同时存在snp和indel的vcf文件进行FILTER标记时可能引入的混淆.

### *安全地使用eval

如果需要对外暴露函数的话, eval是不够安全的, 所以非常有必要对传入的expression进行安全检查.

这里会使用到内置函数`compile`, 并且设置为ast.PyCF_ONLY_AST模式, 然后遍历编译后的每个节点, 检查其安全性.

由于在FILTER标记时, 只会用到大小判断的表达式, 所以在整个表达式中, 除了一些操作语句之外, 只可能出现Num和Name.

当出现这几种之外的元素时, 就应该判断为不安全的表达式, 并报错:

```python
from ast import *

def check_eval(self, node_or_string):
    if isinstance(node_or_string, str):
        node_or_string = compile(node_or_string, '', 'eval', PyCF_ONLY_AST)
    if isinstance(node_or_string, Expression):
        node_or_string = node_or_string.body
    if isinstance(node_or_string, BoolOp):
        for value in node_or_string.values:
            self.check_eval(value)
    elif isinstance(node_or_string, Compare):
        self.check_eval(node_or_string.left)
        for comparator in node_or_string.comparators:
            self.check_eval(comparator)
    elif isinstance(node_or_string, UnaryOp):
        self.check_eval(node_or_string.operand)
    elif isinstance(node_or_string, BinOp):
        self.check_eval(node_or_string.left)
        self.check_eval(node_or_string.right)
    elif isinstance(node_or_string, Subscript):
        self.check_eval(node_or_string.value)
    elif isinstance(node_or_string, (Name, Num)):
        # only Name or Num base element is allowed
        pass
    else:
        raise ValueError("unsupported expression!")
    return None
```

表达式通过上述代码中描述的严格检查之后, 再进行`eval`运算, 从而保证程序的安全.

### 最终实现

除了上述内容之外, 还需要:

1. 加上genotypeFilter之类的特性, 让脚本和gatk的VariantFiltration的调用表现等尽量一致;
2. 添加自定义的默认值参数, 使得脚本可用性更强;

完成这些功能之后, 就可以使用自己编写的脚本来替代gatk的VariantFiltration, 更好地帮助日常工作.