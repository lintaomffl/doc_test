# TDDL学习笔记

Q：1.对于数据库采用读写分离的同时进行分库分表，那么保证slave与master一致的情况下。是否slave也要对应的不断扩充？那么对于服务器的路由查询会不会产生一个很大的代价开销？<br />2.TDDL的主从备份、切换如何实现？<br />3.TDDL的主从读写的路由规则是怎么实现的？<br />4.主从备份是那个层面去做的？Matrix层还是Repo层？<br />可参考的资料：[https://open.atatech.org/articles/169824](https://open.atatech.org/articles/169824)<br />[https://open.atatech.org/articles/144751#77](https://open.atatech.org/articles/144751#77)<br />[http://mw.alibaba-inc.com/products/tddl/_book/?spm=a1zco.8288981.0.0.63a3f7d9z4nIVx](http://mw.alibaba-inc.com/products/tddl/_book/?spm=a1zco.8288981.0.0.63a3f7d9z4nIVx)<br />
<br />TDDL是一种分布式数据库访问引擎
<a name="b9Yeo"></a>
## TDDL整体架构：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591254958324-1a1a1f6a-4396-4305-a52b-ca11c5705e27.png#align=left&display=inline&height=368&margin=%5Bobject%20Object%5D&name=image.png&originHeight=578&originWidth=611&size=45068&status=done&style=none&width=389)
<a name="9xexO"></a>
## TDDL三层DB模型：
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591255001671-1993c182-3470-4487-8fda-bf5bc7aad9dd.png#align=left&display=inline&height=293&margin=%5Bobject%20Object%5D&name=image.png&originHeight=399&originWidth=746&size=203162&status=done&style=none&width=547)<br />Matrix层主要负责SQL的解析、优化、执行；<br />Repo层主要负责数据存储。
<a name="n1ELB"></a>
### Matrix：
Matrix对于SQL进行解析，并利用路由规则对于相应的DB进行查询，但是这一步骤对于应用来说是透明的。然后Matrix再对于SQL进行分发。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591255090286-b2d46741-a22a-42d0-af92-99a5b1c7b1b6.png#align=left&display=inline&height=302&margin=%5Bobject%20Object%5D&name=image.png&originHeight=369&originWidth=525&size=16864&status=done&style=none&width=430)
<a name="rvHeu"></a>
### Group：
这一层主要实现了读写分离和主从备份。Group根据收到的上层的sql，读还是写来对应的去访问Atom层的主从数据库。<br />主从备份有两种情况：1.主从数据库的数据存储方式一致，直接考虑从库重复执行主库的语句。2.两种数据存储方式不同。例如，主库以买家的ID作为主键保存，这样便于买家对于信息进行查询，从库以卖家的ID作为主键进行存储，那么那么从库便于买家进行查询。这样的两种数据库的存储模式，得通过一定的路由方式进行存储，可以利用中间件愚公实现。<br />
<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591255600951-bf05ecf8-3c8c-43fb-b9a9-5ee1fb7c6491.png#align=left&display=inline&height=233&margin=%5Bobject%20Object%5D&name=image.png&originHeight=325&originWidth=608&size=32094&status=done&style=none&width=435)<br />

<a name="dANFK"></a>
### Atom：
这一层就是讲SQL语句进行执行，返回结果。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591255669619-2a53b6e1-79af-469c-ab6a-dc15c3fc14c1.png#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&name=image.png&originHeight=400&originWidth=722&size=47973&status=done&style=none&width=431)<br />
<br />

<a name="meqWn"></a>
## TDDL执行流程
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1591255705393-1b3f2535-6f20-4bf1-ae15-527876930464.png#align=left&display=inline&height=210&margin=%5Bobject%20Object%5D&name=image.png&originHeight=334&originWidth=693&size=106665&status=done&style=none&width=435)
<a name="Q4wyr"></a>
### 流程说明：
<a name="cxc2A"></a>
#### Matrix
1.解析SQL:client的sql请求到达matrix层，Matrix对sql进行解析，生成语法树，并对sql进行优化；<br />2.规则计算：根据生成的语法树，匹配分库分表规则，进行规则计算（具体计算下面详细介绍）<br />3.重组SQL：用规则计算后的物理表名进行替换，生成新的sql；<br />4.分发SQL：选择与规则计算结果匹配的group，将sql转发到对应的group进行处理；
<a name="bIqZr"></a>
#### Group
1.根据权重选择服务器（根据读or写）<br />2.采用重试的查询机制，保证sql语句的执行
<a name="pu8Ie"></a>
#### Atom
1.执行sql<br />2.返回结果<br />

<a name="13KeE"></a>
## TDDL的Sequence生成唯一的ID

<br />参考的网站：配置方式——[http://mw.alibaba-inc.com/products/tddl/_book/Sequence-Quick-Start.html?spm=a1zco.tddl.0.0.3d452588nLt14V](http://mw.alibaba-inc.com/products/tddl/_book/Sequence-Quick-Start.html?spm=a1zco.tddl.0.0.3d452588nLt14V)<br />参见例子，可见按照以上方式配置以后，利用sequence.nextval()即可。
<a name="92"></a>
#### 全局ID原理
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/10039/1562396339459-cd8a9526-35e0-400d-9fde-d2056ead440d.png#align=left&display=inline&height=280&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=359&size=16207&status=done&style=none&width=359)<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/10039/1562396357496-6eefc4af-a5b2-4158-811c-77313d41cee6.png#align=left&display=inline&height=280&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=625&size=28023&status=done&style=none&width=625)
<a name="93"></a>
#### 高并发场景下的唯一ID
可以保证ID唯一但是不一定连续<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/10039/1562396377051-da68ad81-8f80-46a3-b854-0993df3ef3d3.png#align=left&display=inline&height=280&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=570&size=34413&status=done&style=none&width=570)
