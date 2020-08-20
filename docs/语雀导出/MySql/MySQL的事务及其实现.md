# MySQL的事务及其实现

<a name="0"></a>
# 1.事务特性
原子性：事物的操作，要么全部成功，要么全部失败回滚。<br />一致性：事务都是从一个一致性状态转化到另一个一致性状态。<br />隔离性：一个事务在提交之前，对于其他事务是不可感知的<br />永久性：事务提交了以后，就会被保存在数据库里。
<a name="1"></a>
# 2.事务相关
<a name="kZIj2"></a>
## 2.1并发机制控制
<a name="skkak"></a>
#### 乐观锁
它其实并不是一种真正的『锁』，它会先尝试对资源进行修改，在写回时判断资源是否进行了改变，如果没有发生改变就会写回，否则就会进行重试，在整个的执行过程中其实都没有对数据库进行加锁；
<a name="jaNY0"></a>
#### 悲观锁
悲观锁就是一种真正的锁了，它会在获取资源前对资源进行加锁，确保同一时刻只有有限的线程能够访问该资源，其他想要尝试获取资源的操作都会进入等待状态，直到该线程完成了对资源的操作并且释放了锁后，其他线程才能重新操作资源。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1594035874800-15063f9c-59ac-4507-9635-aeedf1466d22.png#align=left&display=inline&height=295&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1360&originWidth=2400&size=372088&status=done&style=none&width=520)
<a name="3"></a>
## 2.2mysql锁<br />
<a name="4"></a>
### 2.2.1锁的类型
<a name="uWzaC"></a>
#### 共享锁(Shared Lock/S锁)
若事务T对对象A加上S锁以后，其他对象只能读取对象A而不能对其进行修改。除非事务T将对对象A的S锁释放。
<a name="3HbNp"></a>
#### 互斥锁(Exclusive Lock/E锁)
若事务T对数据对象A加上X锁，则只允许T读取和修改A，其它任何事务都不能再对A加任何类型的锁，直到T释放A上的锁。它防止任何其它事务获取资源上的锁，直到在事务的末尾将资源上的原始锁释放为止。在更新操作(INSERT、UPDATE 或 DELETE)过程中始终应用排它锁。排他锁会阻止其它事务再对其锁定的数据加读或写的锁，但是不加锁的就没办法控制了。
<a name="5"></a>
### 2.2.2锁的粒度
InnoDB支持行锁和表锁，同时也支持意向锁。<br />意向锁有以下两个原则：<br />a.在事务获取行上s锁之前，必须先获取表的IS。<br />b.在事务获取行上x锁之前，必须先获取表的IX。<br />意向锁也分：共享意向锁和互斥意向锁。事务在对于某一行进行更新的时候，会先添加意向锁、再添加行意向锁，那么还有表锁要对于该表进行操作的时候，就不需要对于每一行都进行扫描验证行锁是否存在，读取到表意向锁的存在就不可以再对该表使用表锁并操作了。
<a name="6"></a>
### 2.2.3锁的范围

- Record Locks(行锁)<br />加在索引行(对！是索引行！不是数据行！)上的锁。比如select * from user where id=1 and id=10 for update，就会在id=1和id=10的索引行上加Record Lock。<br />
- Gap Locks(间隙锁)<br />会锁住两个索引之间的区域。比如select * from user where id>1 and id<10 for update，就会在id为(1,10)的索引区间上加Gap Lock。<br />
- Next-Key Locks(下一键锁)<br />Record Lock + Gap Lock形成的一个闭区间锁。比如select * from user where id>=1 and id<=10 for update，就会在id为[1,10]的索引闭区间上加Next-Key Lock。Next-Key 锁的作用其实是为了解决幻读的问题。<br />
<a name="7"></a>
### 2.2.4加锁操作
在数据库增删改查四种操作中，

- insert、delete和update都是会加排它锁(Exclusive Locks)的
- select只有显式声明才会加锁:
   - select: 即最常用的查询，是不加任何锁的
   - select ... lock in share mode: 会加共享锁(Shared Locks)
   - select ... for update: 会加排它锁(Exclusive Locks)



<a name="8"></a>
## 3.事务日志
<a name="9"></a>
### 重做日志redo log
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1594036786667-9481c2a6-35f8-4772-9905-f0bc1f4697a5.png#align=left&display=inline&height=430&margin=%5Bobject%20Object%5D&name=image.png&originHeight=860&originWidth=2400&size=347499&status=done&style=none&width=1200)<br />redo log重做日志，主要的流程如上图。当得到数据操作的要求时候，从磁盘读取数据，然后在内存缓存中修改数据。并将操作行为记录进redo log buffer，并同时将数据写到磁盘。4、5两部是同时进行的。
<a name="10"></a>
### 回滚日志undo log
undo log主要是对于事务之前的数据状态进行记录。以便在事务提交失败之后，根据undo log进行ROLLBACK。
<a name="7Hb3S"></a>
### 当前读&快照读
对于读取历史数据的方式，我们叫它快照读 (snapshot read)，而读取数据库当前版本数据的方式，叫当前读 (current read)。
<a name="TFoNJ"></a>
#### 快照读：
就是select    
<a name="vNcIw"></a>
#### 当前读：
特殊的读操作，插入/更新/删除操作，属于当前读，处理的都是当前的数据，需要加锁。<br />

