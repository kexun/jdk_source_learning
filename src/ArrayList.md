#ArrayList

ArrayList是List的实现类，可以说是最重用的一个容器之一。他之所以被频繁的使用，必然有其优势之处。下面就来讲讲ArrayList的几个优点：

1. 动态扩容
首先来谈谈ArrayList的数据是如何存储的，他的底层其实就是封装了一个Array数组，数组的类型为Object。  
```
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
     * DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```
   从这段定义可以看出，ArrayList维护了两个数组DEFAULT_CAPACITY和elementData。DEFAULT_CAPACITY是一个空数组，当创建一个空的ArrayList的时候就会使用DEFAULT_CAPACITY，这个时候elementData==DEFAULT_CAPACITY，当在容器中添加一个元素以后，则会使用elementData来存储数据。
   这里值得讨论的是DEFAULT_CAPACITY常量,他代表的含义是一个默认数组大小，当我们创建的容器没用指定容量大小时，就会默认使用这个常量作为数组大小。因此当我们创建一个ArrayList实例的时候，最好考虑一下业务场景，如果我们将频繁的存储大量的元素，那么最好在创建的时候指定一个合理的size。所谓动态扩容，就是当数组中存储的元素达到容量上限以后，ArrayList会创建一个新的数组，新数组的大小为当前数组大小的1.5倍。随后将数组元素拷贝到新数组，如果这个动作频繁执行的话，会消耗性能。























