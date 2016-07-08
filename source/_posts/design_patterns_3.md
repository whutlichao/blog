---
title: 设计模式三——结构型模式
tags: [java, framework]
---
   接上篇总结结构型模式
## 适配器模式（Adapter）
   适配器模式是结构型模式的基础，所以我们先来了解这种设计模式，适配器模式的目的是为了消除不匹配、互补从而达到预期目标的一种模式，适配器模式分为三类：类的适配器模式、对象的适配器模式、接口的适配器模式。
<!--more-->
### 类的适配器模式
   主要是为了针对统一接口而存在的，有时候我们需要把我们需要调用的接口进行统一的封装，从而实现对外的统一。
``` java
public class Base {

	public void method1()
	{
		System.out.println("this is base method");
	}
}

// 对外的统一接口
public interface TargetInterfase {

	void method1();

	void method2();
}

// 适配器，为了匹配缺失接口
public class Adapter extends Base implements TargetInterfase {

	public void method2() {
		System.out.println("this is adapter method");
	}
}

public class TestClient {

	public static void main(String[] args)
	{
		TargetInterfase targetInterfase = new Adapter();
		targetInterfase.method1();
		targetInterfase.method2();
	}
}
```
   因为一个适配器Adapter类，从而使TargetInterface具有了Base类的功能，然而TargetInterface和Base没有直接的继承和聚合关系。
### 对象的适配模式
  对象的适配模式与类的适配模式类似，但是适配器不再继承Base类，而是持有Base类的一个对象，使用对象从而达到调用Base类功能。
``` java
public class Adapter extends Base implements TargetInterfase {

	private Base base = null;

	public Adapter(Base base)
	{
		this.base = base;
	}

	public void method2() {
		System.out.println("this is adapter method");
	}

	public void method1()
	{
		base.method1();
	}
}
```
### 接口的适配器模式
   我们知道一个类实现一个接口，必须重写这个接口的所有抽象方法，但是有时候我们并不会用到所有的抽象方法，这个时候我们就需要用到接口的适配器。
``` java

public interface SouceInterface {

	void method1();

	void method2();
}


public class SorceAdapter implements SouceInterface{

	public  void method1(){};

	public  void method2(){};
}


public class Sub1 extends SorceAdapter{

	public void method1()
	{
		System.out.println("this is sub1");
	}
}

public class Sub2 extends SorceAdapter{

	public void method2()
	{
		System.out.println("this is sub2");
	}
}

public class TestClient {
	public static void main(String[] args)
	{
		SouceInterface souceInterface1 = new Sub1();
		souceInterface1.method1();
		souceInterface1.method2();
		SouceInterface souceInterface2 = new Sub2();
		souceInterface2.method1();
		souceInterface2.method2();
	}
}
```
   总结下三种适配器模式，当我们希望将一个类的功能合并到另一个满足新接口的类时，就使用类的适配器模式。当希望将一个对象转换成满足另一个接口的对象时，可以使用对象的适配器。当我们不需要使用接口中的全部方法，可以使用接口的适配器对接口进行分割。
## 装饰模式（Decorator）
   装饰模式顾名思义，就是指对一个类的功能进行装饰，当我们需要扩展一个类的功能时，我们可以使用装饰模式。这里有人就会问，那我们为什么不继承呢，继承是静态的扩展，不能像装饰模式那样动太的扩展和撤消。
``` java
public interface Base {
	void method();
}

public class Sub1 implements Base{

	public void method() {
		System.out.println("i am sub1");
	}
}


public class Sub2 implements Base{

	private Base sub1 = null;
	public Sub2(Base instance)
	{
		sub1 = instance;
	}
	
	public void method() {
		System.out.println("sub2 start");
		sub1.method();
		System.out.println("sub2 end");
	}
}

public class TestClient {

	public static void main(String[] args)
	{
		Base sub1 = new Sub1();
		Base sub2 = new Sub2(sub1);
		sub2.method();
	}
}
```
   
## 代理模式（Proxy）
   代理模式同装饰模式很类似，装饰模式与代理模式的区别是，主类需要出面，进到化妆室进行装饰后被装饰成另一个新类，而代理模式是主类实例不出面，全权由代理来完成功能，就像中介代理租客找房子一样，找到房子前租客是不需要出面的。
``` java

public interface Base {
	void method();
}

public class Sub implements Base{

	public void method()
	{
		System.out.println("this is sub");
	}
}

public class Proxy implements Base{

	private Base sub = null;
	
	public Proxy()
	{
		this.sub = new Sub();
	}
	
	public void method()
	{
		System.out.println("proxy start");
		sub.method();
		System.out.println("proxy end");
	}
}

public class TestClient {

	public static void main(String[] args) {
		Base proxy = new Proxy();
		proxy.method();
	}
}
```
## 外观模式（Facade）
   当出现需要几个同级别类进行协同工作时，我们将协同关系放到一个第三方的类中，这种类间的组织结构就是外观模式。
``` java
public class cpu {
	public void startup()
	{
		System.out.println("cpu startup");
	}
	
	public void shutdown()
	{
		System.out.println("cpu shutdown");
	}
}

public class Memory {
	public void startup()
	{
		System.out.println("memory startup");
	}
	
	public void shutdown()
	{
		System.out.println("memory shutdown");
	}
}


public class Disk {
	public void startup()
	{
		System.out.println("disk startup");
	}
	public void shutdown()
	{
		System.out.println("disk shutdown");
	}
}

public class Computer {

	private cpu cpu;
	private Memory memory;
	private Disk disk;
	
	public Computer()
	{
		this.cpu = new cpu();
		this.memory = new Memory();
		this.disk = new Disk();
	}
	
	public void startup()
	{
		cpu.startup();
		memory.startup();
		disk.startup();
	}

	public void shutdown()
	{
		cpu.shutdown();
		memory.shutdown();
		disk.shutdown();
	}
}

public class TestClient {

	public static void main(String[] args) {
		Computer computer  = new Computer();
		computer.startup();
		computer.shutdown();
	}

}
```
## 桥模式（Bridge）
## 组合模式（Composite）
## 享元模式（Flyweight）
