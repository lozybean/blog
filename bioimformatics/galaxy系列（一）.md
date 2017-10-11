Title: 使用Galaxy_01_介绍以及搭建
Date: 2016-02-16
Category: bioinformatics
Tags: galaxy, bioinpormatics, python

## 生物信息分析平台galaxy简介

生物信息分析平台主要是将一些生物信息分析过程抽离并且封装成可视化的界面, 使得没有编程经验的生物信息学分析人员, 也可以通过平台上搭建的已有流程进行数据分析;

目前生物信息学分析平台主要有以下几种:

* Galaxy分析平台
* GenePattern分析平台
* DNAnexus分析平台

[galaxy](https://galaxyproject.org)分析平台具有简单易用的特点, 除了有许多开放的公共平台可以使用之外, 搭建自己的平台并且维护其中的流程代码都比较容易, 本文主要介绍如何搭建自维护的平台;

## 初始环境搭建

galaxy分析平台基于python2搭建, 所以首先需要搭建好python环境, 这里推荐使用pyenv来管理python环境:

1.安装pyenv

```git
git clone https://github.com/yyuu/pyenv.git ~/.pyenv
```

2.配置pyenv

编辑`~/.bashrc`, 添加以下几条:

```bash
export PYENV_ROOT="/usr/local/var/pyenv";
export PATH="$PYENV_ROOT/bin:$PATH";
if which pyenv > /dev/null; then
    eval "$(pyenv init -)";
fi
```

并且令其生效:

```bash
source ~/.bashrc
```

3.接下来需要安装相应版本的python, 使用如下命令查看可供安装的python版本:

```bash
pyenv install --list
```

由于galaxy暂时不支持py3, 生物信息流程中会大量用到一些科学计算包, 所以推荐使用anaconda2, 如:

```bash
pyenv install anaconda2-2.4.0
```

4.接下来就可以使用pyenv来管理使用python了, 并且使用conda来安装一些常用包, 如`biopython`等:

```bash
pyenv global anaconda2-2.4.0
conda install biopython
```

## galaxy搭建

galaxy的搭建过程非常简单, 官网也给出了[详细的搭建过程](https://wiki.galaxyproject.org/Admin/GetGalaxy)。这里简单描述一下:

1. 下载galaxy软件包:

```git
git clone https://github.com/galaxyproject/galaxy/ ~/galaxy
```

2. 切换到稳定版本分支:

```bash
cd ~/galaxy
git checkout -b master origin/master
```

3. 确保python环境, 并且启动服务:

```bash
cd ~/galaxy
pyenv local anaconda2-2.4.0
sh run.sh
```

最后一步虽然简单, 但是包含了很多有关python虚拟环境的检查, 以确保整个平台正常运行, 查看`run.sh`文件, 可以具体看到整个平台是如何检查环境, 以及调用各种配置的, 方便根据这些过程简化工作。

等待片刻, 服务启动完成之后查看[http://localhost:8080](http://localhost:8080)在本地运行的平台。

此时只是本地版本, 并不能在外部访问, 按`ctrl-c`关闭服务后, 修改如下配置:

```bash
cp config/galaxy.ini.sample galaxy.ini
vi galaxy.ini
```

找到host一行, 添加:

```
host=0.0.0.0
```

重启服务后, 就可以从外部访问平台了。