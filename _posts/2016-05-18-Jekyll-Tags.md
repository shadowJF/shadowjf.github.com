---
layout: post_layout
title: Jekyll添加Tag分类
time: 2016年05月18日 星期三
location: 北京
pulished: true
tag: jekyll
excerpt_separator: "``"
---

好吧，我还是忍受不了blog没有tag分类，我会很抓狂，然后一直无法心安地想着这事，于是上网扒了扒其他人的实现

然后姑且实现了一版简单的tag分类。

结果如下：

![]({{site.pictureurl}}9.jpg?raw=true)

``

至于怎么做的呢？？

首先，修改menu.html

这里的menu.html也就是你们要做tag分类的那个页面，

我之前的menu.html是按如下方式实现：
    
	\\这里的%前我都加了\是因为不加的话，jekyll会把下面的代码编译执行，变成执行后的形式
    {\% for post in site.posts \%}
    <section class="smenu">
    {{ post.time  | date: "%Y/%m/%d" }}
    &nbsp;&nbsp;
    <a href="{{post.url}}">
    <span class="glyphicon glyphicon-hand-right">  </span>
    &nbsp;&nbsp; {{post.title}} &nbsp;&nbsp; 
    </a>
    </section>
    {\% endfor \%}


然后借助[https://github.com/Huxpro/huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)的帮助，我将其改为：


	<div class="row">
    <!-- 标签云 -->
    			<div id='tag_cloud' class="tags">
    				{\% for tag in site.tags \%}
    				<a  href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}</a>
    				{\% endfor \%}
    			</div>    
    
    <!-- 标签列表 -->
    			{\% for tag in site.tags \%}
    			<div class="one-tag-list">
    			  	<span style="color:#6A5ACD" class="fa fa-tag listing-seperator" id="{{ tag[0] }}">
    <span style="font-size:16px" class="tag-text">{{ tag[0] }}</span>
    </span>
    				{\% for post in tag[1] \%}
    				  <!-- <li class="listing-item">
    				  <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    				  <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    				  </li> -->
    				 <section class="smenu">
    					{{ post.time  | date: "%Y/%m/%d" }}
    					&nbsp;&nbsp;
    					<a href="{{post.url}}">
    						<span class="glyphicon glyphicon-hand-right">  </span>
    						&nbsp;&nbsp; {{post.title}} &nbsp;&nbsp; 
    					</a>
    				</section>
    				<hr>
    				{\% endfor \%}
    			</div>
    			{\% endfor \%}
    
    
    	</div>

可以看到，它这里用到了标签云，总之看上去的效果是：

![]({{site.pictureurl}}10.jpg?raw=true)

然后我们在单纯修改了menu.html之后，标签云并不会像上图中那么好看，而只是单纯的文本，那么我们还需要什么呢？

答案就是css和js

对于我一个前端白痴来说，我只能是照抄过来，原理不要问我 = =

首先，在css中加入如下代码：

```css
    .tags {
      margin-bottom: -5px;
    }
    .tags a,
    .tags .tag {
      display: inline-block;
      border: 1px solid rgba(255, 255, 255, 0.8);
      border-radius: 999em;
      padding: 0 10px;
      color: #ffffff;
      line-height: 24px;
      font-size: 12px;
      text-decoration: none;
      margin: 0 1px;
      margin-bottom: 6px;
    }
    .tags a:hover,
    .tags .tag:hover,
    .tags a:active,
    .tags .tag:active {
      color: white;
      border-color: white;
      background-color: rgba(255, 255, 255, 0.4);
      text-decoration: none;
    }
    @media only screen and (min-width: 768px) {
      .tags a,
      .tags .tag {
    margin-right: 5px;
      }
    }
    #tag-heading {
      padding: 70px 0 60px;
    }
    @media only screen and (min-width: 768px) {
      #tag-heading {
    padding: 55px 0;
      }
    }
    #tag_cloud {
      margin: 20px 0 15px 0;
    }
    #tag_cloud a,
    #tag_cloud .tag {
      font-size: 14px;
      border: none;
      line-height: 28px;
      margin: 0 2px;
      margin-bottom: 8px;
      background: #D6D6D6;
    }
    #tag_cloud a:hover,
    #tag_cloud .tag:hover,
    #tag_cloud a:active,
    #tag_cloud .tag:active {
      background-color: #0085a1 !important;
    }
    @media only screen and (min-width: 768px) {
      #tag_cloud {
    margin-bottom: 25px;
      }
    }
```

然后拷贝如下js文件到你自己的js目录
[https://github.com/Huxpro/huxpro.github.io/blob/master/js/jquery.tagcloud.js](https://github.com/Huxpro/huxpro.github.io/blob/master/js/jquery.tagcloud.js)

最后，在你的html中引用该js，加入如下代码

```js
    <script>
    function async(u, c) {
      var d = document, t = 'script',
      o = d.createElement(t),
      s = d.getElementsByTagName(t)[0];
      o.src = u;
      if (c) { o.addEventListener('load', function (e) { c(null, e); }, false); }
      s.parentNode.insertBefore(o, s);
    }
    </script>
    <!-- jquery.tagcloud.js -->
    <script>
    // only load tagcloud.js in tag.html
    if($('#tag_cloud').length !== 0){
    async('{{ "/assets/js/jquery.tagcloud.js" | prepend: site.baseurl }}',function(){
    $.fn.tagcloud.defaults = {
    //size: {start: 1, end: 1, unit: 'em'},
    color: {start: '#bbbbee', end: '#0085a1'},
    };
    $('#tag_cloud a').tagcloud();
    })
    }
    </script>
```

然后就可以了 = = 好累...