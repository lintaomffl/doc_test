# Notify学习笔记

Notify可以支持事务消息与普通消息的处理，并且针对事务消息会有不同的处理方法。
<a name="b7ZeC"></a>
## Notify框架：

- Producer：消息的上游，消息生产者
- Consumer：消息的下游，消息消费者
- Message: 消息，作为异步通信的数据载体。
- Broker：消息中间件服务器
- Topic：消息主题，用于发布订阅模式
- Queue: 消息队列，用于点对点模式
- Subscription：订阅关系，用于发布订阅模式，表示消费者的感兴趣消息类型
- Exchange:交换器。用于消息订阅计算，找到目标Consumer。

message关键字段：<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1590978923661-e31dead2-9542-4d25-b97c-59acf6ca5292.png#align=left&display=inline&height=138&margin=%5Bobject%20Object%5D&name=image.png&originHeight=276&originWidth=1116&size=55167&status=done&style=none&width=558)<br />

<a name="A0ZjJ"></a>
## Notify对于消息的处理流程：
<a name="mdSTG"></a>
### 对于普通消息的处理：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1590978944428-4554d961-4d94-4cb3-b7f6-29e1e03e94c2.png#align=left&display=inline&height=242&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=1056&size=119269&status=done&style=none&width=528)<br />1.peoducer向Notify服务器发送消息；<br />2.notify对于消息进行持久化处理，采用kv存储的形式<br />3.notify向producer返回一个ack确认；<br />4.notify持久化后执行exchange，这一步会对消费者Subscription的属性过滤规则进行计算，得到目标消费者id集合，找到消费者channel直接push。push完后，内存中维护callback等待消费者ack（成功或者失败），如果没有ack，callback默认等待5秒，作为消费超时处理。<br />5.最多等待5秒后，callback收集到所有ack，如果全部成功，那么直接删除消息记录；如果部分失败，对消息记录进行修改，主要是修改fail target字段，记录失败消费者id。notify会有后台任务扫描未删除记录，重新投递给失败的消费者id，再次执行4、5、6步骤。
<a name="FgApL"></a>
### 对于事务消息的处理：
对于事务消息的处理是需要保证：**事务消息投**递与**本地动作执**行的原子性<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1590979251874-d274ab3f-7eec-48d8-b6e1-b68aa28bd6d5.png#align=left&display=inline&height=242&margin=%5Bobject%20Object%5D&name=image.png&originHeight=483&originWidth=1058&size=145139&status=done&style=none&width=529)<br />对于事务消息，区别之处在于消息生产环节。生产者先发送half消息，当notify收到half消息后，先进行持久化，在kv存储增加一条half（即commited字段为false）记录，然后ack给生产者，此时并不会触发推送消费者事件。生产者half发送成功后，执行本地事务，事务执行成功后，发送commit请求。notify收到commit请求后，将消息的commited字段更新为true，然后就立马执行推送消费者的流程，消费相关逻辑和普通消息无异。<br />在一些场景下，在1、2、3步执行完成后，Notify服务器可能会查询producer的本地动作实行状态——message check机制：为了针对一些producer在commit之前down机的场景，然后notify会主动发起查询producer的情况，已完成message后续的解决（删除或者继续操作）。

二阶段提交：

几个问题：<br />1.什么是二阶段、三阶段提交？<br />2.什么的是幂等？——多次提交的效果与一次提交的效果相同

学习参考的网站：[https://open.atatech.org/articles/95456](https://open.atatech.org/articles/95456) 隆基——阿里消息中间件架构演进之路：notify和metaq<br />[https://blog.csdn.net/lengxiao1993/java/article/details/88290514](https://blog.csdn.net/lengxiao1993/java/article/details/88290514) 
