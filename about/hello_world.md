Title: hello world!
Date: 2015-09-11
Category: about
Tags: pelican, gitcafe, gitpages

```python
print('hello world!')
```

使用gitcafepages + pelican + Macdown 搭建第一个博客：

1. 注册使用gitcafe pages
2. 使用pelican
3. 第一篇文章！
4. 更改主题
5. 添加插件
6. 域名绑定
7. 一些好用的工具
8. 感谢

# 1. 注册使用gitcafe pages：

到`gitcafe`上申请一个账号，如：username，新建仓库：username，gitcafe就自动识别为pages项目啦~

为什么没有用github呢？github访问速度不是很稳定啊（GFW你懂的），并且虽然只是自己的博客，但是不仅仅要考虑自己，也要考虑访问者的体验，所以还是选择gitcafe了。

有了仓库之后，需要添加ssh-key，这样，就可以建立ssh连接，在本地更改远程仓库了。

新建ssh-key：

```bash
ssh-keygen -t rsa -C "youremail@example.com"
```

接着会提示保存的路径以及文件名，密码就留空好了，完成之后，将.pub结尾的那个公钥的内容添加到gitcafe账户中（在settings/SSH keys里面可以找到~），gitcafe pages就建立完成啦~

# 2. 安装pelican

Pelican是一套开源的博客生成软件，使用python编写，所以安装pelican和安装一般的python包是一样的，相信小伙伴们都有各种不同的方式安装了~

比如，我的环境是macos，使用homebrew管理软件，使用pip安装python包：

```bash
brew install python
pip install pelican
pip install markdown
pip install ghp-import
```

markdown支持包是必须的~ 这个ghp-import是可选的，如果想要使用默认的`make github`命令，则需要安装这个包，我感觉还是自己写几行脚本容易理解...

如果成功的话就太好了，不过由于某GFW的原因，pelican可能安装会有一些慢（包括其他的python包），通过使用国内的镜像可以大大提高速度，如使用豆瓣源：

```bash
pip install pelican -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```

pelican安装好之后，选择一个目录就可以初始化博客啦：

```bash
mkdir blog
cd blog
pelican-quickstart
```

之后会问你一系列的问题，按照要求回答就好啦，回答完成之后，会建立以下目录格式：

```
blog/
┣ Makefile             #一些设置，如FTP、SSH等服务器的设置，之前的问题中也会有，如果对自己的回答不满意，也可以在这里更改
┣ content              #博客页面的源文件目录
┣ develop_server.sh    #用于开启测试服务器的脚本
┣ fabfile.py           #一些脚本的配置
┣ output               #生成的输出文件
┣ pelicanconf.py       #主要配置文件
┣ publishconf.py       #发布脚本
```

