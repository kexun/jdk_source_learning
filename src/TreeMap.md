#TreeMap

#####前言：
TreeMap和HashMap一样实现的是Map接口，但两者的实现方式天差地别。HashMap的底层是hash表+单向链表的形式存储数据，TreeMap底层是通过红黑树存储数据。HashMap因为是基于散列表的实现，所以时间开销为O(1)，TreeMap的时间开销是O(lgn)。TreeMap的优势在于他是基于key值排序的。  

红黑树有五大特性：  
1）每个结点要么是红的，要么是黑的。  
2）根结点是黑的。  
3）每个叶结点，即空结点（NIL）是黑的。  
4）如果一个结点是红的，那么它的俩个儿子都是黑的。  
5）对每个结点，从该结点到其子孙结点的所有路径上包含相同数目的黑结点。  

他的所有操作都是围绕着这五大特性展开的。这五大特性的最终目的就是为了维持二叉树的相对平衡性。当每次二叉树操作以后，有可能会出现违反特性的情况（也就是出现了失衡状况），这时二叉树需要通过左旋，右旋，重新着色等系列操作，再次找到平衡点。  

我的理解是：红黑色的操作就两个要点。第一：遵循二叉查找树的规范，对所有元素进行排序，但这里存在着不确定情况，有可能出现左右子树深度极其不对称的情况，导致最坏时间复杂度出现O(n)的情况。  第二：正因为存在二叉树严重不平衡的情况，所以就出现了红黑二叉树，通过标记每个节点的颜色，动态的调整二叉树的结构，使其始终维持在相对平衡的状态，这样做的好处就是查找性能始终维持在O(lgn)的较高水平。  


#####一、添加元素
在TreeMap中插入元素，原理就是在二叉查找树中插入元素。通过对二叉树节点大小的对比插入相应的位置，时间复杂度为O(lgn)。但是在插入完毕以后，有可能会导致红黑树被破坏的情况，因此需要修复当前结构，重新着色。  

put方法：  
```
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}

```

插入元素后调整红黑树结构：  
```
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    root.color = BLACK;
}
```