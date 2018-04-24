---
layout: post_layout
title: 原型模式
time: 2018年04月24日 星期二
location: 北京
pulished: true
tag: java
excerpt_separator: "----"
---

　　原型模式

　　参考：[https://www.cnblogs.com/chenssy/p/3313339.html](https://www.cnblogs.com/chenssy/p/3313339.html)

　　[http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html](http://www.cnblogs.com/java-my-life/archive/2012/04/11/2439387.html)

----

# *原型模式** #

　　所谓原型模式就是用原型实例指定创建对象的种类，并且通过复制这些原型创建新的对象。

　　既然提到复制，那我们就不得不提clone()方法，Object类里面有个clone()方法，是protected native类型，如果我们自己实现的类需要支持可以复制，那么必须实现cloneable接口，然后实现自己的clone()方法。

　　实现cloneable接口，只是为了标识这个接口是可以被复制的，这个接口里面并没有方法，如果不实现这个接口，直接调用clone方法是会报错的。

　　另外提到拷贝，就不得不提深拷贝和浅拷贝，假设一个类，有两个字段，一个int型，一个String类型，通过调用对象的clone方法（其实就是调用Object类的clone方法），得到新的对象，那么这个新的对象的int字段和String字段都被复制了，但是String字段变量和原来那个对象的String字段变量是指向同一个字符串的，这就叫浅拷贝

　　调用Object类的clone方法默认都是浅拷贝，对于类里面非基本类型的变量这样拷贝出来的都是相同的内存空间，这是不合理的。

　　而深拷贝，即对每个变量都进行一次拷贝，即生成新的对象，而不是只是让引用指向原来的对象，这样就需要我们自己去实现类的clone方法

　　那么下面再来看看原型模式：

　　原型模式主要包含如下三个角色：

       Prototype：抽象原型类。声明克隆自身的接口。 
       ConcretePrototype：具体原型类。实现克隆的具体操作。 
       Client：客户类。让一个原型克隆自身，从而获得一个新的对象。

　　具体的例子，java里的cloneable接口就是最好的例子，假设我们要实现一个简历类，支持可以复制：

	public class Resume implements Cloneable {
    private String name;
    private String birthday;
    private String sex;
    private String school;
    private String timeArea;
    private String company;
    
    /**
     * 构造函数：初始化简历赋值姓名
     */
    public Resume(String name){
        this.name = name;
    }
    
    /**
     * @desc 设定个人基本信息
     * @param birthday 生日
     * @param sex 性别
     * @param school 毕业学校
     * @return void
     */
    public void setPersonInfo(String birthday,String sex,String school){
        this.birthday = birthday;
        this.sex = sex;
        this.school = school;
    }
    
    /**
     * @desc 设定工作经历
     * @param timeArea 工作年限
     * @param company 所在公司
     * @return void
     */
    public void setWorkExperience(String timeArea,String company){
        this.timeArea = timeArea;
        this.company = company;
    }
    
    /**
     * 克隆该实例
     */
    public Object clone(){
        Resume resume = null;
        try {
            resume = (Resume) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return resume;
    }
    
    public void display(){
        System.out.println("姓名：" + name);
        System.out.println("生日:" + birthday + ",性别:" + sex + ",毕业学校：" + school);
        System.out.println("工作年限:" + timeArea + ",公司:" + company);
    }

	}

　　客户端：

	public class Client {
    public static void main(String[] args) {
        //原型A对象
        Resume a = new Resume("小李子");
        a.setPersonInfo("2.16", "男", "XX大学");
        a.setWorkExperience("2012.09.05", "XX科技有限公司");
        
        //克隆B对象
        Resume b = (Resume) a.clone();
        
        //输出A和B对象
        System.out.println("----------------A--------------");
        a.display();
        System.out.println("----------------B--------------");
        b.display();
        
        /*
         * 测试A==B?
         * 对任何的对象x，都有x.clone() !=x，即克隆对象与原对象不是同一个对象
         */
        System.out.print("A==B?");
        System.out.println( a == b);
        
        /*
         * 对任何的对象x，都有x.clone().getClass()==x.getClass()，即克隆对象与原对象的类型一样。
         */
        System.out.print("A.getClass()==B.getClass()?");
        System.out.println(a.getClass() == b.getClass());
    }
	}

　　优点

　　1、如果创建新的对象比较复杂时，可以利用原型模式简化对象的创建过程，同时也能够提高效率。

　　2、可以使用深克隆保持对象的状态。

　　3、原型模式提供了简化的创建结构。

　　缺点 

　　1、在实现深克隆的时候可能需要比较复杂的代码。

　　2、需要为每一个类配备一个克隆方法，而且这个克隆方法需要对类的功能进行通盘考虑，这对全新的类来说不是很难，但对已有的类进行改造时，不一定是件容易的事，必须修改其源代码，违背了“开闭原则”。

　　模式使用场景

　　1、如果创建新对象成本较大，我们可以利用已有的对象进行复制来获得。

　　2、如果系统要保存对象的状态，而对象的状态变化很小，或者对象本身占内存不大的时候，也可以使用原型模式配合备忘录模式来应用。相反，如果对象的状态变化很大，或者对象占用的内存很大，那么采用状态模式会比原型模式更好。 

　　3、需要避免使用分层次的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态，通过复制原型对象得到新实例可能比使用构造函数创建一个新实例更加方便。

　　模式总结

　　1、原型模式向客户隐藏了创建对象的复杂性。客户只需要知道要创建对象的类型，然后通过请求就可以获得和该对象一模一样的新对象，无须知道具体的创建过程。

　　2、克隆分为浅克隆和深克隆两种。

　　3、我们虽然可以利用原型模式来获得一个新对象，但有时对象的复制可能会相当的复杂，比如深克隆。