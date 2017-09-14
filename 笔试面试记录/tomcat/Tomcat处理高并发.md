#### 简单的说就是

当Tomcat接收到Client的HTTP请求时，Tomcat从**线程池**中取出一个线程，之后找到该请求对应的Servlet对象并进行初始化，之后调用service()方法。每一个Servlet对象再Tomcat容器中只有一个实例对象，即是**单例模式**。如果多个HTTP请求请求的是同一个Servlet，那么着两个HTTP请求对应的线程将并发调用Servlet的service()方法。