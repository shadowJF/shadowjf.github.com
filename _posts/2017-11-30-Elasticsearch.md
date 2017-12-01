---
layout: post_layout
title: Elasticsearch
time: 2017年11月30日 星期四
location: 北京
pulished: true
tag: open source
excerpt_separator: "----"
---

　　最近要做搜索suggest相关的东西，所以学了一下elasticsearch，之前对于es只在用ELK进行日志实时收集展示的时候用过，这次能够系统的学习一下也是不错的。

----

# **Elasticsearch概述** #

　　Elasticsearch是一个高度可扩展的开源全文搜索分析引擎，它能用来近乎实时的存储、搜索、分析大量数据。通常都是为有复杂的搜索需求的应用来服务的。

### **基本概念** ###

　　**Cluster**

　　一个集群就是一组服务器的集合，每个集群都有一个独特的集群名，默认集群名是“elasticsearch”，当然你可以通过命令行或者改配置文件的方式修改集群名。各个不同的集群必须设置不同的名称，以防节点在加入集群时加入到别的集群去了。

　　当然集群也是允许只有一个节点存在的。

　　**Node**

　　一个节点就是你的集群中的一台服务器，用来存储数据，同时参与到集群的索引和搜索服务中。跟集群一样，节点也是由名称来标识的，每个节点默认的名称是一个uuid，会在启动节点是自动生成，当然你也可以自己指定你想要的节点名

　　**Index**

　　一个索引就是一组文档的集合，索引也是通过名字来标识（必须全部小写）

　　**Type**

　　type是es以前版本的一个概念，现在我用的是6.0.0，type概念已经开始逐渐被废弃掉，type之前是用来作为每个索引中的逻辑分区，来允许你在同一个索引中存储不同类型的文档，而在最新的es中，建议一个索引就一个type，如果是不同类型的文档，就建多个索引就是了（虽然被废弃了，但是实际中还是有使用到，就将type和index名指定为一样就行，可能在后续版本中会完全废弃掉）

　　**Document**

　　文档就是能被索引的信息的基本单元，是通过JSON格式来表述。

　　Shards&Replicas

　　一个索引可能包含很多数据，那么这些数据可能在一个节点上都放不下，为了解决这个问题，es能将一个索引分成多份，这每一份就是一个shards，通过配置文件可以设置创建索引时默认分配多少个shards，也可以在创建索引时，直接指定多少个shards

　　有了shards，能带来两个主要的好处：

　　１.水平扩展
　　２.分布式并行的操作，增加吞吐率

　　而Replicas就是备份啦，当然就是用来容错的，当一个节点down了，不至于数据无法使用，因为有备份就可以继续使用别的节点上的数据了。

　　shards和replicas都可以在创建索引时指定，但是一旦创建，就不能修改shards的数量了，但是reolicas是随时都可以改的。

　　如果不指定数量，es默认是给每个索引分配5个shards和1个备份（即总共两份数据）。在这种情况下，当你有两台以上节点时，每个索引会有5个主要shards和另外5个replica shards，也就是总共10个shards

### **安装** ###
　　这里，我准备搭建一个两个节点的集群环境。

　　首先，es现在是到6.0.0版本了，需要有java8的环境，jdk版本得高于1.8.0_131，所以先保证下这个

　　然后，去[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)下载安装包，下载完后，解压，如果是单机测试，解压完，进到bin目录，直接：

	./elasticsearch

　　启动，即可

　　但是，如果作为生成环境使用，那么集群是必须的，而搭建集群的话，是需要修改一些配置的

　　进入config目录，我们可以看到三个配置文件：

	elasticsearch.yml //es配置
	jvm.options //es使用的jvm配置
	log4j2.properties //log4j日志配置

　　这里我主要说下elasticsearch.yml怎么配置，至于jvm和log4j的配置，自己去看看就行。

　　**path.data & path.logs**

　　这两个参数，指定日志和数据的存储位置，这个如果不指定，默认是在$ES_HOME下的子目录，这样，如果进行es升级什么的，很容易导致数据丢失，所以我们最好给data和log建立两个特定位置的目录，然后通过配置文件设置好位置

	path.data: /data/esData/data
	path.logs: /data/esData/logs

　　**cluster.name**

