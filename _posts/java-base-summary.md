---
title: java基础知识总结
date: 2017-05-03 19:02:19
tags:
---

### java基础

#### 8种基本类型
|基本类型|大小|封装类|
|:---:|:--:|:--:|
|boolean|1位|Boolean|
|byte|8位=1Byte=1字节|Byte|
|char|16位=2字节|Character|
|short|16位=2字节|Short|
|int|32位=4字节|Integer|
|long|64位=8字节   表达时L做后缀|Long|
|float|32位=4字节   表达时F做后缀|Float|
|double	|64位=8字节|Double|
<!-- more -->
- 基本类型都有默认值,比如0,false等等,但是封装类的默认值为null,所以封装类自动拆箱要警惕空指针异常
- jdk1.5后,提供了自动装箱(基本类型->封装类)/拆箱(封装类->基本类型)
    - 自动装箱(valueOf方法)
        - Integer i = 100 -> Integer i = Integer.ValueOf(100)
    - 自动拆箱(xxxValue方法)
        - i++ -> i.intValue()++
    - Integer的自动装箱,在-128到127之间,返回的缓存的Integer对象.不在这个范围内的是一个新的newInteger对象
        ```
        public static Integer valueOf(int i) {
            if(i >= -128 && i <= IntegerCache.high)　　
                return IntegerCache.cache[i + 128];
            else
                return new Integer(i);
        }
        ```

#### equals,hashcode,==
- ==比较的是两个对象的引用(内存地址)是否相同,也可以比较两个基本类型值是不是相等
- equals()比较的是两个对象的值是不是相等.可以重写equals方法来自定义.默认就是用==比较,看地址是否相等.String重写这个方法,比较内容
- hashcode()方法的值为对象在内存的存储地址.native方法.集合中先通过比较hashcode,在调用equals来判断两个元素是否相等.hash的查找快O(1)

#### Object类中的方法
1. hashCode(): 对象在内存的存储地址
2. equals(): 调用==
3. clone(): 返回clone的新对象
4. toString(): 返回类名+@+hashcode的16进制数
5. notify, notifyAll(): 唤醒等待这个对象锁的线程
6. wait(): 使当前拥有这个对象锁的线程等待

#### try-catch的执行顺序和return
- finally块中的内容会先于try中的return语句执行，如果finall语句块中也有return语句的话，那么直接从finally中返回了.如果没有return语句的话,finally块中无法对try中return的值进行改变

#### 接口与抽象类区别
- 接口中没有任何实现,抽象类中可以有部分实现
- 可以实现多个接口,但是只能继承一个抽象类

#### 子类,父类的构造方法,静态块的执行顺序
1. 先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。
2. 执行子类的静态代码块和静态变量初始化。
3. 执行父类的实例变量初始化
4. 执行父类的初始化块
5. 执行父类的构造函数
6. 执行子类的实例变量初始化
7. 执行子类的初始化块
8. 执行子类的构造函数

#### 异常相关的分类 
```
---Throwable
  |----- Error
  |----- Exception
            |--------RuntimeException(ClassNotFoundException, NullPointerException)
            |--------IOException(FileNotFoundException)
            |--------.....Exception
```
- Throwable: 所有异常的接口
- Error: JVM严重错误,不可捕捉,无法恢复,会导致直接退出程序
- Exception: 由于程序设计不合理或者当前环境问题引发的异常.可能可以恢复,可以捕捉
- checked Exception: 在编译期强制检查的异常,比如IOException等,要求对这类异常强制处理,throws或者catch.客户端可以从这些错误中恢复
- unchecked Exception: 可以不必捕获或者处理的异常.都是RunTimeException的子类或者Error类.都是程序代码可以避免.客户端无法从这些错误中恢复
- RuntimeException: 在程序运行时可能会出现的异常

#### overload overrite
- overload: 重载.指一个类中的方法与另一个方法同名，但是参数表不同
- overrite: 重写,覆盖.指子类覆盖父类的同名方法,提供自己的实现

#### jdk不同版本区别
- jdk1.5: 泛型, for each遍历, 自动装包拆包, 枚举类, 可变参数, JUC
- jdk1.7: switch可以用字符串, try with resource
- jdk1.8: lambda表达式, optional接口, stream对集合类拓展

