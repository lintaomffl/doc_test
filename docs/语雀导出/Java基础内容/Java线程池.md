# Java线程池

<a name="sQoZK"></a>
## 线程池的处理流程
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1594295066419-50016081-ef48-49b3-b33d-df3294fe1d10.png#align=left&display=inline&height=265&margin=%5Bobject%20Object%5D&name=image.png&originHeight=375&originWidth=640&size=58414&status=done&style=none&width=453)<br />提交任务的主要处理流程：<br />(1).提交任务之后，先检查当前核心线程池(coreThreadPool)是否已满，若未满则线程池创建新的线程处理任务；否则交由下一流程；<br />(2).检查队列(block queue)是否已满，如果未满，则将任务加入队列；否则交由下一流程；<br />(3).检查线程池(maximunPoolSize)是否已满，如果未满，则创建线程运行任务；否则交给策略处理。<br />(4).任务走到这步将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。根据不同的拒绝策略去处理。

<a name="Ediqu"></a>
## 线程池的使用
<a name="FSTlK"></a>
### <br />
<a name="5QDjR"></a>
### 线程池的创建
```
public ThreadPoolExecutor(
   int corePoolSize,
   int maximumPoolSize,                              
      long keepAliveTime,
   TimeUnit unit,
   BlockingQueue<Runnable> workQueue,
   ThreadFactory threadFactory,
   RejectedExecutionHandler handler){
}
```
1）corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，如果当前poolSize<corePoolSize时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。<br />2）maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。<br />3）keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。<br />4）TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。<br />5）runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择以下几个阻塞队列。

- ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。<br />
- LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。<br />
- SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于Linked-BlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。<br />
- PriorityBlockingQueue：一个具有优先级的无限阻塞队列。<br />

6）ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线程设置有意义的名字，代码如下。
```
new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
```
7）RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。在JDK 1.5中Java线程池框架提供了以下4种策略。

- AbortPolicy：直接抛出异常。<br />
- CallerRunsPolicy：只用调用者所在线程来运行任务。<br />
- DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。<br />
- DiscardPolicy：不处理，丢弃掉。<br />

当然，也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化存储不能处理的任务。<br />

<a name="Qy9n0"></a>
### 向线程池提交任务


可以使用两个方法向线程池提交任务，分别为execute()和submit()方法。这两个方法的区别就是，**execute()用于提交不需要返回值的任务，submit()方法用于提交需要返回值的任务**。
<a name="eFnJk"></a>
#### execute方法
execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功。通过以下代码可知execute()方法输入的任务是一个Runnable类的实例。
```
threadsPool.execute(new Runnable() {
   @Override
public void run() {
// TODO Auto-generated method stub
}
});
```
<a name="GPTOK"></a>
#### submit方法
线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用get（long timeout，TimeUnit unit）方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。
```
Future<Object> future = executor.submit(haveReturnValuetask);
try {
Object s = future.get();
} catch (InterruptedException e) {
// 处理中断异常
} catch (ExecutionException e) {
// 处理无法执行任务异常
} finally {
// 关闭线程池
executor.shutdown();
}
```


<a name="hYCSA"></a>
### 关闭线程池
shutdown与shutdownNow，都是遍历线程池，对每一个线程进行interrupt。
<a name="i6Lit"></a>
#### shutdownNow
shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
<a name="un2Ji"></a>
#### shutdown
