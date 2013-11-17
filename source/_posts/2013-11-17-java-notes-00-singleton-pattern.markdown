---
layout: post
title: "Java Notes - Singleton Pattern(单例总结)"
date: 2013-11-17 14:21
comments: true
sidebar: false
categories: Java
---

这里不赘述单例模式的概念了，直接演示几种不同的实现方式。

## 0)Eager initialization
如果程序一开始就需要某个单例，并且创建这个单例并不那么费时，我们可以考虑用这种方式：
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
 
    private Singleton() {}
 
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```
这种实现方式有几个特点：

<!-- More -->

* 实例一开始就被创建(Eager initialization)。
* `getInstance()`方法不需要加`synchronize`来解决多线程同步的问题。
* `final`关键字确保了实例不可变，并且只会存在一个。

## 1)Lazy initialization
懒加载的方式使得单例会在第一次使用到时才会被创建.先看一种有隐患的写法：
```java
public final class LazySingleton {
	private static volatile LazySingleton instance = null;

	// private constructor
	private LazySingleton() {
	}

	public static LazySingleton getInstance() {
		if (instance == null) {
			synchronized (LazySingleton.class) {
				instance = new LazySingleton();
			}
		}
		return instance;
	}
}
```
**请注意**：上面的写法其实非线程安全的，假设两个Thread同时进入了getInstance方法，都判断到instance==null，然后会因为synchronized的原因，逐个执行，这样就得到了2个实例。解决这个问题，需要用到典型的**double-check**方式，如下：
```java
public class LazySingleton {
    private static volatile LazySingleton instance = null;

    private LazySingleton() {    	
    }

    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton .class) {
                if (instance == null) {
                        instance = new LazySingleton ();
                }
            }
        }
        return instance;
    }
}
```
另外一个更简略直观的替代写法是：
```java
public class LazySingleton {
    private static volatile LazySingleton instance = null;

    private LazySingleton() {    	
    }

    public static synchronized LazySingleton getInstance() {
            if (instance == null) {
                instance = new LazySingleton ();
            }
        return instance;
    }
}
```
## 2)Static block initialization
如果我们对程序的加载顺序有点了解的话，会知道Static block的初始化是执行在加载类之后，Constructor被执行之前。
```java
public class StaticBlockSingleton {
	private static final StaticBlockSingleton INSTANCE;

	static {
		try {
			INSTANCE = new StaticBlockSingleton();
		} catch (Exception e) {
			throw new RuntimeException("Error, You Know This, Haha!", e);
		}
	}

	public static StaticBlockSingleton getInstance() {
		return INSTANCE;
	}

	private StaticBlockSingleton() {
		// ...
	}
}
```
上面的写法有一个弊端，如果我们类有若干个static的变量，程序的初始化却只需要其中的1，2个的话，我们会做多余的static initialization。

## 3)Bill Pugh solution
University of Maryland Computer Science researcher Bill Pugh有写过一篇文章[initialization on demand holder idiom](http://en.wikipedia.org/wiki/Initialization_on_demand_holder_idiom)。
```java
public class Singleton {
    // Private constructor prevents instantiation from other classes
    private Singleton() { }

    /**
    * SingletonHolder is loaded on the first execution of Singleton.getInstance() 
    * or the first access to SingletonHolder.INSTANCE, not before.
    */
    private static class SingletonHolder { 
        public static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
SingletonHolder类会在你需要的时候才会被初始化，而且它不影响Singleton类的其他static成员变量的使用。这个方法是线程安全的并且避免了使用volatile与synchronized。

## 4)Using Enum
这是最简便安全的方法。没有明显的缺点，并且避免了下面要讲到的序列化的隐患。
```java
public enum Singleton {
    INSTANCE;
    public void execute (String arg) {
        // perform operation here 
    }
}
```

## Serialize and de-serialize
在某些情况下，需要实现序列化的时候，普通的单例模式需要添加readResolve的方法，不然会出现异常。
```java
public class DemoSingleton implements Serializable {
	private volatile static DemoSingleton instance = null;

	public static DemoSingleton getInstance() {
		if (instance == null) {
			instance = new DemoSingleton();
		}
		return instance;
	}

	protected Object readResolve() {
		return instance;
	}

	private int i = 10;

	public int getI() {
		return i;
	}

	public void setI(int i) {
		this.i = i;
	}
}
```
仅仅有上面的还不够，我们需要添加serialVersionUID，例子详见下面的总结。

## Conclusion
实现一个功能完善，性能更佳，不存在序列化等问题的单例，建议使用下面两个方式之一:

### Bill Pugh(Inner Holder)
```java
public class DemoSingleton implements Serializable {
	private static final long serialVersionUID = 1L;

	private DemoSingleton() {
		// private constructor
	}

	private static class DemoSingletonHolder {
		public static final DemoSingleton INSTANCE = new DemoSingleton();
	}

	public static DemoSingleton getInstance() {
		return DemoSingletonHolder.INSTANCE;
	}

	protected Object readResolve() {
		return getInstance();
	}
}
```
### Enum
```java
public enum Singleton {
    INSTANCE;
    public void execute (String arg) {
        // perform operation here 
    }
}
```


***
**参考资料**

* [Singleton Pattern](http://en.wikipedia.org/wiki/Singleton_pattern)
* [Singleton Pattern In Java](http://howtodoinjava.com/2012/10/22/singleton-design-pattern-in-java/)

**转载请注明出自[http://kesenhoo.github.com](http:://kesenhoo.github.com)，谢谢**

