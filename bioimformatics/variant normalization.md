Title: [翻译]Variant Normalization
Date: 2016-07-13
Category: bioinformatics
Tags: bioinformatics, variant, normalization, vcf

原文链接: http://genome.sph.umich.edu/wiki/Variant_Normalization

## 介绍

VCF(Variant Call Format)格式文件是一种灵活的格式, 这种格式可以用来表示从SNPs、INDELs到CNVs等很多种变异类型. 然而, VCF文件中的变异表达在参考型和突变型的表示上具有不唯一性. 一次失败的识别很容易造成错误的分析.

这篇Wiki描述了变异标准化(variant normalization)的过程, 该过程不仅适用于双等位基因同时适用于多等位基因. 然后我们对该过程的正确性给出了证明.

## 定义

VCF中的变异标准化涉及两个方面: 

* 变异长度上的**简约性**(parsimony)
* 变异位置**左对齐**(left alignment)

### 简约性(Parsimony)

在变异表达中, 简约性代表用尽可能少但是不为0的核苷酸数量来表示每一个等位基因. 它描述了变异的长度属性, 定义如下:


> 当且仅当一个变异使用了尽可能少的核苷酸, 并且没有一个等位基因的长度为0时, 称该变异为简约的.

并定义:

> 如果每个等位基因的最左侧核苷酸相同, 且移除该核苷酸并不会导致空等位基因, 则称该变异在左侧有多余核苷酸.

如下实例中, 多核苷酸多态性在前三种表示上都有多余核苷酸, 而第四种表示上则是简约的. 当一个变异在左侧具有多余核苷酸时, 称该变异为左侧未简约, 并需要进行左切除. 同理, 右侧未简约也需要进行由切除. 简约性在INDELs一样体现, 在左对齐章节将做示范.

