##### 1、静态代理和动态代理对比。

- http://www.cnblogs.com/cenyu/p/6289209.html这个链接讲的很清楚。

- 什么是代理？

  为某个对象提供一个代理，以控制对这个对象的访问。 代理类和委托类有共同的父类或父接口，这样在任何使用委托类对象的地方都可以用代理对象替代。代理类负责请求的预处理、过滤、将请求分派给委托类处理、以及委托类执行完请求后的后续处理。 

- 区别

  - 两者的具体实现
    - 静态代理类维护一个目标对象，定义同名的方法，方法中调用委托类的方法，在该调用前后做一些事情。
    - 动态代理类维护一个目标对象，之后给目标对象生成代理对象，生成代理对象是通过Proxy.newProxyInstance()方法生成，这个方法需要三个参数，ClassLoader、interfaces和InvocationHandler对象，InvocationHandler对象需要重载invoke()方法，该方法中才是代理类的具体执行逻辑。
  - 代理类的生成时间不同
    - **静态代理**就是由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。 
    - **动态代理**类的源码是在程序运行期间由JVM根据反射等机制动态的生成。代理类和委托类的关系是在程序运行时确定。 
  - 代理多个方法时
    - 静态代理如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。 还有如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。 
    - 动态代理与静态代理相比，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。
  - 静态代理中目标对象和代理对象必须实现同一个接口；动态代理中目标对象必须实现接口，代理对象不需要实现接口。所以java动态代理仅仅局限于接口。

##### 2.1、如何实现动态代理？

```java
//参考代码
/**
 * 创建动态代理对象
 * 动态代理不需要实现接口,但是需要指定接口类型
 */
public class ProxyFactory{
    //维护一个目标对象
    private Object target;
    public ProxyFactory(Object target){
        this.target=target;
    }
   //给目标对象生成代理对象
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开始事务2");
                        //执行目标对象方法
                        Object returnValue = method.invoke(target, args);
                        System.out.println("提交事务2");
                        return returnValue;
                    }
                }
        );
    }
}
```



- 在ProxyFactory类中维护一个目标对象target。
- ProxyFactory类中实现public Object getProxyInstance()方法，该方法返回`java.lang.reflect.Proxy.newProxyInstance()`即一个代理对象。
  - newProxyInstance()方法有三个参数:
    - target.getClass.getClassLoader()
    - target.getClass.getInterfaces()
    - 一个实现`invoke(Object proxy,Method method,Object[] args)`方法的调用处理器对象 ，调用处理器对象是实现了InvocationHandler接口的类对象，InvocationHandler接口要求实现invoke()方法，在该方法的`Object returnValue = method.invoke(target,args)`语句前后，写处理逻辑。

##### 2.2、动态代理的实现原理？

- 代理类对象proxyInstance调用目标对象target要调用的SayGoodBye()方法时，发生了以下几件事：

  - 前提是代理类的类型对象已经生成（二进制字节码存在缓存中），并且通过传入invocationHandler获得了代理类实例proxyInstance。

  - 代理类实现接口的SayGoodBye()方法中有这么一句

    `return (String)this.h.invoke(this, m3, null);`，m3是通过接口反射得到的SayGoodBye()方法对象，该语句的意思是proxyInstance.SayGoodBye()会调用传入的invocationHandler的invoke()，而invocationHandler的invoke()方法中有我们自己写的目标对象执行操作SayGoodBye()方法的前后要执行的操作，然后在这些操作的中间会执行`Object returnValue = method.invoke(target, args);  `，这句其实就是目标对象target调用SayGoodBye()方法。

- 以上步骤的原理细节分析：

  - newProxyInstance()方法中利用getProxyClass0()方法生成了与指定类装载器和一组接口相关的代理类类型对象 。

  - 然后通过反射获取构造函数对象并生成代理类实例 。

    - 先获取代理对象的构造方法。
    - 然后把InvocationHandlerImpl的实例传给该构造方法，生成代理类的实例。

  - 关于利用getProxyClass()方法生成代理类类型对象的具体细节：

    - `getProxyClass0(ClassLoader loader,Class<?>[] interfaces)`方法中调用了`proxyClassCache.get(loader, interfaces) ` 

    - `proxyClassCache.get()`调用了 supplier.get(); 获取动态代理类，其中supplier是Factory,这个类定义在WeakCach的内部。

    - WeakCach的get()方法调用了valueFactory.apply(key, parameter)方法，该方法内有一个语句：

      ```java
      //生成字节码
      byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags);
      ```

    - 通过对该方法生成的字节码进行反编译，可以得到代理类

      ```java
      public final class ProxySubject  
      extends Proxy  
      implements Subject{
        ...
        static  
        {  
          try  {  
       		m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { 				Class.forName("java.lang.Object") });  
          	m3 = Class.forName("jiankunking.Subject").getMethod("SayGoodBye", new 					Class[0]);  	  
          	m4 = Class.forName("jiankunking.Subject").getMethod("SayHello", new Class[] 			{Class.forName("java.lang.String") });  
          	m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]); 
          	m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]); 
            	return;
          }  
          catch (NoSuchMethodException localNoSuchMethodException)  
          {  
            throw new NoSuchMethodError(localNoSuchMethodException.getMessage());  
          }  
          catch (ClassNotFoundException localClassNotFoundException)  
          {  
            throw new NoClassDefFoundError(localClassNotFoundException.getMessage());  
          }  
        }  
        //构造方法
        public ProxySubject(InvocationHandler paramInvocationHandler)  
        {  
          super(paramInvocationHandler);  
        } 
        //这是实现Subject接口中的一个方法
        public final String SayGoodBye()
        {  
          	try{
          		return (String)this.h.invoke(this, m3, null);  
          	}catch (Error|RuntimeException localError){  
          		throw localError;
          	}catch (Throwable localThrowable){  
            		throw new UndeclaredThrowableException(localThrowable);  
            	}  
        }
        ...
      }
      ```

       `return (String)this.h.invoke(this, m3, null);  `这句就是调用我们定义InvocationHandlerImpl的 invoke方法。

##### 2.4动态代理为什么需要接口？

- 因为生成的代理类需要通过反射机制得到method对象，将其传参给invocationHandler增强类，invocationHandler中的invoke方法会通过method对象的invoke（目标对象，参数）方法调用目标对象的业务方法。

##### 3、Cglib实现代理类

上面的静态代理和动态代理模式都是要求目标对象是实现一个接口的目标对象,但是有时候目标对象只是一个单独的对象,并没有实现任何的接口,这个时候就可以使用以目标对象子类的方式类实现代理,这种方法就叫做:Cglib代理

Cglib代理,也叫作**子类代理**,它是在内存中构建一个子类对象从而实现对目标对象功能的扩展.

- JDK的动态代理有一个限制,就是使用动态代理的对象必须实现一个或多个接口,如果想代理没有实现接口的类,就可以使用Cglib实现.
- Cglib是一个强大的高性能的代码生成包,它可以在运行期扩展java类与实现java接口.它广泛的被许多AOP的框架使用,例如Spring AOP和synaop,为他们提供方法的interception(拦截)
- Cglib包的底层是通过使用一个小而快的字节码处理框架ASM来转换字节码并生成新的类.不鼓励直接使用ASM,因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉.

##### 4、三个代理方式的区别

- 静态代理是通过在代码中显式定义一个业务实现类一个代理，在代理类中对同名的业务方法进行包装，用户通过代理类调用被包装过的业务方法；
- JDK动态代理是通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法；
- CGlib动态代理是通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法进行代理；