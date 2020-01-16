## 1.ArrayList

##### 	1.简介

​	-- 是一个数组队列，特点是容量能够**动态增加**

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
/**
1.继承AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
2.实现了RandmoAccess接口，说明具有快速访问能力（注意，RandmoAccess只是一个标记，是标志这个类是有快速访问能力，但是实际上快速访问的能力，还是由类自己去实现的）
3.实现了Cloneable接口，即覆盖了函数clone()，能被克隆;
4.实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输
**/
{	
	//默认容量位10
    private static final int DEFAULT_CAPACITY = 10;
    //一个空数组,如果传入的容量为0时使用
    private static final Object[] EMPTY_ELEMENTDATA = {};
    //一个空数组，传传入容量时使用，添加第一个元素的时候会重新初始为默认容量大小
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //当前数据对象存放地方，当前对象不参与序列化;即存储元素的数组
    transient Object[] elementData; // non-private to simplify nested class access
    //当前数据长度
    private int size;
    //最大长度
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}

```

- DEFAULT_CAPACITY

  -- 默认容量为10，通过new ArrayList()创建时的默认容量

  

- EMPTY_ELEMENTDATA

  --   空的数组，这种是通过new ArrayList(0)创建时用的是这个空数组

  

- DEFAULTCAPACITY_EMPTY_ELEMENTDATA

  --   空的数组，只不过这个是用new ArrayList()创建时用的空数组，与上面的区别在于在添加第一个元素时使用这个空数组的会初始化为DEFAULT_CAPACITY（10）个元素



- elementData

  --   真正存放元素的地方，使用transient是为了不序列化这个字段

- size

  --    真正存储元素的个数，而不是elementData数组的长度

  

  

##### 2.构造函数

```java
//无参构造函数
public ArrayList() {
    //DEFAULTCAPACITY_EMPTY_ELEMENTDATA：这个就是上面我们所说的默认数组
    //无参构造函数，就是直接使用空数据DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
/**
 使用这个数组是在添加第一个元素的时候会扩容到默认大小10，比如下面的例子
 List<String> a = new ArrayList<>();// size=0, elementData.length=0
 a.add("1");//size=1 elementData.length=10
*/

//=========================================================================
//有参 构造函数 初始容量
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;//new arrayList(0)，默认给空数组+
        
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//有参 构造函数 集合
public ArrayList(Collection<? extends E> c) {
    //集合转数组，然后将数组的地址的赋给elementData
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            //如果elementData不是数组类型，则进行深copy
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        //如果为空就默认给将空数组的地址给elementData
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}

```

##### 3.主要方法

###### 1.add(E e)方法

```java
//时间复杂度为O(1)
public boolean add(E e) {
    //检查是否需要扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将元素放在数组的最后面
    //因为++运算符的特点 先使用后运算  这里实际上是
    //elementData[size] = e
    //size+1
    elementData[size++] = e;
    return true;
}
//==========================================================================================
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {//如果时默认的空数组
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);//取传入数组跟默认大小之间的最大
    }
    ensureExplicitCapacity(minCapacity);
}
```

###### 2.ensureExplicitCapacity(int minCapacity ) 扩容方法

```java
private void ensureExplicitCapacity(int minCapacity) {
    //代表修改次数
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)//如果需要的次数大于数组容量，则进行扩容
        grow(minCapacity);
}
//================================================================
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;//旧数组大小
    int newCapacity = oldCapacity + (oldCapacity >> 1);//新的容量=老容量+老容量/2
    if (newCapacity - minCapacity < 0)//如果新容量比老容量大
        newCapacity = minCapacity;//直接赋值新容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)//如果新容量比最大允许容量大
        newCapacity = hugeCapacity(minCapacity);// 则调用hugeCapacity方法设置一个合适的容量
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

//===============================================================
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();//越界
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE ://这两数值差8
    MAX_ARRAY_SIZE;
}
```

###### 3.Arrays.copyOf(elementData, newCapacity)

```java
public static <T> T[] copyOf(T[] original, int newLength) {
    return (T[]) copyOf(original, newLength, original.getClass());
}
//===============================================
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)//判断是不是数组类型，通过判断内存地址
        ? (T[]) new Object[newLength]//如果是就创建一个新的数组
        //(newType.getComponentType()：本地方法，如果不是数组，返回null
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    //https://blog.csdn.net/balsamspear/article/details/85069207  system.arrayCOpy
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
//====================================================================
public static Object newInstance(Class<?> componentType, int length)
    throws NegativeArraySizeException {
    //创建一个数组，类型跟newType一样，长度跟newlength是一样的
    return newArray(componentType, length);
}

