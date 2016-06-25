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
    // 通过对hash与数组长度的与操作，确定key对应的数组位置，然后读取该位置中的元素。
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
    // 此方法也是HashMap留给LinkedHashMap实现的回调方法。透露一下，因为LinkedHashMap在插入元素以后，都会维护他的一个双向链表
    afterNodeInsertion(evict);
    return null;
}

```

#####四、get获取元素
使用HashMap有一个明显的优点，就是他的存取时间开销基本维持在O(1)，除非在数据量大了以后hash冲突的元素多了以后，对其性能有一定的影响。那么现在介绍的get方法很好的体现了这个优势。  
```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

// 同一个key的hash值是相同的，通过hash就可以求出数组的下标，便可以在O(1)的时间内获取元素。
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 在容器不为空，并且对应位置也存在元素的情况下，那么就要遍历链表，找到相同key值的元素。
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 如果第一个元素的key值相同，那么这个元素就是我们要找的。
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 如果第一个元素不是我们要找的，接下来就遍历链表元素，如果遍历完了以后都没找到，说明不存在这个key值
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#####五、remove删除元素
删除元素的实现原理和put，get都类似。remove通过给定的key值，找到在hash表中对应的位置，然后找出相同key值的元素，对其删除。  
```
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

// 通过key的hash值定位元素位置，并对其删除。这里的实现和put基本相同，我只在不同的地方做一下解释。
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 如果找到了相同的key，接下来就要判断matchValue参数，matchValue如果是true的话，就说明
        // 需要检查被删除的value是否相同，只有相同的情况下才能删除元素。如果matchValue是false的话
        // 就不需要判断value是否相同。
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}

```

#####六、resize动态扩容
resize这个方法非常重要，他在添加元素的时候就会被调用到。resize的目的是在容器的容量达到上限的时候，对其扩容，使得元素可以继续被添加进来。这里需要关注两个参数threshold和loadFactor，threshold表示容量的上限，当容器中元素数量大于threshold的时候，就要扩容，并且每次扩容都是原来的两倍。loadFactor表示hash表的数组大小。这两个参数的配合使用可以有效的控制hash冲突数量。  
```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 如果容器并不是第一次扩容的话，那么oldCap必定会大于0
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // threshold和数组大小cap共同扩大为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;
    }
    // 第一次扩容，并且设定了threshold值。
    else if (oldThr > 0)
        newCap = oldThr;
    else {
    	// 如果在创建的时候并没有设置threshold值，那就用默认值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
    	// 第一次扩容的时候threshold = cap * loadFactor
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    // 创建数组
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 如果不是第一次扩容，那么hash表中必然存在数据，需要将这些数据重新hash
    if (oldTab != null) {
    	// 遍历所有元素
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                	// 重新计算在数组中的位置。
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                	// 这里分两串，lo表示原先位置的所有，hi表示新的索引
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 因为cap都是2的幂次，假设oldCap == 10000，
                        // 假设e.hash= 01010 那么 e.hash & oldCap == 0。
                        // 老位置= e.hash & oldCap-1 = 01010 & 01111 = 01010
                        // newCap此时为100000，newCap-1=011111。
                        // 此时e.hash & newCap 任然等于01010，位置不变。
                        // 如果e.hash 假设为11010，那么 e.hash & oldCap != 0
                        // 原来的位置为 e.hash & oldCap-1 = 01010
                        // 新位置 e.hash & newCap-1 = 11010 & 011111 = 11010
                        // 此时 新位置 != 老位置  新位置=老位置+oldCap
                        // 因此这里分类两个索引的链表
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}

```

#####七、遍历
HashMap遍历有三种方式，一种是对key遍历，还有一种是对entry遍历和对value遍历。这三种遍历方式都是基于对HashIterator的封装，三种实现方式大同小异，因此我着重介绍EntryIterator的实现。  


```
// 对HashMap元素进行遍历。
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    // 第一次遍历的时候，实例化entrySet。
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}

final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator();
    }
    public final boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> candidate = getNode(hash(key), key);
        return candidate != null && candidate.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}

final class EntryIterator extends HashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}

// HashMap自己实现的遍历方法。上面的所有方法都是围绕这个类展开的。下面具体讲解这个类的实现原理。
abstract class HashIterator {
    Node<K,V> next;        // 指向下一个元素
    Node<K,V> current;     // 指向当前元素
    int expectedModCount;
    int index;             // 当前元素位置

    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        if (t != null && size > 0) { // 找到table中的第一个元素
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        // 判断当前元素是否为链表中的最后一个元素，如果在链表尾部，那么就需要重新遍历table，
        // 顺序找到下元素的位置。
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }

	// 删除当前遍历的元素。
    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}
```

总结一下这个遍历的过程是 EntrySet -> EntryIterator -> HashIterator。同理对key的遍历过程就是 KeySet -> KeyIterator -> HashIterator。可以看出来不管是哪种遍历，最终都是调用了HashIterator。









