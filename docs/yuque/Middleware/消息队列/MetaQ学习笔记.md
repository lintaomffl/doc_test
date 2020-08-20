# MetaQ学习笔记

MetaQ是一款分布式、队列模型的消息中间件。基于发布订阅模式，有Push和Pull两种消费方式，支持严格的消息顺序，亿级别的堆积能力，支持消息回溯和多个维度的消息查询。<br />

<a name="eIabt"></a>
## MetaQ面向的结构：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1590738039517-e584924e-1c43-46f1-9021-b1fa092a0c70.png#align=left&display=inline&height=354&margin=%5Bobject%20Object%5D&name=image.png&originHeight=342&originWidth=623&size=124129&status=done&style=none&width=644)<br />如上所示，分为三个部分：
<a name="aqzcV"></a>
### 从结构上看：
Broker：又分为Master与Slave，其中master可接收Producer提供的消息并同步给其他slave；另一方面slave与master都会给comsumer提供消息。<br />Client：分为消息提供方Producer与消息的消费方Comsumer。<br />nameServer：用来记录各topic的路由信息，供Client查询topic的路由。
<a name="d48n6"></a>
### 从三个群体的工作过程看：
broker：<br />a.每个broker都与nameServer建立长连接，用来注册topic的路由信息——（如此，则每个nameServer都能关注到所有Broker每个topic的路由信息，client也只需要与任意一个nameSever建立长链接即可得到所有topic信息）<br />Producer：消息提供方<br />a.与任意一nameserver建立长连接，从而掌握topic路由信息；<br />b.与提供topic的master broker建立长连接，并将消息推送过去；<br />c.master Broker负责将消息同步给所有的Broker（是指将消息同步给对应的slave还是所有slave还是所有slave与master？）<br />Comsumer：<br />a.与任意一nameserver建立长连接，从而掌握topic路由信息；<br />b.与提供topic的master和slave的broker都建立长连接，并订阅消息；

<a name="rvAuy"></a>
## 从工作过程的角度看MetaQ的特征
从以上结构可以看出MetaQ的结构是围绕Broker的角度，来提供消息的传递服务的中间件，那么显然我们可以从消息传达、消息同步、Broker消息的情况(消息队列、Broker的cache内存管理)等角度继续分析MetaQ的特性。
<a name="XdKTB"></a>
### 消息传达：
消息实时：利用建立长连接的方式，保证消息的实时性；<br />消息至少发送一次：Comsumer对于消息Pull以后，进行消费后才会返回结果。并且不保证不重复发送（一定程度保证了消息会传达）。并且设立了消息传达不成功也不失败的状态（什么情况下用？——分布式事务消息处理的时候会用，用于消费者回查本地事务消息的状态。）<br />Pull与Push：

   - Push模式：很难掌握消息推送的时机和速率，因为consumer的消费速率不同。
   - Pull模式：consumer可以根据自己的状况选择拉取消息的时机和速率，缺点在于如果服务端没有可供消费的消息，将导致consumer不断轮询，浪费资源。



