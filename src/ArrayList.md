#ArrayList

ArrayList是List的实现类，可以说是最重用的一个容器之一。他之所以被频繁的使用，必然有其优势之处。下面就来讲讲ArrayList的几个优点：

1. 动态扩容
首先来谈谈ArrayList的数据是如何存储的，他的底层其实就是封装了一个Array数组，数组的类型为Object。  
```
    private static final int DEFAULT_CAPACITY = 10;

    private static final Object[] EMPTY_ELEMENTDATA = {};

    transient Object[] elementData;

    private int size;
```
   从这段定义中可以看出，ArrayList维护了两个数组DEFAULT_CAPACITY和elementData。DEFAULT_CAPACITY是一个空数组，当创建一个空的ArrayList的时候就会使用DEFAULT_CAPACITY，这个时候elementData==DEFAULT_CAPACITY，当在容器中添加一个元素以后，则会使用elementData来存储数据。  
   
   这里值得讨论的是DEFAULT_CAPACITY常量,他代表的含义是一个默认数组大小，当我们创建的容器没用指定容量大小时，就会默认使用这个常量作为数组大小。因此当我们创建一个ArrayList实例的时候，最好考虑一下业务场景，如果我们将频繁的存储大量的元素，那么最好在创建的时候指定一个合理的size。所谓动态扩容，就是当数组中存储的元素达到容量上限以后，ArrayList会创建一个新的数组，新数组的大小为当前数组大小的1.5倍。随后将数组元素拷贝到新数组，如果这个动作频繁执行的话，会增大性能开销。  
```
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }

    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```
这三个方法都是ArrayList的构造方法，从前两个方法中可以看出初始化ArrayList的时候是如何指定容器初始大小的，这里也无需多言了。那么我们再看看，当容量达到上限的时候，是如何动态扩充数组大小的呢。  

```
public boolean add(E e) {
    // 每次添加元素之前先动态调整数组大小，避免溢出
    ensureCapacityInternal(size + 1);
    // 为什么ArrayList的元素都是顺序存放的？这就是原因，每次都会把最新添加的元素放到数组末尾。
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    // 如果当前容器为空，那么就先初始化数组，数组大小不能小于DEFAULT_CAPACITY
    if (elementData == EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 容器会在什么时候扩容？ 就是他了！ 如果当前元素数量达到了容器的上限，那么就扩充数组
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // oldCapacity为当前容器大小
    int oldCapacity = elementData.length;
    // oldCapacity >> 1和oldCapacity / 2是等效的，因此newCapacity为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 因为第一次容器有可能为空，elementData.length==0，newCapacity会小于minCapacity
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 当然newCapacity也不能大于MAX_ARRAY_SIZE，因为数组能分配的最大空间就是Integer.MAX_VALUE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 当确定好数组大小后，就可以进行数组拷贝，Arrays.copyOf的底层是一个native方法，后续有机会会讲到他的实现。
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
以上就是ArrayList实现动态扩容的原理。那么我有一个问题，当容器满了以后需要扩容，那当容器元素不足1/2的时候是否需要动态减容呢？
























