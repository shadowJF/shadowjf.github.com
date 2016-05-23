---
layout: post_layout
title: Grok实战
time: 2016年05月23日 星期一
location: 北京
pulished: true
tag: logstash
excerpt_separator: "``"
---

Grok是Logstash中最重要的Filter，它是将非结构化数据转化为结构化数据的关键。

它的功能，就是利用ruby的正则表达式来匹配event，将event中的数据拆分成一个个的字段，因此为了学会Grok，我们得知道ruby里的正则表达式是如何进行匹配的。

### **Grok基本用法** ###

Grok通过将一些text patterns组合在一起，来匹配你的日志内容

一个grok pattern的基本形式为： %{SYNTAX:SEMANTIC}

``

#### **SYNTAX** ####

其中，SYNTAX是pattern的名字，而这些pattern既可以是官方预先定义好的120多种pattern，参考[https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)，也可以是你自己定义好的，至于自己怎么定义，怎么使用自定义的pattern，可以往后看

举个例子，最基本的，3.44将会被**NUMBER**类型的pattern所匹配，而172.20.14.81将会被**IP**类型的pattern所匹配，而当你查看上面官方pattern的定义后你会发现，其实他们的本质如下：

BASE10NUM (?<![0-9.+-])(?>[+-]?(?:(?:[0-9]+(?:\.[0-9]+)?)|(?:\.[0-9]+)))
NUMBER (?:%{BASE10NUM})

IP (?:%{IPV6}|%{IPV4})

（IPV6太长，就不贴出来了，自己去看）

可以看到其实，本质上就是正则表达式而已

#### **SEMANTIC** ####

SEMANTIC则是你赋给这段被匹配的文本的标识符，或者叫字段名。例如，3.44可能代表一个事件的持续时间，因此你给它命名为duration，172.20.14.81代表客户端的ip，因此你给它命名为client，那么，为了匹配如下数据：

	3.44 172.20.14.81

我们需要使用这样的grok表达式：
	
	%{NUMBER:duration} %{IP:client}

#### **类型转换** ####

有的时候，你希望得到的字段的数据类型不是string而是int或者float，那么，你可以在pattern的后面再加一个转换类型，例如

	%{NUMBER:num:int}

这样之后，就会将num转换为int类型，不过目前，只支持转换为int和float

#### **自定义Patterns** ####

虽然说官方已经给我们定义好了120多个pattern，几乎涵盖了我们所需要的内容，但是，我们难免会有自己的数据没法被这些pattern解析，这时候，我们有两种方式来解决：

1. 使用如下形式来匹配
	
		(?<field_name\>the pattern here)

	譬如，我们想要匹配一个queue id，它是长度为10或者11的16进制数，那么我们可以通过这样的模式来匹配：

		(?<queue_id>[0-9A-F]{10,11})

2. 自己像官方一样提前定义好模式，再使用自己的模式匹配
	
	首先，创建一个patterns目录，然后在里面随便创建一个文件，文件名随意

	在该文件中，写下你自己想要使用的patterns，每个pattern的模式为，一个模式名，一个空格，它对应的正则表达式

	例如，对于上面提到的queue id，我们可以添加这样一个模式

		POSTFIX_QUEUEID [0-9A-F]{10,11}

	那么我们在使用grok时，需要制定patterns_dir

		filter {
  			grok {
    			patterns_dir => ["./patterns"]
    			match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
  			}
		}

	其中，SYSLOGBASE、GREEDYDATA都是官方预设的模式，而POSTFIX_QUEUEID是我们自定义的模式，这样我们就能对如下日志内容进行匹配了：

		Jan  1 06:25:43 mailserver14 postfix/cleanup[21403]: BEF25A72965: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>

#### **实战** ####

场景：

一个日志目录下，有三种类型的日志，分别为

- system.log
- business.log
- vertx.log

每种日志的格式还不一样，我要通过一个logstash实例统一对这三类日志进行处理，需要如何设计配置文件呢？

