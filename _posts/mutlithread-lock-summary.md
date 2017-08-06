---
title: 多线程和锁相关总结
date: 2017-03-15 17:22:19
tags:
---

### 多线程与锁

#### java中使用线程的几种方式
1. 线程池 2. 实现runable接口 3. 继承thread类

#### 线程池
1. newFixedThreadPool new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(),threadFactory); 创建固定大小的线程池,不会timeout,基于链表的阻塞队列 FIFO
2. newSingleThreadExecutor new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>()) 大小为1
3. newCachedThreadPool new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>() 无限大小

<!-- more -->
构造方法:
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,RejectedExecutionHandler handler)

1. corePoolSize：线程池的基本大小。提交一个新任务时，若线程池线程数量小于基本大小，则创建一个线程执行，即使其他空闲的线程可以执行新任务
2. maximumPoolSize：线程池最大大小 线程池最多能创建几个线程
3. keepAliveTime：线程活动时间 线程池的工作线程空闲后，保持存活的时间.
默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，直到线程池中的线程数不大于corePoolSize，即当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；
4. TimeUnit：线程活动保持时间的单位，有天、小时、分钟等
5. workQueue：任务队列 保存等待执行任务的阻塞队列
6. handler：表示当拒绝处理任务时的策略，有以下四种取值：
ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。(默认)
ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

一开始线程池中线程数为0,然后有新任务来就不断创建新的线程处理,直到数量到达core size时.再来新任务就进入阻塞队列.当阻塞队列满时,再创建新的线程处理,直到数量达到max size.此时新任务在来时按照handle策略丢弃任务.线程在while循环里面不断通过getTask()去取新的任务来执行，从任务缓存队列里面去取

1. 用execute(Runnable)方法提交任务,没有返回值,不知道任务是否执行 
2. 用submit(Runnable)提交任务，返回future，通过future的get获得返回值

#### 阻塞队列的实现
1.ArrayBlockingQueue 基于数组  FIFO先进先出原则
2.LinkedBlockingQueue：基于链表 FIFO 效率比1高 newFixedThreadPool使用
3.SynchronousQueue：不存储元素的阻塞队列。每个插入操作要等到另一个线程调用移除操作才可以执行 效率比2高 newCachedThreadPool使用
4.PriorityBlockingQueue：具有优先级的无限阻塞队列

1. 如果阻塞立刻抛出异常方法: add(e),remove(),element()
2. 如果阻塞立刻返回布尔: offer(e),poll(),peek()
3. 如果阻塞则等待:put(e),take()

#### 线程池中的线程异常处理

1. 通过submit方法提交任务时会返回future对象,如果线程抛出异常了,future对象就是这个异常

2. 覆盖ThreadPoolExecutor的afterExecuter方法,每个任务跑完都会触发这个方法,参数中有一个throwable对象t,就是抛出的异常

#### Object.wait,notify,notifyall
1. wait()方法：使当前执行代码的线程等待，直到接到通知或者中断为止。 
调用wait之前，必须获得该对象锁，即必须在同步方法中使用。否则会抛异常。  
执行wait方法后，立刻释放锁（sleep方法不会释放锁）
wait(time) 等待一段时间看是否有线程对其唤醒 ，到了时间自动唤醒
当线程在wait时，调用线程的interrupt方法会出现异常

2. notify方法：使停止的线程继续运行  
与wait方法一样 也必须在同步方法中使用，如果没有持有锁 会抛异常  
通知等待该对象锁的进程。如果有多个线程等待，随机运行一个 。别的继续等待
等执行notify的线程执行完退出synchronized块后，才会释放锁，wait状态的线程才能获得锁

3. notifyall 使所有等待该锁的进程变成可运行状态
优先级最高的线程最先执行或随机执行 取决于jvm

#### 线程间通信,同步,互斥
1. 线程间通信可以通过共享数据
2. 同步可以采用wait,nofity或者condition
3. 互斥可以采用synchronized或者lock

