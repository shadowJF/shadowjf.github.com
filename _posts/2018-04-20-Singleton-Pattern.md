---
layout: post_layout
title: 单例模式
time: 2018年04月20日 星期五
location: 北京
pulished: true
tag: java
excerpt_separator: "----"
---

　　单例模式

　　参考：[https://www.jianshu.com/p/b8c578b07fbc](https://www.jianshu.com/p/b8c578b07fbc)

　　以及我自己写的[volatile理解](http://shadowjf.github.io/2016/06/16/Volatile.html)

　　单例模式也算是接触的最早的设计模式了，但是老是会忘记它的几种实现方式，这次一定好好记下来。

----

# **单例模式** #

　　单例模式保证一个类只有一个对象，降低对象之间的耦合度

　　那么先来看看单例模式的几种实现方式：

　　1. 饿汉式（初始化类时即创建单例）

　　这是最简单的单例实现方式，依赖JVM的类加载机制，保证单例只会被创建1次，是线程安全的，因为JVM在类的初始化阶段，即在class被加载后、被线程使用前，会执行类的初始化。

	public class Singleton{
		//1. 加载该类时，单例会自动被创建
		private static Singelton instance = new Singleton();
		//2. 构造函数为私有，防止其他人创建实例
		private Singleton(){};
		//3. 通过调用静态方法获得创建的单例
		public static Singleton getInstance(){
			return instance;
		}
	}

　　这种单例创建方法适用于初始化速度快和占用内存小的对象，因为在没有使用该类的时候，这个单例就已经被创建了，如果内存占用大会造成浪费。

　　2. 枚举类型（初始化类时即创建单例）

　　这种类型的单例，我之前没遇到过，他的实现方式如下：

	public enum Singleton{
		//定义1个枚举的元素，即为单例类的1个实例
		INSTANCE;
		//隐藏了1个空的，私有的构造方法
		//private Singleton(){}
	}
	//获取单例的方法
	Singleton singleton = Singleton.INSTANCE;

　　这是最简洁、易用的单例实现方式

　　3. 懒汉式基础实现（按需、延迟创建单例）

　　与饿汉式最大的区别在于单例创建的时机

　　饿汉式：单例创建时机不可控，类加载时自动创建
　　懒汉式：单例创建时机可控，即有需要时，才手动创建单例

　　具体实现如下：

	public class Singleton{
		//1. 类加载时，先不自动创建单例
		private static Singleton instance = null;
		//2. 构造函数设为私有权限
		//禁止他人创建实例
		private Singleton(){};
		//3. 需要时才手动调用该方法创建单例
		public static Singleton getInstance(){
			//先判断单例是否为空，以避免重复创建
			if(instance==null)
				instance = new Singleton();
			return instance;
		}
	}

　　懒汉式基础式实现方式的缺点在于线程不安全：当有多个线程同时调用获取单例方法时，有可能创建多个实例（即在判断instance==null时都判定成功了）

　　4. 同步锁懒汉式（按需、延迟创建单例） 
　　
　　使用同步锁synchronize锁住创建单例的方法，防止多个线程同时调用，从而避免造成单例被多次创建

	public class Singleton{
		//1. 类加载时，先不自动创建单例
		private static Singleton instance = null;
		//2. 构造方法设为私有，防止别人创建实例
		private Singleton(){};
		//3. 方法上加入同步锁，保证每次只有一个线程能调用
		public static synchronize Singleton getInstance(){
			if(instance==null)
				instance = new Singleton();
			return instance;
		}
		//4. 这是第二种写法，与第一种不同，但效果一致
		public static Singleton getInstance2(){
			synchronize(Singleton.class){
				if(instance == null)
					instance = new Singleton();
			}
			return instance;
		}
	}

　　这种方式虽然保证了线程安全，并且按需创建，但是每次调用getInstance方法都要进行线程同步，即调用synchronize锁，造成过多的同步开销（加锁=耗时耗能），而理想情况下，我们是希望单例创建之后，就不要再加锁控制了。

　　5. 双重校验锁懒汉式（按需、延迟创建单例）

　　在同步锁的基础上，添加一层if判断，若单例已创建，则不需要再执行加锁操作就可以获得单例，从而提供性能，具体实现如下：

	public class Singleton{
		//需要用volatile来修饰
		private static volatile Singleton instance = null;
		private Singleton(){};
	
		public static Singleton getInstance(){
			//第一个if，若单例已经创建，则直接返回已创建的单例
			//无需加锁操作
			if(instance == null){
				//若单例还未创建，则加同步锁，保证只创建一个单例
				synchronize(Singleton.class){
					if(instance == null)
						instance = new Singleton();
				}
			}
			return instance;
		}
	}　　

　　需要注意的是，我们的instance对象，用了volatile关键字来修饰，这是为什么呢？因为java的内存模型中的操作的无序性导致的，假设线程A进入到instance = new Singleton()操作了，但是这个操作并不是一个原子操作，他的实际执行顺序可能是，先给instance分配空间（这里instance已经不是null了，但是还没有执行构造函数初始化），然后才执行构造函数。假设当他刚分配完空间还没有执行构造函数时，线程B来到了第一个if判断，发现instance!=null，那么就会直接返回instance实例，然而此时这个实例还没有执行构造函数，因此是不能合法使用的，就有可能导致程序崩溃。

　　但是从JDK1.5之后，加了volatile关键字，他能保证他所修饰的变量的可见性，因此在线程A，给instance赋值后，线程B能立即感知到，就不会发生这种事情了。

　　这种方法既保证了按需加载单例、线程安全、并不会每次都加锁同步，但是缺点在于实现复杂

　　6. 静态内部类

　　根据静态内部类的特性，同时解决了按需加载、线程安全的问题，同时实现也很简洁

	在静态内部类里创建单例，在装载该内部类时才会去创建单例
	线程安全：类是由 JVM加载，而JVM只会加载1遍，保证只有1个单例

　　具体实现如下：

	public class Singleton{
		private Singleton(){};
		//静态内部类Holder
		private static class Holder{
			//有一个私有的静态Singleton对象，在该内部类加载时会初始化该对象
			private static Singleton instance = new Singleton();
		}
		public static Singleton getInstance(){
			return Holder.instance;
		}
	}


　　以上就是单例模式的讲解了，这下你了解什么是饿汉式、懒汉式、枚举类型、同步锁、双重校验锁了吗？？？