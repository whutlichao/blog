---
title: 设计模式二——创建型模式
tags: [java, framework]
---
   接上篇总结创建型的设计模式
## 工厂方法模式（Factory Method）
   当有大量的不同的产品需要创建时，并且这些产品有共同的接口，则可以通过工厂模式进行创。

 <!--more-->
### 普通工厂模式
   首先，创建一个产品的共同接口：
``` java
public interface Phone {
	void call();

	void takePhoto();

	void listenMusic();
}
```
   产品的具体实现：
``` java

public class Apple implements Phone{

	public void call() {
		System.out.println("call from iphone");
	}

	public void takePhoto() {
	}

	public void listenMusic() {
	}
}


public class SumSung implements Phone{

	public void call() {
		// TODO Auto-generated method stub
		System.out.println("call from Sumsung");
	}

	public void takePhoto() {
	}

	public void listenMusic() {
	}
}
```
   工厂类：
``` java

public class PhoneFactory {

	public Phone product(String tag)
	{
		if (tag.equals("Apple"))
		{
			return new Apple();
		}
		else if (tag.equals("Sumsung"))
		{
			return new SumSung();
		}

		return null;
	}
}
```
   测试时需要先实例化工厂，才能生成不同的产品。
``` java
public class TestClient {

	public static void main(String[] args)
	{
		PhoneFactory phoneFactory = new PhoneFactory();
		Phone phone = phoneFactory.product("Apple");
		phone.call();
	}
}
```
### 静态工厂模式
   在普通工厂模式中需要每次都实例化一个工厂，如果我们生产方法使用静态的，则不需要实例化一个工厂。
``` java

   public class PhoneFactory {

   	public static Phone product(String tag)
   	{
   		if (tag.equals("Apple"))
   		{
   			return new Apple();
   		}
   		else if (tag.equals("Sumsung"))
   		{
   			return new SumSung();
   		}

   		return null;
   	}
   }
```

## 抽象工厂模式（abstract Factory)；
   抽象工厂模式是对普通工厂模式的一种改进，普通工厂模式有弊端，如果我们再新增一种产品，比如新增一个HuaWei的产品类，则需要修改工厂类，这是与开闭原则相违背，所以我们对普通工厂模式进行改进，将工厂也进行抽象。
 ``` java
public interface PhoneFactory {

	Phone product();
}

public class AppleFactory implements PhoneFactory{

	public Phone product() {
		return new Apple();
	}

}

public class SumsungFactory implements PhoneFactory{

	public Phone product() {
		return new SumSung();
	}
}


public class TestClient {

	public static void main(String[] args)
	{
		PhoneFactory phoneFactory = new AppleFactory();
		Phone phone = phoneFactory.product();
		phone.call();
		phoneFactory = new SumsungFactory();
		phone = phoneFactory.product();
		phone.call();
	}
}

```
   如果后续需要增加新的产品，只需要新增一个工厂类和一个产品类就可以了，不需要再对现有代码进行修改，增加了可扩展性。

## 创建者模式（builder)
   工厂模式是为了创建单个实例，如果我们需要批量的创建实例，则此时就需要用到创建者模式。
``` java
public interface Builder {
	List productPhones(int count);
}

public class AppleBuilder  implements Builder{

	public List productPhones(int count) {
		ArrayList phones = new ArrayList();
		for (int i = 0; i < count; i++)
		{
			phones.add(new Apple());
		}
		return phones;
	}

}

public class SumsungBuilder implements Builder {

	public List productPhones(int count) {
		ArrayList phones = new ArrayList();
		for (int i = 0; i < count; i++) {
			phones.add(new SumSung());
		}
		return phones;
	}
}


public class TestClient {

	public static void main(String[] args)
	{
		Builder builder = new AppleBuilder();
		builder.productPhones(10);
	}
}
```

## 原型模式（Prototype）
   原型模式只指将一个对象作为原形，从而克隆一个同原型对象类似的新对象，这里我们需要充分利用到Object类中的clone方法，Object的clone方法可以复制一个实现cloneable接口的类的实例。