#### BIO
- 传统IO模型,同步阻塞(Blocking IO)
- 基于字节流(inputstream/outputstream)和字符流(Reader/Writer,字节流+编码)
- 面向流
- socket.accept(),read(),write()三个方法是同步阻塞的.因为不知道什么时候可以读,可以写,只能傻等.所以当单线程处理一个连接时,io会阻塞当前线程
- 基于上面,可以使用多线程,每一个连接对应一个线程来处理
- 使用线程池的问题是:在连接数大的时候,会同时维护很多线程,线程的创建,销毁,切换成本很高,且占用大量内存

#### 字节流
- :outputstream,inputstream 是二进制流,字节流更加底层

#### 字符流
- write,read 一个字符两个字节,是根据字节流加编码转换过来

#### NIO
- 同步非阻塞IO模型,支持io多路复用,可以解决高并发大量链接问题
- 面向缓冲buffer,数据块
- 同步: 当数据准备好以后,还是要主动同步进行io读写.而在真正的异步io中,用户线程不会进行数据的拷贝读写
- 非阻塞: 在获取数据的时候,向selector注册channel后,不会一直阻塞等到数据来,而是数据来的时候主动通知,这样就是非阻塞
- 单线程处理多个连接
- 存在问题:连接数少的时候,并发程度不高时性能反而低.另外操作系统io实现有差异(通过netty,mina解决)
- zero-copy.内存映射.直接将文件从硬盘映射到内存,减少一次从硬盘到内核io的操作

#### 常见io模型
io操作的两个阶段:1.等待数据,等待系统网卡可以读写|2.真正读写(数据从内核复制到用户空间)
- 阻塞IO和非阻塞IO的区别在于第一步，发起IO请求是否会被阻塞，如果阻塞直到完成那么就是传统的阻塞IO，如果不阻塞，那么就是非阻塞IO。
- 同步IO和异步IO的区别就在于第二个步骤是否阻塞，如果实际的IO读写阻塞请求进程，那么就是同步IO，因此阻塞IO、非阻塞IO、IO复用、信号驱动IO都是同步IO，如果不阻塞，而是操作系统做完IO两个阶段的操作再将结果返回，那么就是异步IO。
- 阻塞io:从第一阶段就一直阻塞,等待数据到来完成读写.关心我要开始读了
- 非阻塞io:第一阶段不断检查轮询,检查到可以读写时,完成读写.关心我可以读了
- io复用:进程受阻于select调用,等待多个套接字的一个变成可读.1个进程可以监视多个描述符
- 信号驱动io:信号通知发起可以读写了,开始第二阶段.没有第一阶段
- 异步io:第一阶段和第二阶段都是非阻塞异步的,只关心读完了的通知

#### select,poll,epoll
linux支持io多路复用的系统内核调用有三种:select,poll,epoll.都是先锁住block,再将block拷贝到用户态
- select:基于数组遍历去实现,有最大连接数限制(因为数组大小有限制,2048)
- poll:基于链表遍历去实现,没有最大限制
- epoll:通过哈希表实现,事件通知方式回调

#### 两种io多路复用模式: Reactor模式和 Proactor模式
- 标准的经典的 Reactor模式:可以开始读写了
    1. 等待事件 (Reactor 的工作)
    2. 发”已经可读”事件发给事先注册的事件处理者或者回调 ( Reactor 要做的)
    3. 读数据 (用户代码要做的)
    4. 处理数据 (用户代码要做的)
- Proactor模式:已经完成读写了
    1. 等待事件 (Proactor 的工作)
    2. 读数据(看，这里变成成了让 Proactor 做这个事情)
    3. 把数据已经准备好的消息给用户处理函数，即事件处理者(Proactor 要做的)
    4. 处理数据 (用户代码要做的)

#### NIO组成(Reactor模式)
1. channel:读写数据的通道,双向.可以往buffer写数据,也可以从buffer读数据
2. buffer:存放数据的地方
3. selector:单线程处理多个channel

#### String,Stringbuilder,Stringbuffer
- String: 字符串常量,不可变
- StringBuilder: 字符串变量,可变(append时不会创建新的字符串),不支持并发,单线程用,速度快    
- StringBuffer: 字符串变量,可变,支持并发(方法用sync锁修饰),可以在多线程使用,速度慢
