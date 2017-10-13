Title: 集群管理
Date: 2017-10-13
Category: Pages
Tags: cluster, ganglia, fabric


本文介绍三种集群管理的工具:
1. ganglia
2. fabric
3. rsync+sshpass

## 1. 使用ganglia监控集群性能

ganglia是一款集群性能监控软件, 由一台主控节点和若干监控节点组成. 根据SGE集群的结构, 使用管理节点作为主控节点, 其他计算机作为监控节点.

ganglia不支持SELINUX, 所以首先要关闭SELINUX:

```
vi /etc/selinux/config
-> SELINUX=disable
```

重启生效.

同步服务器时间:

```
yum install -y ntpdate
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime -f
ntpdate us.pool.ntp.org
```

并且将时间同步加入计划任务:

```
echo "30 5 *  *  *  * ntpdate us.pool.ntp.org" >>/etc/crontab
service crond restart
```

为了各个节点之间的通讯, 需要建立一个UID相同的用户: ganglia, 如496:

```
groupadd -g 496 ganglia
useradd -u 496 ganglia -g ganglia
```

注意每个机器上都必须保持相同的UID和GID.

在主控节点安装:

ganglia是基于*rrdtool*数据库的, 所以需要安装*rrdtool*

```
# 安装rrdtools
yum install -y rrdtool rrdtool-devel
# 安装ganglia
yum install -y ganglia-web ganglia-gmetad ganglia-gmond
```

ganglia由gmetad和gmond两个组件组成, 主控节点运行gmetad监控各个监控节点的性能, 监控节点运行gmond来搜集性能信息.

在主控节点上, 配置*gmetad.conf*和*gmond.conf*两个文件.

```bash
cd /etc/ganglia
vi gmetad.conf
# 编辑gmetad.conf, 修改data_source行, 添加集群名以及集群的计算机名称, 默认的端口
-> data_source "bio_cluster" master:8649 exec_01:8649 exec_02:8649

vi gmond.conf
# 编辑gmond.conf, 按以下方式修改, 
# cluster的name必须和gmetad中的集群名称相同, owner默认为ganglia, 该用户需要在所有节点上都有相同UID和GID
# host的IP地址为主控节点的IP地址, 根据实际情况修改填写;
# prot默认使用8649端口,
-> cluster {
  name = "bio_cluster"
  owner = "ganglia"
  latlong = "unspecified"
  url = "unspecified"
}
-> udp_send_channel {
  host = 172.30.184.61
  port = 8649
  ttl = 1
}
-> udp_recv_channel{
  port = 8649
}
```

ganglia是一个web服务, 但是并不自带后台, 所以需要依赖于某一个web后台服务, 如Apache.

Apache的安装非常简单, 在系统软件仓库中直接安装即可:

```bash
yum install httpd
```

配置apache, 并指定分配一个URL给ganglia服务:

```bash
vi /etc/httpd/conf.d/ganglia.conf

-> <Location /ganglia>
  Order deny,allow
  Allow from all
</Location>
```

重启各个服务, 并且使其开机启动:

```bash
service gmetad restart
service gmond restart
service httpd restart

chkconfig  --level 35 gmetad on
chkconfig  --level 35 gmond on
chkconfig  --level 35 httpd on
```

自此, ganglia主控节点就已经配置完毕, 如果服务器上还有防火墙, 则需要将80端口(apache服务端口)以及8469端口(ganglia默认端口)开启.

ganglia的监控节点配置非常简单, 只需要安装gmond, 并且使用相同的gmond配置文件, 可以通过`async`的方式从主控节点传输. 配置完毕后, 开启gmond服务即可.

在浏览器中, 输入: http://{主控节点IP}/ganglia 查看服务器资源使用情况.

## 2. 使用fabric管理集群

fabric尚不支持python3, 所以安装前需要先切换到python2环境, 然后直接使用`pip`安装即可:

```
pip install fabric
```

fabric最简单的应用是直接运行一下常用的任务.

编辑*fabfile.py*文件, 在其中输入:

```python
from fabric.api import *


def first_task():
    with cd('/'):
        run("echo Hello World!")
```

上述代码中, first_task就是一个最简单的任务形式, 其中`cd`是fabric中的一个api, 顾名思义就是切换工作目录, 并且`cd`还具有上下文管理功能, 使用`with`语句可进行上下文管理.

在命令行中输入:

```bash
fab first_task
```

即可自动运行该任务.

其中`fab`是fabric的命令程序, 使用`fab --help`可以看到其使用说明. fabric默认会在当前目录下寻找fabfile.py来作为任务入口, 也可以通过`-f`参数来指定另外一个文件.

fabric的主要功能就是远程部署, 也就是直接在本地操作远程计算机上的内容. 

使用`fab -H [hosts]`的形式, 可以将远程计算机的地址传入进来, 当然同时还需要访问密码等.

更加常用的方式是, 使用装饰器, 在任务上指定运行的服务器.

如:

```python
from fabric.api import *

env.roledefs['remote_hosts'] = ['host1', 'host2', 'host3']
env.password = 'password'

@roles('remote_hosts')
def remote_task():
    run("echo OK")
```

如上形式, 先定义一系列远程服务器, 并给一个分组名称如`remote_hosts`, 在需要远程执行的任务前添加装饰器:`roles('remote_hosts')`, 这样在执行`fab remote_task`时, 就会自动到远程服务器上运行.

当远程服务器分组很多时, 可借助configparser来配置服务器.

首先建立*hosts*文件, 输入内容为:

```config
[cluster1]
user@host1:22 =
user@host2:22 =
user@host3:22 =

[cluster2]
user@host1:22 = 
user@host5:22 = 
user@host8:22 =
```

在*fabfile.py*中, 增加以下代码:

```python

from configparser import ConfigParser
config = ConfigParser()
config.read('hosts')
for section in config.sections():
    hosts = [host for host in config.options(section)]
    env.roledefs[section] = hosts

env.password = 'password'
```

即可利用*hosts*文件对远程服务器进行批量管理.

当一组中远程服务器较多时, 顺序进行比较浪费时间, `fabric`中也准备好了简单的并行处理装饰器:`parallel`, 也只需在任务前加入该装饰器即可.

```python
from fabric.api import *
from configparser import ConfigParser
config = ConfigParser()
config.read('hosts')
for section in config.sections():
    hosts = [host for host in config.options(section)]
    env.roledefs[section] = hosts

env.password = 'password'

@roles('remote_hosts')
@parallel(pool_size=10)
def remote_task():
    run("echo OK")
```

如上例中, 即可实现10个服务器同时处理任务.

命令也可以传参, 如:

```python
def task(a, b):
    run('echo %d' % (a + b))
```

可以通过:

```bash
fab task:a=1,b=2
```

这样的形式传参.

对于某些不常使用的任务, 可以使用通用的命令来运行, 如:

```python
@parallel(pool_size=10)
def run_command(command):
    run(command)
```

```bash
fab --roles cluster1 run_command:command="echo OK"
```

就可以对某个分组并行地运行同一个任务.

是不是有种指挥大军的感觉?

## 3. 使用rsync和sshpass传输文件

rsync是一个文件同步工具, sshpass可以使用密码的方式连接ssh, 直接在软件仓库中就可以安装这两个软件:

```bash
yum install -y rsync sshpass
```

使用方式非常简单, 如需要服务器中复制hosts文件:

```bash
RSYNC_PASSWORD="root_password"
for host in exec_01 exec_02 exec_03 exec_04
do
    sshpass -p $RSYNC_PASSWORD rsync -avz /etc/hosts root@${host}:/etc/hosts
done
```