然后，到`output`文件夹下面，添加gitcafe仓库，并且设置本地git配置，git的教程大家可以看廖雪峰的博客：[廖雪峰的官方网站](http://www.liaoxuefeng.com)。

```bash
git config --local user.name 'yourname'
git config --local user.email 'youremail@example.com'
git init
git remote add gitcafe 'git@gitcafe.com:usrname/usrname.git'
git checkout -b gh-pages
touch Readme.md
git add *
git commit -m 'init repo'
git push -u gitcafe gh-pages
```

注意了，这里的分支一定要填写gh-pages，不仅可以兼容github，而且这样才可以启用gitcafe的pages服务，master是不能用pages服务的。

不同的情况下，添加远程仓库的方式会不一样，但是总之这一步要正确添加远程仓库就是了~

完成这些之后，就可以动手写博客啦~

# 3. 第一篇文章！

随便选择并且使用一款markdown编辑器，动手写吧~

在你的博客开头，添加以下的内容：

```
Title: Hello World
Date: 2015-09-11
Category: pages
Tags: pelican, gitcafe, gitpages
```

这些内容大家应该能猜到是什么作用了吧？

再下面就是你的博客正文啦，在完成大作之后，到项目目录下，执行：

```bash
cd ..
make html
```

此时，就在output文件夹下面成功生成了博客页面了，在项目路径下，执行：`sh develop_server.sh start`,然后通过[127.0.0.1:8000](127.0.0.1:8000)就可以看到你的页面效果啦~

当然，这个时候博客还是在你的本地，如果想让其他人也访问到你的博客的话，就需要同步到gitcafe pages上，到output文件夹下面，执行：

```bash
git add -A
git commit -m 'my first page'
git push -u gitcafe gh-pages
```

然后访问`yourname.gitcafe.io`即可成功访问第一篇文章啦~

但是你一定不满足那么多的操作步骤吧？下面编辑Makefile，定制我们的工作流程：

编辑Makefile，添加以下几行：

```makefile
gitcafe: publish
    cd $(OUTPUTDIR); git add -A; git commit -m 'Generate Pelican site'; git push -u gitcafe gh-pages
```

同时，修改`publishconf.py`，将其中的`DELETE_OUTPUT_DIRECTORY`设置为`False`，否则每次`make publish`的时候，都会把output目录删除掉，这对我使用自定义的make命令会造成困扰，不如自己管理`git`仓库。

做完上述工作，以后就可以通过下面的命令一键同步啦：

`make gitcafe`

# 4. 更改主题

可以用一些其他人已经设计好的主题，如使用pelican-bootstrap3主题等；首先需要下载主题：

```bash
mkdir themes
cd thems
git clone git://github.com/getpelican/pelican-themes.git ./
```

在其中挑选一个主题，并且使用下面的命令安装：

```bash
pelican-themes -i pelican-bootstrap3
pelican-themes -l
```

第二条命令可以看到当前安装的主体列表，选择想要更改的主体，并且修改`pelicanconf.py`，添加`THEME = 'pelican-bootstrap3'`一行，然后重新`make html`，试试效果吧~

# 5. 添加插件

首先还是先下载插件:

```
mkdir plugins
cd plugins
git clone git://github.com/getpelican/pelican-plugins.git ./
```

然后在`pelicanconf.py`中，以安装sitemap为例，修改下面的内容：

```python
PLUGIN_PATHS = [u'plugins']
PlUGINS = ['sitemap']

SITEMAP={
    'format' : 'xml',
    'priorities' : {
        'articles' : 0.7,
        'indexes' : 0.5,
        'pages' : 0.3,
    },
    'changefreqs':{
        'articles' : 'monthly',
        'indexes' : 'daily',
        'pages' : 'monthly',
    }
}
```

之后再执行`make html`，通过[http://127.0.0.1:8000/sitemap.xml](http://127.0.0.1:8000/sitemap.xml)即可看到效果啦~

# 6. 申请购买域名

到这里为止，博客的基本内容就完成的差不多了，但是个性化的域名还是比较重要的，到[`Godaddy`](https://www.godaddy.com)上面，注册并且申请一个域名，通过支付宝付款购买之后，就有自己的独立域名了，但是光有域名还不行，还要和空间绑定起来，这里其实就拿`gitcafe`当做空间，做一个绑定；

首先，到[`DNSPOD`](https://www.dnspod.cn)上面，注册一个账号，然后添加域名解析服务，添加刚购买的域名，会将之前的信息导入，此时，会新增两条NS记录，到`Godaddy`上的域名管理界面，在`DNS Manager-Settings`中，将`Nameservers`修改为`custom`，并且添加在DNSPOD上产生的两条NS记录；之后，解析的工作就交给`DNSPOD`了；添加一个A记录，指向地址为：`117.79.146.98`，添加一个CNAME记录，主机记录为`www`，记录值：`gitcafe.io`；到这里就完成域名部分的工作了；

回到`gitcafe`的项目管理目录，在`Pages 服务`下面，添加购买的域名，访问你购买的域名，看看是不是有效果了~

# 7. 一些好用的工具

好用的markdown编辑器：MacDown，也就是我现在正在使用的编辑器，安装非常简单：

```bash
brew cask install macdown
```

就完成啦，关键还是免费开源的，必须要赞一个~

国内可用的图床：`新浪微博`

# 8. 感谢

第一次搭建博客，花了整整一天时间，期间借鉴了不少前辈的经验：`lizherui,cold,poem_of_sunshine`，非常感谢~