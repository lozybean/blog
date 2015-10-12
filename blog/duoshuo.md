Title: 在pelican中使用多说
Date: 2015-09-18
Category: pages
Tags: comment, pelican, duoshuo

本文记录在pelican中实现评论系统的过程

[`disqus`](https://disqus.com)是pelican默认支持的评论系统，然而由于墙内的原因，访问总是会有些问题（最近ss也不行了），国内的[`多说`](http://duoshuo.com)也是不错的选择，为了使用`多说`，首先需要在多说上创建一个账号，填写使用的站点等信息之后，多说将会自动提示复制以下js代码：

```html
<!-- 多说评论框 start -->
	<div class="ds-thread" data-thread-key="请将此处替换成文章在你的站点中的ID" data-title="请替换成文章的标题" data-url="请替换成文章的网址"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"lyon0804"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0]
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
<!-- 多说公共JS代码 end -->
```

根据该代码内容的提示，需要替换以及修改部分内容，如：

```html
<!-- 多说评论框 start -->
	<div class="ds-thread" data-thread-key="{{ article.slug }}" data-title="{{ article.title }}" data-url="{{ SITEURL }}/{{ article.url }}"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"lyon0804"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0]
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
<!-- 多说公共JS代码 end -->
```

## 在pelican网页模板中添加代码

由于每个主题生成的最终网页不同，pelican的网页模板当然是存放在每个主题中，根据下面的思路：

1. 找到pelican的安装地址，如：`/usr/local/Cellar/python/2.7.10_2/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages/pelican`,主题的默认安装位置在该目录下的`themes`中，到主题目录中，找到自己使用的主题，发现有一个`templates`文件夹，这里面肯定就是存放模板了；
2. 由于评论应该显示在文章中，所以想当然应该查看一下`article.html`中的内容，找到显示文章主题的部分，并且在最后，发现以下内容：`{% include 'includes/comments.html' %}`，所以应该是在该文件中加载评论信息；
3. 编辑该文件，第一行显示：`{% if DISQUS_SITENAME %}`，应该是使用全大写的全局常量来控制是否显示评论，由于默认支持的是`Disqus`，这里检查的也就是`DISQUS_SITENAME`是否设置了；

找到模板之后就简单了，照葫芦画瓢，在相同文件下，添加以下内容：

```html
{% if DUOSHUO_SITENAME %}
    <hr/>
    <section class="comments" id="comments">
        <h2>Comments</h2>
		<div class="ds-thread" data-thread-key="{{ article.slug }}" data-title="{{ article.title }}" data-url="{{ SITEURL }}/{{ article.url }}"></div>
		<script type="text/javascript">
    		var duoshuoQuery = {short_name:"{{ DUOSHUO_SITENAME }}"};
    		(function() {
    			var ds = document.createElement('script');
    			ds.type = 'text/javascript';ds.async = true;
    			ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    			ds.charset = 'UTF-8';
    			(document.getElementsByTagName('head')[0]
    			 || document.getElementsByTagName('body')[0]).appendChild(ds);
    		})();
    	</script>
    	<noscript>Please enable JavaScript to view the <a href="http://duoshuo.com/">comments powered by
        Duoshuo.</a></noscript>
    	<a href="http://duoshuo.com" class="dsq-brlink">comments powered by <span>Duoshuo</span></a>

    </section>
{% endif %}
```

其中最后第二行当用户关闭js的时候弹出提示，最后一行显示多说的链接，但是如果想要显示多说的logo，还需要在css中添加多说logo的字图；

接下来，到pelicanconf.py中，添加DUOSHUO_SITENAME，让这段代码通过判断，试试效果吧~

## 异步加载

为了让评论模块不影响页面加载速度，使用异步加载的方式，点击显示评论的时候才加载评论框，js还没怎么学，幸好已经[有人](http://liam0205.me/2014/07/22/duoshuo-delay/)已经在hexo上实现了，根据pelican的特点，模仿者写了一段js：

```html
<script type="text/javascript">
    function toggleDuoshuoComments(container, id, title, url){
        if(jQuery(container).has("div").length>0){
            jQuery(container).empty();
            return;
        }
        var el = document.createElement('div');
        el.setAttribute('class','ds-thread');
        el.setAttribute('data-thread-key', id);
        el.setAttribute('data-title',title);
        el.setAttribute('data-url', url);
        DUOSHUO.EmbedThread(el);
        jQuery(container).append(el);
    }
</script>
<a href="javascript:void(0);" onclick="toggleDuoshuoComments('#comment-box', '{{ article.slug }}', '{{ article.title }}' , '{{ SITEURL }}/{{ article.url }}');">
查看评论</a>
<div id="comment-box" ></div>
<hr/>
```

然后将之前的评论框加载注释掉：

```html
<!--         <div class="ds-thread" data-thread-key="{{ article.slug }}" data-title="{{ article.title }}" data-url="{{ SITEURL }}/{{ article.url }}"></div> -->
```

就完成异步加载的工作了

## 代码一览

最终总体代码看起来就像这样：

```html

{% if DUOSHUO_SITENAME %}
    <hr/>
    <section class="comments" id="comments">
        <h2>Comments</h2>
<!--         <div class="ds-thread" data-thread-key="{{ article.slug }}" data-title="{{ article.title }}" data-url="{{ SITEURL }}/{{ article.url }}"></div> -->
		<script type="text/javascript">
     		var duoshuoQuery = {short_name:"{{ DUOSHUO_SITENAME }}"};
    		(function() {
    			var ds = document.createElement('script');
    			ds.type = 'text/javascript';ds.async = true;
    			ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    			ds.charset = 'UTF-8';
    			(document.getElementsByTagName('head')[0]
    			 || document.getElementsByTagName('body')[0]).appendChild(ds);
    		})();
    	</script>
        <script type="text/javascript">
            function toggleDuoshuoComments(container, id, title, url){
                if(jQuery(container).has("div").length>0){
                    jQuery(container).empty();
                    return;
                }
                var el = document.createElement('div');
                el.setAttribute('class','ds-thread');
                el.setAttribute('data-thread-key', id);
                el.setAttribute('data-title',title);
                el.setAttribute('data-url', url);
                DUOSHUO.EmbedThread(el);
                jQuery(container).append(el);
            }
        </script>
        <a href="javascript:void(0);" onclick="toggleDuoshuoComments('#comment-box', '{{ article.slug }}', '{{ article.title }}' , '{{ SITEURL }}/{{ article.url }}');">
        查看评论</a>
        <div id="comment-box" ></div>
        <hr/>
    	<noscript>Please enable JavaScript to view the <a href="http://duoshuo.com/">comments powered by
        Duoshuo.</a></noscript>
    	<a href="http://duoshuo.com" class="dsq-brlink">comments powered by <span>Duoshuo</span></a>

    </section>
{% endif %}
```