#### 中断线程和取线程结果
1. 用stop停止.不推荐
2. 用interrupt设置标记位,在线程run方法中判断标记位如果为ture,就直接ruturn退出
3. 在线程run方法中抛出异常并trycatch也会停止线程

通过submit提交的callable会返回future对象,通过future.get()方法可以拿到线程结果,这个方法是阻塞方法,会一直等到线程运行结束

####  Thread.sleep,join,yield
1. sleep
暂停当前线程,单位是毫秒,使线程转到timed wait状态。时间结束后转为就绪状态.不会释放锁.native方法
2. join
将这个线程加入当前线程。只有这个线程结束才会继续执行当前线程后面的代码.当前线程为wait状态
join内部使用wait方法等待，所以也会释放锁.synchronized使用this进行同步
3. yield
暂停当前正在执行的线程对象，放弃当前cpu,把执行机会让给相同或者更高优先级的线程。自己返回就绪状态.native方法

#### synchrinized对象锁
- sync对象锁:修饰非static方法,或者synchronized(this),或者synchronized(object) 如果多个线程访问同一个对象中的同步方法 则要等待.如果访问同个对象的非同步方法 也可以直接读
- sync类锁:修饰static方法,或者synchronized(object.class)
如果多个线程访问属于同一个类不同对象的同步方法,都要等待

带有锁的A方法中可以调用带有锁的B方法.可重入锁
出现异常时，锁会自动释放
synchronized方法子类继承后没有同步效果

1. 关键字volatile是线程同步的轻量级实现，性能比synchronized好,并且volatile只能修饰变量
2. 多线程访问volatile不会阻塞 synchronized会阻塞
3. volatile不会保证原子性,但是synchronized可以 

#### ReentrantLock与Condition
ReentrantLock功能比sync更多.通过lock.lock()加锁,需在finally块中lock.unlock解锁,reenttrantLock继承AbstractQueuedSynchrinizer AQS,lock使用cas方式加锁
1. 和synchronized一样也是可重入锁
2. 等待可中断: 当其他线程长时间等待时,可以选择放弃等待锁,处理别的事情.对同步块执行时间长的有帮助.使用lock.lockInterruptibly()加锁而不是lock.lock()
3. 公平锁:多个线程在等待同一个锁时,可以按照申请顺序依次获得锁.synchronized是非公平的.reentrantlock也是默认非公平的.可以参数指定new reentrantlock(true)
4. 绑定多个条件: 一个ReentrantLock对象可以绑定多个condition条件.synchronized不行.lock.newCondition()
5. trylock()方法尝试去获取锁
6. 可重入锁的实现:每个锁都有一个计数器和一个所有者线程,当计数值为0的时候,这个锁就没有被任何线程只有.当线程请求一个未被持有的锁时,JVM将记下锁的持有者,并且将获取计数值置为1,如果同一个线程再次获取这个锁,技术值将递增,退出一次同步代码块,计算值递减,当计数值为0时,这个锁就被释放

Condition中，用await()替换wait()，用signal()替换notify()，用signalAll()替换notifyAll().与object方法区别的是同一个lock锁可以绑定多个condition,更加灵活

#### 读锁,写锁
reentrantreadwritelock 读写锁,和reentrantlock一样也是基于AQS实现
读锁:lock.readlock.lock() lock.readlock.unlock() 
写锁:lock.weritelock.lock() lock.writelock.unlock()
使用场景:(读多写少的情况)多线程涉及到对一些共享资源的读和写操作，且写操作没有读操作那么频繁。在没有写操作的时候，两个线程同时读一个资源没有任何问题，所以应该允许多个线程能在同时读取共享资源。但是如果有一个线程想去写这些共享资源，就不应该再有其它线程对该资源进行读或写
读-读能共存，读-写不能共存，写-写不能共存
同一个线程在获取到读锁后去加写锁会产生死锁,因为加写锁必须没有读锁,否则会等待,当前线程等待的话就永远不会unlock读锁,造成死锁.不允许锁升级
但是允许在写锁后去加读锁,此时相当与锁降级为读锁,必须显示释放写锁

