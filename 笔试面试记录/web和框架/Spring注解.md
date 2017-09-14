> 本篇的大多数内容来自于：http://www.cnblogs.com/digdeep/p/4525567.html

#### 一、先回答这几个问题：

#####1、知道哪些spring的原生注解？

#####2、自己实现过注解吗？

#####3、@resource和@autowired注解有什么区别？

#### 二、先了解下java的注解

##### 2.1、java元注解

@Retention @Target @Document @Inherited

**@Retention**：注解的保留位置

```java
@Retention(RetentionPolicy.SOURCE)   //注解仅存在于源码中，在class字节码文件中不包含
@Retention(RetentionPolicy.CLASS)  // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得
@Retention(RetentionPolicy.RUNTIME)  // 注解会在class字节码文件中存在，在运行时可以通过反射获取到
```

**@Target **：注解的作用目标

```
@Target(ElementType.TYPE)   //接口、类、枚举、注解
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包  
```

**@Document **：说明该注解将被包含在javadoc中

**@Inherited**：说明子类可以继承父类中的该注解

**举例说明元注解的使用**

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface AnnatDemo{
	public int value();
}
//以上代码定义了@AnnatDemo注解，作用目标是 用于对方法注解，并且保留在运行时的环境中，我们可以利用反射获得一个方法上的注解  调用定义的方法,

//比如@AnnatDemo 作用于以下方法：
public interface IClientProtocolEx extends IProtocol {
　　int METHOD_START=0;
　　@AnnatDemo(METHOD_START)
　　public String say(String person);
}

//那么可以利用以下代码进行反射：
Class ipt=IClientProtocalEx.class;
Method[] mts=ipt.getMethod();
for(Method mt:mts){
	AnnatDemo ad=mt.getAnnotation(AnnatDemo.class);//如果方法上没有该注解，则返回null
	int value=ad.value();
　　 System.out.println("value:"+value);
}
//注解是用于建设基础jar包的一部分，项目都有自己的框架，若运用恰当，注解则为其中良好的一部分！
```

