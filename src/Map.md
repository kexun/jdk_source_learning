#Map

和Collection一样，Map也是集合容器的一个顶层接口。Map是通过key-value方式存储数据，key值都是唯一的，但key是否能为空，则要看他的不同子类的实现。我们可以把Map看成一个小型的数字字典，通过key值的方式存储数据性能非常快，比如他的子类Hashmap，底层就是通过散列表来实现存储，他的时间复杂度是O(1)。另一个典型的子类Treemap是基于红黑树实现的，时间复杂度为O(log n)。以下将介绍部分方法。

1.size() 获取容器中元素的个数
2.isEmpty() 判断容器中是否保函元素，如果不包含元素，返回true
3.containsKey(Object key)判断是否保函key值
4.containsValue(Object value)判断是否保函Value
5.get(Object key)通过key值获取元素
6.put(K key, V value)将一个元素通过键值对的方式存入容器，如果以存在这个key，则会替换value。
7.remove(Object key)移除一个元素
8.putAll(Map<? extends K, ? extends V> m)将容器中所有元素拷贝到当前容器。
9.interface Entry<K,V> 这个内部类其实也是一个顶层接口，所有存入容器的元素首先都会被包装成一个entry元素。Map的不同子类都会实现这个接口，而且实现各不相同。原因很简单，每个子类都有独特的数据结构来存储数据，TreeMap用的是红黑树，HashMap用的是散列表，而LinkedHashMap则是用散列表+双向链表实现的。
######TreeMap：
```
***省略***
K key;
V value;
Entry<K,V> left = null;
Entry<K,V> right = null;
Entry<K,V> parent;
boolean color = BLACK;
***省略***
```
######HashMap：
```
***省略***
final int hash;
final K key;
V value;
Node<K,V> next;
***省略***
```
######LinkedHashMap
```
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```