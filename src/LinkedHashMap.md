#LinkedHashMap

#####前言：
本文讲述的LinkedHashMap是HashMap的子类，他不仅实现了HashMap的所有功能，更是维护了元素的存储顺序。LinkedHashMap维护元素顺序的方式有两种，一种是维护他的存入顺序，另一种则是维护元素的读取顺序。基于这种功能，LinkedHashMap可以在用于LRU算法的实现。LRU为何物？LRU的全称是Least Recently Used翻译过来就是最近最久未使用，他通常应用于缓存的一种实现方式，当缓存数据满时，删除最少使用的缓存数据。  
LinkedHashMap的结构是HashMap+双向链表。他通过继承HashMap得到了用hash表存储数据的能力，同时他又维护了一个双向链表实现了对元素的排序功能。HashMap部分上文已经介绍了，本文着重要介绍的是双向链表部分实现（这里有必要说明一下，我写的系列文章是基于jdk1.8的。jdk1.8和之前版本的实现有不少差异，LinkedHashMap部分就改动了不少，有兴趣的同学可以对照1.7的链表实现和1.8的链表实现，你会发现是两者差异很大，对于两种实现的优缺点可以自行思考哦）。下面就来一起研究研究吧。

#####一、双向链表结构
jdk1.8的链表结构和1.7的差异很大，可以看出来1.8中的实现简化了不是，只维护了两个指针，befor和after。在整个链表中维护了head（头指针）和tail（尾指针）。这两个指针是有讲究的，head所指向的是eldest元素，也就是最老的元素，tail指向youngest元素，也就是最年轻的元素。在这个链表中，都是在队尾添加元素，队头删除元素，这种方式很像队列，但是还是有点区别。  
```
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// 指向eldest元素
transient LinkedHashMap.Entry<K,V> head;
// 指向youngest元素
transient LinkedHashMap.Entry<K,V> tail;
```

#####二、LinkedHashMap实例创建
LinkedHashMap的创建和HashMap没什么两样，就是这个构造方法中，加入了acessOrder的参数，告诉LinkedHashMap以哪种方式维护顺序。
```
// 元素遍历顺序，true维护元素的访问顺序，最新访问的放入队尾，false维护元素的插入顺序，最新插入的在队尾。
final boolean accessOrder;

public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

#####三、get获取元素

LinkedHashMap的get方法几乎就是复用了HashMap。唯一的区别就是多了一个accessOrder判断，如果accessOrder==true说明他需要维护元素的访问顺序，而afterNodeAccess是HashMap提供的回调方法，他也会在put元素的时候调用。afterNodeAccess方法的作用就是将当前访问的元素添加到队尾，因为这个链表都是从头部删除，因此这个元素会在最后才被删除。
```
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

void afterNodeAccess(Node<K,V> e) { // 将访问元素添加到队尾
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        // 如果当前元素是头元素，那么就将head指向他的下一个节点
        if (b == null)
            head = a;
        else
            b.after = a;
        // 如果当前元素是尾元素，那么就将last指向他的上一个节点
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

#####四、put元素
LinkedHashMap并没有自己实现put方法，完完全全是复用了HashMap的，因为HashMap提供了两个回调方法作为他的扩展，LinkedHashMap只需要实现这两个方法即可，从这里也可以学到如何提供代码的扩展性，预先留出回调接口也是个不错的选择哦。在HashMap的put方法中，调用了两个回调方法，afterNodeAccess和afterNodeInsertion。第一个方法已经介绍了，下面就介绍afterNodeInsertion，这个方法的主要目的就是在map添加元素以后，维护链表的顺序，同时也会控制了对链表头元素的删除与否。  
```
// 在插入元素以后，判断当前容器的元素是否已满，如果是的话，就删除当前最老的元素，也就是队头元素。
void afterNodeInsertion(boolean evict) {
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

// 这是用户实现的回调方法，判断当前最老的元素是否需要删除，如果为true，就删除链表头元素
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

#####五、删除元素
在删除元素以后，LinkedHashMap需要维护当前链表的指针，也就是双向链表的head和tail指针的指向问题  
```
void afterNodeRemoval(Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    // 如果当前元素是头元素，那么head指向他的下一个节点
    if (b == null)
        head = a;
    else
        b.after = a;
        
    // 如果当前元素是尾元素，那么tail指向他的上一个节点
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

#####六、LinkedHashIterator遍历
容器的遍历是一个亘古不变的话题，然而LinkedHashMap的遍历方式有他的特殊性。因为他在hash表的基础之上又维护了一个双向链表，而这个链表维护这元素的遍历顺序，因为LinkedHashMap在遍历的时候，只能遍历这个链表，而不能像HashMap一样遍历hash表。  
```
abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next;
    LinkedHashMap.Entry<K,V> current;
    int expectedModCount;

    LinkedHashIterator() {
    	// 第一次从头开始遍历
        next = head;
        expectedModCount = modCount;
        current = null;
    }

    public final boolean hasNext() {
        return next != null;
    }

	// 对链表从头到尾开始遍历，顺序遍历的方式很简单就是next = e.after
    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
        return e;
    }

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




























