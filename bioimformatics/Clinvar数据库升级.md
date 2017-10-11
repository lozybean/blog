Title: clinvar数据库升级
Date: 2017-09-08
Category: bioinformatics
Tags: clinvar, database, bioinformatics


## Clinvar数据库

Clinvar数据库是一个变异致病性数据库, [之前的文章](http://www.lyon0804.com/clinvarshu-ju-ku-de-shi-yong.html) 有讲到如何正确使用clinvar数据库. 本文将在前文基础上, 总结clinvar数据库使用中的问题, 并且介绍如何使用vcf2.0数据库以及submission数据库解决这些问题.

除了前文中提到的问题之外, Clinvar提供的vcf数据库还有其他的问题存在, 这些问题有:

### 标注不清

clinvar旧版本的vcf数据库中, 以一个rs号为单位, 使用rs number作为变异的ID, 记录中会展现每个rs号的所有Allele, 而实际上**CNLHGVS**注释显示, 注释内容只对其中一个Allele有效, 或者不同的Allele结论不同. 在[之前的文章](http://www.lyon0804.com/clinvarshu-ju-ku-de-shi-yong.html)中有更加详细的问题描述以及解决方式.

而在实际的clinvar数据库中(以及网页上), 有一个存在的唯一标识: **Variation ID**, 该ID信息在旧版本的vcf文件中丢失, 只提供Variant Accession的唯一标识, 该标识是变异+疾病(Variation + Condition)的识别标识, 而非Variation的识别标识.

### CLNSIG描述不清

旧版本的vcf文件中, CLNSIG的注释以数字标明, 各数字意义为:

* 0 - Uncertain significance
* 1 - not provided
* 2 - Benign
* 3 - Likely benign
* 4 - Likely pathogenic
* 5 - Pathogenic
* 6 - drug response
* 7 - histocompatibility
* 255 - other

并且在vcf文件中, CLNSIG的注释以REVIEW为单位, 同一个位点的同一种疾病对应的结果, 如果有不同的结论, 则直接标注为255(other), 这对注释结果带来困扰.

如VariationID为134651, `NM_001127500.2(MET):c.4141G>A (p.Ala1381Thr)` [clinvar链接](https://www.ncbi.nlm.nih.gov/clinvar/variation/134651/) 这一条变异, 网页显示为

![134651](http://wx2.sinaimg.cn/large/95202659gy1fjp4au3lj4j20h108z3yq.jpg)

而在下载vcf文件中的显示为:

```
7       116436092       rs45578433      G       A       .       .       RS=45578433;RSPOS=116436092;dbSNPBuildID=127;SSR=0;SAO=1;VP=0x050068000a05040436100100;GENEINFO=MET:4233;WGT=1;VC=SNV;PM;PMC;NSM;REF;ASP;VLD;HD;KGPhase1;KGPhase3;LSD;CLNALLE=1;CLNHGVS=NC_000007.13:g.116436092G>A;CLNSRC=.;CLNORIGIN=1;CLNSRCID=.;CLNSIG=1|255;CLNDSDB=MedGen|MedGen:OMIM:Orphanet:SNOMED_CT;CLNDSDBID=CN169374|C0007134:605074:ORPHA217071:41607009;CLNDBN=not_specified|Renal_cell_carcinoma\x2c_papillary\x2c_1;CLNREVSTAT=no_assertion|conf;CLNACC=RCV000121350.1|RCV000205626.4;CAF=0.9988,0.001198;COMMON=0
```

其中的CLNSIG显示`1|255`, CLNREVSTAT显示`no_assertion|conf`, 经查, 发现CLNSIG标注255的原因是因为对于*Renal cell carcinoma, papillary, 1*该疾病, 有两个Submissions分别报告为*Benign*和*Likely Benign*; 在vcf文件中简单判定该位点具有冲突结论, 故标注为255;

对于实际解读来讲, 仅标记other(255)很难进行准确的判断, 只有将各种Submission的情况全部注释, 才能进行准确判读.

另外一个更加深刻的例子:

VariationID为127827, `NM_000546.5(TP53):c.91G>A (p.Val31Ile)` [clinvar链接](https://www.ncbi.nlm.nih.gov/clinvar/variation/127827/)这一条变异, 网页显示为

![127827](http://wx2.sinaimg.cn/large/95202659gy1fjp4awe236j20hy0ahweu.jpg)

该位点网页判读为有冲突结论, 而在下载的旧版本vcf文件中, 该位点显示为:

```
17      7579705 rs201753350     C       A,T     .       .       RS=201753350;RSPOS=7579705;dbSNPBuildID=137;SSR=0;SAO=1;VP=0x050068420a05040436100100;GENEINFO=TP53:7157;WGT=1;VC=SNV;PM;PMC;NSM;REF;U5;R5;ASP;VLD;HD;KGPhase1;KGPhase3;LSD;CLNALLE=1,2;CLNHGVS=NC_000017.10:g.7579705C>A,NC_000017.10:g.7579705C>T;CLNSRC=.,UniProtKB_(protein);CLNORIGIN=1,1;CLNSRCID=.,P04637#VAR_044554;CLNSIG=0,0|3|3|255;CLNDSDB=MedGen:SNOMED_CT,MedGen:SNOMED_CT|MedGen|MedGen:Orphanet:SNOMED_CT|MedGen:OMIM;CLNDSDBID=C0027672:699346009,C0027672:699346009|CN169374|C0085390:ORPHA524:428850001|C1835398:151623;CLNDBN=Hereditary_cancer-predisposing_syndrome,Hereditary_cancer-predisposing_syndrome|not_specified|Li-Fraumeni_syndrome|Li-Fraumeni_syndrome_1;CLNREVSTAT=single,single|single|mult|conf;CLNACC=RCV000222526.1,RCV000115742.7|RCV000122173.2|RCV000123100.4|RCV000409871.1;CAF=0.9982,.,0.001797;COMMON=1
```

首先该记录具有具体注释每个Allele的情况, 另外, 这里的CLNSIG对于`T`这个ALT, 仅注释为`0|3|3|255`, 原因是对于四种疾病, 前三种没有冲突, 可以明确注明, 但是最后一种疾病有两个冲突的Submission, 分别报道为: *Likely pathogenic* 和 *Uncertain significance*, 故注明255. 如果不去细究这个冲突, 仅仅是看vcf文件的结果, 很有可能趋向于Likely Benign(3), 而忽略该位点具有疑似致病报道.

## 数据库升级

为了解决旧版本vcf数据库中的问题, clinvar官方团队在2017年2月份在FTP站点中, 额外提供了一个vcf2.0版本的数据库. 该数据库目前为beta(测试)版本, 并且在说明文件中指出, 目前可能有不完善的地方.

![vcf2.0](http://wx1.sinaimg.cn/large/95202659gy1fjp4ayz5mfj20bh0fy751.jpg)

该版本的数据库解决了之前数据库中存在的若干问题:

1. 新版本数据库以Variation ID为单位, 对每一个位置的每一个allele分别注释, 避免以rs号为单位, Allele和注释冲突的问题;
2. 新版本数据库中CLNSIG使用了更加明确的文字描述, 当出现有冲突时, 明确标明*Conflicting_interpretations_of_pathogenicity*, 而不是仅标注other(255), 如第二个例子;
3. 当就版本数据库中, 冲突不严重时, 如第一个例子, 新版本数据库直接注释为*Benign/Likely_benign*, 和网页上描述一致, 不再标注other(255);
4. 当变异属于某个单体型时, 添加该单体型的致病描述;

新版本数据库中, CLNSIG使用了更准确的文字描述, 分为:

* Benign
* Benign/Likely_benign
* Likely_benign
* Likely_pathogenic
* Pathogenic
* Pathogenic/Likely_pathogenic
* Uncertain_significance
* Conflicting_interpretations_of_pathogenicity
* Affects
* association
* drug_response
* protective
* risk_factor
* not_provided
* other

具体含义见[clinvar网站](https://www.ncbi.nlm.nih.gov/clinvar/docs/clinsig/)

前两个例子, 在新版本的vcf文件中, 都会

第一个例子显示为:

```
7       116436092       134651  G       A       .       .       AF_EXAC=0.00044;AF_TGP=0.0012;ALLELEID=138390;CLNDISDB=MedGen:C0007134,OMIM:605074,Orphanet:ORPHA217071,SNOMED_CT:41607009|MedGen:CN169374;CLNDN=Renal_cell_carcinoma,_papillary,_1|not_specified;CLNHGVS=NC_000007.13:g.116436092G>A;CLNREVSTAT=criteria_provided,_multiple_submitters,_no_conflicts;CLNSIG=Benign/Likely_benign,_not_provided;CLNVC=single_nucleotide_variant;CLNVCSO=SO:0001483;CLNVI=Illumina_Clinical_Services_Laboratory,Illumina:104349;GENEINFO=MET:4233;MC=SO:0001583|missense_variant;ORIGIN=1;RS=45578433
```

其中CLNSIG为*Benign/Likely_benign,_not_provided* (旧版本为`1|255`(not provided, other))

第二个例子显示为:

```
17      7579705 127827  C       T       .       .       AF_TGP=0.0018;ALLELEID=133284;CLNDISDB=MedGen:C0027672,SNOMED_CT:699346009|MedGen:C0085390,Orphanet:ORPHA524,SNOMED_CT:428850001|MedGen:C1835398,OMIM:151623|MedGen:CN169374;CLNDN=Hereditary_cancer-predisposing_syndrome|Li-Fraumeni_syndrome|Li-Fraumeni_syndrome_1|not_specified;CLNHGVS=NC_000017.10:g.7579705C>T;CLNREVSTAT=criteria_provided,_conflicting_interpretations;CLNSIG=Conflicting_interpretations_of_pathogenicity;CLNVC=single_nucleotide_variant;CLNVCSO=SO:0001483;CLNVI=Illumina_Clinical_Services_Laboratory,Illumina:663209,UniProtKB_(protein):P04637#VAR_044554;GENEINFO=TP53:7157;MC=SO:0001583|missense_variant,SO:0001623|5_prime_UTR_variant,SO:0001636|2KB_upstream_variant;ORIGIN=1;RS=201753350
```

该位点在新版本中, 仅allele: T单独作为一个记录显示, 并且CNLSIG表示为*Conflicting_interpretations_of_pathogenicity*, 而不是`255`(other).

## Submissions

新版本的数据库中, 虽然对冲突的情况使用了更加明确的标识, 但是仍然没有展现每个Submission详细的信息, 只能通过网页查找更详细的Submission信息才能更好做出判断.

由于新版本数据库添加了**Variation ID**这个唯一识别标识, 使用该标识, 从FTP提供的*submission_summary.txt.gz*文件中, 通过该标识可以提取所有的Submissions.

```
127827  Uncertain significance  Dec 10, 2015    -       OMIM:151623     C1835398:Li-Fraumeni syndrome 1 criteria provided, single submitter     clinical testing        unknown:na      Counsyl SCV000487895.1  TP53
127827  Uncertain significance  Feb 09, 2017    -       Hereditary cancer-predisposing syndrome C0027672:Hereditary cancer-predisposing syndrome        criteria provided, single submitter     clinical testing        germline:1      Ambry Genetics  SCV000187523.4  TP53
127827  Likely pathogenic       Mar 18, 2016    -       OMIM:151623     C1835398:Li-Fraumeni syndrome 1 criteria provided, single submitter     reference population    germline:3      Soonchunhyang University Bucheon Hospital,Soonchunhyang University Medical Center       SCV000267534.1  TP53
127827  Likely benign   Apr 12, 2017    This variant is considered likely benign or benign based on one or more of the following criteria: it is a conservative change, it occurs at a poorly conserved position in the protein, it is predicted to be benign by multiple in silico algorithms, and/or has population frequency not consistent with disease.    not specified   CN169374:not specified  criteria provided, single submitter     clinical testing        germline:na     GeneDx  SCV000149651.5  TP53
127827  not provided    Sep 19, 2013    -       AllHighlyPenetrant      CN169374:not specified  no assertion provided   reference population    germline:na     ITMI    SCV000086388.1  -
127827  Likely benign   Jun 14, 2016    -       Li-Fraumeni Syndrome    C0085390:Li-Fraumeni syndrome   criteria provided, single submitter     clinical testing        germline:na     Illumina Clinical Services Laboratory,Illumina  SCV000407074.2  TP53
127827  Likely benign   Jan 18, 2017    -       MedGen:C0085390 C0085390:Li-Fraumeni syndrome   criteria provided, single submitter     clinical testing        germline:na     Invitae SCV000166401.4  TP53
```

## 注释方法升级

### 建立数据库

为了更加明确地注释clinvar中的内容, 在注释时应当使用新版本vcf数据库进行注释, 并且在遇到*Conflicting_interpretations_of_pathogenicity*时, 额外注释submissions以便解读.

为了实现该注释方式, 首先需要下载vcf2.0中的vcf文件, 以及tab_delimited中的`submission_summary.txt.gz`文件, 并分别建立MongoDB数据库.

为了提升查询的速度, 对`submission_summary.txt.gz`建立数据库时, 应当建立**VariationID**的索引. vcf2.0数据库的索引按照查询习惯建立, 一般都会建立`chr+pos+ref+alt`的索引.

数据库建立示例代码:

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- \#
"""
@author = 'liangzb'
@date = '2017/9/7 0007'

Requirements:
1. python3.6 +
2. pyvcf
3. pymongo

This database is stand on ClinVar's tab-delimiter files;
 AlleleID can be find in clinvar *vcf2.0* database,
 The transformation of VariationID and AlleleID is from *hgvs4variation.txt.gz*;
 Submission database is from *submission_summary.txt.gz*

All the above files can be found on ClinVar ftp server.
"""

import csv
import gzip
from collections import namedtuple

from vcf import Reader
from pymongo import MongoClient

server_ip = 'xx.xx.xx.xx'
client = MongoClient(server_ip, 27017, document_class=OrderedDict)
clinvar_tab_version = '20170904'
clinvar_version_2 = '20170905'


def trans_b37_hg19(chrom):
    if not chrom.startswith('chr'):
        if chrom == 'MT':
            return 'chrM'
        else:
            return 'chr' + chrom
    return chrom


def admin_acquire(fn):
    def wrapper(self, *args, **kwargs):
        self.db.logout()
        self.db.authenticate(name='admin', password='123')
        fn(self, *args, **kwargs)
        self.db.logout()
        self.db.authenticate(name='user', password='123')

    return wrapper


class Base(object):
    def __init__(self):
        self.db = client.clinvar
        self.db.authenticate(name='user', password='123')
        self.prefix = ''
        self.collection_name = None

    def check_version(self):
        if self.collection_name in self.db.collection_names():
            return True
        else:
            enable_versions = ', '.join([v for v in self.db.collection_names() if v.startswith(self.prefix)])
            raise ValueError(f"version do not exists in database: {self.collection_name}! "
                             f"please check and use the version in:[ {enable_versions} ]")


class IDTable(Base):
    """
        This database is not necessary;
        according to README_VCF.txt:
        *some Variation IDs may be missing from the files*,
        maybe it can be supplementary someway.
    """

    def __init__(self, version=clinvar_tab_version, assembly='GRCh37'):
        super().__init__()
        self.prefix = 'id_table_'
        self.collection_name = f'{self.prefix}{version}'
        self.collection = self.db[self.collection_name]
        self.assembly = assembly

    def create_index(self):
        self.collection.create_index([("allele_id", 1), ("assembly", 1)])
        self.collection.create_index([("variation_id", 1), ("assembly", 1)])

    @admin_acquire
    def insert(self, file_name):
        self.collection.drop()
        with gzip.open(file_name, 'rt') as fp:
            for line in fp:
                if line.startswith('#'):
                    continue
                symbol, gene_id, variation_id, allele_id, type_, assembly, *_ = line.strip().split('\t')
                if type_ != 'genomic':
                    continue
                document = {
                    'variation_id': variation_id,
                    'allele_id': allele_id,
                    'assembly': assembly,
                }
                self.collection.insert(document)
        self.create_index()

    def get_allele_id(self, variation_id: str):
        return self.collection.find_one({
            'variation_id': variation_id,
            'assembly': self.assembly
        })['allele_id']

    def get_variation_id(self, allele_id: str):
        return self.collection.find_one({
            'allele_id': allele_id,
            'assembly': self.assembly
        })['variation_id']


class SubmissionSummary(Base):
    def __init__(self, version=clinvar_tab_version):
        super().__init__()
        self.prefix = 'submission_'
        self.collection_name = f'{self.prefix}{version}'
        self.collection = self.db[self.collection_name]

    def create_index(self):
        self.collection.create_index([("VariationID", 1)])

    @admin_acquire
    def insert(self, file_name):
        self.collection.drop()
        self.create_index()
        with gzip.open(file_name, 'rt') as fp:
            # find header line
            for line in fp:
                if line.startswith('##'):
                    continue
                if line.startswith('#VariationID\t'):
                    header = line.strip('#\n').split('\t')
                    break
            Row = namedtuple('Row', header)
            csv_reader = csv.reader(fp, delimiter='\t')
            for r in csv_reader:
                row = Row(*r)._asdict()
                variation_id = row.pop('VariationID')
                self.collection.update({'VariationID': variation_id}, {
                    "$push": {'Submissions': row}
                }, upsert=True)

    def get_submissions(self, variation_id: str):
        record = self.collection.find_one({"VariationID": variation_id})
        if not record:
            return []
        else:
            return record["Submissions"]


class Clinvar(Base):
    def __init__(self, version=clinvar_version_2):
        super().__init__()
        self.prefix = 'clinvar_v2_'
        self.collection_name = f'{self.prefix}{version}'
        self.collection = self.db[self.collection_name]

    def create_index(self):
        self.collection.create_index([('chr', 1), ('pos', 1), ('ref', 1), ('alt', 1)])

    @admin_acquire
    def insert(self, vcf_file):
        self.collection.drop()
        with open(vcf_file, 'r') as fp:

            vcf_reader = Reader(fp)
            vcf_reader.metadata['reference'] = 'hg19'
            for record in vcf_reader:
                document = {
                    'chr': trans_b37_hg19(record.CHROM),
                    'pos': int(record.POS),
                    '_id': record.ID,
                    'ref': str(record.REF),
                    'alt': str(record.ALT[0]),
                }
                for key, value in record.INFO.items():
                    if isinstance(value, list):
                        value = ','.join([v for v in value if v])
                    document.update({key: value})
                self.collection.insert(document)
        self.create_index()

    def search(self, chrom: str, pos: int, ref: str, alt: str):
        return self.collection.find_one({'chr': chrom,
                                         'pos': pos,
                                         'ref': ref,
                                         'alt': alt})

    def search_with_submission(self, chrom: str, pos: int, ref: str, alt: str):
        record = self.search(chrom, pos, ref, alt)
        variation_id = record['_id']
        record['Submissions'] = submission.get_submissions(variation_id)
        return record

    def __getitem__(self, variation_id: str):
        return self.collection.find_one({'_id': variation_id})


def get_submissions_with_allele_id(allele_id):
    variation_id = id_table.get_variation_id(str(allele_id))
    return submission.get_submissions(variation_id)


id_table = IDTable()
submission = SubmissionSummary()
clinvar = Clinvar()
```

### 查询方法升级

在查询方法上, 为了最大限度清晰地注释变异的结果, 除了vcf2.0数据库的注释之外, 还应当在发现有冲突结论时, 遍历*submissions*数据库(如上面代码中的`search_with_submission`方法), 并将可能致病的结论明确表示出来.
