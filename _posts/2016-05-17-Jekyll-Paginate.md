---
layout: post_layout
title: Jekyll首页文章分页
time: 2016年05月17日 星期二
location: 北京
pulished: true
categories: Jekyll
excerpt_separator: "``"
---

Jekyll自带分页功能，不过只支持对首页的文章分页，也就是index.html中的文章，而如果想对某个分类下或某个标签下的文章分页，则是不支持的。

不过也无所谓，先对首页分页了再说

首先，在_config.yml中添加：
 
    gems: [jekyll-paginate]
    paginate: 5
    paginate_path: "page:num"

``

其中，paginate设置每页的文章个数

这时候，如果我们在本地用jekyll测试时

执行 jekyll -serve 

发现报错：

 Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one o
 its dependencies installed. In order to use Jekyll as currently configured, yo
'll need to install this gem. The full error message from Ruby is: 'cannot load
such file -- jekyll-paginate' If you run into trouble, you can find helpful res
urces at http://jekyllrb.com/help/!

这时候我们只要执行

gem install jekyll-paginate 即可

接着我们需要在原来的index.html中 添加如下代码：

    {% if paginator.total_pages > 1 %}
    <div class="pagination">
      {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&laquo; Prev</a>
      {% else %}
    <span>&laquo; Prev</span>
      {% endif %}
    
    {% for page in (1..paginator.total_pages) %}
    {% if page == paginator.page %}
      <em>{{ page }}</em>
    {% elsif page == 1 %}
      <a href="{{ '/index.html' | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
    {% else %}
      <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
    {% endif %}
    {% endfor %}
    
      {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">Next &raquo;</a>
      {% else %}
    <span>Next &raquo;</span>
      {% endif %}
    </div>
    {% endif %}

然后将原来的

for post in site.posts

改为

for post in paginator.posts

就大功告成了