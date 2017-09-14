### java设计模式

> 设计模式分
>
> 创建型模式、单例模式、工厂模式、建造者模式和原型模式
>
> 结构型模式、适配器模式、装饰者模式、代理模式、外观模式、桥接模式、组合模式、享元模式
>
> 行为型模式、观察者模式、策略模式、模板方法模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

#### 1、单例模式

- 懒汉式的线程安全写法,懒加载模式,影响性能。即时实例已经存在，还是只能有一个线程调用getInstance()。

  ```
  public class Singleton{
    private static Singleton instance = null;
    priavte Singleton(){  
    }
    public static synchronized Singleton getInstance(){
      if(instance==null){
        instance = new Singleton();
      }
      return instance;
    }
  }
  ```

  ​

- 饿汉式,某些场景无法适用。static final

  ```
  public class Singleton{
    private static final Singleton instance = new Singleton();
    private Singleton(){
    }
    public static Singleton getInstance(){
      return instance;
    }
  }
  ```

  ​

- 懒汉式的双重检查锁定写法

```
public class Singleton{
  private volatile static Singleton instance = null;
  private Singleton(){
  }
  public static Singleton getInstance(){
    if(instance==null)
    {
      synchronized(Singleton.Class)
      {
        if(instance==null)
        {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}
```

- 懒汉式的静态内部类

  ```
  public class Singleton{
    private static class SingletonHolder{
      private static final Singleton instance = new Singleton();
    }
    private Singleton (){}
    public static final singleton getInstance(){
      return SingletonHolder.instance;
    }
  }
  ```

- 枚举

  枚举 Enum

  用枚举写单例实在太简单了！这也是它最大的优点。下面这段代码就是声明枚举实例的通常做法。

  ```
  public enum EasySingleton{
      INSTANCE;
  }
  ```

  我们可以通过EasySingleton.INSTANCE来访问实例，这比调用getInstance()方法简单多了。创建枚举默认就是线程安全的，所以不需要担心double checked locking，而且还能防止反序列化导致重新创建新的对象。但是还是很少看到有人这样写，可能是因为不太熟悉吧。

#### 2、工厂模式

简单工厂模式；工厂方法模式；抽象工厂模式。

#### 3、代理模式

java的静态代理和动态代理。

#### 4、观察者模式

- 概念

观察者和被观察者，被观察者维护一个观察者列表，一旦自己的状态发生变化就告知观察者。还分推模式和拉模式。

观察者采用**注册**(_Register_)或者成为**订阅**(_Subscribe_)的方式告诉被观察者：我需要你的某某状态，你要在它变化时通知我。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。

- 观察者模式优点

观察者模式将观察者和主题（被观察者）彻底解耦，主题只知道观察者实现了某一接口（也就是Observer接口）。并不需要观察者的具体类是谁、做了些什么或者其他任何细节。任何时候我们都可以增加新的观察者。因为主题唯一依赖的东西是一个实现了`Observer`接口的对象列表。

####5、