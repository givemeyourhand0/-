> 转载自：http://blog.csdn.net/woshixuye/article/details/8230407

###1、基本概念

![java异常](E:\笔记\笔试面试记录\图片\java异常.png)

Throwable是所有异常的根，java.lang.Throwable
Error是错误，java.lang.Error
Exception是异常，java.lang.Exception

#### 2、Exception

一般分为Checked异常和Runtime异常，所有RuntimeException类及其子类的实例被称为Runtime异常，不属于该范畴的异常则被称为CheckedException。

##### 2.1、**Checked异常**

只有java语言提供了Checked异常，Java认为Checked异常都是可以被处理的异常，所以Java程序必须显示处理Checked异常。如果程序没有处理Checked异常，该程序在编译时就会发生错误无法编译。这体现了Java的设计哲学：没有完善错误处理的代码根本没有机会被执行。对Checked异常处理方法有两种

1 当前方法知道如何处理该异常，则用try...catch块来处理该异常。
2 当前方法不知道如何处理，则在定义该方法是声明抛出该异常。

我们比较熟悉的Checked异常有

Java.lang.ClassNotFoundException
Java.lang.NoSuchMetodException
java.io.IOException

##### 2.2、**RuntimeException**异常

Runtime如除数是0和数组下标越界等，其产生频繁，处理麻烦，若显示申明或者捕获将会对程序的可读性和运行效率影响很大。所以由系统自动检测并将它们交给缺省的异常处理程序。当然如果你有处理要求也可以显示捕获它们。

我们比较熟悉的RumtimeException类的子类有

Java.lang.ArithmeticException
Java.lang.ArrayStoreExcetpion
Java.lang.ClassCastException
Java.lang.IndexOutOfBoundsException
Java.lang.NullPointerException



#### 3、Error

当程序发生不可控的错误时，通常做法是通知用户并中止程序的执行。与异常不同的是Error及其子类的对象不应被抛出。

Error是throwable的子类，代表编译时间和系统错误，用于指示合理的应用程序不应该试图捕获的严重问题。

Error由Java虚拟机生成并抛出，包括动态链接失败，虚拟机错误等。程序对其不做处理。

