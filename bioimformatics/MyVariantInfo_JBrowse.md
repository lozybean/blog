Title: MyVariantInfo以及JBrowse
Date: 2017-08-17
Category: pages
Tags: variant, annotation, jbrowse

## 基于web的变异注释工具: MyVariantInfo

如果你是没有生物信息背景, 当看到一个位点, 或者一个变异的时候, 要如何快速找到该位点或者该变异位于那个基因, 是什么功能, 人群频率是多少等等呢?

没有必要再去各种数据库搜索, [MyVariantInfo](http://myvariant.info) 网站提供了RESTFUL接口, 只需要输入简单的网页地址, 即可看到你想要的所有信息. 在浏览器输入[http://myvariant.info/demo/](http://myvariant.info/demo/), 快速感受一下使用示例吧!

如通过一个rs号来查询相关信息. rs号应该是最常见的位点表示形式了, 查询形式也很简单:

```
http://myvariant.info/v1/query?q=rs75321043
```

在上面网页地址的`q=`后面, 输入你想要查询的rs号, 然后到浏览器中, 就可以看到该位点的所有注释结果了.

当你看到一大段'代码'的时候, 别慌! 这是一种json格式, 只要有点耐心, 你就会发现不管是1000g/exac等人群频率, polyphen/sift等各种预测值, dbsnp/ensembl/clinvar等各种数据库的注释结果, 几乎涵盖了所有生物信息注释的内容.

看到这里, 你是不是觉得: 

> 这个网站好是好, 就是json格式不容易看了, 如果弄成可视化的就好了!

那么以下内容你就要好好介绍给生物信息的同学, 让他们来帮忙搭建一个定制的基因组浏览器.

## JBrowse

JBrowse是一个可定制的基因组浏览器, 该浏览器可以通过定制显示的内容, 展现各种个性化的内容, 上文提到的MyVariantInfo当然不在话下.

### 下载

JBrowse下载非常简单, 使用`wget`直接下载安装包即可:

```
wget -c http://jbrowse.org/releases/JBrowse-1.12.3/JBrowse-1.12.3.zip
```

### 安装

JBrowse是一个网页服务, 所以首先要安装一个网页后台服务, 比如最常见的`apache`:

```bash
yum install apache
```

JBrowse本身的安装就异常简单了, 只需要直接运行安装包中的安装文件即可:

```bash
unzip JBrowse-1.12.3.zip
cd JBrowse-1.12.3.zip
./setup.sh
```

### 配置

JBrowse的关键步骤在于配置, 首先, 需要配置`apache`, 开一个url供JBrowse使用:

```bash
echo "
Alias /JBrowse /path/to/JBrowser/JBrowse-1.12.3

<Location /JBrowse>
  Order deny,allow
  Allow from all
</Location>

" >/etc/httpd/conf.d/jbrowse.conf
```

然后重启`apache`:

```bash
service httpd restart
```

接下来直接打开`http://localhost/JBrowse`页面, 就可以看到最初的JBrowse了, 当然现在还没有任何定制的内容.

### 加入参考基因组

使用JBrowse自带的脚本, 添加本地参考基因组:

```
./bin/prepare-refseqs.pl --fasta <reference.fa>
```

该命令会生成一个`data`目录, 之后我们的所有应用配置也在该目录中.

重新打开页面, 是不是看到了可选的序列了呢?

### 加入MyVariantInfo

有了序列查询功能当然不够, 重要的是每个位点的注释信息, 本文的重头戏来了, 下面会说明如何配置MyVariantInfo到JBrowse中.

#### 安装MyVariantViewer插件

在JBrowse目录下输入:

```
git clone https://github.com/elsiklab/myvariantviewer.git plugins/MyVariantTviewer
```

然后编辑`jbrowse.conf`文件, 加入:

```
[ plugins.MyVariantViewer ]
location = plugins/MyVariantViewer
```

### 配置Track

JBrowse有许多的Track, 每个track可承载独立的内容, 要将MyVariantInfo的信息整合到JBrowse中, 就必须要添加Track, 如想要添加`exac`的信息, 则需要编辑`data/trackList.json`, 在`tracks`数组中, 加入:

```json
{
	"style" : {
		"color" : "#b33"
	},
	"baseUrl" : "https://myvariant.info/v1/",
	"storeClass" : "MyVariantViewer/Store/SeqFeature/Variants",
	"urlTemplate" : "query?q={refseq}:{start}-{end} AND _exists_:exac&size=1000&email=colin.diesh@gmail.com&fields=exac",
	"type" : "CanvasFeatures",
	"category": "MyVariant.info",
	"optimizer": true,
	"label" : "MyVariant.info exac",
	"glyph" : "MyVariantViewer/View/FeatureGlyph/Diamond"
}
```

通过定制`urlTemplate`以及其他的相关设置, 即可将MyVariantInfo中可以注释的内容, 全部搬到JBrowse中.

### 添加vcf的track

除了MyVariantInfo之外, 本地有建立好的vcf文件也可以通过JBrowse进行加载, 但是JBrowse只支持bgzip压缩, 且建立tabix索引的vcf文件, 所以首先要对vcf文件进行处理:

```bash
bgzip myfile.vcf
tabix -p vcf myfile.vcf.gz
```

然后在`tracks.json`的`tracks`数组中添加类似下面的内容即可: 

```json
{
   "label": "mytrack",
   "urlTemplate": "myfile.vcf.gz",
   "storeClass": "JBrowse/Store/SeqFeature/VCFTabix", 
   "type": "CanvasVariants"
}
```

## 效果展示

示例效果: 如搜索13号染色体上的某一段内容, JBrowse可自动从MyVariantInfo中拉取相关信息, 并全数展示到网页上, 点击每一个小方块, 可显示注释的详细信息, 当然网页上的其他小按钮小功能就不一一介绍了~

![效果](http://wx3.sinaimg.cn/large/95202659gy1fjp7mjhvc2j21kw0zpwtj.jpg)

示例的配置文件可参考:

* [trackList.json](https://gist.githubusercontent.com/lozybean/5a2fa731d544997dd88796ed72b3c695/raw/bebfa7a72bb02076104c5fa93ffe9a34d3d4cf2d/trackList.json)
* [functions.conf](https://gist.githubusercontent.com/lozybean/5a2fa731d544997dd88796ed72b3c695/raw/bebfa7a72bb02076104c5fa93ffe9a34d3d4cf2d/functions.conf)
* [tracks.conf](https://gist.githubusercontent.com/lozybean/5a2fa731d544997dd88796ed72b3c695/raw/bebfa7a72bb02076104c5fa93ffe9a34d3d4cf2d/tracks.conf)