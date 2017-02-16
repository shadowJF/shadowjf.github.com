---
layout: post_layout
title: Chat Robot
time: 2017年02月08日 星期三
location: 北京
pulished: true
tag: AI
excerpt_separator: "----"
---

　　最近在写聊天机器人，就记录下开发聊天机器人的一些基本知识等等的，首先给出一些我看过的不错的文章，我的记录也是在这些前人的基础上丰富起来的。
 
　　[自己动手做聊天机器人](http://www.shareditor.com/blogshow/?blogId=63) 这是一个国人自己搭建的blog，这一系列记录了他自己学习搭建聊天机器人的过程，技术栈是python，不过看了下虽然有很多干货，包括代码什么，但是总觉得不够连贯。

　　[聊天機器人的開發思路](http://zake7749.github.io/2016/12/17/how-to-develop-chatbot/#u8A9E_u610F_u5716) 这是一个湾湾程序猿的github博客，在github上他自己有个chatbot的工程，因此他记了些博客，对我也是很有帮助的。

　　[AIML](http://www.alicebot.org/aiml.html) 一种基于规则的人工智能标记语言

　　[结巴分词](https://github.com/fxsjy/jieba)结巴分词

　　[opencc](https://github.com/BYVoid/OpenCC)中文简繁转换(巨难编译，最后搞不定只能用yum install 了老版的opencc)

----

##机器人模型##

　　聊天机器人大体上分为三种模型：

- 规则式模型(Rule-based model)
- 检索式模型(Retrieval-based model)
- 生成式模型(Generative model)

　　下面我们一一来介绍

###规则式模型###

　　所谓规则式模型，是最简单的一种，就是通过设计规则，来让机器人知道，遇到什么词该给出什么样的回答，例如：

```Java

	if "天氣" in user.query:
	chatbot.say("今天天氣真好")

```

　　而提到规则式模型，就不得不提AIML(Artificial Intelligence Mark-up Language)，它是一种人工智能标记语言，最简单的形式如下：

```xml

	<aiml version = "1.0.1" encoding = "UTF-8"?>
   		<category>
      		<pattern> HELLO </pattern>
      		<template>
         		WORLD !
      		</template>
   		</category>
	</aiml>
```

　　按照上述规则，机器人在遇见HELLO后，就会回复WORLD。

　　AIML以XML文件来定义规则，最重要的几个标签，在上面都有展示：

- \<aiml> 标识AIML文件开始和结束的标签
- \<category> 某一种规则(或知识)的开始和结束的标签
- \<pattern> 用来匹配用户输入的模式
- \<template> 回复的内容

　　当然，AIML还有20多种其余的标签，都有各自的作用，具体可以参考如下文档

　　[http://www.alicebot.org/TR/2005/WD-aiml/WD-aiml-1.0.1-008.html#section-introduction](http://www.alicebot.org/TR/2005/WD-aiml/WD-aiml-1.0.1-008.html#section-introduction)

　　[http://www.pandorabots.com/pandora/pics/wallaceaimltutorial.html](http://www.pandorabots.com/pandora/pics/wallaceaimltutorial.html)

　　当然，这个AIML只是一种语言，那么如果用它来构建一个机器人，需要一种该语言的解释器，在ALICE的官网上提供了各种语言的[AIML解释器](http://www.alicebot.org/downloads/programs.html)，我试用了一下其中的[chatterbean](http://www.geocities.ws/phelio/chatterbean/#INTRODUCTION)，是java版本的。


####chatterbean####

　　按照chatterbean官网介绍，有两个版本下载，一个是binary，一个是source，binary就是两个jar包，而source则包含了所有的java源文件以及alice的aiml文件。

　　我首先下了binary版本，先试用一下，打开压缩包后，就两个jar包，一个bsh.jar,一个chatterbean.jar。 官网说我们只要执行：

	java -cp bsh.jar -jar chatterbean.jar path_to/properties.xml

　　其中，path_to/properties.xml是机器人的配置文件，但是官网居然没说这配置文件是干嘛的，需要配什么，只给了个示例，是一个机器人[Ifurita](http://www.geocities.ws/phelio/chatterbean/#BOTS)的下载，我又把这个机器人下载下来，发现里面确实有properties.xml，内容如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
	<properties>
  		<comment>Ifurita's configuration properties</comment>
  		<entry key="context">context.xml</entry>
  		<entry key="splitters">splitters.xml</entry>
  		<entry key="substitutions">substitutions.xml</entry>
  		<entry key="categories">./</entry>
	</properties>

　　Ifurita里面确实也有context.xml、splitters.xml、substitutions.xml文件，那么我们把这些都复制到之前的binary目录下，然后再执行

	java -cp bsh.jar -jar chatterbean.jar properties.xml

　　这时候，发现并没有报错，但是，我们输入任何文字后，会报一个空指针，想了想，我们最关键的aiml文件并没有指定啊，所以肯定报空指针，那么怎么指定我们所要使用的AIML文件呢，其实在properties.xml里面最后不是有一个categories条目吗，这里指定一个目录路径，那么chatterbean就会在该目录下寻找以aiml为文件后缀的文件做为机器人的AIML文件，那么我们将Ifurita机器人的ifurita.xml文件拷贝过来，改为ifurita.aiml,然后再执行上面的java命令，就可以愉快地聊天了，如下：

![]({{site.pictureurl}}64.jpg?raw=true)

　　那如果我自己再添加一个我自己的AIML文件呢？ 如下：

	<?xml version="1.0" encoding="ISO-8859-1"?>

	<aiml>
  	<!-- Categories for saying goodbye. -->
  		<category>
    		<pattern>HI</pattern>
    		<template>hi there</template>
  		</category>
  
  		<category>
    		<pattern>WHO ARE YOU</pattern>
    		<template>I'm shadow</template>
  		</category>
  
  		<category>
    		<pattern>WHERE DO YOU WORK</pattern>
    		<template>I work in YonYou</template>
  		</category> 
	</aiml>

　　其实最开始，我试了中文，发现不生效，原因以及如何让AIML支持中文参考[http://blog.csdn.net/zhang_hui_cs/article/details/22686951](http://blog.csdn.net/zhang_hui_cs/article/details/22686951)

　　而且，AIML文件中的pattern标签里的英文必须是大写。。。。不然也不生效，这样之后，我们再问它这些问题，就会给出相应回答了

![]({{site.pictureurl}}65.jpg?raw=true)

　　AIML有一些还是比较有用的标签的，想srai、think、that等等，帮助进行同义转换、上下文理解等等有帮助，这里不详解，自己去上面提供的网址看吧。

###检索式模型###

　　所谓检索式模型，也挺好理解的，它跟搜索引擎挺像的，它有一个问题库，每个问题都有一个回答，这样，当用户输入对话后，就用该句子去问题库中寻找最相似的问题，以此来获得回答。

　　所以这种模型的关键在于，如何计算句子之间的相似度。常用的文本相似度计算方法如下：

- TF/IDF
- 编辑距离
- 向量相似度
- 主题模型

####TF/IDF####
　　TF/IDF就是词频、逆文档频率，是计算文档相似度的常用算法，当然也有其改进算法[OKapi BM25](https://en.wikipedia.org/wiki/Okapi_BM25)(考虑了文本长度的改进算法),不过在我开发机器人的过程中，并没有使用这种算法，机器人场景下，待计算相似度的文本都是短文本，这种计算方法的准确率并不高。
####编辑距离####
　　编辑距离就是要将句子A改成B，至少需要进行几步操作，这里的操作包括：替换、删除、添加，不过这种方法并没办法考虑到语义的情况，因此在做聊天机器人时，这种算法也不太适用
####向量相似度####
　　向量相似度的计算方法有很多，例如：余弦相似度、jaccard相似性、欧几里得距离、曼哈顿距离等等、那么我们的重点在于如何将文本转为向量，在这方面的研究，提的比较多的就是word2vec以及sentence2vec（doc2vec），这两种算法在[gensim](https://radimrehurek.com/gensim/index.html)的python库中都有实现，原理比较复杂，后面会继续研究，其实单独看一个词，你也许不能知道他们是否相似，所以我们可以结合上下文，来分析单词的语义，上下文语境越相似的词相似度应该是越高的，这就是word2vec的思想，同理doc2vec也一样。

　　不过，word2vec和doc2vec都是无监督学习，因此需要大量的语料，才能训练出来比较好的结果。

　　关于word2vec的使用，最上面的两篇分享里面都有

　　其实，短句子的相似度计算，用word2vec就能达到较好的效果，我们只要获得句子所有词的向量，然后对所有词向量做一个平均，就生成了句向量，然后利用余弦相似度计算句子间的相似度即可，但是这只适用于短句子，长的文档则还是用doc2vec来计算比较合适，因为如果用word2vec会丢失掉很多上下文信息。

　　下面就介绍下我用word2vec来计算句子间相似度的过程(基于python)：

　　１ 下载语料库（wiki）

　　word2vec是无监督训练，需要大量语料来训练，因此我们选择从wiki上下载需要的[wiki语料库](https://zh.wikipedia.org/wiki/Wikipedia:%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%8B%E8%BD%BD)，其中有各种语言的，我们选择中文的，然后我下载了[20170201的备份](https://dumps.wikimedia.org/zhwiki/20170201/),记得下载zhwiki-20170201-pages-articles.xml.bz2，但是这个比较大，我用来做实验，所以下载了下面的那个分段后的第一部分即zhwiki-20170201-pages-articles1.xml.bz2

　　２　安装gensim

　　gensim是一个很有名的主题模型函数库，里面提供了各种有用的工具，像word2vec、doc2vec、lda等等，有了python环境，直接执行pip3 install --upgrade gensim即可安装，但是后面你会发现有坑的，那是因为通过这种方式安装的gensim，虽然装了numpy库，但是在windows上装这个有个坑，就是你没有装mkl库，因此后面会报错，所以我们需要从下面这个地址去下载[numpy‑1.12.0+mkl‑cp34‑cp34m‑win_amd64.whl](http://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy)，然后执行pip3 install numpy‑1.12.0+mkl‑cp34‑cp34m‑win_amd64.whl来完成对numpy+mkl的安装（linux安装numpy和scipy也是各种坑，这两个都是安装gensim包必备的 = =后面详解）

　　３　wiki语料xml转为txt文本

　　第一步下载下来的wiki语料是xml文档，里面有很多无用的标签，我们需要对它进行处理，来得到纯粹的文本数据，好在gensim已经提前为我们准备了这样的工具，因此我们只要按照下述程序代码，去执行一遍，即可得到处理好的文本文档：

```python
	# -*- coding: utf-8 -*-
	import logging
	import sys

	from gensim.corpora import WikiCorpus

	def main():

    	if len(sys.argv) != 2:
        	print("Usage: python " + sys.argv[0] + " wiki_data_path")
        	exit()

    	logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
    	wiki_corpus = WikiCorpus(sys.argv[1], dictionary={})
    
    	texts_num = 0
    
    	with open("wiki_texts.txt",'w',encoding='utf-8') as output:
        	for text in wiki_corpus.get_texts():
            	output.write(b' '.join(text).decode('utf-8') + '\n')
            	texts_num += 1
            	if texts_num % 10000 == 0:
                	logging.info("已处理 %d 篇文章" % texts_num)
	if __name__ == "__main__":
    	main()
```

　　转换后的文本如下：

![]({{site.pictureurl}}66.jpg?raw=true)

　　４ 繁简转换

　　从上图可以看出，大量的繁体字出现，如果不处理，那么繁体和简体相同的词会被当做两个词，因此，我们还需要对这个文档进行繁体转简体的处理，繁简转换大家都推荐用[opencc](https://github.com/BYVoid/OpenCC)，然而，我上主页看了之后，下载了源码，怎么编译都不通过，不管是在window上（我尝试了用MSYS和Visual Studio去编译）还是linux上编译，全都报各种错，试错了一天还是没搞定，后来发现，yum源search了一下opencc，有如下结果：

![]({{site.pictureurl}}67.jpg?raw=true)

　　虽然是0.4版本的，属于老版，但是能用就好，直接yum install opencc之后，还不行，还得装一个opencc-tools，因此在yum install opencc-tools.x86_64，这样之后，在/usr/share/opencc下会有opencc需要用到的转换的配置文件，我们只要执行：

	opencc -i input_file -o output_file -c mix2zhs.ini

　　就可以将文本转换为简体了，注意配置文件必须是mix2zhs.ini，代表将混合有简体和繁体的文本转换为简体，而不能用zht2zhs.ini（繁体转换为简体），我试了并没有转换成功。这也是老版的问题吧，我觉得，没关系，用mix2zhs就行。

　　５ 分词

　　英文用空格就可以标识每个词，但是中文必须得借助分词工具，来分词，现在开源的分词工具有很多，结巴、word、hanlp等等，这里为了方便，直接使用结巴分词，非常简单，首先pip3 install jieba安装下载结巴分词，然后执行如下python代码：

```python
	# -*- coding: utf-8 -*-

	import jieba
	import logging

	def main():

    	logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)

    	# load stopwords set
    	stopwordset = set()
    	with open('stopwords.txt','r',encoding='utf-8') as sw:
        	for line in sw:
            	stopwordset.add(line.strip('\n'))

    	output = open('wiki_seg.txt','w',encoding='utf-8')
    
    	texts_num = 0
    
    	with open('wiki_texts_sim.txt','r',encoding='utf-8') as content :
        	for line in content:
            	words = jieba.cut(line, cut_all=False)
            	for word in words:
                	if word not in stopwordset:
                    	output.write(word +' ')
            	texts_num += 1
            	if texts_num % 10000 == 0:
                	logging.info("已完成前 %d 行分词" % texts_num)
    	output.close()
    
	if __name__ == '__main__':
		main()
```

　　这段代码，需要注意的是，stopwords.txt是停用词，直接去结巴分词github上就能找到，或者用自己的，无非就是你我他之类的这些词，然后还需要注意的是，在open()文件时，一定要加上encoding="utf-8"，不然就会报编码之类的错误，所以这点需要注意。分词完后，文件如下：

![]({{site.pictureurl}}68.jpg?raw=true)

　　6 训练词向量

　　这是最关键的一步了，我们需要对文本进行训练，获得每个词的向量表示，然而代码其实很简单，如下：

```python
	# -*- coding: utf-8 -*-

	from gensim.models import word2vec
	import logging

	def main():

    	logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
    
    	sentences = word2vec.Text8Corpus("wiki_seg.txt")
    	model = word2vec.Word2Vec(sentences, size=250)
    	model.save_word2vec_format(u"med250.model.bin", binary=True)

    	# how to load a model ?
    	# model = word2vec.Word2Vec.load_word2vec_format("your_model.bin", binary=True)

	if __name__ == "__main__":
    	main()
```

　　虽然代码很简单，但是内容其实很多，最关键的一行代码是：

	model = word2vec.Word2Vec(sentences, size=250)

　　参数的含义如下：

	sentences:待训练的句子集
	size:這表示的是訓練出的詞向量會有幾維
	alpha:機器學習中的學習率，這東西會逐漸收斂到 min_alpha
	sg:這個不是三言兩語能說完的，sg=1表示採用skip-gram,sg=0 表示採用cbow
	window:窗口参数，往左往右看幾個字的意思，印象中原作者論文裡寫 cbow 採用 5 是不錯的選擇
	workers:執行緒數目，除非電腦不錯，不然建議別超過 4
	min_count:若這個詞出現的次數小於min_count，那他就不會被視為訓練對象

　　但是在训练过程中，说我的电脑上没有c编译器，因此采用的是slow training，如果要用fast training，需要安装c编译器后再重新安装gensim，一开始我以为忍忍吧，就让它慢慢跑得了，结果发现，在windows，没有c编译器的slow training真心太slow，顾及得跑三四天，好吧，还是去linux上再重新来一遍吧。

　　首先，安装python3.4，虽然linux上已经有python2.7了，gensim也支持python2.7，但是为了和windows上版本一致，因此还是装一个python3.4吧。不用删除原来的2.7，直接在官网下载一个python的[source包](https://www.python.org/downloads/release/python-343/)，然后解压，之后执行：

	./configure
	make
	make install

　　装完后，接着就要进入艰难的安装gensim的步骤了，因为它依赖numpy和scipy，这两个东西安装起来各种坑。。

　　按照gensim官方[安装指导](http://radimrehurek.com/gensim/install.html)，我们其实可以直接：

	easy_install-3.4 -U gensim

　　来安装，注意这里是easy_install_3.4，因为我们之前有2.7版本，所以easy_install是2.7版本的。但是这样安装，会一下把gensim依赖的东西一起安装，出错了信息会非常多，不好排查，因此，我们还是按照后面提供的方法，先装numpy和scipy，最后再装gensim

　　执行：

	easy_install-3.4 numpy

　　理所当然的报了一堆错。。。不要慌，打开报错的信息一步步排查，最后发现，就是缺少了一些依赖，首先是需要装subversion

	yum install subversion

　　然后，还需要gfortran的编译环境，所以

	yum install gcc-gfortran

　　然而，天不助我，我们内网的源没有这个，好吧，直接去网上搜rpm包，有个网站很赞[http://rpm.pbone.net/](http://rpm.pbone.net/)，直接搜你要的即可，注意gfortran的版本要和你的gcc版本一致。我们下载了：

	gcc-gfortran-4.8.5-4.el7.x86_64.rpm
	libquadmath-devel-4.8.5-4.el7.x86_64.rpm

　　后面那个是gfortran依赖的包，因此也下载了，然后执行：

	rpm -i xx.rpm

　　安装完毕，基本上就可以正常安装numpy了，如果再报错，你就去看看报的啥错，缺啥装啥吧 = =，然后安装scipy

	easy_install-3.4 scipy

　　scipy安装过程中可能会需要装一个atlas的东西，直接yum源安装即可，需要注意的是，在安装过程中，可能会报很多的warning，我一开始以为出错，后来发现只是warning而已，不要在意。。。。

　　最后，终于可以装gensim了

	easy_install-3.4 --upgrade gensim

　　如果numpy和scipy装成功了，这一步基本上不会有问题了，然后装个结巴分词

	pip3 install jieba

　　然后，将我们的train.py和分好词的语料，拷贝到linux上，执行

	python3 train.py

　　发现，又出了个之前没出现过的错，ImportError: No module named bz2，google之百度之，解决方法为，首先，安装bzip2-devel

	yum install bzip2-devel

　　然后，对python进行重新编译，cd到python source目录，然后执行：

	make
	make install

　　然后，终于可以执行train.py了，这时发现，我去，速度快得一比啊，几分钟就训练完了！！训练完得到模型文件：med250.model.bin

　　7 计算句子间的相似度

　　有了训练好的模型，我们就可以开始计算句子间的相似度，其实就是计算向量间的相似度，向量间的相似度计算方法有很多，我们采用最常见最简单的余弦相似度，执行下面的python代码：

```python
	# -*- coding: utf-8 -*-

	from gensim.models import word2vec
	import jieba
	import logging
	import sys

	def get_sentence_vector(sentence,model,stopwordset):
		word_vector_list = []
	
		words = jieba.cut(sentence, cut_all=False)
	
		for word in words:
			if word not in stopwordset:
				word_vector_list.append(model[word])
	
		result_vector = [0] * 250
	
		for i in range(250):
			for vector in word_vector_list:
				result_vector[i] += vector[i]
			if(len(word_vector_list)!=0):
				result_vector[i] /= len(word_vector_list)
	
		return result_vector	

	def calc_sim(vector1,vector2):
		dot_product = 0.0  
		normA = 0.0  
		normB = 0.0  
		for a,b in zip(vector1,vector2):  
			dot_product += a*b  
			normA += a**2  
			normB += b**2  
		if normA == 0.0 or normB==0.0:  
			return None  
		else:  
			return dot_product / ((normA*normB)**0.5)  

	def main():

		logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
	
		if(len(sys.argv)!=3):
			logging.info("usage: python " + sys.argv[0] + " sentence1 sentence2")
			exit()

		# load stopwords set
		stopwordset = set()
		with open('stopwords.txt','r',encoding='utf-8') as sw:
			for line in sw:
				stopwordset.add(line.strip('\n'))
			
		model = word2vec.Word2Vec.load_word2vec_format("med250.model.bin", binary=True)
		
		sent1 = sys.argv[1]
		sent2 = sys.argv[2]
	
		vector1 = get_sentence_vector(sent1,model,stopwordset)
		vector2 = get_sentence_vector(sent2,model,stopwordset)

		similarity = calc_sim(vector1,vector2)
	
		print(similarity)

	if __name__ == "__main__":
		main()
```

　　上面的代码主要干了这些事：首先，从传入的参数中读入待比较的两个句子，然后，获取停用词，这里和之前分词用到的停用词一样，然后利用word2vec.Word2Vec.load_word2vec_format来加载模型到内存，这一步如果模型很大会比较慢且占较多内存，不过我们测试的数据量并不大，所以很快，然后通过get_sentence_vector函数，将句子转化为向量，具体做法就是对句子分词，然后去掉停用词，对每个词再获取其词向量，最后对所有这些词向量做个平均。获得两个句子的词向量后，我们就可以通过余弦相似度算法计算他们之间的相似度了

　　下面是一些结果展示：

	我爱读书 我爱游泳 0.912990954057
	我爱读书 你是傻子 0.769530908508
	我爱读书 我爱看书 0.954195134156
	我爸爸是天底下最好的人 我超级喜欢我爸爸 0.823780526446
	北京公积金查询 工作居住证办理 0.439870274941
	北京公积金查询 上海公积金提取 0.662318980781

　　如果输入句子的分词结果中有模型中不包含的词会报错，当然如果你的训练语料足够丰富，一般是不会出现这种情况的。通过上述结果，我们发现，虽然我们的语料很有限，只用了wiki语料的很少一部分，但训练出的结果还是比较令人满意的。可以预见，如果针对特定场景或领域，我们的训练语料足够丰富的情况下，这样的相似度计算效果是很好的。


####主题模型####
　　每个句子都有自己的主题，利用主题模型来对句子进行分类，获取句子的类型后，在抽取句子特征，最后根据特征，获取回答，这是主题模型的处理流程，这其中涉及到的关键技术在于分类和特征抽取，分类算法有很多，例如最常见的朴素贝叶斯等等，而特征抽取则涉及到实体识别。

　　现在很多开源的NLP工具都已经实现了一部分的文本相似度计算算法， 例如word分词、Hanlp等等，我之前就用的hanlp的文本推荐功能来计算句子之间的相似度，hanlp的文本推荐计算相似度是结合了三种相似度算法来计算的，一是语义相似度、一是编辑距离、一是拼音相似度，后来我对这个算法进行了一些修改，以让它来适应机器人短文本相似度计算的特点。

　　现在我们的智能聊天机器人，就是一个检索式模型的机器人。它由这样一些组件组成：

- 大脑：核心处理器，对话理解、响应、知识库-场景导入、存储、重载等等
- 眼睛：文本输入
- 嘴巴：音频输出
- 耳朵：音频输入

　　而其中大脑是最核心最重要的组件，它由如下模块组成：

- 问答模块：
- 问答预处理模块：
- 知识模块：
- 场景模块：
- 聊天记录模块：
- 长时记忆模块：
- 短时记忆模块：

　　其中知识模块，就涉及到利用检索模型，去知识库中检索相关知识，并给出回答。

###生成式模型###

　　这种应该是技术含量最高的模型了，但是这种一般是用于日常生活对话的聊天型机器人，如果是针对特定领域的商务型机器人则不会使用这种模型。

　　生成式模型我也不太了解，简单谈一谈，貌似自从google发表了Sequence to Sequence Learning with Neural Networks的论文后，大家都开始用这种模型去实现机器人了，像github上就有很多高完成度应用，如[DeepQA](https://github.com/Conchylicultor/DeepQA)，这也是图灵后台使用的关键技术之一。Sequence to Sequence 的基本概念是串接兩個 RNN/LSTM，一個當作編碼器，把句子轉換成隱含表示式，另一個當作解碼器，將記憶與目前的輸入做某種處理後再輸出，不過這只是最直觀的方式，其實解碼器還有很多種作法，如果想了解細節與效能上的差異，我推薦[這篇文章](http://jacoxu.com/?p=1852)。

　　

三种模型就介绍到这里吧，后续再继续研究关于机器人上下文处理、自学习相关方面的内容