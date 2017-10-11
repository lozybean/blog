Title: [翻译]LiftOver
Date: 2016-09-21
Category: bioinformatics
Tags: bioinformatics, LiftOver

原文链接: http://genome.sph.umich.edu/wiki/LiftOver

## 介绍

LiftOver是实现统一遗传学分析的参考基因组版本的重要一步. LiftOver可能应用于三种情况:

1. 将基因组位置从一个基因组版本转换到另外一个版本;
	最常见的一个情景是, 我们有NCBI build 36版本的基因组位置(UCSC hg18), 然后想要把这些位置转变到NCBI build 37版本(UCSC hg19).
2. 将dbSNP的rs号从一个版本转换到另外一个版本;
3. 将基因组位置和dbSNP的rs号同时在不同的版本之间转换;

我们将会看到一些诸如Merlin/PLINK格式的数据.

我们将解释上述三种应用的工作流程. 在文章的剩余部分, 将给出从低版本到高版本转换的例子, 这也是普遍的实践.

使用不同的工具可以使liftOver变得简单. 比如UCSC的liftOver工具就可以用来在不同的基因组版本之间转换[BED格式](http://genome.ucsc.edu/FAQ/FAQformat.html#format1)的文件. 使用我们自己的脚本, 我们还可以转换rs号和Merlin/PLINK的文件.

## 转换基因组位置

基因组位置使用[BED格式](http://genome.ucsc.edu/FAQ/FAQformat.html#format1)有最好的表示方式. UCSC提供工具, 用来转换从一个基因组版本到另外一个版本的BED文件.

### 二进制liftOver工具

我们需要UCSC的[二进制liftOver](http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/liftOver)工具, 以及[hg18到hg19的chain file](http://hgdownload.cse.ucsc.edu/goldenPath/hg18/liftOver/hg18ToHg19.over.chain.gz).

提供BED格式的文件(如: input.bed)

注意: 在染色体名字前加上'chr'

```
chr1	743267	743268	rs3115860
chr1	766408	766409	rs12124819
chr1	773885	773886	rs17160939
```

运行liftOver:

```
liftOver input.bed hg18ToHg19.over.chain.gz output.bed unlifted.bed
```

unlifted.bed文件将会包含所有不能被转换的基因组位置. 原因有很多种. 失败的原因将在后文说明.

### 网络接口

另外一种选择是, 可以使用下面的网络接口来转换BED文件: [连接](http://genome.ucsc.edu/cgi-bin/hgLiftOver). 网络接口会告诉你为什么一些基因组位置, 点击"Explain failure messages"可以查看.

## 转换dbSNP rs号

rs号是由dbSNP给出. UCSC从每个dbSNP版本制作了自己的拷贝. 注意两个中心发布的相同版本的dbSNP并不相同. 但我们将rs号从低版本转换到高版本时, 实际上有两种方式.

### 使用RsMergeArch和SNPHistory

有必要快速归纳一下dbSNP的rs号是如何合并/重新激活的(re-activate):

1. 当同一个的SNP找到了对应不同的rs号, 则较高的rs号将被合并到较低的rs号, 合并过程会在*RsMergeArch.bcp.gz*文件中记录.
2. 当rs号必定是可伸缩(retracted)时, rs号将在*SNPHistory.bcp.gz*文件中记录.
3. 可伸缩的SNP会以添加备注的方式, 在*SNPHistory.bcp.gz*文件中重新激活([re-activated](http://www.ncbi.nlm.nih.gov/books/NBK44496))

记住以上几点后, 我们就可以合并两个表格以获得高版本rs号和新版本rs号之间的关系. 我们开发了一个脚本(供内部使用), 叫[liftRsNumber.py](http://genome.sph.umich.edu/wiki/LiftRsNumber.py), 用来在不同的版本之间转换rs号. 该脚本需要*RsMergeArch.bcp.gz*和*SNPHistory.bcp.gz*, 这些文件可以在下文的资源列表中找到.

输入示例:

```
3000
3001
3002
```

命令:

```
python liftRsNumber.py input.rs > output.rs
```

输出示例:

```
unchanged	3000
lifted	2032
unchanged	3002
```

## Merlin/PLINK 格式转换

在Merlin/PLINK .map文件中, 每一行包含了基因组位置和dbSNP的rs号. 我们在这里的目标是使用两者的信息来尽可能多地转换位置. 有三种liftOver的方法, 我们推荐前两种. 第一种方法是常见并且使用最多的情况的, 并且在我们的观察中, 这种方法可以转换最多的基因组位置, 然而, 他并不能反映不同dbSNP版本之间的rs号改变. 第二种方法更加强健, 这种方法转换的每个rs号都有有效的基因组位置, 这是由于它将使用dbSNP数据转换旧版本rs号作为第一步. 第三种方法不够简单明确, 我们只会简要提及.

### 转换Merlin格式

PLINK格式和[Merlin格式](http://www.sph.umich.edu/csg/abecasis/Merlin/tour/input_files.html)基本是相同的. 区别在于Merlin的.map文件有四列. 我们将展示如何转换PLINK格式的过程, 然后你可以使用以下命令:

```
awk '{print $1,$2,"\t",$3;}' PLINK.map >Merlin.map
```

来获得Merlin的.map文件.

### 转换PLINK格式

[PLINK](http://pngu.mgh.harvard.edu/~purcell/plink/data.shtml)格式通常指*.ped*和*.map*文件.

#### 方法1

我们主要使用UCSC的LiftOver二进制工具来帮助转换. 我们有一个脚本[liftMap.py](http://genome.sph.umich.edu/wiki/LiftMap.py), 但是还是建议一步一步地理解这个工作:

1. 将.map文件转换为.bed文件;
   重新排列.map文件的列顺序, 即可得到标准的BED格式文件.
2. LiftOver .bed 文件;
   使用上文提到的方法将.bed文件从一个版本转换到另外一个版本.
3. 将.bed文件转换回.map文件;
   重新排列.bed文件的列来获得新版本的.map文件.
4. 修改.ped文件
   .ped文件是一个有很多列的文件. 按照约定, 前六列分别为: *family_id*, *person_id*, *father_id*, *mother_id*, *sex* 以及 *phenotype*. 在第七列有两位字母/数字来表示唯一的基因型. 在第二步, 由于一些基因组位置不能被转换到新的版本, 我们需要将.ped文件中的相关列丢弃来保持一致性. 可以使用`PLINK --exclude`参数获得这些snps, 参考[去除SNPs子集](http://pngu.mgh.harvard.edu/~purcell/plink/dataman.shtml#exclude).
5. (可选项)改变.map文件中的rs号
   和人类参考基因组版本类似, dbSNP同样会有不同的版本. 你需要根据需求从旧版本rs号转换到新版本. 这些步骤见上文描述.

#### 方法2

该方法是使用[liftRsNumber.py](http://genome.sph.umich.edu/wiki/LiftRsNumber.py)来将旧版本的rs好转换为新版本, 使用数据文件*b132_SNPChrPosOnRef_37_1.bcp.gz*(一个包含dbSNP和相应NCBI build 37版本基因组位置信息的数据文件), 然后相应地调整.map和.ped文件.

1. 提取并且转换rs号;
   使用[liftRsNumber.py](http://genome.sph.umich.edu/wiki/LiftRsNumber.py)将map文件中的rs号从旧版本转换到新版本.
2. 使用rs号查找SNP位置;
   dbSNP提供[b132_SNPChrPosOnRef_37_1.bcp.gz](http://qbrc.swmed.edu/zhanxw/software/liftOver/b132_SNPChrPosOnRef_37_1.bcp.gz)包含rs号, 染色体以及染色体位置信息. 和在第一步获得的新的rs号一起使用这个文件. 事实上, 一些rs号在build 132版本中并不存在, 或者并不合适(如: 这些点并不位于人类参考基因组上, 或者它们被比对到多个位置, 这些情况在染色体一列使用"AltOnly", "Multi", "NotOn", "PAR", "Un"这些值来标注), 我们可以在liftover的步骤丢弃这些点. 我们在这一步骤之后获得新版本的rs号和对应的染色体位置.
3. 转换.map文件和.ped文件;
   为了转换.map文件, 我们可以逐行扫描文件的内容, 然后跳过没有转换的rs号. 相对应的, 有必要丢弃.ped文件中没有转换的SNP基因型.

#### 方法3

<del>该方法中dbSNP提供的文件已经被移除, 不再适用.</del>

## 转换失败的各种原因

### 基因组位置不能被转换

当一个SNP只在旧版本的基因组的contig上存在时, liftOver不能给出新的基因组版本对应位置.

你可以尝试在UCSC的liftOver在线工具上转换下面的SNP(BED格式):

```
20	56737667	56737668	rs1073519
```

错误信息将会是: "Sequence intersects no chains".

### 在非参考基因组位置上的高版本的SNP

一些SNP不在NCBI build 37版本常染色体或者性染色体上. dbSNP没有包括这些点. 你不能使用dbSNP的数据库来根据rs号找到对应的染色体位置.

以rs1006094为例: 在NCBI dbSNP网页, 这个SNP被报道为"Mapped unambiguously on non-reference assembly only", 因此转换这个SNP可能并没有太大的意义.

### rs号该新版本的dbSNP中改变了

新版本dbSNP有可能没有一些rs号. 当dbSNP发布一个新版本时, 高版本的rs号可能归并到低版本rs号, 因为这些SNP实际上是相同的. 这个归并操作可能会比较复杂. 简短的介绍可以参考上文中*使用RsMergeArch和SNPHistory*一节. 细节信息可以参考:

<del>Finding Specific Data in dbSNP's FTP Files (该页面已经被移除)</del>

[Merging RefSNP Numbers and RefSNP Clusters](http://www.ncbi.nlm.nih.gov/books/NBK44468/#Build.can_two_id_numbers_correspond_to_t)

举个例子:

rs3001 被归并到 rs2032.

### 不同的dbSNP版本

NCBI发布dbSNP132(VCF格式), UCSC也发布了他们的dbSNP132版本(txt格式). 这两个数据库文件不只是格式上不同, 内容上也有差异.

关于NCBI发布的版本, 将不会包括:

* 被列为微卫星或者named variations的SNPs
* 多等位基因并且邻近碱基对未知的SNPs
* 没有比对到GRCh37参考基因组版本的SNPs

关于UCSC发布的版本, 参考[UCSC dbSNP track note](http://genomewiki.ucsc.edu/index.php/DbSNP_Track_Notes).

拿rs1054140作为例子:

NCBI dbSNP网站上给出一个位置: [参考链接](http://www.ncbi.nlm.nih.gov/projects/SNP/snp_ref.cgi?rs=1054140)

NCBI dbSNP的VCF文件上没有记录

UCSC基因组浏览器网站上给出了两个位置: [参考链接](http://genome.ucsc.edu/cgi-bin/hgTracks?clade=mammal&org=Human&db=hg19&position=rs1054140&hgt.suggest=&hgt.suggestTrack=knownGene&pix=800&Submit=submit&hgsid=205770459&hgt.newJQuery=1)

UCSC dbSNP文件给出两个位置:

```
721	chr10	17842693	17842694	rs1054140	0	+	T	T	A/T	genomic single	by-cluster,by-submitter ...
723	chr10	18089681	18089682	rs1054140	0	+	T	T	A/T	genomic single	by-cluster,by-submitter ...
```

## 资源

* [liftRsNumber.py](http://genome.sph.umich.edu/wiki/LiftRsNumber.py)
* [liftMap.py](http://genome.sph.umich.edu/wiki/LiftMap.py)
* <del> NCBI provisional map file and info 该文件已经被移除</del>
* NCBI RgMergeArch [文件](ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606/database/organism_data/RsMergeArch.bcp.gz)以及[概述](http://www.ncbi.nlm.nih.gov/SNP/snp_db_table_description.cgi?t=RsMergeArch)
* NCBI SNPHistory [文件](ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606/database/organism_data/SNPHistory.bcp.gz)以及[概述](http://www.ncbi.nlm.nih.gov/SNP/snp_db_table_description.cgi?t=SNPHistory)
* NCBI SNPChrPosOnRef build 132 [文件](http://qbrc.swmed.edu/zhanxw/software/liftOver/b132_SNPChrPosOnRef_37_1.bcp.gz)以及[概述](http://www.ncbi.nlm.nih.gov/SNP/snp_db_table_description.cgi?t=SNPChrPosOnRef). 若该链接失效, 考虑使用升级的文件([参考链接](ftp://ftp.ncbi.nih.gov/snp/organisms/human_9606/database/organism_data/b144_SNPChrPosOnRef_107.bcp.gz))
  // 译者注: 最后的链接中, dbSNP版本已经更新, 链接不再完整有效, 请到对应目录下查找相应文件; 另: 截止翻译时, NCBI默认使用GRCh38版本参考基因组, 下载时需要注意.
* UCSC和NCBI发布的dbSNP有哪些差别 [UCSC dbSNP track note](http://genomewiki.ucsc.edu/index.php/DbSNP_Track_Notes)
* dbSNP比对过程[参考链接](http://www.ncbi.nlm.nih.gov/books/NBK44455)
* <del>NCBI dbSNP 132版本 过时的版本</del>
* <del>UCSC dbSNP 132版本 过时的版本</del>

## 维护者

该页面由[Xiaowei Zhan](zhanxw@umich.edu)维护.

## 译者注1

该wiki发布于2015年7月15号, 里面部分数据库的版本相对过时, 实际操作中请配合目前版本需求;

## 译者注2 VCF格式转换

使用gatk提供的picard可以进行VCF格式的版本转换, 需要准备的文件有:

1. 下载相应地转换文件, 可以从[gatk的github](https://github.com/broadgsa/gatk/tree/master/chainFiles)上下载得到;
2. 目标基因组版本的基因组序列;
3. 待转换的VCF文件;

使用以下命令在不同版本之间转换VCF格式(以GRCh37的cosmic数据库转换到hg19为例):

```
java -jar picard.jar LiftoverVcf I=GRCH37_cosmic.vcf O=hg19_cosmic.vcf C=b37tohg19.chain R=ucsc.hg19.fasta REJECT=reject.vcf
```

转换结果将输出到`hg19_cosmic.vcf`, 转换失败的结果将输出到`reject.vcf`.

## 申明

原文地址: http://genome.sph.umich.edu/wiki/LiftOver

本文由本人(Lyon0804)翻译并发布, 如果侵犯原文权利请告知.

转载请注明本文地址: http://www.lyon0804.com/fan-yi-liftover.html