<a name="YZAg1"></a>
### 消息同步：（保证消息可靠性）
1.从提供方到Broker：<br />Broker收到消息以后会向producer返回SendResult。整个过程同步，保证消息可靠地传输给Broker；<br />2.Broker对于消息的持久化：<br />Broker对于消息持久化地保持在本地磁盘，从而保证消息的持久化地在Broker进行保存；即利用**持久化到文件**的方式进行持久化。<br />3.Consumer对于消息进行拉取消费：<br />a.顺序消费：消费者是一条接着一条地顺序消费消息，只有在成功消费一条消息后才会接着消费下一条。<br />b.可靠消费：如果在消费某条消息失败（如异常），则会尝试重试消费这条消息（默认最大5次），超过最大次数后仍然无法消费，则将消息存储在消费者的本地磁盘，由后台线程继续做重试。而主线程继续往后走，消费后续的消息。由此来保证消息的可靠消费。
<a name="NeIo8"></a>
### 消息队列：
消息队列的存储方式：图片链接：[http://blog.ludanyang.love/images/metaq.png](http://blog.ludanyang.love/images/metaq.png)<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591090896009-531051c8-1975-4280-b66e-3c37f8617084.png#align=left&display=inline&height=268&margin=%5Bobject%20Object%5D&name=image.png&originHeight=536&originWidth=1127&size=94808&status=done&style=none&width=563.5)<br />该类消息队列形式可以保证局部有序。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591103134878-1ea6c3c7-9ef4-4984-b060-acf299c34fed.png#align=left&display=inline&height=339&margin=%5Bobject%20Object%5D&name=image.png&originHeight=678&originWidth=1214&size=210807&status=done&style=none&width=607)<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591103168077-cd76e662-9b7b-4c65-b620-3b2f3ad0272f.png#align=left&display=inline&height=337&margin=%5Bobject%20Object%5D&name=image.png&originHeight=674&originWidth=1206&size=133691&status=done&style=none&width=603)<br />metaq的逻辑存储结构是一种物理队列+逻辑队列的结构。<br />[![](https://yuque.antfin.com/attachments/lark/0/2020/png/304852/1590740634791-4ab84f80-a973-464a-b646-a82ff031136b.png#align=left&display=inline&height=571&margin=%5Bobject%20Object%5D&originHeight=571&originWidth=626&size=0&status=done&style=none&width=626)](https://yq.aliyun.com/go/articleRenderRedirect?spm=a2c4e.11153940.0.0.29c058f8tMQzhC&url=http%3A%2F%2Fimg2.tbcdn.cn%2FL1%2F461%2F1%2F3d92994d64253f1a3ea37ca8b8267220e188319f)<br />物理队列只有一个，采用固定大小的文件顺序存储消息。逻辑队列有多个，每个逻辑队列有多个分区，每个分区有多个索引。<br />a.消息顺序写入物理文件里面，每个文件达到一定的大小，新建一个文件继续顺序写数据（消息的写入是串行的，避免了磁盘竞争）。<br />b.消息的索引则顺序的写入逻辑文件中，并不存放真正的消息，只是存放指向消息的索引。<br />metaq对于客户端展现的是逻辑队列就是消费队列，consumer从消费队列里顺序取消息进行消费。<br />这种设计是把物理和逻辑分离，消费队列更加轻量化。所以metaq可以支撑更多的消费队列数，提升消息的吞吐量，并且有一定的消息堆积能力。<br />**缺点** ：<br />写虽然是顺序写，但是读却是随机读的<br />**解决办法** ：尽可能让读命中pageCache，减少磁盘IO次数 (参考下文所述：Linux的文件Cache管理) <br />metaq的所有消息都是持久化的，先写入系统PAGECACHE（页高速缓存），然后刷盘，可以保证内存与磁盘都有一份数据，访问时，直接从内存读取。<br />刷盘策略分为异步和同步两种。<br />

<a name="12"></a>
### Linux的文件Cache管理
在 Linux 操作系统中，为了加快文件的读写，当应用程序需要读取文件中的数据时，操作系统先分配一些内存，将数据从存储设备读入到这些内存中，然后再将数据分发给应用程序；当需要往文件中写数据时，操作系统先分配内存接收用户数据，然后再将数据从内存写到磁盘上。<br />文件 Cache 管理就是对这些由操作系统分配，并用来存储文件数据的内存的管理。 <br />Cache 管理的优劣通过两个指标衡量：

- Cache 命中率：Cache 命中时数据可以直接从内存中获取，不再需要访问低速外设，因而可以显著提高性能；
- 有效 Cache 的比率：有效 Cache 是指真正会被访问到的 Cache 项。如果有效 Cache 的比率偏低，则相当部分磁盘带宽会被浪费到读取无用 Cache 上，而且无用 Cache 会间接导致系统内存紧张，最后可能会严重影响性能。 在 Linux 的实现中，文件 Cache 分为两个层面，一是 Page Cache，另一个 Buffer Cache。 每一个 Page Cache 包含若干 Buffer Cache。
<a name="13"></a>
### 通过内存映射的方式读写文件
metaq在文件读写操作上做了一定的优化，使用内存映射的方式完成读写，替代了传统的IO操作，从而大大的减少了文件读写系统调用的次数，提升了IO的性能。<br />传统的文件访问：

- 系统调用打开文件，获取文件描述符
- 使用read write 系统调用进行IO
- 系统调用关闭文件

这种方式是非常低效的, 每一次I/O操作都需要一次系统调用。 另外, 如果若干个进程访问同一个文件, 每个进程都要在自己的地址空间维护一个副本, 浪费了内存空间<br />内存映射的方式：

- 打开文件，得到文件描述符。
- 获取文件大小
- 把文件映射成虚拟内存（mmap）
- 通过对内存的读写来实现对文件的读写（memset或memcpy）
- 卸载映射
- 关闭文件

首先建立好虚拟内存和磁盘文件之间的映射（mmap系统调用），当进程访问页面时产生一个缺页中断，内核将页面读入内存(也就是说把磁盘上的文件拷贝到内存中)，并且更新页表指向该页面。<br />所有进程共享同一物理内存，物理内存中可以只存储一份数据，不同的进程只需要把自己的虚拟内存映射过去就可以了，这种方式非常方便于同一副本的共享，节省内存。<br />经过内存映射之后，文件内的数据就可以用内存读/写指令来访问，而不是用Read和Write这样的I/O系统函数，从而提高了文件存取速度。

学习框架内容主要参考：[https://yq.aliyun.com/articles/2892](https://yq.aliyun.com/articles/2892)<br />一个介绍的很清楚的博客：[http://blog.ludanyang.love/2016/07/25/AliBaBa/metaQ%E5%AD%A6%E4%B9%A0%E6%8A%A5%E5%91%8A/](http://blog.ludanyang.love/2016/07/25/AliBaBa/metaQ%E5%AD%A6%E4%B9%A0%E6%8A%A5%E5%91%8A/)<br />MetaQ源码分析：[https://blog.csdn.net/weixin_36372879/article/details/94436781](https://blog.csdn.net/weixin_36372879/article/details/94436781)<br />可能是更好的消息中间件学习资料：[https://open.atatech.org/articles/95456](https://open.atatech.org/articles/95456)    [https://open.atatech.org/articles/170795](https://open.atatech.org/articles/170795)<br />Q：<br />1.broker 中master与slave的对应关系？1对1，一对多？