![parsimony](http://genome.sph.umich.edu/w/images/9/98/Normalization_mnp.png)

图中展示了多核苷酸多态性的多种展示, 最左列用颜色区分了四种可能存在的不同表示方式. 右边一列展示了在VCF文件中的相应表示. 最后一列展示了四种方式的简约性表现.

在简约性的定义的基础上, 容易看到:

> 如果一个变异是非简约的, 则所有的等位基因长度必须大于1.

逆命题不成立.

### 左对齐(Left alignment)

将一个变异左对齐表示将该变异的起始位置尽可能地向左移动. 这是插入(insertion)缺失(deletion)中的相关概念, 描述了变异的位置属性. 为了区分左对齐和简单的对一个变异左填充, 给出定义如下:

> 在保持所有等位基因长度的前提下, 当且仅当一个变异不可能再往左移动时, 称该变异为左对齐的.

下图展示了一个没有标准化的短串联重复序列(一种特殊的indel). 并且用和图中颜色对应的文字对变异进行描述.

* <span style="color:#ff0000">VCF文件要求任意一个等位基因都不能以空字符串的形式表示(空等位基因). 红色的indel是非法的VCF表示方式.</span>
* <span style="color:#008000">绿色的变异并不是左对齐的, 你可以在该变异的每个等位基因左边加上一个核苷酸, 并移除每个等位基因右边的C. 但该变异是简约的.</span>
* <span style="color:#F9A908">橙色的变异是左对齐的, 但是右侧并不简约.</span>
* <span style="color:#0000FF">蓝色的变异是左对齐的, 但是左侧并不简约.</span>
* <span style="color:#800080">褐红色的变异是左对齐并且简约的.</span>

![left_alignment](http://genome.sph.umich.edu/w/images/5/53/Normalization_str.png)

图中展示了CA微卫星(Short Tendem Repeat)的多种表示方式, 最左列用颜色区分了五种可能存在的不同表示方式. 右边一列展示了在VCF文件中的相应表示. 最后一列展示了该微卫星的五种方式的左对齐以及简约性表现.

## 区分是否标准化

> 当且仅当一个变异是简约且左对齐时, 称该变异是标准化的.

### 引论

为了检测一个变异是否标准化, 我们首先需要证明以下引论(1):

> 当且仅当变异是非左对齐或者非右简约时, 该变异的所有等位基因都以同样的核苷酸结尾.

从**=>**方向证明:

* 如果一个indel是左对齐并且右侧简约时, 该变异的所有等位基因不以同样的核苷酸结尾.

首先假设一个indel已经左对齐并且右侧简约. 当任一等位基因长度大于1时, 由于该indel是右侧简约的, 很显然每个等位基因不能以同样的核苷酸结尾. 当其中某一等位基因长度为1时, 并假设所有的等位基因都以A结尾. 此时, 为了避免出现空等位基因的情况, 右侧已经不再能去除相同的A核苷酸, 故右侧仍然是简约的. 在任意等位基因左侧延长一个核苷酸(复制参考基因组), 此时在右侧就出现了多余的核苷酸. 将该核苷酸去除, 结果导致indel的每个等位基因在保持长度不变的前提下向左移动了一个位置, 这和该indel是左对齐矛盾. 因此, 所有的等位基因都不能以相同的核苷酸结尾.

从**<=**方向证明:

* 如果一个indel是非左对齐或者非右简约时, 该变异的所有等位基因都以同样的核苷酸结尾.

假设该变异是非左对齐的, 则对该变异进行左对齐操作时, 为了保证长度不变, 则每移动一个核苷酸就必须从右侧移除一个相同的核苷酸, 因此该变异右侧至少存在一个相同的核苷酸, 否则无法进行左对齐操作.

假设该变异是非右简约的, 根据定义可知, 该变异的所有等位基因中, 最右侧的碱基都必须相同以期望被移除.

### 必然性

> 一个变异是标准化的当且仅当:
> 1. 在左侧没有多余的碱基;
> 2. 任意一个等位基因都以不同的核苷酸结尾;

证明:

一个变异是标准化的当且仅当该变异简约并且左对齐

=> 一个变异是标准化的当且仅当该变异左简约并且右简约并且左对齐

=> 一个变异是标准化的当且仅当该变异左侧没有多余核苷酸并且右侧没有多余核苷酸并且左对齐

=> 一个变异是标准化的当且仅当该变异左侧没有多余的核苷酸并且所有等位基因不以相同的核苷酸结尾 (引论1)

### 唯一性

变异的标准化得到唯一的变异表达是非常重要的, 在我们证明之前, 先凭直觉接受: 一个变异的任意表达都可以通过同时在等位基因一端添加参考核苷酸或者同时在等位基因的一端移除相同的核苷酸, 从而得到另外一种表达.

现在假设有变异的两个标准化A和B. 不失一般性, 假设B在A的右侧, 由于标准化的定义, A和B都是左对齐的, 如果A和B不在同一个位置则意味着B可以通过向左对齐变成A(由于A和B表示了相同的变异), 与B是标准化矛盾, 所以A和B必然在相同的位置.

假设A和B虽然在同一个位置, 但是长度不同, 不失一般性认为B比A长, 则B肯定是非简约并且可以通过去除核苷酸达到和A相同的长度, 与B标准化矛盾, 所以A和B长度必然相同.

由于A和B必然在同一个位置并且长度相同, 故变异的标准化表示是唯一的.

## 实施

[vt](http://genome.sph.umich.edu/wiki/Vt#Normalization)中已经得到了实施.

### 标准化的算法

现在我们知道了如何区分一个变异是否标准化, 我们只需要操作变异表达方式, 直到最后侧的核苷酸不同并且去除左侧多余的核苷酸, 即可得到一个标准化后的变异表达. 标准化双等位基因和多等位基因的算法表示如下:

![algorithm](http://genome.sph.umich.edu/w/images/9/9b/Variant_normalization_algorithm.png)

1到8行进行了左对齐操作并且保证右侧简约. 9到11行保证左侧的简约.

### 比较

#### 2014年5月20号

下面的表格展示了对一个不公开数据进行标准化时, 正确标准化的变异数量. 该比较在2014年5月20号完成.

| Dataset  | bcftools | gatk | vt | comments   |
--------- | :------: | ---: | --: | ----------------------------------------------: |
 #normalized| - |	18794  | 18849 | bcftools的标准化存在bug, 会去除不同的核苷酸.|
 #normalized after bcftools | - | - | -  | - |
 #normalized after gatk     | - | 0 | 57 | 57个gatk的标准化结果被vt重新左对齐. 其中有6个是双等位基因, 51个是多等位基因. 注意有两个变异被gatk修改但是没有完全标准化 |
 #normalized after vt       | - | 0 | 0  | vt的标准化结果没有被进一步标准化 |

使用的命令为:

* bcftools norm -f ref.fa in.vcf -O z > out.vcf.gz
* java -jar GenomeAnalysisTK.jar -T LeftAlignAndTrimVariants --trimAlleles -R ref.fa --variant in.vcf.gz -o out.vcf.gz
* vt normalize -r ref.vcf.gz -o out.vcf.gz

使用的版本为:

* bcftools v0.2.0-rc8-5-g0e06231 (using htslib 0.2.0-rc8-6-gd49dfa6)
* GATK v3.1-1-g07a4bf8
* vt normalize v0.5

问题已经在2015年5月20号传达给bcftools和gatk

#### 2014年5月22号

下面的表格展示了对一个不公开数据进行标准化时, 正确标准化的变异数量. 该比较在2014年5月22号完成. bcftools更新了.

| Dataset  | bcftools | gatk | vt | comments   |
--------- | :------: | ---: | --: | ----------------------------------------------: |
 #normalized| 18849 |	18794  | 18849 | bcftools的标准化和vt已经使用相同的算法. |
 #normalized after bcftools | 0 | 0 | 0  | bcftools的标准化结果没有被进一步标准化 |
 #normalized after gatk     | - | 0 | 57 | gatk的写了只使用双等位基因的文档. 57个gatk的标准化结果被vt重新左对齐. 其中有6个是双等位基因, 51个是多等位基因. 注意有两个变异被gatk修改但是没有完全标准化 |
 #normalized after vt       | - | 0 | 0  | vt的标准化结果没有被进一步标准化 |	

使用的命令为:

* bcftools norm -f ref.fa in.vcf -O z > out.vcf.gz
* java -jar GenomeAnalysisTK.jar -T LeftAlignAndTrimVariants --trimAlleles -R ref.fa --variant in.vcf.gz -o out.vcf.gz
* vt normalize -r ref.vcf.gz -o out.vcf.gz

使用的版本为:

* bcftools v0.2.0-rc8-5-g0e06231 (using htslib 0.2.0-rc8-6-gd49dfa6) [updated non release development version]
* GATK v3.1-1-g07a4bf8
* vt normalize v0.5

## 一个该标准化算法失效的例子

我们用下面的规则区分变异的标准化(normalization)和分解/组合(decomposition/reconstruction):

标准化代表着将一个变异的多个表达降低为一个权威的表达. 标准化可以适用于双等位基因和多等位基因. 变异标准化问题已经被解决, 并且有一个唯一的表示方式(左对齐并且简约的表达). 数学证明已经发表.[1](http://bioinformatics.oxfordjournals.org/content/suppl/2015/02/19/btv112.DC1/VtNormApplicationNote_supp_20141113_1346.pdf)

变异的分解代表着将一个变异的表达拆分成多个变异. 这个分解可以是纵向的 —— 将一个多等位基因分解成双等位基因. 也可以是横向的 —— 将一个包含多个indel和snp群的复合变异分解成多个单一变异. 横向分解常常不仅有一个结果. 相似地, 将多个变异组合重构成一个变异也有横向和纵向两种方式. 讲一个复合变异纵向分解是一个多对一的函数, 将多个双等位基因组合为一个复合变异则并不具有唯一解, 需要考虑到所有等位基因可能的排列.

如果你的例子中包含了变异的分解和组合, 那么你就有可能发现前后不一致.

区分标准化和分解组合的不同点是非常重要的. 标准化的概念意味着变异有标准表示格式. 如果在你的标准化概念里加入分解和组合, 由于可辨别性的内在差异(是否具有唯一解等), 你将会得到前后不一致的情况.

当进行分解组合操作时, 我认为应当考虑以下几点:

* 你的变异仅仅用来表示单个个体还是一个群体？
* 这些基因型是否在独立的阶段产生?

在不同的情况下可能得到不同的答案.

[An example of inconsistent variant representation due to using vt normalize](https://github.com/atks/vt/issues/16)

## 引用

[Adrian Tan, Gonçalo R. Abecasis and Hyun Min Kang. (2015) Unified Representation of Genetic Variants. Bioinformatics.](http://bioinformatics.oxfordjournals.org/content/31/13/2202)

## 维护者

该页面由[Adrian](atks@umich.edu)维护.

## 申明

原文地址: http://genome.sph.umich.edu/wiki/Variant_Normalization

本文由本人(Lyon0804)翻译并发布, 如果侵犯原文权利请告知.

转载请注明本文地址: http://www.lyon0804.com/fan-yi-variant-normalization.html