我们先把最后的结果列出来：

    input {
    	file {
    		path => "D:/DevTools/logstash-all-plugins-2.3.1/logstash-2.3.1/logs/gateway-logs/*.log"
    		start_position => beginning
    		ignore_older => 0
    	}
    }
    filter {
    	if [path] =~ "server.system" {
    		mutate { replace => { type => "server_system" } }
    		grok {
    			match => {"message" => "%{TIMESTAMP_ISO8601:logtime} %{LOGLEVEL:loglevel}  %{JAVACLASS:javaclass} - %{GREEDYDATA:logmessage}"}
    		}
    	}else if [path] =~ "server.business" {
    		mutate { replace => { type => "server_business" } }
    		grok{
    			patterns_dir => ["../patterns"]
    			match => {"message" => "%{TIMESTAMP_ISO8601:logtime} %{LOGLEVEL:loglevel}(\s)+%{JAVACLASS:javaclass} - %{GREEDYDATA:busiop} %{BUSI_RESULT:busires} -> accountID=%{GREEDYDATA:accountid},gatewayID=%{GREEDYDATA:gatewayid},cloudService=%{MYGREEDYDATA:cloudservice}(,resultSize=%{INT:resultsize})?(,timeCost=%{INT:timecost})?(,statusMessage=%{GREEDYDATA:statusmessage},statusCode=%{GREEDYDATA:statuscode})?"}
    		}
    	}else if [path] =~ "server.vertx" {
    		mutate { replace => { type => "server_vertx" } }
    		grok{
    			match => {"message" => "%{TIMESTAMP_ISO8601:logtime} %{LOGLEVEL:loglevel}  %{JAVACLASS:javaclass} - %{GREEDYDATA:logmessage}"}
    		}
    	}
    	
    	date {
    		match => [ "logtime", "yyyy-MM-dd HH:mm:ss,SSS" ]
    	}
    	
    	
    }
    output {
    	if "_grokparsefailure" not in [tags] {
    		elasticsearch {}
    	}else{
    		stdout { codec => rubydebug }
    	}
    }
	
首先，我们对于input要指定，日志文件存放的路径，start\_position要设为beginning，ignore_older设为0，因为不设为0，默认会忽略一天以上没有修改过的日志，那么可能以前的日志就不会被处理

接着在Filter中，我们首先通过条件判断，来区分出不同的日志类型

这里我们用的判断条件是

	[path] =~ "server.system"

这里path是每个event对应的文件路径， =~代表匹配，也就是如果路径中包含有server.system字符串，那么就会按照system日志去处理，同理，business日志和vertx日志也会按照自己的逻辑去处理。

接下来我们只需要针对每类日志的格式，设计自己的处理方式

接下来我们介绍我是如何对业务日志进行处理的：

我们的业务日志格式如下：

	2016-05-20 14:56:30,193 INFO  com.yonyou.nccpub.gateway.rmcdispatcher.handler.RmcDispatchHandler - ERP remote call Succeed -> accountID=oojTT8fX,gatewayID=37f162f7-3a32-4b7a-8c42-9bed714d5f1c,cloudService=null

当然，如果业务操作失败，那么这此之后还会带上statusMessage和statusCode信息

如果业务操作成功，且有返回值，那么这之后还会带上resultSize和timecost信息

现在来看我使用的grok模式为：

	grok{
			patterns_dir => ["../patterns"]
			match => {"message" => "%{TIMESTAMP_ISO8601:logtime} %{LOGLEVEL:loglevel}(\s)+%{JAVACLASS:javaclass} - %{GREEDYDATA:busiop} %{BUSI_RESULT:busires} -> accountID=%{GREEDYDATA:accountid},gatewayID=%{GREEDYDATA:gatewayid},cloudService=%{MYGREEDYDATA:cloudservice}(,resultSize=%{INT:resultsize})?(,timeCost=%{INT:timecost})?(,statusMessage=%{GREEDYDATA:statusmessage},statusCode=%{GREEDYDATA:statuscode})?"}
		}

