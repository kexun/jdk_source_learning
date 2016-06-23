#HashMap
#####前言：
从今天开始我将介绍Map系列接口，我认为Map是集合类中概念最多，实现最复杂的一类接口。在讲解过程中会涉及到不少数据结构的知识，这部分知识点需要读者额外花一定时间系统学习。  

HashMap是Map的一个实现类，这个类很重要，是很多集合类的实现基础，底层用的就是他，比如前文中讲到的HashSet，下文要讲到的LinkedHashMap。我们可以将HashMap看成是一个小型的数字字典，他以key-value的方式保存数据，Key全局唯一，并且key和value都允许为null。  

HashMap底层是通过维护一个数据来保存元素。当创建HashMap实例的时候，会通过指定的数组大小以及负载因子等参数创建一个空的数组，当在容器中添加元素的时候，首先会通过hash算法求得key的hash值，再根据hash值确定元素在数组中对应的位置，最后将元素放入数组对应的位置。在添加元素的过程中会出现hash冲突问题，冲突处理的方法就是判断key值是否相同，如果相同则表明是同一个元素，替换value值。如果key值不同，则把当前元素添加到链表尾部。这里引出了一个概念，就是HashMap的数据结构其实是：hash表+单向链表。通过链表的方式把所有冲突元素放在了数组的同一个位置。但是当链表过长的时候会影响HashMap的存取效率。因此我们在实际使用HashMap的时候就需要考虑到这个问题，那么该如何控制hash冲突的出现频率呢？HashMap中有一个负载因子(loadFactor)的概念。容器中实际存储元素的size = loadFactor * 数组长度，一旦容器元素超出了这个size，HashMap就会自动扩容，并对所有元素重新执行hash操作，调整位置。好了说了这么多，下面就开始介绍源码实现。  

#####一、Node结构介绍
Node类实现了Map.Entry接口，他是用于存放数据的实体，是容器中存放数据的最小单元。Node的数据结构是一个单向链表，为什么选用这种结构？那是因前文讲到的，HashMap存放数据的结构是：hash表+单向链表。下面给出定义Node的源码：  
```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}

```
这个结构非常简单，定义了一个hash和key，hash值是对key进行hash以后得到的。value保存实际要存储的对象。next指向下一个节点。当hash冲突以后，就会将冲突的元素放入这个单向链表中。  

#####二、创建HashMap
创建HashMap实例有四个构造方法，这里着重介绍一个，看源码：  
```
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

// HashMap的数组大小是有讲究的，他必须是2的幂，这里通过一个牛逼哄哄的位运算算法，找到大于或等于initialCapacity的最小的2的幂
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```
构造方法中有两个参数，第一个initialCapacity定义map的数组大小，第二个loadFactor意为负载因子，他的作用就是当容器中存储的数据达到loadFactor限度以后，就开始扩容。如果不设定这样参数的话，loadFactor就等于默认值0.75。但是细心的你会发现，容器创建以后，并没有创建数组，原来table是在第一次被使用的时候才创建的，而这个时候threshold = initialCapacity * loadFactor。 这才是这个容器的真正的负载能力。  
tableSizeFor这个方法的目的是找到大于或等于initialCapacity的最小的2的幂，这个算法写的非常妙，值得我们细细品味。  
假设cap=7  
第一步 n = cap -1 = 6 = 00000110  
第二步 n|= n>>>1:  
n>>>1表示无符号右移1位，那么二进制表示为00000011，此时00000110 | 00000011 = 00000111  
第三步 n|=n>>>2:  
00000111 & 00000001 = 00000111  
第四部 n|=n>>>4：  
00000111 & 00000000 = 00000111  
第五步 n|=n>>>8;  
00000111 & 00000000 = 00000111  
第六步 n|=n>>>16;  
00000111 & 00000000 = 00000111  
最后 n + 1 = 00001000  
其实他的原理很简单，第一步先对cap-1是因为如果cap原本就是一个2的幂，那么最后一步加1，会使得这个值变成原来的两倍，但事实上原来这个cap就是2的幂，就是我们想要的值。接下来后面的几步无符号右移操作是把高位的1补到低位，经过一系列的位运算以后的值必定是000011111...他的低位必定全是1，那么最后一步加1以后，这个值就会成为一个00010000...(2的幂次)，这就是通过cap找到2的幂的方法。看到如此简约高效的算法，我服了。

#####三、put添加元素
添加一个元素是所有容器中的标配功能，但是至于添加方式那就各有千秋，Map添加元素的方式是通过put，向容器中存入一个Key-Value对。下面我将详细介绍put的实现过程，这个方法非常重要，吃透了这个方法的实现原理，基本也就能搞懂HashMap是怎么一回事了。  
```
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

// 获取key的hash值，这里讲hash值的高16位右移和低16位做异或操作，目的是为了减少hash冲突，使hash值能均匀分布。
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 如果是第一次添加元素，那么table是空的，首先创建一个指定大小的table。
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 通过对hash值取模的方法，确定key对应的数组位置，然后读取该位置中的元素。
    if ((p = tab[i = (n - 1) & hash]) == null)
    	// 如果当前位置为空，那么就在当前数组位置，为这个key-value创建一个节点。
        tab[i] = newNode(hash, key, value, null);
    else {
    	// 如果当前位置已经存在元素，那么就要逐个读取这条链表的元素。
        Node<K,V> e; K k;
        // 如果key和hash值都等于当前头元素，那么这存放的两个元素是相同的。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果当前位置的链表类型是TreeNode，那么就讲当前元素以红黑树的形式存放。
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
            	// 遍历链表的所有元素，如果都未找到相同key的元素，那么说明这个元素并不在容器中存在，因此将他添加到链表尾部，并结束遍历。
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果在遍历过程中，发现了相同的key值，那么就结束遍历。
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 如果e != null 说明在当前容器中，存在一个相同的key值，那么就要替换key所对应的value
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 这是专门留给LinkedHashMap调用的回调函数，LinkedHashMap会实现这个方法。从这里可以看出，HashMap充分的考虑了他的扩展性。
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 这里判断当前元素的数量是否超过了容量的上限，如果超过了，就要重新进行扩容，并对当前元素重新hash，所以再次扩容以后的元素位置都是会改变的。
    if (++size > threshold)
        resize();
    // 此方法也是HashMap留给afterNodeInsertion扩招的回调方法。透露一下，因为afterNodeInsertion在插入元素以后，都会维护他的一个双向链表。
    afterNodeInsertion(evict);
    return null;
}

```














