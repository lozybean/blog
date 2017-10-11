Title: Linux文件下载方式汇总
Date: 2016-05-19
Category: pages
Tags: linux, download

## wget: 直观轻便地下载工具

`wget`是最简单的下载工具, 但是功能却很强大。

优点:

* 支持http/https/ftp
* 支持用户名密码验证
* 支持递归下载完整目录: -r
* 支持断点续传: -c
* 支持使用代理服务器下载

缺点:

* 不支持sftp
* 不支持多线程下载, 下载速度较慢

更多的功能可以man查看.

wget虽然简单易用, 但是一般而言只适用于小文件的快速下载, 过大的文件应该尝试其他的方式.

## Axel: 轻量多线程下载工具

wget的其中一个缺点是不支持多线程下载, axel工具相对wget弥补了多线程下载的功能缺陷(但是其他方面的特性比wget要少很多):

优点:

* 支持http/https/ftp
* 支持多线程下载
* 支持断点续传
* 支持代理服务器

缺点:

* 不支持sftp
* 不支持遍历目录, 必须指定确定URL
* 不支持用户名密码验证

axel使多线程下载加速成为可能, 但是必须建立在服务器支持的前提之下, 如果服务器不支持多线程下载, 则不论设置多少线程, 最终都只能使用单线程下载, 完全起不到加速的效果.

## lftp: 扩展sftp站点工具

前两种方式都不支持sftp下载, 需要下载sftp站点上的内容, 有以下三种备选方案:

* 当文件较小, 且环境允许的情况下, 使用FileZilla等可视化工具下载, 往往能够得到最好的体验
* 当环境不允许时(没有GUI), 可以使用sftp命令登陆到目标站点, 然后通过交互式命令

在文件较大, 需要长时间下载时, 以上两种都不是特别好的方式;

lftp工具是一个支持文件协议更加丰富的下载工具, 使用lftp可以比较轻松地下载sftp站点的内容:

优点:

* 支持ftp/ftps/http/https/hftp/sftp等
* 支持多线程下载
* 支持断点续传
* 支持同步

缺点:

* 是单独的shell, 自动化操作不方便
* 多线程下载命令pget不支持通配符
* 速度偏慢

lftp并不是一个简单的下载工具, 可以看做一个登录器, 使用lftp命令会开启一个单独的交互窗口, 然后通过在窗口中输入交互式命令来进行文件的上传/下载.

可以通过简单的脚本来实现lftp的自动化交互:

```bash
lftp -u ${USER},${PASS} sftp://${HOST} << EOF
cd /remote/path
lcd /local/path
mget -c *
pget -n 10 somefile.gz
bye
EOF
echo "done"
```

上例中, mget命令支持通配符, 但是不支持多线程, pget支持多线程, 但是不支持通配符. 和Axel一样, 多线程下载需要得到服务器支持.

## rsync: 更好的同步方案

如果服务器支持rsync服务的话, 应该优先考虑使用rsync.

rsync是独立的文件传输协议, 使用该协议同步的文件会保持和服务器上文件相同的时间戳, 是真正意义上的同步, 支持该协议的服务器在使用该服务传输文件时往往具有质变的下载速度。

## ASCP: NCBI御用下载工具

ascp是NCBI提供的文件下载工具, 建立在ssh和ftp的基础上, 使用前先下载安装Aspera, 并找到安装后的sshkey文件, 下面提供一个我实际使用的示例脚本, 专门用来下载NCBI的ftp站点内容, 并保持一致的目录结构:

```bash
#!/bin/bash
usage(){
	echo "Usage: $0 NCBI_FTP_FILEPATH"
	echo "for example: $0 /sra/sdk/2.3.2/sratoolkit.2.3.2-centos_linux64.tar.gz"
	exit 1
}

[[ $# -eq 0 ]] && usage

ascp=/home/galaxy/.aspera/connect/bin/ascp
ssh_key=/home/galaxy/.aspera/connect/etc/asperaweb_id_dsa.openssh
remote_path=$1
local_dir=${PWD}/$(dirname $1)
mkdir -p ${local_dir}

${ascp} -k 1 -T -l10M -i ${ssh_key} anonftp@ftp.ncbi.nlm.nih.gov:${remote_path} ${local_dir}
```

