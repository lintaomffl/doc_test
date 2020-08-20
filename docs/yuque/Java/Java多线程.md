# Java多线程

比较简洁的多线程介绍：[https://www.cnblogs.com/wxd0108/p/5479442.html](https://www.cnblogs.com/wxd0108/p/5479442.html)
<a name="JLjha"></a>
### 线程的状态：
<a name="Q3B4E"></a>
#### 状态枚举
![](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593676982449-a28a031a-3882-43d3-b21c-4e068f128253.png#align=left&display=inline&height=548&margin=%5Bobject%20Object%5D&originHeight=548&originWidth=1656&size=0&status=done&style=none&width=1656)
<a name="o98jW"></a>
#### 线程状态转换
![](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593676982448-29361051-0ee3-49f5-ba1e-3e90d5625c07.png#align=left&display=inline&height=419&margin=%5Bobject%20Object%5D&originHeight=419&originWidth=605&size=0&status=done&style=none&width=605)<br />各种状态一目了然，值得一提的是"blocked"这个状态：<br />线程在Running的过程中可能会遇到阻塞(Blocked)情况

1. 调用join()和sleep()方法，sleep()时间结束或被打断，join()中断,IO完成都会回到Runnable状态，等待JVM的调度。
1. 调用wait()，使该线程处于等待池(wait blocked pool),直到notify()/notifyAll()，线程被唤醒被放到锁定池(lock blocked pool )，释放同步锁使线程回到可运行状态（Runnable）
1. 对Running状态的线程加同步锁(Synchronized)使其进入(lock blocked pool ),同步锁被释放进入可运行状态(Runnable)。

此外，在runnable状态的线程是处于被调度的线程，此时的调度顺序是不一定的。Thread类中的yield方法可以让一个running状态的线程转入runnable。
<a name="EfMku"></a>
### 线程对象的一些规则：
<a name="Auel9"></a>
#### A.synchronized
**synchronized原理**<br />在java中，每一个对象有且仅有一个同步锁。这也意味着，同步锁是依赖于对象而存在。<br />当我们调用某对象的synchronized方法时，就获取了该对象的同步锁。例如，synchronized(obj)就获取了“obj这个对象”的同步锁。<br />不同线程对同步锁的访问是互斥的。也就是说，某时间点，对象的同步锁只能被一个线程获取到！通过同步锁，我们就能在多线程中，实现对“对象/方法”的互斥访问。 例如，现在有两个线程A和线程B，它们都会访问“对象obj的同步锁”。假设，在某一时刻，线程A获取到“obj的同步锁”并在执行一些操作；而此时，线程B也企图获取“obj的同步锁” —— 线程B会获取失败，它必须等待，直到线程A释放了“该对象的同步锁”之后线程B才能获取到“obj的同步锁”从而才可以运行。<br />**synchronized基本规则**<br />**Synchronized 作用在代码块的时候，线程在访问的时候都只能互斥地访问。**<br />**Synchronized 作用在对象的时候，则遵循以上三条规则。**<br />我们将synchronized的基本规则总结为下面3条，并通过实例对它们进行说明。<br />**[第一条](https://www.cnblogs.com/skywang12345/p/3479202.html#a21)**: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对**“该对象”的该“synchronized方法”或者“synchronized代码块”的访问**将被阻塞。<br />第二条: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然**可以访问“该对象”的非同步代码块**。<br />第三条: 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对**“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问**将被阻塞。

三条规则的实例说明：[https://www.cnblogs.com/skywang12345/p/3479202.html#p4](https://www.cnblogs.com/skywang12345/p/3479202.html#p4)<br />

<a name="nfMN0"></a>
#### B.Volatile
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593835084804-60021959-8006-4c97-8dc7-d2b8173462b7.png#align=left&display=inline&height=151&margin=%5Bobject%20Object%5D&name=image.png&originHeight=302&originWidth=535&size=104436&status=done&style=none&width=267.5)<br />java通常的内存模型如上，在以上模型中，各线程都会从主内存中拷贝一份变量副本到自己的工作内存之中，然后对于共享内存进行作用。<br />**Volatile主要作用：**<br />该关键字修饰的变量绕过了工作内存，使得线程对于共享变量的读取都直接针对主内存中的共享变量。但是**volatile只实现了可读性，没有实现原子性操作**，是轻量级的synchronized.<br />
<br />**Synchronized与volatile区别** <br />1.volatile只能修饰变量，而synchronized可以修改变量，方法以及代码块<br /> <br />2.volatile在多线程中不会存在阻塞问题，synchronized会存在阻塞问题<br /> <br />3.volatile能保证数据的可见性，但不能完全保证数据的原子性，synchronized即保证了数据的可见性也保证了原子性<br /> <br />4.volatile解决的是变量在多个线程之间的可见性，而sychroized解决的是多个线程之间访问资源的同步性
