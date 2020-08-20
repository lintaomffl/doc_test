# MySQL系统架构

主要参考文章:[https://open.atatech.org/articles/174197](https://open.atatech.org/articles/174197)<br />MySQL的架构也分为In-Memory 和On-Disk两部分。本文主要总结了前者的特征。<br />![b427b303a057900c6364b9e74e3594ce7481a65e.jpeg](https://intranetproxy.alipay.com/skylark/lark/0/2020/jpeg/304852/1594016620729-30e7dd87-2aba-49b0-86c3-1b2d1bce3852.jpeg#align=left&display=inline&height=570&margin=%5Bobject%20Object%5D&name=b427b303a057900c6364b9e74e3594ce7481a65e.jpeg&originHeight=570&originWidth=738&size=137180&status=done&style=none&width=738)
<a name="1"></a>
## Buffer Pool(以下简称bp）
占整个内存系统的80%左右的空间，以数据页的形式缓存数据。<br />整个bp就是一个大list，可以分为如下图所示的两个部分sublist，一端为热端（被频繁访问的部分），一端为冷端（随时要被驱逐出bp的部分）。所以每次插入数据也会在5/8处插入新的数据。<br />![6d1261c9251b317a08a701ff7c23eb72d3dc184c.jpeg](https://intranetproxy.alipay.com/skylark/lark/0/2020/jpeg/304852/1594017165441-58c1c04e-79b0-453a-8239-71fd0a9beab2.jpeg#align=left&display=inline&height=634&margin=%5Bobject%20Object%5D&name=6d1261c9251b317a08a701ff7c23eb72d3dc184c.jpeg&originHeight=634&originWidth=624&size=59014&status=done&style=none&width=624)
<a name="2"></a>
## Change Buffer（以下简称cb）
cb是bp中的一部分，用来缓存不在bp中存储的二级索引变化的page，是DML的结果，定期合并、读到bp中。<br />![5b941b3de44db13e0b2d00f25509d267a8c4b639.jpeg](https://intranetproxy.alipay.com/skylark/lark/0/2020/jpeg/304852/1594018157415-4ec61823-2def-4652-a08b-e56903cee1e6.jpeg#align=left&display=inline&height=673&margin=%5Bobject%20Object%5D&name=5b941b3de44db13e0b2d00f25509d267a8c4b639.jpeg&originHeight=673&originWidth=1127&size=97068&status=done&style=none&width=1127)
<a name="3"></a>
## Adaptive Hash Index
mysql本身不支持哈希索引，但内部有使用到自适哈希索引，维护hash表。可以通过 innodb_adaptive_hash_index 来启用，也可以动态修改。<br />hash index通过B-TREE索引键前缀创建，基于需求和最经常访问的搜索pattern。mysql基于自己的算法来自行创建哈希索引，并且hash table是分区的，通过控制innodb_adaptive_hash_index_parts来设置分区数量，在特定情况下可以启动加速查询的效果，对like和%不友好
<a name="4"></a>
## Redo Log Buffer
redo log也叫WAL，用来缓存数据变化，当脏数据写入文件时，先写入redo log buffer，在写入redo log中，一个目的是用来保证数据一致性，保证数据库crash以后能够通过redo log进行恢复。第二个主要用来降低IO，如果每次数据库的DML都直接写入到数据文件中，则会大大影响性能。
