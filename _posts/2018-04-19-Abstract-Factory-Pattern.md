---
layout: post_layout
title: 抽象工厂模式
time: 2018年04月19日 星期四
location: 北京
pulished: true
tag: java
excerpt_separator: "----"
---

　　抽象工厂模式

　　参考：[http://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html](http://www.cnblogs.com/java-my-life/archive/2012/03/28/2418836.html)

　　有些地方，把抽象工厂模式跟工厂方法模式搞混了，这篇算讲的不错的。

----

# **抽象工厂模式** #

　　我们来举个例子，假设我们现在要生产cpu和主板，都分别有intel和AMD两种类型，假设按照静态工厂模式，我们需要两个工厂类，一个cpu工厂类，根据传入的参数不同决定是生产intel的cpu还是AMD的cpu，一个主板工厂，根据传入的参数不同决定是生产intel的主板还是AMD的主板（工厂方法模式类似，两个抽象工厂，四个具体工厂类）。

　　这样，我们生产电脑时，要生产cpu和主板，就得调这两个工厂的produce方法，来得到cpu和主板。

　　但是，存在一个问题：intel的cpu只能和intel的主板配套，AMD的cpu只能和AMD的主板配套，但是，现在cpu的工厂和主板的工厂是分开的，并没有约束关系，那么就存在生产出来的cpu和主板不配套的问题。

　　这时，就需要抽象工厂模式了，在介绍抽象工厂模式之前，我们先来了解下产品族和产品等级结构

　　所谓产品族，是指位于不同产品等级结构中，功能相关联的产品组成的家族。比如AMD的主板、芯片组、CPU组成一个家族，Intel的主板、芯片组、CPU组成一个家族。而这两个家族都来自于三个产品等级：主板、芯片组、CPU。一个等级结构是由相同的结构的产品组成，示意图如下：

![]({{site.pictureurl}}125.jpg?raw=true)

　　工厂方法模式是针对单一产品等级结构的，也就是工厂只生产cpu或者只生产主板。而抽象工厂模式针对不同产品族的多个产品等级结构。

　　抽象工厂模式的抽象工厂里的接口是负责生产某一个产品族下的所有产品的，那么每个产品族都有一个对应的工厂子类实现了该抽象工厂的所有方法，用于生产该产品族下的每个产品。

　　还是拿cpu和主板举例，假设有两个产品族，intel和AMD，每个产品族下有两个产品等级结构，cpu和主板。

　　那么，我们的抽象工厂里有两个接口：生产cpu和生产主板

　　intel有一个工厂类，AMD有一个工厂类，当你要组装基于intel硬件的电脑时就调用intel工厂类的生产方法，当你要组装基于AMD硬件的电脑时就调用AMD的工厂类的生产方法，具体代码如下：

	public interface AbstractFactory {
    /**
     * 创建CPU对象
     * @return CPU对象
     */
    public Cpu createCpu();
    /**
     * 创建主板对象
     * @return 主板对象
     */
    public Mainboard createMainboard();
	}

	
	public class IntelFactory implements AbstractFactory {

    @Override
    public Cpu createCpu() {
        // TODO Auto-generated method stub
        return new IntelCpu(755);
    }

    @Override
    public Mainboard createMainboard() {
        // TODO Auto-generated method stub
        return new IntelMainboard(755);
    }

	}


	public class AmdFactory implements AbstractFactory {

    @Override
    public Cpu createCpu() {
        // TODO Auto-generated method stub
        return new IntelCpu(938);
    }

    @Override
    public Mainboard createMainboard() {
        // TODO Auto-generated method stub
        return new IntelMainboard(938);
    }

	}


	public class ComputerEngineer {
    /**
     * 定义组装机需要的CPU
     */
    private Cpu cpu = null;
    /**
     * 定义组装机需要的主板
     */
    private Mainboard mainboard = null;
    public void makeComputer(AbstractFactory af){
        /**
         * 组装机器的基本步骤
         */
        //1:首先准备好装机所需要的配件
        prepareHardwares(af);
        //2:组装机器
        //3:测试机器
        //4：交付客户
    }
    private void prepareHardwares(AbstractFactory af){
        //这里要去准备CPU和主板的具体实现，为了示例简单，这里只准备这两个
        //可是，装机工程师并不知道如何去创建，怎么办呢？
        
        //直接找相应的工厂获取
        this.cpu = af.createCpu();
        this.mainboard = af.createMainboard();
        
        //测试配件是否好用
        this.cpu.calculate();
        this.mainboard.installCPU();
    }
	}

　　**优点：**

　　**分离接口和实现**

　　客户端使用抽象工厂来创建需要的对象，而客户端根本就不知道具体的实现是谁，客户端只是面向产品的接口编程而已。也就是说，客户端从具体的产品实现中解耦。

　　**使切换产品族变得容易**

　　因为一个具体的工厂实现代表的是一个产品族，比如上面例子的从Intel系列到AMD系列只需要切换一下具体工厂。如果有新的产品族增加，那么也只需要新增工厂类即可，不需要修改原来的代码

　　**缺点：**

　　**不太容易扩展新的产品**

　　如果需要给整个产品族添加一个新的产品，那么就需要修改抽象工厂，这样就会导致修改所有的工厂实现类。