#### ThreadLocal
使每个线程都有一个自己的共享变量，而不是所有线程共享一个变量,不同线程隔离数据
里面有一个threadlocalmap(数组实现),key是线程,value是变量. 
比如simpledataformat 时间转换这个类不是线程安全的 ,但我需要在多线程中使用这个类(format parse)会报错NumberFormatException: multiple points或者时间错误.因为里面用了Calendar这个对象是全局变量,所有线程共享的 set的时候.可能两个线程同时修改就会不安全.可以使用threadlocal<simpledataformat>使每一个线程都有自己的对象,不会影响其他线程 所以没有并发问题

#### 闭锁和栅栏
都是继承AQS,cas实现
闭锁:CountLatchDown. 类似于一扇门：在闭锁到达结束状态之前，这扇门一直是关闭着的，不允许任何线程通过，当到达结束状态时，这扇门会打开并允许所有的线程通过。且当门打开了，就永远保持打开状态。
作用：可以用来确保某些活动直到其他活动都完成后才继续执行.比如某个操作需要的资源初始化完毕后再进行另一操作.玩游戏中需要所有玩家准备后才能开始
内部实现:包含一个计数器，该计算器初始化为一个正数，表示需要等待事件的数量 CountDownLatch latch = new CountDownLatch(3)。latch.countDown()方法递减计数器，表示有一个事件发生，而在其他线程先latch.await()方法等待计数器到达0，所有await的线程都会阻塞,直到所有需要等待的事情都已经完成.然后再写继续的逻辑

栅栏:CyclicBarrier 与闭锁原理相似,内部也有一个计数器,有await方法没有countdown方法,不过是栅栏是等待别的线程完成.闭锁是等待别的事件完成.闭锁await通过别的线程事件调用countdown()方法,直到计数器为0然后不阻塞.栅栏await直到别的线程也都运行到await后,再继续一起运行

#### 乐观锁和悲观锁
1. 悲观锁:只要是共享数据可能会出现竞争就加锁,不管有没有真的发生.比如synchronized
2. 乐观锁: 先进行操作.如果没有其他线程用共享数据操作就成功,否则就采取补偿措施,比如不断重试

#### cas原理
cas:compare and swap 比较和交换
1. VAB:V变量的内存地址. A:旧值  B:新值
2. 当需要对V更新时,取出值A.更新后的值为B.在更新时看现在的值还是不是A.如果是A表示其他线程没有修改,则更新为B值,否则不更新.
3. concurrent包里的AtomInteger类就采用了cas操作.getandincrement.另外AQS类也是使用cas操作,基本上concurrent包中类都使用aqs,比如reentrantlock,reentrantreadwritelock,1.8的concurrenthashmap
4. 漏洞:ABA: 如果一个变量V一开始是A,后来更新操作时还是A,但是他的值不一定没有被更改过,可能另一个线程改成B后又改成A.这时就会误认为没有改变过值.可以使用带标记的类,里面有一个变量记录值变更过的版本来保证

#### 原子类
concurrent包下面的原子类 包括:AtomInteger,AtomicLong等
可以实现i++的原子操作,在多线程环境下保证数据正确
getAndIncrement:加1返回原值
getAndAdd:加n返回原值
使用cas实现

#### 自旋锁
有时候共享数据的锁时间很短,这个时候其他线程需要不断阻塞和唤醒线程,开销很大.所以可以让等待锁的线程不阻塞,还在runing状态.自旋等待一段时间,不放弃cpu.直到拿到锁.自旋的次数默认为10次
对于那些锁竞争不是很激烈，锁占用时间很短的并发线程适用用自旋锁
jdk1.7后,所有的锁都会使用自旋锁

#### 偏向锁
若某一锁被线程获取后，便进入偏向模式，当线程再次请求这个锁时，就无需再进行相关的同步操作了，从而节约了操作时间，如果在此之间有其他的线程进行了锁请求，则锁退出偏向模式。在JVM中使用-XX:+UseBiasedLocking开启偏向锁 
偏向锁适合用在单个线程中操作一个对象的同步方法,在竞争激烈的场合没有优化效果
