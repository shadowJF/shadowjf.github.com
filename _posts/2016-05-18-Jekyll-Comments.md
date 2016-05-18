---
layout: post_layout
title: Jekyll添加评论系统
time: 2016年05月18日 星期三
location: 北京
pulished: true
excerpt_separator: "``"
---


到目前为止，我觉得我的这个博客还差点东西，一是评论功能，一是文章按tag分类

对于我这个对前台一窍不通的的来说，要自己去实现个按tag分类文章的页面，目前还有点难度 = =|| ，所以先把评论功能搞定吧

Jekyll本身不带评论功能，它只是一个不带数据库的静态网站，为了支持评论功能，我们需要借助的别的社交评论平台。

像是Disqus、多说等等，前一个是国外的主要评论系统，而多说则是中文版Disqus

至于选择哪个，各有各的好处吧

``

Disqus评论时登录的账号都是国外常用的社交系统，像是facebook、twitter、disqus、google，而多说支持国内大部分社交系统的账号登录
所以优劣一目了然

另外，Disqus存在被墙的风险，虽然我现在能访问，但说不定什么时候就不能访问了。

但最后我还是选了Disqus，为啥呢。。。

因为网上看了篇文章，说多说会同步数据到他们自己的服务器，这样你写的文章博客就都成了他们的资源....

虽然我自己本身没有去验证过，但是印象分减10，那就愉快地决定用Disqus了

Disqus评论系统添加
----
首先，先去Disqus官网注册一个账号：

[https://disqus.com/](https://disqus.com/)

注册完后，点击右上角的设置按钮，选择

Add Disqus To Site

然后填写下面的信息：

![](/_assets/4.jpg)

点击next

出现如下信息

![](/_assets/5.jpg)

然后，选择personnal site，并且回答两个简单的问题后，就进入到下一步

![](/_assets/6.jpg)

这里你需要选择一个你的网站的类型，从而它会自动生成你需要的js代码

我们这里选择第一个**Universal Code**即可

接着就进入到最后一步：

![](/_assets/7.jpg)

在这里，我们把第一个框框里的代码拷贝下来，我的生成的代码如下，仅供参考：

    <div id="disqus_thread"></div>
    <script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
    this.page.url = {{page.url}};  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = {{page.title}}; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };*/
    
    (function() {  // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    
    s.src = '//shadowjf.disqus.com/embed.js';
    
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
    </script>
    <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>

然后，把这个代码，放到你的文章的layout的html代码中

至于上面那个注释说，最好是能把

    this.page.url = {}
    this.page.identifier = {}

替换成你的页面的动态生成的url和identifier，防止出现多线程的一些问题，可能造成评论丢失等问题。

但是我不知道怎么把当前页面的url和文章title设置到JS代码中 = =||

所以我没有替换，因为这个问题只出现在一个页面能通过多个url访问的情况中，所以我不管它也是能行的。。。。

好了最后，我们来看看效果吧

![](/_assets/8.jpg)

可以看到，效果还是挺美的，而且我没有改任何配置，因为我觉得他肯定是支持自定义外形的，正好和我blog还挺搭的

好了，快来给我评论吧  ^_^