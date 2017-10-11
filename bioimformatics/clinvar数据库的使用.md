Title: clinvar数据库的使用
Date: 2016-07-15
Category: bioinformatics
Tags: bioinformatics, clinvar, ANNOVAR

## clinvar数据库

clinvar数据库记录基因组变异和人类健康的关系, 在遗传变异与疾病的研究中非常重要.

### 数据库获取

该数据库可以[FTP站点](ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/)下载到. 

该数据库大约每月更新一次, 以保证数据实时性.

### 数据库构成

在下载到的clinvar_20160705.vcf(或者其他日期版本)中, 是在dbsnp的基础上进行注释, 实际内容在INFO中体现. 

由于dbsnp中一条记录可能包含多种基因型的变异, 显然, 不同的基因型表现出的疾病关系很可能是不同的, 所以在下载的vcf文件中, vcf文件条目和实际的clinvar记录条目是一对多的关系. 意识到这一点非常重要, 由于很多工具并不会针对clinvar进行特殊处理, 忽略这个特性很有可能会得到前后矛盾的注释结果.

## ANNOVAR的问题

在ANNOVAR中, 作者提供了切分和向左标准化的clinvar文件, 但是在转换的同时CLINHGVS信息丢失, 所以注释的结果中并不能准确表示clinvar数据库的原始表达.

如:

chr7:150648198这个位点, A>G表现为良性, 而A>T则表现为致病, 在原始文件中的记录是这样的:

```
7       150648198       rs1137617       A       C,G,T   .       .       RS=1137617;RSPOS=150648198;RV;dbSNPBuildID=86;SSR=0;SAO=1;VP=0x05017000030515053e110100;GENEINFO=KCNH2:3757;WGT=1;VC=SNV;PM;TPA;SLO;REF;SYN;ASP;VLD;G5;HD;GNO;KGPhase1;KGPhase3;LSD;OM;CLNALLE=2,3;CLNHGVS=NC_000007.13:g.150648198A>G,NC_000007.13:g.150648198A>T;CLNSRC=.,.;CLNORIGIN=1,1;CLNSRCID=.,.;CLNSIG=2,5;CLNDSDB=MedGen,MedGen;CLNDSDBID=CN169374,CN221809;CLNDBN=not_specified,not_provided;CLNREVSTAT=single,single;CLNACC=RCV000181727.1,RCV000181832.1;CAF=0.2278,.,0.7722,.;COMMON=1
```

在ANNOVAR转换后的clinvar中, 该记录是这样的:

```
7       150648198       150648198       A       C       Benign\x2cPathogenic    Cardiac_arrhythmia\x2cCardiac_arrhythmia        RCV000181727.1,RCV000181832.1   MedGen:OMIM\x2cMedGen:OMIM      CN029864:115000\x2cCN029864:115000
7       150648198       150648198       A       G       Benign\x2cPathogenic    Cardiac_arrhythmia\x2cCardiac_arrhythmia        RCV000181727.1,RCV000181832.1   MedGen:OMIM\x2cMedGen:OMIM      CN029864:115000\x2cCN029864:115000
7       150648198       150648198       A       T       Benign\x2cPathogenic    Cardiac_arrhythmia\x2cCardiac_arrhythmia        RCV000181727.1,RCV000181832.1   MedGen:OMIM\x2cMedGen:OMIM      CN029864:115000\x2cCN029864:115000
```

虽然ANNOVAR对clinvar进行了切分, 将原始文件中的复合记录切分为三条, 但是在切分的过程中并没有区分CLNHGVS和其他clnvar记录的对应关系;

如在原始文件中, NC_000007.13:g.150648198A>G对应的CLNSIG是2(Benign), 而NC_000007.13:g.150648198A>T对应的CLNSIG则是5(Pathogenic), 而在ANNOVAR转换之后, 虽然拆分成了A>C, A>G, A>T三条记录, 但是关键的clinvar信息却是简单拼接的;

可想而知, 使用ANNOVAR对chr7:150648198A>C这条变异的结果进行注释时, 会得到致病的判断, 这显然是错误的.

## Clinvar的切分和标准化

由于clinvar提供的vcf文件特点, 一条记录中可能包含同一个位点的不同变异, 这样并不便于后续的注释, 所以切分是必要的, 但是在切分的过程中, 需要根据CLNHGVS对应的基因型, 正确反应原始记录.

### 根据HGVS判断变异

CLNHGVS使用HGVS标准来表示一个变异, 不同的变异类型其表示方式分别举例有:

