#Map

和Collection一样，Map也是集合容器的一个顶层接口。Map是通过key-value方式存储数据，key值都是唯一的，但key是否能为空，则要看他的不同子类的实现。我们可以把Map看成一个小型的数字字典，通过key值的方式存储数据性能非常快，比如他的子类Hashmap，底层就是通过散列表来实现存储，他的时间复杂度是O(1)。另一个典型的子类Treemap是基于红黑树实现的，时间复杂度为O(log n)。以下将介绍部分方法。

| method | desc |
|--------|--------|
|size()| 获取容器中元素的个数  |
|isEmpty()| 判断容器中是否保函元素，如果不包含元素，返回true  |
|containsKey(Object key)|判断是否保函key值  |
|containsValue(Object value)|判断是否保函Value|
|get(Object key)|通过key值获取元素 |
|put(K key, V value)|将一个元素通过键值对的方式存入容器，如果以存在这个key，则会替换value。|
|remove(Object key)|移除一个元素|
|putAll(Map<? extends K, ? extends V> m)|将容器中所有元素拷贝到当前容器。|
|interface Entry<K,V> |这个内部类其实也是一个顶层接口，所有存入容器的元素首先都会被包装成一个entry元素。Map的不同子类都会实现这个接口，而且实现各不相同。原因很简单，每个子类都有独特的数据结构来存储数据，TreeMap用的是红黑树，HashMap用的是散列表，而LinkedHashMap则是用散列表+双向链表实现的。下面是这三个类对Entry的部分实现代码：|
 
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

#AbstractMap
AbstractMap这个类没特别的地方，他相当于一个骨架，对Map中的接口方法做了一个最简化的实现，设计他的目的是为了最小化接口实现的压力。

#SortedMap
SortedMap是Map的子接口，他是一个排序接口，所有实现他的子类都事排序的，通常都是根据key进行排序。因此所有key值都必须实现Comparator接口。下面列举几个他特有的排序方法。

| method | desc |
|--------|--------|
|firstKey()|获取第一个元素|
|lastKey()|获取最后一个元素|