``` java
public class ProtoType implements Cloneable{

	public Object clone()
	{
		ProtoType proto = null;
		try {
			proto = (ProtoType)super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return proto;
	}
}
```
   上面的例子只是浅复制，而有时候我们需要深复制，浅复制和深复制有什么区别呢？Object中的clone是一个native方法，为什么会使用native方法呢，是因为C中对浅复制和深复制更容易操作。
   浅复制：比如我们对A对象进行浅复制产生一个A1对象，可以理解为内存中新增了一个A1指针指向A对象，A1中的所有属性都还是A的，比如A中对于对象C的引用，则在A1中也同样使用这个引用。
   深复制：比如我们对A对象进行深复制产生一个A1对象，可能理解为A对象的内存直接被复制了一份新的赋给了A1，但实际上比这个复杂的多，如果A中有引用B，B又引用C，则深复制时会对引用也进行递归深复制，A1中有引用B1，B1又引用C1。
   由此可见深复制的性能比浅复制更加的耗时，但两种复制方式应对不同的场景，所以我们需要精准的确认到底使用哪种复制方式，否则会对带来性能问题。
 ``` java
 public class ProtoType implements Cloneable, Serializable{

	private static final long serialVersionUID = 1L;

	public Object clone()
	{
		ProtoType proto = null;
		try {
			proto = (ProtoType)super.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		return proto;
	}

	public Object deepClone() throws IOException, ClassNotFoundException
	{
		ByteArrayOutputStream bos = new ByteArrayOutputStream();  
		ObjectOutputStream oos = new ObjectOutputStream(bos);  
		oos.writeObject(this);  

		ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());  
        ObjectInputStream ois = new ObjectInputStream(bis);
        return ois.readObject();  
	}
}
```
## 单例模式（Singleton）
   单例模式在java中使用非常频繁，它有什么好处呢：当某个类比较庞大时，单例模式可以避免多次实例化导致不必要的开销；某些特殊的类不能被多次实例化，否则会引起业务逻辑错误。但是单例模式也不能随便乱用，否则轻则线程阻塞、性能有损，重则业务逻辑错误。在使用单例模式时我们不得不考虑的一个重要问题就是线程安全，因为只有一个实例，所以在处理多线程并发时要做线程同步或加锁。
### 简单单例模式
   单例模式需要三个重要元素，1、private的对象实例。2、私有的构造方法，防止其它地方使用默认的公共构造方法实例化对象。3、公共静态的获取实例的入口。先来看一个例子：
``` java
public class Singleton {

	private static Singleton instance = null;

	private Singleton()
	{

	}

	public static Singleton getInstance()
	{
		if (instance == null)
		{
			instance = new Singleton();
		}

		return instance;
	}
}
```
### 线程安全的单例模式
   但是大家会发现一个问题，当这个类被放到多线程环境下时，就会出现问题，因为毫无线程安全的保护，所以可以改进如下：
``` java
public static synchronized Singleton getInstance()
{
  if (instance == null)
  {
    instance = new Singleton();
  }

  return instance;
}
```
   但是问题又来了，这样直接在方法上加同步关键字，支将整个对象都锁住的，多线程并发性能会大大降低，因为每次调用获取对象实例时都会给对象加锁。我们再改进一下，只在第一次创建对象时对对象加锁。
``` java
public static Singleton getInstance()
{
  if (instance == null)
  {
    synchronized(instance)
    {
      if (instance == null)
      {
        instance = new Singleton();
      }
    }
  }
  return instance;
}
```
   但是问题又来了（好烦啊，问题真多），在java中创建对象和赋值操作是分两步进行的，但是这两个步骤的先后顺序是无法判断的，也就是说有可能先给instance分配好了空白内存空间再生成对象，或者先生成了对象再将这内存分配给instance。当两上线程同时进入了第一个instance==null的判断时，若A线程先给instance分配了内存空间后放开了instance的锁，但是还没有做创建对象的操作，B线程发现instance不为空（只是有指向的空白内存而已），如果这时候B线程要使用insance时会发现instance没有被实例化。我们再做如下的优化。
``` java
public class Singleton {
	private Singleton()
	{
	}
	private static class SingletonFactory
	{
		private static Singleton instance = new Singleton();
	}

	public static Singleton getInstance()
	{
		return SingletonFactory.instance;
	}
}
```
   我们使用一个静态内部类来维护这个对象，为什么这样做可以线程安全呢，因为JVM会提前加载静态变量，而JVM加载类的时候是线程互斥的，所以 我们利用了JVM的这样一个机制从而保证线程的安全和性能的要求。
   还有一种方法是将获取和创建过程分离，单独为创建加锁，这样也可以实现安全的单例模式，同时不会降低性能。
``` java
public class Singleton {
	private static Singleton instance = null;
	private Singleton()
	{
	}

	public static Singleton getInstance()
	{
		if (instance == null)
		{
			SyncInit();
		}
		return instance;
	}

	private static synchronized void SyncInit() {
		if (instance == null)
		{
			instance = new Singleton();
		}
	}
}   
```
