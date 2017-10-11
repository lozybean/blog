Title: SGE极简安装指南
Date: 2017-07-19
Category: pages
Tags: linux, sge, sun grid engine


## 计算集群
通过集群软件, 将很多的计算机整合到一起, 使用者只需要在单一计算机投递任务, 任务即可根据资源状态分派到不同的机器上进行计算任务.

计算集群除了很小的本地硬盘之外(用于系统和存放工作环境所需的软件), 往往不具有大型存储, 需要结合分布式存储等存储方案使用.

### SGE介绍
SGE原名: Sun Grid Engine, 是Sun公司开发的集群管理工具. 在Sun公司被Oracle收购之后, 更名为Oracle Grid Engine, 并不再提供免费版本. 

Liverpool大学维护了一个分支, 改名为[Son of Grid Engine](https://arc.liv.ac.uk/trac/SGE)并保持着免费版本的更新.

本文在8.1.9(更新时间2016-03-02)版本上进行的极简安装说明.

### 集群组成
集群组成指计算集群在物理上的结构, 表示集群由哪些节点(计算机)组成.

本文方案由下列类型的节点组成集群:

1. 控制节点
运行sge_qmaster进程, 负责SGE软件管理的主机, 并负责集群资源管理分发等任务.

2. 计算节点
运行sge_execd进程, 负责进行实际计算任务, 一般而言具有较高性能.

3. 登陆节点
投递主机, 该主机是用户登入集群的入口, 用户在该主机投递任务, 进行日常操作等.


### SGE元素

SGE元素指在SGE软件中的名词概念:

1. admin user
可修改任务状态、SGE配置的用户.

2. user
一般用户, 可使用集群, 并投递任务的用户. 一般用户在第一次投递任务之后会自动加入到用户列表中. **所有用户必须在所有主机上添加, 并保持uid和gid一致**.

3. administrative host
只有当主机是admin hosts时, 才可以安装SGE软件, 查看和修改集群状态等. 控制节点必然是administrative host.

4. exec host
当主机是exec host时, 该主机执行具体计算任务, exec host必须是计算节点, 即必须运行sge_execd进程.

5. submit host
该主机是具有投递任务权限的主机, 一般而言, 所有节点都会设置为submit host.

6. queue
主机队列, 若干主机可以属于一个队列. 通过队列可以将不同类型的主机进行分类, 在投递任务时, 有针对性地投递到相应的队列上.

## SGE安装

### 安装环境

本文的教程环境, 使用虚拟化技术, 在实体机上最小化安装4台CentOS 6.7系统的虚拟机.

分别作为一台管理节点, 两台计算节点, 一台登陆节点. 

所有虚拟机都必须设置在相同局域网网段中, 本文中的网段为`172.30.184.0/24`.


### STEP1: 节点准备

所有节点添加HOSTS，注意HOST NAME和虚拟机名称必须相同. 主机名称可以使用`hostname`命令查看.

```bash
echo "
172.30.184.61 master
172.30.184.62 exec_01
172.30.184.63 exec_02
172.30.184.64 submit_01
" >> /etc/hosts
```

在所有节点上安装NFS.

```bash
yum -y install nfs-utils rpcbind
```

在所有节点上添加sgeadmin用户, 作为集群的admin user.

```bash
groupadd -g 501 sgeadmin
useradd -u 501 sgeadmin -g sgeadmin
```

在管理节点上启动NFS服务, 并使得/sge_rpm和/opt/sge两个目录共享.

```bash
echo "
/sge_rpm  172.30.184.0/24(rw,insecure,no_all_squash,no_root_squash,sync)
/opt/sge  172.30.184.0/24(rw,insecure,no_all_squash,no_root_squash,sync)
">/etc/exports

service rpcbind start
service nfs start
chkconfig rpcbind on
chkconfig nfs on
```

在管理节点上, 编辑/etc/sysconfig/iptables, 在reject上方添加以下内容:

```bash
# 其中2049和20049是NFS端口, 6444和6445是SGE相关端口, 根据实际环境做相应调整
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2049 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 2049 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 20049 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 20049 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6444 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 6444 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6445 -j ACCEPT
-A INPUT -m state --state NEW -m udp -p udp --dport 6445 -j ACCEPT
```

并重启防火墙

```bash
service iptables restart
```

在管理节点上, 下载RPM, 下载地址为: https://arc.liv.ac.uk/downloads/SGE/releases/

```bash
yum install -y wget
mkdir /sge_rpm
cd /sge_rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-8.1.9-1.el6.x86_64.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-debuginfo-8.1.9-1.el6.x86_64.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-devel-8.1.9-1.el6.noarch.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-drmaa4ruby-8.1.9-1.el6.noarch.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-execd-8.1.9-1.el6.x86_64.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-guiinst-8.1.9-1.el6.noarch.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-qmaster-8.1.9-1.el6.x86_64.rpm
wget -c https://arc.liv.ac.uk/downloads/SGE/releases/8.1.9/gridengine-qmon-8.1.9-1.el6.x86_64.rpm
```

在所有计算节点和登陆节点挂载`/sge_rpm`:

```bash
mkdir /sge_rpm
mount master:/sge_rpm /sge_rpm
```

在所有节点上安装rpm包:

```bash
cd /sge_rpm
yum install -y epel-release
yum localinstall -y gridengine-*
```

### STEP2: Master Host 

安装SGE qmaster, `->`开头的命令是指安装过程中的交互式过程简要描述, 除了`!`开头行之外, 可全部默认.

```bash
cd /opt/sge
./install_qmaster
-> Agree lincese
-> Install other than >root< >> y
-> ! enter a valid user name >> sgeadmin
-> Install path: /opt/sge [default]
-> SGE TCP/IP communication server: Using a network service [2]
-> Cell name: default [default] 
-> Cluster name: p6444 [default]
-> qmaster spool directory: /opt/sge/default/spool/qmaster [default]
-> Install Windows Execution Hosts: n  
-> Verify and set the file permissions: y  
-> All hosts in a single domain: y  
-> Enable the JMX MBean server: n  
-> spooling method: classic [default]
-> SGE group id range: 20000-20100 [default]
-> spool directory for execution hosts: /opt/sge/default/spool [default] 
-> ! administrator_mail: example@foo.com
-> change configuration parameters: n
-> start qmaster at machine boot: y
-> use a file which contains the list of hosts: n
-> ! adding admin and submit hosts:
Hosts: master
Hosts: exec_01
Hosts: exec_02
-> add shadow host: n
-> creating default queue: all.q
-> Scheduler Tuning: 1) Normal
-> Agree: y
```

**注意**: SGE安装的节点必须是admin hosts, 所以在计算节点安装完成之前, 所有的计算节点也必须是admin hosts. 若以上步骤没有添加计算节点, 则可以使用` qconf -ah exec_01` 命令, 将计算节点添加到admin hosts列表中.

添加环境变量, 本示例中, 使用BASH登陆服务器, 其他SHELL客户端参考安装最后一步的说明添加.

```bash
. /opt/sge/default/common/settings.sh
cp /opt/sge/default/common/settings.sh /etc/profile.d/sge.sh
```


### STEP3: 计算节点

挂载并自动挂载`/opt/sge`

```bash
mount master:/opt/sge /opt/sge
echo "master:/opt/sge /opt/sge nfs defaults 0 0">>/etc/fstab
```

添加环境变量

```bash
. /opt/sge/default/common/settings.sh
cp /opt/sge/default/common/settings.sh /etc/profile.d/sge.sh
```

安装SGE execd, 可一直使用默认值, 一路回车即可.

```bash
cd /opt/sge
./install_execd
-> Agree lincese
-> $SGE_ROOT: /opt/sge [default]
-> cell name: default [default]
-> SET sge_execd TCP/IP port: 6445 [default]
-> check hostname resolving
-> change spool directory: n [default]
-> start execd at machine boot: y [default]
-> starting execution daemon
-> add queue for this host: n
-> see previous screen: n
```

将计算节点添加为submit host

```bash
qconf -as exec_01
```

### STEP4: 登陆节点 

挂载并自动挂载`/opt/sge`

```bash
mount master:/opt/sge /opt/sge
echo "master:/opt/sge /opt/sge nfs defaults 0 0">>/etc/fstab
```

添加环境变量

```bash
. /opt/sge/default/common/settings.sh
cp /opt/sge/default/common/settings.sh /etc/profile.d/sge.sh
```

进入管理节点, 并输入以下命令:

```bash
qconf -as submit_01
```

即可将该节点设置为登陆节点.

## 集群管理

在admin host上, 使用管理员账号可进行集群管理. 主要使用`qconf`命令进行管理, 该命令完整的介绍见: [Grid Engine man pages](http://gridscheduler.sourceforge.net/htmlman/manuals.html).

一般而言, `qconf`命令的参数以`s`开头的是查看, `a`开头的是直接添加, 以`A`开头的是通过配置文件添加, 以`m`开头的是修改, 以`M`开头的是通过配置文件修改, 以`d`开头的是删除.

以下介绍添加队列时的一个trick.

可使用

``` 
qconf -sq queue_name >/tmp/tmp.q
```

命令, 将一个和将要添加的队列相近的队列信息打印到临时文件中, 并在该文件中修改队列名称、使用的shell、队列中的主机列表(slots项)等进行修改.

再使用

```
qconf -Aq /tmp/tmp.q
```

命令添加队列.