---
title: java集合总结
date: 2017-03-20 15:07:41
tags:
---



### 集合

#### 集合类图
![集合类图](https://note.wiz.cn/api/document/files/unzip/880d2cf7-790f-4848-acb8-b6d646096ddf/e408f6af-d2c8-4293-92e5-6cec6d49d39c.2087/index_files/ebf43c38-8cb5-4a8b-9d57-d20e4c2c6bca.png)

Iterator: 迭代器接口,定义了hasNext,next,remove三个方法.所有集合类是实现iterator接口
Collection: 集合类根接口,Set和List的父接口,定义了add,remove方法
List: 有序,可以重复的元素集合接口,定义了get,set等方法,实现collection接口
Set: 无序,不能重复的元素集合接口,实现collection接口
Map: 包含key-value对的集合接口,定义了get,put等方法

<!-- more -->

#### List

- ArrayList 原理 实现 扩容 初始大小 是否线程安全 哪里线程不安全了 是否有线程安全的类 怎么实现线程安全的 区别
	- ArrayList是一个基于动态数组实现的列表,容量可以自动增长
	- 位于java.util包下,继承AbstractList类,实现List接口,RandomAccess接口(通过下标快速访问),Cloneable接口(允许被克隆),Serializable(支持序列化传输)
	- ArrayList线程不安全,在多线程环境下可以使用Collections.synchronizedList(通过每个方法加锁实现线程安全)和concurrent.CopyOnWriteArrayList(add时加reentrantlock锁,先复制一个新数组,再添加元素到新数组,再返回新数组)
	1. 当list一开始初始化时,数组长度为0.添加一个元素时,数组长度改为初始大小10.以后扩容为原有长度的1.5倍,若1.5倍还不够,则扩容至需求大小.扩容时使用Arrays.copyof进行复制扩容

	2. ArrayList是线程不安全的，比如add方法：先在size下标处添加元素，再修改size++。当两个线程A、B，A在size处添加元素，还没修改size时，B又添加元素。随后AB都将size+1，则丢失了A添加的元素并且size错误。解决方案：在方法中使用synchronized (mutex) {}加锁
	- fast-fail:非线程安全的类都有可能抛出.在迭代时如果另一个线程在修改list结构,比如删除或者插入数据,就会抛出ConcurrentModificationException异常 快速失败.
		- 原因是因为在迭代时next()或者remove时,会先检测modCount == expectedModCount.如果有别的线程修改了 modcount会++,不等于期望的modcount,就会抛错
		- 解决方案:
		1. 在所有修改modcount的地方加锁,使迭代和插入数据不会同时发生,避免fastfail.比如synchronizedlist.但是这样迭代会很慢
		2. 在迭代的时候先对原数组进行复制,然后在对复制的数组进行迭代遍历.比如使用copyonwritearraylist.任何对array在结构上有所改变的操作（add、remove、clear等），CopyOnWriterArrayList都会copy现有的数据，再在copy的数据上修改，这样就不会影响COWIterator中的数据了，修改完成之后改变原有数据的引用即可。缺点是会产生大量对象并且数组copy需要时间

- CopyOnWriteArrayList
线程安全的arraylist,不会发生fast-fail.
add时加reentrantlock锁,先复制一个新数组,再添加元素到新数组,再返回新数组
缺点:内存占用大,并且只能保证数据最终一致性,因为只是在add时加锁,在get时没有加锁,如果一个线程写另一个线程同时读,只能读到旧数据

- LinkedList
	- LinkedList以双向链表实现,并保留头尾指针firstlast,链表无容量限制.按下标访问元素—get(i)/set(i,e) 访问时要遍历链表将指针移动到位（如果i>数组大小的一半，会从末尾移起）.读慢,但是插入快
	
#### Map

- hashmap 头插法 1.8改变红黑树
1. 位于java.util包下,基于Hash表的map接口实现,根据Hash算法计算keyvalue位置
2. 线程不安全,可以使用SynchronizedMap或者ConcurrentHashMap实现线程安全.HashTable不允许null的key其他和HashMap一样并且线程安全
3. 实现了map接口,clone接口和serializable接口,继承AbstractMap类
4. HashMap的迭代器(Iterator)是fail-fast迭代器
5. 最多只允许一条记录的键为Null，允许多条记录的值为Null，是非同步的
	- 实现原理
	hashmap底层是一个数组table,数组默认大小为16.负载因子是0.75 数组里的元素是Entry类,有指向下一个节点的指针,数组的每一个位置叫做桶bucket,后面挂了一条Entry链.添加节点时如果key=null,则添加到table第一个位置.按key的hashcode值在做一遍hash计算存放在数组的位置,遍历该位置的每一个entry,如果key相同则新值替换旧值,如果key不同,则将新的entry放入头部,原有的元素挂在next上(头插法,因为1.最新插入的元素可能最快读取,节省查找耗时2. 这样插入效率最高,如果插到末尾还要遍历).如果数组容量不够,则扩容2倍再添加
 	jdk1.8时 在同一个bucket位置的链表长度大于8时会将链表转换成红黑树实现
    - 怎么解决hash冲突
    散列值的冲突问题，通常是两种方法：链表法和开放地址法。链表法就是将相同hash值的对象组织成一个链表放在hash值对应的槽位；开放地址法是通过一个探测算法，当某个槽位已经被占据的情况下继续查找下一个可以使用的槽位
	hashmap用的就是链表法
    - 为什么要用hash
    因为好的hash算法查找的速度是O(1)
- hashtable
Hashtable与HashMap类似，是HashMap的线程安全版，它支持线程的同步，即任一时刻只有一个线程能写Hashtable，因此也导致了Hashtale在写入时会比较慢，它继承自Dictionary类，不同的是它不允许记录的键或者值为null，同时效率较低

- concurrenthashmap
线程安全，并且锁分离。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行

- treemap
TreeMap实现SortMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序（自然顺序），也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。不允许key值为空，非同步的

- linkedhashmap
LinkedHashMap保存了记录的插入顺序，在用Iteraor遍历LinkedHashMap时，先得到的记录肯定是先插入的，在遍历的时候会比HashMap慢，有HashMap的全部特性。

#### BlockingQueue阻塞队列
一般用于生产者消费者模式和线程池,当队列为空时,消费者消费会阻塞,当队列满时,生产者生产会阻塞
1. ArrayBlockingQueue 基于数组  FIFO先进先出原则
2. LinkedBlockingQueue：基于链表 FIFO 效率比1高 newFixedThreadPool使用
3. SynchronousQueue：不存储元素的阻塞队列。每个插入操作要等到另一个线程调用移除操作才可以执行 效率比2高 newCachedThreadPool使用
4. PriorityBlockingQueue：具有优先级的无限阻塞队列

|        |   返回异常 |  返回布尔  |    阻塞等待|
|:--------:|:--------:|:---------:|:--------:|
|入队     |   add(e)  |  offer(e) | put(e) |
|出队     |  remove() |   poll()  | take() |
|查看     |  element()|   peek()  |  无     |



