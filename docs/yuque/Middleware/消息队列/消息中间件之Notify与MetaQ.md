# 消息中间件之Notify与MetaQ

Q：1.Notify的exchange是怎么计算出路由的，是否需要维护一个nameserver来查询订阅者的ID？——是的，也要对应地维护订阅关系。<br />2.两种模式下面，消费者消费失败消息以后的处理办法。<br />
<br />消息中间件主要为了完成：解耦、异步、并行。<br />其中**解耦**是为了是的上下游的处理过程相互不干扰，**异步**和**并行**都是为了提高处理的效率。<br />但是为了保证消息传达与消费的可靠性，不能仅仅只考虑这三个层面，同样要用一种合理的机制来促使消息可靠传达且消费。<br />所以可以从一下几个方面去考虑两者的异同：<br />**消息过滤与订阅方式、结构以及工作过程、保证消息可靠传达消费的机制、消息在中间件服务器的持久化方式**<br />Notify——适用于交易系统，对于下游类似支付、SNS等可以并行化的流程进行解耦与异步。
<a name="QVRSd"></a>
## 消息过滤与订阅方式：
|  | Notify | MetaQ |
| --- | --- | --- |
| 话题模式 | 1.采用订阅模式；<br /> | 1.订阅Topic模式<br /> |
| 消息传递方法 | 2.Nofity框架中的Broker维护一个exchange，查阅nameServer(也可能是维护在Broker的表格)中Provider与Consumer之间的订阅关系来计算出对应的Consumer的路由，并将消息Push到对应的Consumer。 | 2.将Provider与Consumer都与NameServer连接，前者注册自己发布的话题，后者订阅自己感兴趣的话题。<br />但是有PULL与PUSH两种模式，Provider将消息发布到Broker以后，**PULL模式**等待Consumer主动拉取消息，**PUSH模式**则为Broker将消息主动PUSH给Consumer。 |
| 消息过滤 | Broker根据订阅关系规则过滤消息，并且找到目标Consumer进行消息投递。 | a.服务器端，消息的tag转换成hashcode。并将hashcode与订阅hashcode进行对比过滤。<br />b.消费者，也对于自己订阅话题的tag与传来的消息tag进行对比。 |

<a name="qnGbC"></a>
## 结构以及过程：
<a name="lJ8ky"></a>
## 保证消息传达与消费可靠性质的机制：
|  | Notify | MetaQ |
| --- | --- | --- |
| Provider给Broker传递消息 | 传达消息以后，确认成功传达。<br />记录sendResult，若成功则令isSuccess为yes<br /> | 消息传达以后，确认消息传达<br /> |
| Broker的本地持久化消息 | 1.Notify通常利用KV存储消息。<br />2.对于事务消息的特殊的传递模式。 | 利用保存到本地磁盘的方式进行持久化消息。<br />有同步持久化模式和异步持久化模式。前者当即写入，后者定时强制写入。前者更能保证可靠。 |
| Broker给Consumer传递消息与Consumer的消息消费 | Broker在给Consumer投递完消息以后，维护一个callback等待消费者消费完以后返回的ack；<br />根据callback的情况来决定删除kv的消息还是更新kv消息状态。 | Consumer将消息拉下以后，进行顺序消费，从而保证消息都能够被消费；<br />若某条消息消费失败，则会重试（最多5次），都失败以后则将消息保存本地，继续消费后续的消息。 |

<a name="T8Ssb"></a>
## 消息在中间件服务器的持久化方式：
|  | Notify | MetaQ |
| --- | --- | --- |
| 持久化方式 | 通常利用kv存储<br /> | 通常进行写入本地磁盘来进行持久化<br />队列存储方式 |

