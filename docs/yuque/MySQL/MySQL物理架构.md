# MySQL物理架构

主要参考文章：[https://open.atatech.org/articles/174662](https://open.atatech.org/articles/174662)<br />

<a name="0"></a>
# 表
不同于其他RDB，MySQL独有的存储引擎，与服务端分离。不同业务场景下可以使用不同的存储引擎。<br />其中，MySQL用到的主要引擎为Innodb，Innodb不管是表还是index，都是根据主键按B-tree索引排列的，在Mysql中primary key和cluster index可以认为是一个东西，cluster index叶子节点数据结构包含了所有的列和数值（其实就是表数据），如果是secondary index，则只包含索引列和主键列。<br />变长的列（大字段 VARCHAR, VARBINARY, and BLOB and TEXT types）对于index来说是个例外，变长的列对于index page来说太长，在index page中只会存一部分数据（一般为768 bytes），剩余的存储在overflow page（溢出页）中，每个溢出页也有一个自己的list。这样的列也叫做off-page columns。其实不只是大字段会导致这个情况，只要值大小超过768 bytes，那么就会导致这个情况发生。<br />**数据库IO最小单位为page，默认为16k，文件扩展的单元是extent 默认为1M。再往上是表空间和schema。**

<a name="1"></a>
# 索引
MySQL innodb是强烈依赖于主键（非空、唯一、少更新的列）的数据库，主要是两周索引cluster index和secondary index。<br />**Cluster Index，**如果表上没有定义PK，那么innodb会自动定位非空的unique索引作为cluster 索引。如果没有PK也没有非空唯一索引，则innodb内部自己生成一个隐藏的cluster index叫做GEN_CLUST_INDEX，在一个自动生成的列上，里面存的是ROW ID。表里面行都是根据row id进行排序，row id是一个6byte的单调递增值，在物理存储上连续。 因为当通过cluster index访问一行的时候，索引直接指向了含有行数据的页，精准获取到需要的数据，IO也是最少的。<br />**Secondary Index**，则只包含索引列和主键列。通过主键再去查找cluster index。

当新的row插入到索引页中时，innodb会试图保留1/16的空间给将来的插入和更新操作，如果是顺序插入，则索引页最大为15/16满，如果是随机插入到索引页，那么索引页可能是1/2到15/16满。

**Sorted Index Build 分3个步骤：**<br />1、扫描cluster index，放入到sort buffer中，如果sort buffer满了，会写入到一个临时中间文件。<br />2、合并过个临时中间文件到一个文件中。<br />3、插入到B-tree中。<br />在引入sorted index build以前，索引条目通过insert API一次只能插入一条，sorted index build大大缩短时间和资源cost。在执行sorted index build的时候，也会执行一次checkpoin刷脏。

<a name="3"></a>
# 表空间（tablespace）
<a name="4"></a>
## system tablespace
system tablespace是用来存储change buffer（上一篇中有提到）和创建中系统表空间中的表和索引。
<a name="5"></a>
## File-Per-Table Tablespaces
该表空间通过 innodb_file_per_table来开启。开启以后innodb创建的表都会创建一个新的表空间，并且物理上是一个新的文件，文件后缀为.ibd。关闭该参数以后，innodb创建的表会在system 表空间。
<a name="6"></a>
## general 表空间
顾名思义，general表空间就是用create tablespace创建出来的表空间，给不同表共享。<br />比File-Per-Table有个潜在的内存使用上的优势，多表少表空间的情况下，表空间元数据使用的内存会比File-Per-Table少。
<a name="7"></a>
## undo 表空间
undo表空间包含了undo log。
<a name="8"></a>
## temporary表空间
临时表空间用来存储session级别用户常见的临时表和优化器创建的内部临时表。<br />会话级别的临时表全部是on-disk的，当会话disconnect，临时表也会被truncate和释放，归还给pool。<br />当服务启动时，会创建包含10个临时表空间的pool，pool中数量只会增加不会减少。后缀为.ibt。<br />可以通过innodb_temp_tablespaces_dir制定创建的目录。
<a name="10"></a>
# redo log
在普通的DML操作中，reg log记录数据的变化和LSN(log sequence number)，在数据库crash的时候用来恢复，也是实现replication的重要组件。<br />在会话执行修改事务的时候，不管事务有没有提交，首先会把修改的数据记入到redo log buffer，然后redo log buffer有专门的线程按照特定的条件（比事务提交）写入到redo log中，并且会把同一时间提交的不同事务一同刷新到redo log（组提交）。
<a name="11"></a>
# undo log
undo log是用来记录一个事务（未提交）中最近改变的撤销日志。
