[spring](http://lib.csdn.net/base/javaee)的面试题部分

###1、为什么要用spring，spring的优缺点有哪些？

​	1.使用Spring的IOC容器，将对象之间的依赖关系交给Spring，降低组件之间的耦合性，让我们更专注于应用逻辑

​	2.可以提供众多服务，事务管理等。

​	3.AOP的很好支持，方便面向切面编程。

​	4.对主流的框架提供了很好的集成支持，如Hibernate,Struts2,JPA等

​	5.Spring DI机制降低了业务对象替换的复杂性。

​	6.Spring属于低侵入，代码污染极低。

​	7.Spring的高度可开放性，并不强制依赖于Spring，开发者可以自由选择Spring部分或全部

### 2、Spring的IOC容器初始化流程

- Spring中有两个主要的容器系列：

  - 1、实现BeanFactory接口的简单容器；
  - 2、实现ApplicationContext接口的高级容器。
  - ApplicationContext比较复杂，它不但继承了BeanFactory的大部分属性，还继承其它可扩展接口，扩展的了许多高级的属性。

- 整个IOC初始化，大致分为资源定位、资源装载、资源解析、Bean生成，Bean注册这几个过程。

- 关于Spirng IoC容器的初始化过程在《Spirng技术内幕：深入解析Spring架构与设计原理》一书中有明确的指出，IoC容器的初始化过程可以分为三步：

  - Resource定位（Bean的定义文件定位）

  - 将Resource定位好的资源载入到BeanDefinition

  - 将BeanDefiniton注册到容器中

  - **第一步 Resource定位**

    Resource是Sping中用于封装I/O操作的接口。正如前面所见，在创建spring容器时，通常要访问XML配置文件，除此之外还可以通过访问文件类型、二进制流等方式访问资源，还有当需要网络上的资源时可以通过访问URL，Spring把这些文件统称为Resource

  - **第二步 通过返回的resource对象，进行BeanDefinition的载入**

    - BeanDefinition相当于一个数据结构，这个数据结构的生成过程是根据定位的resource资源对象中的bean而来的，这些bean在Spirng IoC容器内部表示成了的BeanDefintion这样的数据结构，IoC容器对bean的管理和依赖注入的实现都是通过操作BeanDefinition来进行的。
    - 在Spring中，配置文件主要格式是XML，对于用来读取XML型资源文件来进行初始化的IoC 容器而言，该类容器会使用到AbstractXmlApplicationContext类，该类定义了一个名为loadBeanDefinitions(DefaultListableBeanFactory beanFactory) 的方法用于获取BeanDefinition，此方法在具体执行过程中首先会new一个与容器对应的BeanDefinitionReader型实例对象，然后将生成的BeanDefintionReader实例作为参数，传入loadBeanDefintions(XmlBeanDefinitionReader)继续往下执行载入BeanDefintion的过程。例如FileSystemXmlApplicationContext容器会在此方法中new一个XmlBeanDefinitionReader对象，这个对象专门用来载入所有的BeanDefinition。

  - **第三步，将BeanDefiniton注册到容器中**

    - 最终Bean配置会被解析成BeanDefinition并与beanName,Alias一同封装到BeanDefinitionHolder中,之后beanFactory.registerBeanDefinition(beanName, bdHolder.getBeanDefinition())，注册到DefaultListableBeanFactory.beanDefinitionMap中。之后客户端如果要获取Bean对象，Spring容器会根据注册的BeanDefinition信息进行实例化。

###3、Spring的IOC容器实现原理，为什么可以通过byName和ByType找到Bean？

- Spring的IOC利用hashMap存储bean，key为bean的id（没有指定id就是bean的name），value为bean。


- 在应用中，我们常常使用<ref>标签为JavaBean注入它依赖的对象。但是对于一个大型的系统，这个操作将会耗费我们大量的资源，我们不得不花费大量的时间和精力用于创建和维护系统中的<ref>标签。实际上，这种方式也会在另一种形式上增加了应用程序的复杂性，那么如何解决这个问题呢？[spring](http://lib.csdn.net/base/javaee)为我们提供了一个自动装配的机制，尽管这种机制不是很完善，但是在应用中结合<ref>标签还是可以大大的减少我们的劳动强度。前面提到过，在定义Bean时，<bean>标签有一个autowire属性，我们可以通过指定它来让容器为受管JavaBean自动注入依赖对象。

  <bean>的autowire属性有如下六个取值，他们的说明如下： 

  1、 No：即不启用自动装配。Autowire默认的值。 

  2、 byName：通过属性的名字的方式查找JavaBean依赖的对象并为其注入。比如说类Computer有个属性printer，指定其autowire属性为byName后，Spring IoC容器会在配置文件中查找id/name属性为printer的bean，然后使用Seter方法为其注入。 (如果容器中存在一个与指定属性类型相同的bean，那么将与该属性自动装配；如果存在多个该类型bean，那么抛出异常，并指出不能使用byType方式进行自动装配；如果没有找到相匹配的bean，则什么事都不发生，也可以通过设置)

  3、 byType：通过属性的类型查找JavaBean依赖的对象并为其注入。比如类Computer有个属性printer，类型为Printer，那么，指定其autowire属性为byType后，Spring IoC容器会查找Class属性为Printer的bean，使用Seter方法为其注入。 

  4、 constructor：通byType一样，也是通过类型查找依赖对象。与byType的区别在于它不是使用Seter方法注入，而是使用构造函数注入。 

  5、 autodetect：在byType和constructor之间自动的选择注入方式。 

  6、 default：由上级标签<beans>的default-autowire属性确定。

#### 1.Spring的aop你怎样实现?

用动态代理和cglib实现,有接口的用动态代理,无接口的用cglib

#### 2.Spring在SSH起什么作用

为大部分框架提供模版,常见的核心类提供初始化,并且整合三层框架

####3.Spring容器内部怎么实现的

内部用Map实现,或者说HashMap

#### 4.怎么样理解IOC与AOP

IOC是一种控制反转的思想,降低了对象的耦合度,AOP是面向切面编程,非侵入式编程,实现了非业务性编程(公共功能),譬如日志,权限,事务等等

####5.Spring的事务，事务的作用。

Spring里面的事务分为编程式事务和声明式事务,一般用声明式事务,用来控制数据操作的完整性,一致性

####6.Spring的IOC和AOP你在项目中是怎么使用的？

IOC主要来解决对象之间的依赖问题,把所有的bean的依赖关系通过配置文件或者注解关联起来,降低了耦合度,AOP一般用来整合框架时候都可以用得到,

事务用的最多,还有个别日志,权限功能也可以用到

####7Spring主要使用了什么模式？

工厂模式-->每个Bean的创建通过方法

单例模式-->默认的每个Bean的作用域都是单例

代理模式-->关于AOP的实现是通过代理,体现代理模式

###8.Spring bean的作用域.

Scope作用域有4种,常见的有单例或者多例,默认是单例

####9.Spring的事务是如何配置的？

1.先配置事务管理器TransactionManager,不同的框架有不同属性

2.再配置事务通知和属性,通过tx:advice

3.配置<aop:config>,设置那些方法或者类需要加入事务

###10.Spring的配置文件最好使用什么文件？

xml,因为它是最简单,最流行的数据格式

###11.你使用过Spring中的哪些技术？

bean的管理,AOP技术,IOC技术 ,事务等

###12.为什么要用Spring

降低对象耦合度,让代码更加清晰,提供一些常见的模版

###13.说下Spring的注解

1.bean的标记注解

@Component 通用注解 @Repository 持久层注解 @Service 业务层注解 @Controller:表现层注解

2.bean的自动装配注解

称

@AutoWired 默认是按照类型装配,如果有多个类型实现可以用Qualifier来指定名

@Resource 默认是按照名称来装配,是JDK里面自带的注解,默认情况下用@AutoWired注解

###15.写过类似Spring AOP的操作吗？

简单的写过,譬如前置通知,后置通知的方法,环绕通知,事务就是典型的AOP的实现

###16.Spring中的AOP在你项目中是怎么使用的，用在哪里？

Struts2和[hibernate](http://lib.csdn.net/base/javaee)整合时候都可以用得到, 事务用的最多,还有个别日志,权限功能也可以用到

###17.Spring的事务（传播属性，隔离级别）。

七大传播属性,四大隔离级别

###19.Spring DI的几种方式

setter注入和构造器注入,一般用setter注入

###20.依赖注入的原理

就是通过反射机制生成想要的对象注入

###21.说一下整合Spring的核心监听器。

这个是在SSH整合的时候使用,是整个WEB项目启动的时候初始化Spring的容器. 具体是在web.xml里面配置的ContextLoaderListener

Spring配置文件中的核心是个监听器，是用来初始化Spring的容器

###22.Spring你们为什么用配置文件而不使用注解?

配置文件耦合度低,容易维护,尤其是在切面或者事务的时候,只配置一次就可以让很多代码拥有事务,

###23.Spring和Hibernate的事务有什么区别？

Spring的事务提供了统一的事务处理机制,包含了JDBC,Hibernate,IBatis等事务实现,而Hibernate只处理自己事务

###24.Struts2与Spring整合先启动那个容器。

先启动监听器,因为先要初始化容器,初始化容器了以后Action才能从容器里面获得

###26.让你写Spring的容器，你是怎样实现的？

我们可以写一个HashMap,如果并发考虑的话要写并发的Map,把bean的名字放在map的key,bean的实现map的value

###27.谈谈Spring的IOC和AOP，如果不用Spring，怎么去实现这两个技术。

ioc用反射实现 ,AOP用动态代理实现

###28.Spring事务和Hibernate事务的操作上面的区别？

hibernate的事务只能手动显示代码的方式控制创建事务与提交事务以及回滚。

Spring可以通过配置文件设定一类class事务的创建与提交以及回滚，也可以显示代码方式控制。

###29.讲下Spring的七大事务传播

有七个,常用有两个REQUIERD, REQUIRED_NEW,REQUIERD表示两个事务的方法调用的时候,前面的时候和后面的合并成一个事务,REQUIRED_NEW是重启一个事务,各干各的

###30.在同一进程里，有A，B两个方法都对不同的表进行更新数据，假如A方法出异常了，若要B方法执行，怎样配置事务级别，若不要B方法执行，又该怎样配置？

前者用REQUIRED_NEW,后者用REQUIRED

###31.事务并发会引起什么问题,怎么解决

事务并发会引起脏读,幻读,不可重复读等问题,设定事务的隔离级别就可以解决

###32.事务的隔离级别

Spring定义有四种,但是常见的是READ_COMMIT,Oralce有两种实现,[MySQL](http://lib.csdn.net/base/mysql)有四种

###33.Spring的IOC容器与工厂类有什么区别？

IOC(Inversion of Control)对Bean的控制能力更强,能控制对象自动注入,还可以控制生命周期,而工厂类只是简单的创建一个对象,没有什么控制能力

###34.事务的安全问题：锁机制的实现原理及在项目中的使用

锁有悲观锁和乐观锁,悲观锁一般假设每个人都会修改数据,默认情况下把数据都锁住,影响性能,但安全性高.

乐观锁是假设每个人都只读下数据,不会修改数据,性能比较高,但是安全性较低,一般通过增加类似于[版本控制](http://lib.csdn.net/base/git)里面版本号来解决问题

###35.讲下BeanFactory和ApplicationContext的区别

BeanFactory是Spring容器顶级核心接口,比较早,但功能比较少,getBean就是BeanFactory定义的,

ApplicationContext是Spring里面的另外一个容器顶级接口,它继承于BeanFactory,但是提供的功能譬如校验,国际化,监听,

对Bean的管理功能比较多,一般使用ApplicationContext

###f-sm-1. 讲下SpringMvc和Struts1,Struts2的比较的优势

性能上Struts1>SpringMvc>Struts2 开发速度上SpringMvc和Struts2差不多,比Struts1要高

###f-sm-2. 讲下SpringMvc的核心入口类是什么,Struts1,Struts2的分别是什么

SpringMvc的是DispatchServlet,Struts1的是ActionServlet,Struts2的是StrutsPrepareAndExecuteFilter

###f-sm-3. SpringMvc的控制器是不是单例模式,如果是,有什么问题,怎么解决

是单例模式,所以在多线程访问的时候有线程安全问题,不要用同步,会影响性能的,解决方案是在控制器里面不能写字段

###f-sm-4. SpingMvc中的控制器的注解一般用那个,有没有别的注解可以替代

一般用@Controller注解,表示是表现层,不能用用别的注解代替.

###f-sm-5. @RequestMapping注解用在类上面有什么作用

用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

###f-sm-6. 怎么样把某个请求映射到特定的方法上面

直接在方法上面加上注解@RequestMapping,并且在这个注解里面写上要拦截的路径 

###f-sm-7. 如果在拦截请求中,我想拦截get方式提交的方法,怎么配置

springMVC模式的面试题部分

可以在@RequestMapping注解里面加上method=RequestMethod.GET

###f-sm-8. 如果在拦截请求中,我想拦截提交参数中包含"type=test"字符串,怎么配置

可以在@RequestMapping注解里面加上params="type=test"

###f-sm-9. 我想在拦截的方法里面得到从前台传入的参数,怎么得到

直接在形参里面声明这个参数就可以,但必须名字和传过来的参数一样

###f-sm-10. 如果前台有很多个参数传入,并且这些参数都是一个对象的,那么怎么样快速得到这个对象

直接在方法中声明这个对象,SpringMvc就自动会把属性赋值到这个对象里面 ###f-sm-11. 怎么样在方法里面得到Request,或者Session

直接在方法的形参中声明request,SpringMvc就自动把request对象传入

###f-sm-12. SpringMvc中函数的返回值是什么.

返回值可以有很多类型,有String, ModelAndView,List,Set等,一般用String比较好,如果是AJAX请求,返回的可以是一个集合

###f-sm-13. SpringMvc怎么处理返回值的

SpringMvc根据配置文件中InternalResourceViewResolver(内部资源视图解析器)的前缀和后缀,用前缀+返回值+后缀组成完整的返回值

###f-sm-14. SpringMVC怎么样设定重定向和转发的

在返回值前面加"forward:"就可以让结果转发,譬如"forward:user.do?name=method4" 在返回值前面加"redirect:"就可以让返回值重定向,譬如"redirect:http://www.uu456.com" 

###f-sm-15. SpringMvc用什么对象从后台向前台传递数据的

通过ModelMap对象,可以在这个对象里面用addAttribute()方法,把对象加到里面,前台就可以通过el表达式拿到

###f-sm-16. SpringMvc中有个类把视图和数据都合并的一起的,叫什么

ModelAndView

###f-sm-17. 怎么样把数据放入Session里面

可以声明一个request,或者session先拿到session,然后就可以放入数据,或者可以在类上面加上@SessionAttributes注解,

里面包含的字符串就是要放入session里面的key

###f-sm-18. SpringMvc怎么和AJAX相互调用的

通过Jackson框架就可以把[Java](http://lib.csdn.net/base/java)里面的对象直接转化成[js](http://lib.csdn.net/base/javascript)可以识别的Json对象 具体步骤如下 :

1.加入Jackson.jar

2.在配置文件中配置json的映射

3.在接受Ajax方法里面可以直接返回Object,List等,但方法前面要加上@ResponseBody注解

###f-sm-19. 当一个方法向AJAX返回特殊对象,譬如Object,List等,需要做什么处理

要加上@ResponseBody注解

###f-sm-20. SpringMvc里面拦截器是怎么写的

有两种写法,一种是实现接口,另外一种是继承适配器类,然后在SpringMvc的配置文件中配置拦截器即可:

<!-- 配置SpringMvc的拦截器 --> <mvc:interceptors> <!-- 配置一个拦截器的Bean就可以了 默认是对所有请求都拦截 -->

<bean id="myInterceptor" class="com.et.action.MyHandlerInterceptor"></bean>

<!-- 只针对部分请求拦截 --> <mvc:interceptor> <mvc:mapping path="/modelMap.do" /> <bean class="com.et.action.MyHandlerInterceptorAdapter" /> </mvc:interceptor> </mvc:interceptors>

###f-sm-21. 讲下SpringMvc的执行流程

系统启动的时候根据配置文件创建spring的容器, 首先是发送http请求到核心控制器DispatcherServlet，spring容器通过映射器去寻找业务控制器，

使用适配器找到相应的业务类，在进业务类时进行数据封装，在封装前可能会涉及到类型转换，执行完业务类后使用ModelAndView进行视图转发，

数据放在model中，用map传递数据进行页面显示。

