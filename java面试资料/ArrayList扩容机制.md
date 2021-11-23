# ArrayList扩容机制

以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即**向数组中添加第一个元素时，数组容量扩为10**。 

```java
/**
 * 默认初始容量大小
/
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 *默认构造函数，使用初始容量10构造一个空列表(无参数构造)
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**

 * 带初始容量参数的构造函数。（用户自己指定容量）
   */
   public ArrayList(int initialCapacity) {
   if (initialCapacity > 0) {//初始容量大于0
       //创建initialCapacity大小的数组
       this.elementData = new Object[initialCapacity];
   } else if (initialCapacity == 0) {//初始容量等于0
       //创建空数组
       this.elementData = EMPTY_ELEMENTDATA;
   } else {//初始容量小于0，抛出异常
       throw new IllegalArgumentException("Illegal Capacity: "+
                                          initialCapacity);
   }
  }

   /**
    *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
    *如果指定的集合为null，throws NullPointerException。
    */
     public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## 1、add 方法

```java
/**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
   //添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }
```

## 2、ensureCapacityInternal() 方法

当 要 add 进第1个元素时，minCapacity为1，在Math.max()方法比较后，minCapacity 为10。

```java
//得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

    ensureExplicitCapacity(minCapacity);
}
```

## 3、ensureExplicitCapacity() 方法

```java
//判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

​    // overflow-conscious code
​    if (minCapacity - elementData.length > 0)
​        //调用grow方法进行扩容，调用此方法代表已经开始扩容了
​        grow(minCapacity);
}
```

添加前10个元素时，容量始终为10；

直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。

## 4、grow() 方法

```java
/**

要分配的最大数组大小
/
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**

ArrayList扩容的核心方法。
*/
  private void grow(int minCapacity) {
  	// oldCapacity为旧容量，newCapacity为新容量
  	int oldCapacity = elementData.length;
 	 //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
  	//我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
  	int newCapacity = oldCapacity + (oldCapacity >> 1);
	//然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
	 if (newCapacity - minCapacity < 0)
 		 newCapacity = minCapacity;
 	// 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
 	//如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 	`Integer.MAX_VALUE - 8`。
  	 if (newCapacity - MAX_ARRAY_SIZE > 0)
  			 newCapacity = hugeCapacity(minCapacity);
  	// minCapacity is usually close to size, so this is a win:
  	 elementData = Arrays.copyOf(elementData, newCapacity);
 }
```

## 

 5、hugeCapacity() 方法

如果新容量大于 MAX_ARRAY_SIZE,进入(执行) hugeCapacity() 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 Integer.MAX_VALUE - 8。

 ```java
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
 ```

## 6、System.arraycopy() 和 Arrays.copyOf()方法

- ### System.arraycopy() 方法

```java
/**
在此列表中的指定位置插入指定的元素。
先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //arraycopy()方法实现数组自己复制自己
    //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

- ### Arrays.copyOf()方法

```java
/**
     以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）; 返回的数组的运行时类型是指定数组的运行时类型。
     */
    public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
        return Arrays.copyOf(elementData, size);
    }
```

## 7、ensureCapacity方法

该方法没有调用过，留给用户使用。可以用来在添加大量元素之前使用，避免频繁的扩容！

```java
/**
    如有必要，增加此 ArrayList 实例的容量，以确保它至少可以容纳由minimum capacity参数指定的元素数。
     *

@param   minCapacity   所需的最小容量
/
    public void ensureCapacity(int minCapacity) {
int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
    // any size if not default element table
    ? 0
    // larger than default for default empty table. It's already
    // supposed to be at default size.
    : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

## 8、通过反射获取ArrrayList容量

```java
public static Integer getCapacity(ArrayList list) {
    Integer length = null;
    Class clazz = list.getClass();
    Field field;
    try {
        field = clazz.getDeclaredField("elementData");
        field.setAccessible(true);
        Object[] object = (Object[]) field.get(list);
        length = object.length;
        return length;
    } catch (Exception e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return length;
}
```