//================================================================
/**
从源数组src取元素，范围为下标srcPos到srcPos+length-1，取出共length个元素，存放到目标数组中，存放位置为下	标destPos到destPos+length-1

1.src:源数组
2.srcPos:在源数组中，开始复制的位置
3.dest：目标数组
4.destPos 在目标数组中，开始赋值的位置
5.length 被复制的数组元素的数量

当数组为一维数组，且元素为基本类型或String类型时，属于深复制，即原数组与新数组的元素不会相互影响
数组为多维数组，或一维数组中的元素为引用类型时，属于浅复制，原数组与新数组的元素引用指向同一个对象
**/
public static native void arraycopy(Object src,  int srcPos, Object dest, int destPos, int length);


```

###### 4.add(int index, E element)

```java
//时间复杂度O(n)
public void add(int index, E element) {
    //判断是否越界，<0或者大于size  
    rangeCheckForAdd(index);
   //判断是否扩容 
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将inex及其之后的元素往后挪一位，则index位置处就空出来了
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}
//===================================================================================
//判断是否出现下标是否越界
private void rangeCheckForAdd(int index) {
  //如果下标超过了集合的尺寸 或者 小于0就是越界  
  if (index > size || index < 0)
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
//如果越上界抛出IndexOutOfBoundsException异常，如果越下界抛出的是ArrayIndexOutOfBoundsException异常//
```

###### 5.addAll(Collection<? extends E> c)

```java
public boolean addAll(Collection<? extends E> c) {
    //也是调用了copy方法
    Object[] a = c.toArray();//将其转换为数组类型
    int numNew = a.length;//插入数据的类型
    //判断是否需要扩容
    ensureCapacityInternal(size + numNew);  // Increments modCount
    //在原数组的末尾插入数组
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
   // 如果c不为空就返回true，否则返回false
    return numNew != 0;
}
```

###### 6.get(int index)

```java
//时间复杂度O(1)
public E get(int index) {
    //查看是否越界
    rangeCheck(index);
    return elementData(index);
}
//==================================================
E elementData(int index) {
    return (E) elementData[index];
}
```

###### 7.remove(int index) 

```java
public E remove(int index) {
    //判断是否越界
    rangeCheck(index);
    //更改次数+1
    modCount++;
    //取出旧值
    E oldValue = elementData(index);
    //判断是不是最后一个元素，总共5个元素，插入remove(4),5-4-1=0,说明移除的时最后一个元素
    int numMoved = size - index - 1;
    if (numMoved > 0)//如果不是最后一个元素，则需要将后面的元素移到前面一位
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //--size
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

##### 4.总结

```java
/**
	1.底层数组实现，使用默认构造方法初始化出来的容量是10
	2.扩容默认是1+0.5，除了初始扩容
	3.实现了RandomAccess接口，底层又是数组，所以查找方便
	4.线程不安全
	5.因为add跟remove,实质都是要创建一个新的数组，然后将旧的数组复制到新的数组中；所以插入跟删除性能都很差
	6.在插入跟remove的时候的复制，基本类型是深copy,其他类型是浅copy
**/
```

###### 1.面试问题

1.为什么ArrayList的elementData是用transient修饰的？

```java
/**
1.用transient修饰，代表着这个字段不被序列化，是为了节约时间和空间.
		因为elementData不是所有的元素都有值，打个比方，一个List扩容了比如原来是16，现在是24，那么现在是只有17个元素有值，7个元素是没有值的(size=17,length=24)；


但是 是不是elementData就不能序列化？并不是，ArrayList重写了writeObject 保证序列化的时候虽然不序列化全部 但是元素有值的都序列化
**/
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);//序列化
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}


```

2.ArrayList和Vector的区别

```java
/**
1.arrayList是线程不安全的，Vector是线程安全的
2.arrayList扩容每次是1+0.5，而Vector是1+1
**/


//对比
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

//============================================总结
/**
就代码实现上来说，和ArrayList并没有很多逻辑上的区别，但是在Vector的关键方法都使用了synchronized修饰
**/

```

3.什么情况下你会使用ArrayList？什么时候你会选择LinkedList？

```java
/**
1.频繁查询的时候用arrayList
因为Array是基于索引(index)的数据结构，它使用索引在数组中搜索和读取数据是很快的。Array获取数据的时间复杂度是O(1),但是要删除数据却是开销很大的，因为这需要重排数组中的所有数据。
2.频繁插入或者删除的时候采用LinkedList
相对于ArrayList，LinkedList插入是更快的。因为LinkedList不像ArrayList一样，不需要改变数组的大小，也不需要在 数组装满的时候要将所有的数据重新装入一个新的数组，这是ArrayList最坏的一种情况，时间复杂度是O(n)，而LinkedList中插入或删除 的时间复杂度仅为O(1)。ArrayList在插入数据时还需要更新索引（除了插入数组的尾部）。
**/
```

##### 5.补充说明

```java
//关于数组可以参考
http://www.dczou.com/viemall/220.html
```

## 参考

```
https://juejin.im/post/5a58aa62f265da3e4d72a51b
http://cmsblogs.com/?p=4727
```

