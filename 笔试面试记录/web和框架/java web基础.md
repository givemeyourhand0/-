##### 1、Cookie和Session

见百度脑图

##### 2、Servlet是线程安全的吗？

- Servlet不是线程安全的。
- 要了解Servlet是不是线程安全的，就需要知道Servlet容器是如何响应http请求的。当Tomcat收到http请求时，会从线程池中取出一个线程，之后找到对应的Servlet对象并进行初始化，之后调用service()方法。注意每个Servlet对象在Tomcat容器中只能有一个实例，即单例模式。如果多个请求请求的是同一个Servlet，那么多个请求对应的线程将并发的调用Servlet的service()方法。

##### 3、如何保证Servlet线程安全？

- 首先要保证变量的作用域合理，线程私有变量要定义在方法中。
- 共享变量要保证线程安全，可以使用锁，volatile关键字或atomic实现。