首先，我们用TIMESTAMP_ISO8601来匹配时间戳，并置为logtime字段

接一个空格

然后我们用LOGLEVEL来匹配日志级别，并设为loglevel字段

接至少一个空格 (\s)+ 这里没有继续接一个空格，是因为当log级别不一样时，这里产生的空格数量不一样

然后用JAVACLASS匹配java类，并设为javaclass字段

接一个 - 

然后用GREEDYDATA匹配业务操作类型，设为busiop

接一个空格

然后用BUSI\_RESULT匹配业务操作结果，设为busires，然而这里的BUSI_RESULT并不是官方预设的pattern而是我自己定义的，可以参考最后的patterns文件

接一个 ->

然后用GREEDYDATA匹配accountid，gatewayid

用MYGREEDYDATA匹配cloudservie

这里MYGREEDYDATA也是我自己定义的模式，它与GREEDYDATA的唯一区别就是MYGREEDYDATA不能包含逗号，这里为什么要这么做呢？

因为到这里为止，前面的内容都是日志中必定出现的内容，因此匹配不会出问题，但是当我们开始进一步匹配后面的resultSize、timecost、statusMessage、statusCode时，他们是有可能出现，有可能不出现的。

所以我们都用圆括号括起来，并加上?，代表有可能出现一次，也有可能不出现。

但是当日志中出现resultSize时，GREEDYDATA是贪婪匹配，它会把resultSize的内容也匹配到cloudservice中去，这样便会导致resultsize匹配不到具体内容，因此，要使用MYGREEDYDATA，来防止这种情况的出现。

>我的自定义模式文件

	BUSI_RESULT (Succeed|Failed)
	MYGREEDYDATA ([^,])*

当然，在grok处理后，我通过mutate filter来将各类日志的type字段的值替换为相应的类型，这样，方便对不同类型的日志进行分类处理

**日期处理**

可以看到，在对三个日志进行grok处理后，最后我还加了date filter

这个date filter是干啥的呢？

其实他就是对event的时间来进行规范化的

首先，我们要知道，logstash处理完事件，默认会打上一个时间戳，当你没有使用date filter时，这个时间戳的值就是logstash处理该事件的时间

但是，加入在这样一个场景下，我们有一批老的日志，现在需要交给logstash处理，它们实际产生的时间都不一样，但是经由logstash处理后，发现时间戳都变成今天处理的时间了，这对我们进行日志分析是没有意义的。

所以我们希望event的时间戳能和日志中的时间戳一致，而这就是date filter能做的事

我们只要指定event中解析出的日志产生时间戳的字段，再指定一个format就行了，而这个format必须按照[https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html](https://www.elastic.co/guide/en/logstash/current/plugins-filters-date.html)中的规则来写，大小写都不能错，而且很坑爹的是你会发现，年月日的大小写还不一样，我当时就在这调了好久。。

**解析失败处理**

有时候我们的日志中难免会有非定制格式的数据出现，例如，异常信息会分布在多行时，这时很有可能出现grok解析出错的情况

这时，我们只要在output中添加条件判断，

	if "_grokparsefailure" not in [tags]{
	}

grok处理失败的话，会在event中添加一个字段tags，里面有一个\_grokparsefailure的值，所以我们可以通过这个来判断是否解析成功，解析成功的就放到elasticsearch中，解析不成功的就打印到控制台



最后，推荐一个很好用的grok匹配测试的网址：

[http://grokdebug.herokuapp.com](http://grokdebug.herokuapp.com)

有了它，就可以快速方便的调试，你使用的grok模式是不是能正确匹配你的日志文本了。而且，它也支持自定义pattern，所以非常赞，如果没有这个工具，真不知道要调到啥时候才能让logstash正确解析日志了。