　　该参数指定节点所要加入的集群名，如果不指定默认为elasticsearch，所以最好是修改为你自己的集群名，防止别人的节点加入等等错误情况的发生。

	cluster.name: xxx

　　**node.name**

　　节点名，如果不指定，会默认生成uuid，并取前7位字符，但是最好自己也指定有意义的节点名

	node.name: es-node-1

　　**bootstrap.memory_lock**

　　操作系统会将不经常使用的内存swap到硬盘上，这样对es的性能影响会较大，因此将该参数设为true可以将内存锁住，不会swap到磁盘上，但是也有别的方法来达到此目的，参考[https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#mlockall](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#mlockall)

　　**network.host**

　　默认情况下，这个是绑定到回路地址，即只能通过localhost或127.0.0.1去访问es，而外面是无法访问到它的，所以需要将该地址设为本机的ip地址或者0.0.0.0，这样才能组成集群，同时让外面能访问到。

　　在不修改该值的时候，es认为你是在测试环境，有很多检测即使不通过也会让你启动起来，但是一旦你改了该参数，es就会认为你现在工作在生产模式下，那么它在启动时会进行很多检测，一旦检测出问题，就会打出日志，并启动失败。

　　**discovery.zen.ping.unicast.hosts**

  当节点要和别的server上的节点组成集群时，这里要写上别的server的ip地址（也可以指定端口，如果不指定，默认使用transport.profiles.default.port或者transport.tcp.port的配置）

	discovery.zen.ping.unicast.hosts: ["xxx.xxx.xx.xxx"]

　　**discovery.zen.minimum_master_nodes**

　　这个参数说是为了防止“split brain”而设置的，设置成(master_eligible_nodes /2+1)即可，其中master_eligible_nodes 是可选为master节点的节点数，默认情况下每个节点都是master_eligible_nodes，所以这里我们设置为2

　　elasticsearch.yml文件比较重要的配置就是上面这些，然而，光配置这些还不够，es对系统配置还有些要求，假设我们不修改系统配置，直接启动，看看会报什么错

　　**对了，es启动不能用root用户，必须先建一个es用户，以es用户身份才能启动！**

![]({{site.pictureurl}}122.jpg?raw=true)

　　发现在bootstrap check时，有两个错误，导致无法启动es，第一个错误为max file descriptors设的不够大，只有65535，但es要求最小是65536，我们可以通过如下方式修改：

	第一种方式：换到root用户，执行ulimit -n 65536
	第二种方式：修改/etc/security/limits.conf文件，将nofile设为65536

　　对于第二个问题，max virtual memory areas vm.max_map_count [65530] is too low，我们可以按照如下方式：

	切换到root用户，执行：sysctl -w vm.max_map_count=262144

　　然后我们就可以启动es了

　　如果是直接启动，非后台方式：

	./elasticsearch

　　如果是后台方式：

	./elasticsearch -d

　　两台机器都启动后，我们随便在一台机器上输入如下指令查看节点状态：

	curl -XGET 'localhost:9200/_cat/nodes?v&pretty'

　　显示结果如下：

![]({{site.pictureurl}}123.jpg?raw=true)

　　这样我们的es集群环境就算是搭建完毕

### **ES使用** ###

　　es提供了完整REST API接口来对es进行操作，不管是查询状态、创建索引、删除索引、CRUD所有操作通通通过REST接口进行操作。具体怎么操作可以查看ES文档：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)

　　这里主要说一下使用java如何对es进行操作：

　　在之前的版本，es一直提供了TransportClientes一直提供了[TransportClient](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/java-api.html)，自从5.6.0之后，就提供了 Java High Level REST Client[Java High Level REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/6.0/java-rest-high.html)，而REST Client还是比较通俗易懂，也好用些，但是缺点是，api不太全，只提供了基本的索引、搜索相关接口，而对索引自身的管理等接口还没有提供，我当时都是自己看他的源码，继承了它的client写的

　　他自己提供的JAVA High Level REST Client本质是就是通过http request进行请求，那么我们其实自己也可以写，但是在集群环境下，我们只能指定一个特定的ip去访问，一旦那台机器down了，就无法对es进行操作了，但是使用es提供的REST Client，他自己能自动选择当前可用的server进行操作，这也是我改它的代码，而不是自己实现的原因。

### **ES Suggest** ###

　　参考ES的[completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html)

　　做搜索suggest的


	

　　

	
