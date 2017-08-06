---
title: ArrayList( jdk8)解析
date: 2016-10-04 10:23:49
tags:
---


### ArrayList( jdk8)解析



- ArrayList是一个基于动态数组实现的列表,容量可以自动增长

- 位于java.util包下,继承AbstractList类,实现List接口,RandomAccess接口(通过下标快速访问),Cloneable接口(允许被克隆),Serializable(支持序列化传输)

- ArrayList线程不安全,在多线程环境下可以使用Collections.synchronizedList和concurrent.CopyOnWriteArrayList

<!-- more -->



##### 属性

```

    /**

     *  默认创建的数组长度为10

     */

    private static final int DEFAULT_CAPACITY = 10;



    /**

     *  初始化长度为0时返回的数组

     */

    private static final Object[] EMPTY_ELEMENTDATA = {};



    /**

     *  初始化时不传长度参数时返回的数组

     */

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};



    /**

     *   实际存放列表元素的数组

     */

    transient Object[] elementData; 



    /**

     *  当前数组存放元素个数

     *

     * @serial

     */

    private int size;

```




##### 构造方法

```

    /**

     *  带长度的构造方法

     *  当长度小于0时报错

     *  长度等于0时，elementData是一个空数组

     *  长度大于0时，elementData初始化一个指定长度的数组

     */

    public ArrayList(int initialCapacity) {

        if (initialCapacity > 0) {

            this.elementData = new Object[initialCapacity];

        } else if (initialCapacity == 0) {

            this.elementData = EMPTY_ELEMENTDATA;

        } else {

            throw new IllegalArgumentException("Illegal Capacity: "+

                                               initialCapacity);

        }

    }



    /**

     * 不带长度的构造方法

     * elementData是一个空数组

     */

    public ArrayList() {

        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;

    }



    /**

     *  将一个Collection转换成一个ArrayList,调用toArray方法

     */

    public ArrayList(Collection<? extends E> c) {

        elementData = c.toArray();

        if ((size = elementData.length) != 0) {

            if (elementData.getClass() != Object[].class)

                elementData = Arrays.copyOf(elementData, size, Object[].class);

        } else {

            this.elementData = EMPTY_ELEMENTDATA;

        }

    }

```

#### add方法

```

    /**

     *  添加一个元素

     *  首先调用ensureCapacityInternal,确保数组长度足够.当数组长度不够时,调用grow方法扩容

     *  再往数组中添加元素,size++

     */

    public boolean add(E e) {

        ensureCapacityInternal(size + 1);  // Increments modCount!!

        elementData[size++] = e;

        return true;

    }



    /**

    *   最小需要数组大小=当前数组大小+插入元素个数

    *   当当前数组不是空数组,或者为空数组并且最小需要数组大小(minCapacity)大于10,则ensureExplicitCapacity参数为最小需要数组大小

    *   当最小需要数组大小小于10时,ensureExplicitCapacity参数为10

    */

    private void ensureCapacityInternal(int minCapacity) {

        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {

            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);

        }



        ensureExplicitCapacity(minCapacity);

    }



    /**

    *   修改次数加1

    *   如果当前数组大小足够,就不需要调用grow.数组大小不够时,则调用grow扩容

    */

    private void ensureExplicitCapacity(int minCapacity) {

        modCount++;



        // overflow-conscious code

        if (minCapacity - elementData.length > 0)

            grow(minCapacity);

    }



    /**

     *   扩容数组长度(newCapacity)为原有数组长度的1.5倍

     *   当扩容数组长度不够最小需要数组大小时,将数组扩容至最小需要数组大小,当扩容数组长度够时使用扩容数组长度

     *   当扩容数组长度大于最大数组长度时,若最小需要数组大小溢出int值时,报OOM错.否则扩容至int最大值

     *   调用Arrays.copyOf进行复制扩容

     */

    private void grow(int minCapacity) {

        // overflow-conscious code

        int oldCapacity = elementData.length;

        int newCapacity = oldCapacity + (oldCapacity >> 1);

        if (newCapacity - minCapacity < 0)

            newCapacity = minCapacity;

        if (newCapacity - MAX_ARRAY_SIZE > 0)

            newCapacity = hugeCapacity(minCapacity);

        // minCapacity is usually close to size, so this is a win:

        elementData = Arrays.copyOf(elementData, newCapacity);

    }



    private static int hugeCapacity(int minCapacity) {

        if (minCapacity < 0) // overflow

            throw new OutOfMemoryError();

        return (minCapacity > MAX_ARRAY_SIZE) ?

            Integer.MAX_VALUE :

            MAX_ARRAY_SIZE;

    }

```



#### remove方法



```

    /**

     *  首先通过rangeCheck方法检查下标index有没有越界

     *  如果移除的是末尾的元素，直接将数组最后一个元素赋null，size减1，不改变大小

     *  如果不是末尾元素，调用System.arraycopy将数组从删除元素至末尾的元素复制一遍到前一位，将最后一个元素赋null，size减1

     *  最后返回删除的那个元素

     */



    public E remove(int index) {

        rangeCheck(index);



        modCount++;

        E oldValue = elementData(index);



        int numMoved = size - index - 1;

        if (numMoved > 0)

            System.arraycopy(elementData, index+1, elementData, index,

                             numMoved);

        elementData[--size] = null; // clear to let GC do its work



        return oldValue;

    }



    /**

     *  检查数组下标是否越界，越界抛IndexOutOfBoundsException异常

     */

    private void rangeCheck(int index) {

        if (index >= size)

            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

    }

```



#### 注意点

1. 当list一开始初始化时,数组长度为0.添加一个元素时,数组长度改为初始大小10.以后扩容为原有长度的1.5倍,若1.5倍还不够,则扩容至需求大小

2. ArrayList是线程不安全的，比如add方法：先在size下标处添加元素，再修改size++。当两个线程A、B，A在size处添加元素，还没修改size时，B又添加元素。随后AB都将size+1，则丢失了A添加的元素并且size错误。解决方案：在方法中使用synchronized (mutex) {}加锁


