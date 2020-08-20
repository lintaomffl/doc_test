# Tair学习笔记

主要参考：[https://open.atatech.org/articles/144751#41](https://open.atatech.org/articles/144751#41)<br />
<br />Tair是一个类似于map的key/value结构存储系统（也就是缓存系统），具备标准的特性是：高性能、高扩展、高可靠，也就是传说中的三高产品，支持分布式集群部署。官网说目前支持java和c这两个版本。<br />高性能——基于高速缓存、内存或者ssd<br />高扩展——轻量中间件+三种数据引擎+负载均衡<br />高可用——各种容灾部署方式和解决方案<br />主要功能：<br />**数据库缓存**——作为数据库与dao层之间的中间缓存，降低对后端数据库的访问压力，高速缓存能使得访问速度达到1ms级别，例如高频率的数据库查询；<br />**临时数据存储**——应用程序需要维护大量临时数据，将临时数据存储在mdb中，可以降低内存管理的开销，改进应用程序工作负载。例如：在分布式系统中，同一个用户的不同请求可能会发送到不同的服务器上，这时可以用mdb作为全局存储，用于保存Session数据、用户的Token、权限信息等数据。【通常将缓存和临时数据存储统称为“非持久化存储”】<br />**持久化存储**——此时类似于传统的数据库，将数据存入磁盘中做持久化存储，例如广告推荐类需要离线计算大量数据以及榜单的生成（注意：由于此时采用的数据库引擎ldb是NoSQL类型的，所以不支持sql查询）
<a name="42"></a>
### Tair架构
<a name="43"></a>
#### 物理架构
Tair是Master/Slave结构。Config Server管理Data Server节点、维护Data Server的状态信息；Data Server负责数据存储，按照Config Server的指示完成数据复制和迁移工作，并定时给Config Server发送心跳信息。Config Server是单点，采用一主一备的方式保证可靠性。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/10039/1562381381535-d468529d-0cdc-4b38-a379-77c323fc47c8.png#align=left&display=inline&height=470&margin=%5Bobject%20Object%5D&name=image.png&originHeight=470&originWidth=832&size=204193&status=done&style=none&width=832)
<a name="44"></a>
#### 逻辑架构
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2019/png/10039/1562381606207-b295cc06-8a43-48b9-8478-f9169bba5eed.png#align=left&display=inline&height=1064&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1064&originWidth=1917&size=996497&status=done&style=none&width=1917)<br />三种存储模式的对比：

| **名称** | **基于数据库** | **特点** | **适用场景** |
| --- | --- | --- | --- |
| MDB | Memcached | 基于内存；仅支持k-v类型数据；不支持持久化 | 数据小而简单，读多（QPS 万以上）写少，且偶尔数据丢失不应该对业务产生较大影响；例如访问量显示，session manager等功能 |
| RDB | Redis | 基于内存；除了kv，还支持string, list, hash, set, sortedset等数据类型；<br />支持一定程度的持久化 | 数据形式复杂，偶尔数据丢失不应该对业务产生较大影响；例如排行榜、最新项目检索、地理位置存储及range查询。 |
| LDB | LevelDB | 基于ssd硬盘；仅支持k-v类型数据；支持持久化 | 数据简单，有持久化需求，且读写QPS较高（万级别）但存储数据较简单的应用场景；例如订单计数，库存记录等功能，更新非常频繁。 |


<br />Tair<br />采用乐观锁机制，来保证写入数据时候的数据一致性；<br />利用一致性hash实现负载均衡；<br />利用多级缓存结构来解决单点热点数据访问问题。<br />