* snp: NC_000001.10:g.955597G>T;
* ins: NC_000001.10:g.2337967_2337968insC, 注意C插入的位置是在2337967之后;
* del: NC_000001.10:g.2340153delA, NC_000001.10:g.6529183_6529185delTCC;
* mnp: NC_000001.10:g.1273413_1273420delTAGGCAGGinsC, NC_000001.10:g.201334375_201334376delinsAC;
* dup: NC_000001.10:g.949699dupG, NC_000001.10:g.3334509_3334510dupCC;
* 野生型: NC_000001.10:g.25717365C\x3d;

通过HGVS可以准确定位到染色体位置, 并获取变异的相关信息;

### 通过CLNHGVS进行切分

在INFO中CLN开头的字段表示clinvar的注释信息, 由于一个记录可能对应多个变异, 所以CLN开头的字段都是逗号分隔的列表; 

根据CLNHGVS可以获得有注释的基因型列表, 通过该列表对clinvar原始文件进行切分而不是通过ALT进行切分, 好处是可以准确表达clinvar中存在记录的变异;

CLN开头的字段列表是一一对应的, 所以在切分之后, 需要一一获取其中的信息;

如:

```
7       150648198       rs1137617       A       C,G,T   .       .       RS=1137617;RSPOS=150648198;RV;dbSNPBuildID=86;SSR=0;SAO=1;VP=0x05017000030515053e110100;GENEINFO=KCNH2:3757;WGT=1;VC=SNV;PM;TPA;SLO;REF;SYN;ASP;VLD;G5;HD;GNO;KGPhase1;KGPhase3;LSD;OM;CLNALLE=2,3;CLNHGVS=NC_000007.13:g.150648198A>G,NC_000007.13:g.150648198A>T;CLNSRC=.,.;CLNORIGIN=1,1;CLNSRCID=.,.;CLNSIG=2,5;CLNDSDB=MedGen,MedGen;CLNDSDBID=CN169374,CN221809;CLNDBN=not_specified,not_provided;CLNREVSTAT=single,single;CLNACC=RCV000181727.1,RCV000181832.1;CAF=0.2278,.,0.7722,.;COMMON=1
```

以上记录中, 正确切分后应该为:

```
chr7    150648198       rs1137617       A       G       .       .       dbSNPBuildID=86;RV;RSPOS=150648198;KGPhase1;SSR=0;ASP;OM;GNO;RS=1137617;CLNORIGIN=1;CAF=0.2278,.,0.7722,.;WGT=1;CLNACC=RCV000181727.1;SAO=1;VLD;CLNDBN=not_specified;G5;PM;CLNREVSTAT=single;LSD;COMMON=1;CLNDSDB=MedGen;REF;TPA;VP=0x05017000030515053e110100;SYN;VC=SNV;GENEINFO=KCNH2:3757;CLNSIG=2;SLO;CLNHGVS=NC_000007.13:g.150648198A>G;CLNSRCID=.;KGPhase3;HD;CLNSRC=.;CLNDSDBID=CN169374
chr7    150648198       rs1137617       A       T       .       .       dbSNPBuildID=86;RV;RSPOS=150648198;KGPhase1;SSR=0;ASP;OM;GNO;RS=1137617;CLNORIGIN=1;CAF=0.2278,.,0.7722,.;WGT=1;CLNACC=RCV000181832.1;SAO=1;VLD;CLNDBN=not_provided;G5;PM;CLNREVSTAT=single;LSD;COMMON=1;CLNDSDB=MedGen;REF;TPA;VP=0x05017000030515053e110100;SYN;VC=SNV;GENEINFO=KCNH2:3757;CLNSIG=5;SLO;CLNHGVS=NC_000007.13:g.150648198A>T;CLNSRCID=.;KGPhase3;HD;CLNSRC=.;CLNDSDBID=CN221809
```

可见在切分之后不仅注释结果更加清晰, 还去掉了A>C等不存在记录的变异;

### 记录的标准化

在CLNHGVS中, 很多的表示都并非标准化的表示, 为了方便注释, 应该在切分之后进行标准化操作(左对齐, 简约化);

标准化操作可以参考[之前的文章](http://www.lyon0804.com/fan-yi-variant-normalization.html);

##  使用MongoDB进行注释

当完成clinvar的标准化之后, 就可以简单地对变异进行搜索, 由于同一个位点的表示总是有一个唯一的标准化的表示, 通过`CHROM`、`POS`、`REF`、`ALT`这四个键完成数据库查询, 通过建立以上四个键的组合索引, 可以较快地在MongoDB中进行查询操作;

借助PyVCF的VCFReader和VCFWriter, 对VCF文件进行注释并且输出注释后的VCF文件;

## 实现

具体实现脚本可以参考本人的这篇[gist](https://gist.github.com/lozybean/9bb99e48d9d155510383ac9533741113);

其中参考基因组的读取借助了`samtools faidx`命令, 这需要安装samtools并且对参考基因组fasta建立索引.