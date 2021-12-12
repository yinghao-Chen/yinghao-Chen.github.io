---
title: 弄懂JVM
author: 不好听
img: /2018/12/30/nong-dong-jvm/images/image1.png
top: true
categories: java
tags:
  - jvm
date: 2018-12-30 12:07:15
---

### 1.class Loader（类加载器）  
我们编写的java文件，编辑器会编译成<font color=#0099ff face="微软雅黑">*.class</font>的字节码文件；在我们执行代码时，会由<font color=#0099ff face="微软雅黑">类加载器</font>将文件加载到运行时数据区（内存）中去执行。  
我们先来看一段简单的代码：  
{% codeblock %}
public class Test {
	public static void main(String[] args) {
		System.out.println(Test.class.getClassLoader());
	}
}
{% endcodeblock %}  
执行将打印如下内容：  
![打印](images/image2.PNG "打印")  
打印出了加载这个类所用到的类加载器为<font color=#0099ff face="微软雅黑">AppClassLoader</font>，那么它具体是怎么加载的呢,加载过程是什么样的呢？  
打开命令行工具，cd进入Test.class所在目录，执行java指令:<font color=#0099ff face="微软雅黑">java -verbose test.Test</font>  
![打印](images/3.png "打印")  
有没有一种很熟悉的感觉？对，首先打开了rt.jar，然后就去加载了Object类...  
回到最开始执行代码打印的sun.misc.Launcher$AppClassLoader@73d16e93，在命令行中找到它:  
![打印](images/4.png "打印")  
可以看出，加载了这么几个类 启动类加载器（bootstrapClassLoader）、扩展类加载器(ExtClassLoader)、系统类加载器(AppClassLoader)   
在通过一段代码，探索一下他们的关系：  
{% codeblock %}
public class Test {
	public static void main(String[] args) {
		ClassLoader c = Test.class.getClassLoader();
		while(c != null) {
			System.out.println(c);
			c = c.getParent();
		}
	}
}
{% endcodeblock %}  
打印如下：  
![打印](images/6.png "打印")  
很明显，AppClassLoader的上级为ExtClassLoader（<font color=#0099ff face="微软雅黑">注意：不是继承关系，只是一种组合关系</font>）。那么ExtClassLoader的上级加载器是什么呢？  来看一下源码：  
![打印](images/5.png "打印")  
![打印](images/7.png "打印")  
上面这段代码，<font color=#0099ff face="微软雅黑">synchronized</font>保证了只有一个线程进行加载，然后loadclass方法是一个递归方法，一直找到最上层加载器才去进行加载（双亲委派机制），然后找不到上级的时候，最终去找到了一个<font color=#0099ff face="微软雅黑">native</font>本地方法（根据操作系统不同，实现也不一样，一般为c++实现的，是java实现跨平台的原因），即我们的启动类加载器。  
现在很清楚了，来总结一下，用下面的一张图进行说明：  
![打印](images/8.png "打印")  
启动类加载器，负责将JAVA_HOME/lib下的类库（如rt.jar）加载到内存中，这是一个本地私有方法，不同通过程序引用去调用执行。  
扩展类加载器，负责将JAVA_HOME/lib/ext或者由系统变量java.ext.dir指定位置的类库加载到内存。  
应用程序类加载器（即系统类加载器，getSystemClassLoader()的返回值），负责将classpath类路径下的类加载到内存。  

双亲委派机制： 加载时，自上而下；检查时，自下而上。  这也是java虚拟机为了保证程序执行安全的一种方式。下面我来简单解释一下：  
来看一段代码，新建一个名称为List的类：  
![打印](images/9.png "打印")  
这段代码虽然可以编译通过，但是却不能够正常执行。就是因为是自上而下加载的，List已经由上级加载器 加载过了（rt.jar中），就不能进行加载了。保证了在写出了不安全的程序时，JDK不让它执行。  

### 2.Runtime Data Area（运行时数据区） 
借助一张图来进行说明：  
![打印](images/10.png "打印")  
1. 堆。 对于大多数应用程序来说，堆是java虚拟机管理的内存中最大的一块。 堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。 此区域的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。 如果堆中没有内存完成实例分配，并且堆也无法再扩展时，就会抛出OutOfMemoryError异常。  
2. 方法区。 此区域同样是线程共享的区域。 被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等都存储在这里。 当方法去无法满足内存分配需求时，会抛出OutOfMemoryError异常。  
3. 程序计数器。 是一块较小的内存空间，当前线程所执行的字节码行号指示器。  
进去cmd,执行指令： javap -c Test  
![打印](images/11.png "打印")  
可以看到，编译后的每一条执行指令都有对应一个编号。 就是指向执行指令的地址，执行后做累加操作。使得程序按照一定的步骤去执行。  
4. 栈。 每当启动一个新线程的时候，java虚拟机都会为它分配一个栈。java以栈帧为单位来保存线程的运行状态。虚拟机只会为栈执行两种操作：以栈帧为单位的入栈和出栈。  

最后，下面这张图对于大家进行jvm调优肯定会有帮助的：  
![打印](images/12.png "打印")  
![打印](images/13.png "打印")  

![打印](images/14.png "打印")  
