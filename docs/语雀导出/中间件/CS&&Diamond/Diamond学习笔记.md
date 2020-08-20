# Diamond学习笔记

<a name="hjeLB"></a>
## Diamond基本介绍
<a name="rzQOj"></a>
### Diamond提供的主要功能：
——**配置持久化管理**与动**态配置推送服务**。<br />**配置持久化管理**：应用方发布的配置会通过持久化存储保存，与发布者的生命周期无关。<br />**动态配置推送服务**：开发人员可以通过控制台、客户端API，发布、变更配置值，借助Diamond提供配置变更推送功能，可以实现不重启应用而获取最新的配置值，则是Diamond的核心功能，在淘宝内部有很多应用场景，如数据库动态切换和扩容，业务系统开关配置运行时变更等。
<a name="RnPIX"></a>
### Diamond适用范围：

- 1.**低频变更**的动态配置管理；
- 2.对于配置的变更**只保证最终一致性**，不保证过程一致性；
- 3.要求接入的**业务满足幂等性。**<br />
<a name="D7r60"></a>
### Diamond限流阈值：

- 1.单个配置写的频率不超过1分钟1次
- 2.批量配置下发不应该超过10QPS
<a name="mDqly"></a>
### Diamond使用约束：

- 1.配置中心只保存最新数据，中间数据的变更过程并不保留，只会确保订阅者得到最新的状态变更信息；
- 2.仅支持幂等性的业务场景；
- 3.订阅者获得消息存在先后顺序，业务场景下如若各个订阅者之间收到消息的顺序存在逻辑性则不适合diamond
- 4.不适合过大的消息发送，不大于100k，单个应用的数据总和不超过1MB；
- 5.配置中心不保证读写一致性，发布者发布数据后，即使同一节点上的订阅者也不一定能马上收到更新。但配置中心将确保全局稳态下的最终一致性。
> 如果把订阅和发布分别看作读和写，那么写入新数据后立即读取的数据可能还是更新前的数据，而不是新写入的数据。在正常情况下，经过短暂的时间后，应可读到更新后的数据。



<a name="Ynn9c"></a>
## Diamond架构


<a name="p3t6a"></a>
### 组成部分：
**服务地址组件：**用于发现服务，供应用方进行通过服务地址组件来找到Diamond服务器，为运维人员提供了集群扩容，也让应用方透明地进行下线运维。<br />**Diamond-Server组件：**<br />可进行配置消息传递的主要部分。Diamond-Client通过调用客户端的API来发布配置；Server集群对发布要求进行响应，且各个服务器之间相互同步信息，更新各自的缓存；<br />**MySQL存储部分：**<br />用来对于配置信息的持久化处理。<br />

<a name="YMb66"></a>
### 主要部分的工作流程：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591152702447-d3333199-0f30-498d-a8e7-476520f783ae.png#align=left&display=inline&height=380&margin=%5Bobject%20Object%5D&name=image.png&originHeight=567&originWidth=925&size=180185&status=done&style=none&width=620)<br />如上图可见。<br />

<a name="cK8Mg"></a>
#### 消息（配置信息）传递过程：
**A.配置信息从客户端发起到服务器的部分——**<br />1.需要更变配置时，利用API发布事件变更消息到Diamond服务器集群；<br />2.服务器集群收到消息以后，利用MySQL来持久化处理；<br />3.然后再返回响应给客户端；<br />
<br />**B.集群之的动作——**这一部分的机制实现了去中心化<br />1.各个集群集群进行变更事件的同步；<br />2.各集群将相关信息保存到本地缓存。<br />
<br />**C.客户端获得消息配置信息——**<br />1.轮训获得最新值；<br />2.服务端用本地缓存直接相应<br />

<a name="3ug9E"></a>
### 保证消息(配置信息)可靠性传递的机制：容灾
配置信息消息在：客户端缓存、服务器磁盘、数据库Sql持久化保存、容灾目录都有保存；<br />主备数据库都有对消息进行了持久化处理；<br />各个服务器集群进行了同步；<br />客户端留有缓存、数据库SQL的保存。<br />

<a name="RxvP2"></a>
### 配置信息推送机制：
在Diamond产品发展过程中，先后使用了2种思路。

1. 基于拉模型的客户端轮询的方案，如图-2
   - 客户端通过轮询方式发现服务端的配置变更事件。轮询的频率决定了动态配置获取的实时性。
   - 这种方案最大的好处就是简单、可靠。但是，当用户量增加时，较高的轮询频率给整个Diamond服务带来的压力也不断增加。另外，从配置中心的应用场景上来看，是一种写少读多的系统，客户端大多数轮询请求都是没有意义的，因此这种方案不够高效。
2. 基于推模型的客户端长轮询的方案，如图-3
   - 当前淘宝生产环境中的Diamond服务，采用了这种方案来实现动态配置实时推送。
   - 基于Http长轮询模型，实现了让客户端在没有发生动态配置变更的时候减少轮询。这样减少了无意义的轮询请求量，提高了轮询的效率；也降低了系统负载，提升了整个系统的资源利用率。
   - 另外，这种推拉结合的策略，做到了在长连接和短连接之间的平衡，实现上让服务端不用太关注连接的管理，效果上又获得了类似TCP长连接的信息推送的实时性。


<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591153690273-e3080bd4-8aea-42e7-a43f-c62870e8ba37.png#align=left&display=inline&height=474&margin=%5Bobject%20Object%5D&name=image.png&originHeight=762&originWidth=925&size=150689&status=done&style=none&width=576)<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591153701186-8aab54d6-ca7e-478f-bdcb-632a9cad86d3.png#align=left&display=inline&height=429&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=925&size=124823&status=done&style=none&width=580)<br />
<br />

<a name="HvsX6"></a>
### 高效的Diamond
Diamond的高效，主要体现在它提供的高吞吐率的配置获取，以及实时的动态配置推送能力上。
> 引入cache，借助zero-copy

为了提高数据读取吞吐率，引入cache是一个普遍的思路。Diamond的持久化配置保存在mysql中，额外的服务端每个节点都以本地磁盘文件的形式保存了全量的配置数据作为cache。当客户端请求读取配置数据时，服务端直接读取本地磁盘文件即可。 这样不但减少了对数据库单点的访问压力，也可以方便的实现配置获取服务的水平扩展。 引入服务端配置数据的缓存文件后，一次配置数据的读取请求，其实就是一次静态文件的Http请求。借助0拷贝的优化，减少了配置数据在应用态和内核态之间无用的内存拷贝，大大提升了数据传输效率。 借助这上述两点优化，diamond-server单节点（4核，8G虚拟机）的配置（1KB）读取能力在qps 6000+。<br />
<br />
<br />参考学习的资料源：<br />介绍文档：[http://mw.alibaba-inc.com/products/diamondserver/_book/programer-magazine-configserver.html?spm=a1zco.8292286.0.0.1ad52588rh3sbv](http://mw.alibaba-inc.com/products/diamondserver/_book/programer-magazine-configserver.html?spm=a1zco.8292286.0.0.1ad52588rh3sbv)<br />内部Wiki：[http://gitlab.alibaba-inc.com/middleware/diamond/wikis/Diamond-Intro](http://gitlab.alibaba-inc.com/middleware/diamond/wikis/Diamond-Intro)<br />一个源码分析学习的网站：[https://blog.csdn.net/rainforc/article/details/84769981](https://blog.csdn.net/rainforc/article/details/84769981)