<a name="HlguN"></a>
## 4.事务原理
<a name="jWYgd"></a>
### 4.1隔离性问题
脏读：<br />在一个事务中，读取了其他事务未提交的数据<br />不可重复读：<br />在一个事务中，中间有其他事务提交了修改，第一次读的数据和第二次读的数据不一样<br />幻读：<br />在一事务中，同样的条件,中间有其他事务提交了插入/删除，第一次和第二次读出来的记录数不一样<br />

<a name="l4SrJ"></a>
### 4.2基于锁的隔离实现
| 隔离级别 | 解释 | 存在问题 | 锁实现 |
| :--- | :--- | :--- | :--- |
| RAED UNCOMMITED | 读未提交记录，始终是读最新记录 | 脏读、不可重复读、幻读 | 读的过程不加S锁 |
| READ COMMITED | 读已提交记录 | 不可重复读、幻读 | 读的过程加 S锁，无论事务是否结束，SELECT 语句一旦结束，立马释放S锁，不会等到事务结束才释放锁 |
| REPEATABLE READ | 可重复读记录 | 幻读 | 读的过程加S锁，直到事务结束，才释放S锁 |
| SERIALIZABLE | 序列化读记录。并发度最差，除非明确业务需求及性能影响才使用 | 不存在 | 读的过程添加S锁+范围锁；修改数据的过程中，添加 X 锁+间隙锁 |



<a name="19"></a>
### 4.3基于MVCC的事务隔离
Innodb MVCC主要是为REPEATABLE READ、READ COMMITED事务隔离级别做的。如果使用锁机制来实现这种隔离级别，会极大的降低数据库的并发能力。出于性能考虑，Mysql等数据库使用了以乐观锁为理论基础的MVCC（多版本并发控制）来避免。<br />而其他两种不需要MVCC：

- Read Uncommitted每次都读取记录的最新版本，会出现脏读，未实现MVCC
- Serializable对所有读操作都加锁，读写发生冲突，不会使用MVCC
<a name="20"></a>
#### MVCC实现
InnoDB为数据库中存储的每行数据添加三个字段来实现MVCC：

- 事务版本号（DB_TRX_ID ），记录了数据的最后一次更新（INSERT/UPDATE)的事务ID
- 回滚指针（DB_ROLL_PTR） ，指向当前数据的undo log记录
- 删除位（DELETE BIT），标识该记录是否被删除，物理删除是在mysql进行数据的GC，清理历史版本数据的时候。

在可重读Repeatable reads事务隔离级别下：

- SELECT：
   - 读取创建版本号<=当前事务版本号。确保事务读取的行，要么是在事务开始前已经存在，要么是事务自身插入或者修改过的。
   - 删除版本号为空或>当前事务版本号。确保事务读取到的行，在事务开始之前未被删除。
- INSERT：创建一条新数据，DB_TRX_ID设置为当前事务ID，DB_ROLL_PT为NULL，即没有需要回滚的数据指向
- DELETE：将当前行的DB_TRX_ID设置为当前事务ID，DELETE BIT设置为1
- UPDATE：复制了一行，新行的DB_TRX_ID为当前事务ID，DB_ROLL_PT指向了上一个版本的undo log，事务提交后DB_ROLL_PT置为NULL

通过MVCC，可以减少锁的使用，大多数读操作都不用加锁，读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行，也只锁住必要行。具有以下特性：

- 每一个写操作都会创建一个新版本的数据，读写能够并发进行
- 每个读事务相当于看到数据当前的一个快照
- 读写操作之间的冲突不再需要被关注，只需要管理和快速挑选数据的版本
- serializable snapshot isolation 可串行化快照隔离
<a name="21"></a>
#### MVCC与版本获取
MVCC实现了多个并发事务更新同一行记录会时产生多个记录版本，那么，新开始的事务如果要查询这行记录，应该获取到哪个版本呢？<br />每个事务在第一个SQL开始的时候都会根据当前系统的活跃事务链表创建一个read_view。在每种隔离级别：

- REPEATABLE READ：InnoDb检查每行数据，确保它们符合两个标准：
   - 只查找创建时间早于当前事务ID的记录，这确保当前事务读取的行都是事务之前已经存在的，或者是由当前事务创建或修改的行
   - 行的DELETE BIT为1时，查找删除时间晚于当前事务ID的记录，确定了当前事务开始之前，行没有被删除
- READ COMMITED：每次重新计算read-view，read-view的范围为InnoDb中最大的事务ID，为避免脏读读取的是DB_ROLL_PT指向的记录
