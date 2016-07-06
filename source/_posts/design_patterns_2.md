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
