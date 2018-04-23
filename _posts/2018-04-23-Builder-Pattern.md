---
layout: post_layout
title: 建造者模式
time: 2018年04月23日 星期一
location: 北京
pulished: true
tag: java
excerpt_separator: "----"
---

　　建造者模式

　　参考：[http://www.cnblogs.com/java-my-life/archive/2012/04/07/2433939.html](http://www.cnblogs.com/java-my-life/archive/2012/04/07/2433939.html)

　　[https://www.jianshu.com/p/be290ccea05a](https://www.jianshu.com/p/be290ccea05a)

----

# **建造者模式** #

　　所谓建造者模式，就是将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

　　这样说其实很抽象，完全不知道是什么意思，感觉又和工厂模式有点相同。

　　那么我们先来看下他的UML图：

![]({{site.pictureurl}}126.jpg?raw=true)

　　该模式下，有4个角色：Director、Builder、ConcreteBuilder、Product，当然还有一个client，这是我们的客户

　　客户如果有需求，要创建一个什么东西，他是直接跟Director沟通，然后Director将客户创建产品的需求划分为各个部件的建造请求（这一部分通过Builder提供的多个方法来实现），最后各个部件的建造请求委派到具体的建造者（ConcreteBuilder），各个具体建造者负责进行产品部件的构建，最终构建成具体产品。

　　ok，是不是还有点不太理解，那我们结合例子来看：假设客户去买电脑，Director就是电脑老板，老板和客户沟通（你买来是打游戏、还是学习等），Director了解需求后，就会将客户需要的主机划分为各个部件的建造请求，并且通过一个具体的装机人员（ConcreteBuilder）去构建组件，将组件组装起来得到产品Product

　　那么Builder就是一个接口或者抽象方法，他定义了讲产品划分为哪几部分进行构建，如下：

	public  abstract class Builder {  

	//第一步：装CPU
	//声明为抽象方法，具体由子类实现 
    	public abstract void  BuildCPU()；

	//第二步：装主板
	//声明为抽象方法，具体由子类实现 
    	public abstract void BuildMainboard（）；

	//第三步：装硬盘
	//声明为抽象方法，具体由子类实现 
    	public abstract void BuildHD（）；

	//返回产品的方法：获得组装好的电脑
    	public abstract Computer GetComputer（）；
	}

　　而ConcreteBuilder则是继承该Builder的具体的建造者，因为对于某一个产品可能有多种组装方式，例如，用来打游戏的电脑需要好点的cpu和主板，而用来学习的可能就不需要那么好的cpu和主板，因此，这可能是两种不同的ConcreteBuilder，那么我们这里可以实现其中的一种具体建造者：

	public class ConcreteBuilder extend  Builder{
    	//创建产品实例
    	Computer computer = new Computer();

    	//组装产品
    	@Override
    	public void  BuildCPU(){  
       		computer.Add("组装CPU")
    	}  

    	@Override
    	public void  BuildMainboard（）{  
       		computer.Add("组装主板")
    	}  

    	@Override
    	public void  BuildHD（）{  
       		computer.Add("组装主板")
    	}  

    	//返回组装成功的电脑
     	@Override
      	public  Computer GetComputer（）{  
      		return computer
    	}  
	}

　　那么老板Director是干什么的呢，老板了解用户需求后，指定一个具体建造者即可，如下：

	public class Director{
		//指挥装机人员组装电脑
		public void Construct(Builder builder){             
			builder. BuildCPU();
            builder.BuildMainboard();
            builder. BuildHD();
        }
	}

　　可见，Director接受一个Builder参数，然后只会Builder进行产品构建即可。

　　当然我们还需要定义我们的产品Product

	public class Computer{
    
    	//电脑组件的集合
    	private List<String> parts = new ArrayList<String>()；
     
    	//用于将组件组装到电脑里
    	public void Add(String part){
    		parts.add(part);
		}
    
    	public void Show(){
          for (int i = 0;i<parts.size();i++){    
          System.out.println(“组件”+parts.get(i)+“装好了”);
          }
          System.out.println(“电脑组装完成，请验收”);
		}
	}

　　最后，我们来展示下客户到电脑城找老板买电脑的整个过程：

	public class Builder Pattern{
		public static void main(String[] args){

			//逛了很久终于发现一家合适的电脑店
			//找到该店的老板和装机人员	
			Director director = new Director();
			Builder builder = new ConcreteBuilder();

			//沟通需求后，老板叫装机人员去装电脑
			director.Construct(builder);

			//装完后，组装人员搬来组装好的电脑
			Computer computer = builder.GetComputer();
			//组装人员展示电脑给小成看
			computer.Show()；
    	}
	}

　　到这里我们就知道整个建造者模式的过程了，那么简单总结下，什么是建造者模式：

　　1. 建造过程分为多个组件的构建，而且可能组件之间还有依赖约束关系，整个构建过程比较复杂，对客户完全透明
　　2. 建造模式必定存在两个部分，一个部分是部件构造和产品装配，这是由Builder来决定的，另一个部分是整体构建的算法，这是由Director来决定的，而在建造模式中，强调的是固定整体构建的算法，而灵活扩展和切换部件的具体构造和产品装配方式。这也是Builder可以被继承的原因，而director负责的整体构建算法则是基本上固定的。

　　那么我们就知道在什么情况下需要使用建造者模式了：

　　1. 需要生成的产品对象具有复杂的内部结构
　　2. 产品独享的属性相互依赖，建造者模式可以强制实行一种分步骤进行的建造过程，因此如果产品对象的一个属性必须在另一个属性被赋值后才可以被赋值，则使用建造模式是一个很好的设计思想

　　那么建造者模式的优缺点是什么呢？

　　优点：

　　易于解耦：将产品本身与产品创建过程进行解耦，可以使用相同的创建过程来得到不同的产品。也就说细节依赖抽象。

　　易于精确控制对象的创建：将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰

　　易于拓展：增加新的具体建造者无需修改原有类库的代码，易于拓展，符合“开闭原则“。

　　缺点：

　　建造者模式所创建的产品一般具有较多的共同点，其组成部分相似；如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。

　　如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大。

　　这两个缺点感觉说的是一回事，就是因为director负责的整体构造算法不变，所以造出来的产品肯定是都要具备这些属性的，只不过单个组件的实现方式不同，所以适用范围没有那么广。万一产品结构变化了，那么可能就不适用了。

　　那么JDK里面有哪些建造者模式的应用实例呢？我看网上都说StringBuilder和StringBuffer（线程安全）的append方法是建造者模式。

　　append方法是可以接受很多不同类型的输入，然后针对不同类型，来将数据append到字符数组上，这样说来，他自己本身即是director又是builder，他是通过方法重载来实现不同的builder的？