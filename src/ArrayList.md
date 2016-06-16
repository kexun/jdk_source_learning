#ArrayList

ArrayList是List的实现类，可以说是最重用的一个容器之一。他之所以被频繁的使用，必然有其优势之处。下面就来讲讲ArrayList的几个优点：

#####一、 动态扩容
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
以上就是ArrayList实现动态扩容的原理。那么我有一个问题，当容器满了以后需要扩容，那当容器元素不足1/2或者更少的时候是否需要动态减容呢？

#####验证：
下面写了若干测试代码，分别给出了3中情况，创建的时候设定容器大小和使用默认大小，然后通过逐个增加元素，观察数组大小变化。
```
List<Integer> list1 = new ArrayList<Integer>(1);
    list1.add(1);
    list1.add(2);

    List<Integer> list2 = new ArrayList<Integer>();
    list2.add(1);

    List<Integer> list3 = new ArrayList<Integer>(11);
    for (int i = 0; i < 11; i++) {
        list3.add(i);
    }
    list3.add(22);
    
    for (int i = 11; i > 0; i--) {
        list3.remove(i);
    }
```
1.第一种情况，默认创建一个大小为1的容器，数组大小就是1。  
![](/img/2.png)  
![](/img/3.png)  

2.第二种情况，创建一个默认大小的容器，则数组大小为10。  
![](/img/4.png)  

3.第三种情况，创建一个大小为11的容器，则数组大小为11。  
![](/img/5.png)  

4.第三种情况下，向容器中添加元素，当添加到第12个的时候，数组动态扩容到16，正好符合之前描述的，扩容1.5倍的说法。  
![](/img/6.png)  

5.第三种情况下，删除容器中的元素，数组大小并不会因此而减小。  
![](/img/7.png)![]  
(/img/8.png)  

#####二、添加元素
其实ArrayList的add，set方法都非常简单。一句话概括，就是对数组元素的操作。  

1.add方法：  
```
public boolean add(E e) {
	// 检查扩容
    ensureCapacityInternal(size + 1);
    elementData[size++] = e;
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1);
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}

public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);

    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```
这里给出了四种add方法，add(E e)添加到数组末尾，add(int index, E element)添加到指定位置。添加元素的时候，首先都要检查扩容，而add(int index, E element)方法中多一步操作，就是将指定位置以后的所有元素向后移动一位，留出当前位置用来存放添加的元素。后面两种addAll方法原理和前两种一样，无非他是添加一个集合元素的区别。  

2.set方法：  
```
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```
set和add的区别就是，add是添加一个元素，而set是替换元素，size不变。

#####三、删除元素
remove方法和add方法类似，也是对数组的一系列组合操作。删除也分为对单个元素的删除和集合删除。下面就分别来看看这两类方法的具体实现。  

1.remove单个元素：  
```
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
    	// 直接进行数组拷贝操作，把index后的所有元素向前移动一位。
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // 把元素设空，等待垃圾回收

    return oldValue;
}

public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

// 之所以叫做快速删除，是因为他被设置为一个私有方法，只能在内部调用，删除元素的时候，省去了数组越界的判断。也不返回被删除的元素，直接进行数组拷贝操作。
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null;
}
```
2.删除集合元素  
removeAll和remove方法思想也是类似的，但是这里有个细节我认为作者处理的非常妙，有必要拿出来品味一下。那么妙在哪里呢？原来这里有两个方法removeAll和retainAll他们正好是互斥的两个操作，但是底层都调用了同一个方法来实现，请看！  
```
// 删除包含集合C的元素
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}

// 除了包含集合C的元素外，一律被删除。也就是说，最后只剩下c中的元素。
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
        	// 我认为这里有两点值得我们学习
            // 第一，作者巧妙的提取了逻辑上的最大公约数，仅通过一行逻辑判断就实现了两个互斥的效果。
            // 第二，作者的所用操作都集中于elementData一个数组上，避免了资源的浪费。
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // 理论上r==size 只有当出现异常情况的时候，才会出现r!=size，一旦出现了异常，
        // 那么务必要将之前被修改过的数组再还原回来。
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // 被删除部分数组后，剩余的所有元素被移到了0-w之间的位置，w位置以后的元素都被置空回收。
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```
==当我读到这段代码的时候，我忍不住赞叹，代码之美，美在逻辑的严谨，美在逻辑的简约，也终于明白了，何为对称美。==










