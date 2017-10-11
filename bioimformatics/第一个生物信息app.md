Title: 第一个生物信息app
Date: 2016-05-30
Category: bioinformatics
Tags: pyqt, bioinformatics, mongodb

## 引子

本文将介绍一个桌面应用的创建全过程, 展示生物信息除了拒人于千里之外命令行界面之外, 对无IT经验的用户更加友好的展示方式;

设计环境以及技术:

* CentOS6.7(服务端)
* Windows10(客户端)
* Python3.5
* PyQt5.6
* Mongodb3
* pymongo3
* pyinstaller3.2

最终完成的功能:

* 根据基因名称, 检索所有的转录本编号, 以及对应的外显子的长度和数量

## 数据获取

首先从[NCBIFTP站点](ftp://ftp.ncbi.nlm.nih.gov/genomes/H_sapiens/ARCHIVE/BUILD.37.3/GFF/ref_GRCh37.p5_top_level.gff3.gz)下载准备文件:*ref_GRCh37.p5_top_level.gff3.gz*, 参考[之前的文章](http://www.lyon0804.com/linuxwen-jian-xia-zai-fang-shi-hui-zong.html)的方式来下载该文件到服务器.

该文件中包含人类基因组GRCh37版本的注释信息, 利用该信息可以检索到每个基因、转录本、外显子、CDS等情况;

预览该文件可以发现, 除了注释行之外, 每行开头有**NC_**、**NM_**等标记, 这些标记是NCBI的genbank标记, 其具体意义为:

* `AC_`(Genomic): Complete genomic molecule, usually alternate assembly
* `NC_`(Genomic): Complete genomic molecule, usually reference assembly
* `NG_`(Genomic): Incomplete genomic region
* `NT_`(Genomic): Contig or scaffold, clone-based or WGS
* `NW_`(Genomic): Contig or scaffold, primarily WGS
* `NS_`(Genomic): Environmental sequence
* `NZ_`(Genomic): Unfinished WGS
* `NM_`(mRNA): normal used
* `NR_`(RNA): normal used
* `XM_`(mRNA): Predicted model
* `XR_`(RNA): Predicted model
* `AP_`(Protein): Annotated on AC_ alternate assembly
* `NP_`(Protein): Associated with an NM_ or NC_ accession
* `YP_`(Protein)
* `XP_`(Protein): Predicted model, associated with an XM_ accession
* `ZP_`(Protein): Predicted model, annotated on NZ_ genomic records

其中**NC_**开头的序列通常作为标准参考, NC_000001到NC000024分别表示24条染色体(包含XY性染色体), 是我们的重点关注对象;

## 创建数据库

为了方便检索信息, 需要选择一款数据库软件, 这里选择Mongodb来完成这次设计, mongodb的安装不再赘述;

在完成Mongodb的安装之后, 别忙着启动数据库, 先编辑mongodb.conf配置, 确保修改以下内容:

```
bind_ip = 0.0.0.0
port = 27017
fork = true
auth = true
```

然后使用`mongod -f mongodb.conf`启动数据库服务;

并且在本地启动一个mongo shell: `mongo 127.0.0.1:27017`以便配置用户信息:

```javascript
# 添加管理员用户
use admin
db.addUser('root', '123')

# 创建reference数据库
use reference
# 添加只读用户
db.addUser('user', '123', true)
```

以上配置为了客户端和服务器的权限分离, 防止客户端修改数据库;

利用一个小脚本, 将我们需要的信息添加到数据库中:

```python
import gzip
from pymongo import MongoClient

client = MongoClient('127.0.0.1', 27017)
db = client.reference
db.authenticate('root', '123')

with gzip.open('ref_GRCh37.p5_top_level.gff3.gz', 'rt') as fp:
    for line in fp:
        if not line.startswith('NC'):
            continue
        (assembly, _, type_, start, end, _,
         strand, _, items) = line.rstrip().split('\t')
        chr_ = 'chr%d' % int(assembly.split('.')[0][3:])
        if type_ == 'exon' or type_ == 'mRNA':
            item_dict = {
                'chr': chr_,
                'assembly': assembly,
                'start': int(start),
                'end': int(end),
                'strand': strand,
            }
        else:
            continue
        for item in items.split(';'):
            key, value = item.split('=')
            if key == 'Dbxref':
                result_value = {}
                for subitem in value.split(','):
                    subkey, subvalue = subitem.split(':')
                    result_value[subkey] = subvalue
                value = result_value
            item_dict[key] = value
        db[type_].insert(item_list)

db.exon_info.create_index([('transcript_id', 1)])
db.mRNA.create_index([('gene', 1)])
```

## 连接数据库查询

搭建数据库服务器的工作完成之后, 就需要进行本地windows环境的开发工作, 首先要实现的是内核部分的查询操作:

1. 首先, 我们需要根据基因名称, 列出所有的转录组编号;
2. 根据上一步查询到的转录组信息, 列出转录组编号为上述转录组的所有外显子信息;

得益于我们创建数据库时的索引方式, 上述查询非常简单快速, 其基本代码为:

```python
from pymongo import MongoClient
client = MongoClient('192.168.6.4', 27017)
db = client.reference
db.authenticate('user', '123')
mRNA_info = db.mRNA_info
exon_info = db.exon_info
gene_name = 'BRCA1'
cursor_mRNA = mRNA_info.find({'gene': gene_name}, {'Name': 1, '_id': 0})
for q in cursor_mRNA:
    mRNA = q['Name']
    exon_length = 0
    exon_num = 0
    cursor_exon = exon_info.find({'transcript_id': mRNA}, {'start': 1, 'end': 1, '_id': 0})
    for q in cursor_exon:
        exon_length += q['end'] - q['start'] + 1
        exon_num += 1
	 print(gene_name, mRNA, exon_length, exon_num, sep='\t')
```

## PyQt设计界面

至此, 核心任务已经完成, 但是达到文首的目的, 所以还需要再设计一个桌面应用, 我选择使用PyQt5来完成GUI部分的工作;

首先, 利用QtDesigner, 设计一个粗糙简单的界面:

![QtDesigner设计结果](http://ww3.sinaimg.cn/large/95202659gw1f4dffbs4vmj21hc0sx0wv.jpg)

由于本人艺术细胞缺乏, 就只弄一个粗糙的界面了;

保存为: main.ui, 再使用以下命令将其转换为ui模块:

```python
python -m PyQt5.uic.pyuic main.ui -o my_design.py
```

生成的my_design.py中就包含了我们设计的界面信息;

## 添加界面逻辑

接下来要做的工作就是将界面和核心代码融合, 首先我们要确定界面中信号的传递方式:

1. 首先通过按钮来获得查询信号;
2. 然后从输入栏中获得基因名称;
3. 再利用该基因名称获取查询结果并格式化为字符串;
4. 将字符串展示到展示框中;

详细代码为:

```python
import sys
from pymongo import MongoClient
from PyQt5.QtWidgets import *
from my_design import Ui_MainWindow

client = MongoClient('192.168.6.4', 27017)
db = client.reference
db.authenticate('user', '123')
mRNA_info = db.mRNA_info
exon_info = db.exon_info


class MyWindow(QMainWindow, Ui_MainWindow):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.pushButton.clicked.connect(self.show_search_result)
    
    def show_search_result(self):
        gene_name = self.lineEdit.text()
        search_result = self.search(gene_name)
        self.textBrowser.setText(search_result)
    
    def search(self, gene_name):
        result = ''
        cursor_mRNA = mRNA_info.find({'gene': gene_name}, {'Name': 1, '_id': 0})
        for q in cursor_mRNA:
            mRNA = q['Name']
            exon_length = 0
            exon_num = 0
            cursor_exon = exon_info.find({'transcript_id': mRNA}, 
                                         {'start': 1, 'end': 1, '_id': 0})
            for q in cursor_exon:
                exon_length += q['end'] - q['start'] + 1
                exon_num += 1
            result += '{gene_name}\t{mRNA}\t{exon_length}\t{exon_num}\n'.format_map(vars())
        return result


if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MyWindow()
    window.show()
    sys.exit(app.exec_())
```

## 打包发布

到这里基本工作已经完成, 但是目前只是在开发环境下运行, 如果要release到其他机器上, 需要对依赖的库进行打包;

使用**pyinstaller**工具可以非常方便地完成这项工作;

1. 首先创建一个打包后的目标目录, 如: app;
2. 进入该目录, 运行: pyinstaller -w script_name.py; (主程序文件替换为完整目录)
3. 等待5分钟左右, 即可在该目录下看到两个文件夹: build、dist, 其中dist目录下的exe文件即最终打包好的文件;

然而普通的电脑上运行这个exe文件还是会有报错: 

```
could not be located in the dynamic link library
api-ms-win-crt-runtime-|1-1-0.dll
```

这是因为运行的机器上没有安装vc++ 2015的依赖库, 去以下地址下载并且安装即可: 

* [64位下载地址](http://download.microsoft.com/download/9/3/F/93FCF1E7-E6A4-478B-96E7-D4B285925B00/vc_redist.x64.exe)
* [32位下载地址](http://download.microsoft.com/download/9/3/F/93FCF1E7-E6A4-478B-96E7-D4B285925B00/vc_redist.x86.exe)

双击运行, 并输入BRCA1基因测试:

![BRCA1 result](http://ww2.sinaimg.cn/large/95202659gw1f4dfx7y7nfj20m90hj0sz.jpg)

## 后记

至此所有工作已经完成了, 但是这个软件并不完美(不仅仅是界面), 后续可以往上面添加更多的内容, 如异常处理, 通过基因别名查询, 更多的查询方式等等功能;
