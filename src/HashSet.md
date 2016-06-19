#HashSet

#####前言：
先说说HashSet的继承关系，HashSet继承了AbstractSet抽象类并实现了Set接口，AbstractSet的子类还包括TreeSet，里面实现了两个类公共的一部分方法，后面也会略有介绍。  
那么HashSet到底一个怎么样的存在呢?HashSet顾名思义就是通过Hash表的方式存储数据，既然提到hash，那么肯定少不了HashMap，其实HashSet很聪明，他只需在内部维护了一个HashMap实例，将数据存储在了map中，并且集合元素不能重复。因此HashSet可以说是集合类中实现最简单的一个，他基本就是实现了List接口中定义的几个方法。  
那么我想说HashSet和HashMap有什么区别呢？  
1.HashMap提供键值对的方式存储数据，而HashSet仅仅提供数据存储，并没有键值对应。他获取元素的方式也只能通过遍历的方式逐个获取  
2.HashMap在存入数据的时候是更加key值的hash值判断，而HashSet需要重写hashCode和equals两个方法，如果不重写则会调用默认的实现，用户在使用HashSet的时候要特别注意元素的euqals判断，有必要的话要重写一个，以免出现问题。  

#####一、HashSet创建
HashSet的创建其实就是实例化一个HashMap，可以创建默认大小的HashMap实例，也可以指定initialCapacity和loadFactor，这两个参数的具体含义会在介绍HashMap的文章中讲解。  
```
public HashSet() {
    map = new HashMap<>();
}
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

```
#####二、在HashSet中添加元素
HashMap中真正存放元素的地方就是HashMap，但是HashMap是K-V的形式，而HashSet中没有Key，那么他们是如何将HashSet中的元素映射到HashMap中的呢？原来作者将HashSet中的元素存放到了map的key中，而value则存放一个无意义的Object对象。这么做的好处还可以保证HashSet中的value值唯一。了解HashMap的同学知道，HashMap中校验key值唯一性的方式是通过hash值，然后根据hash值定位数组中的位置，但是也存在hash冲突的情况，那么解决hash冲突的方式就是在原来数组的位置增加一个链表，将hash值冲突，但是key值不同的元素存放在链表中。那么在判断key值是否相同的时候就用到了equals方法。从上面的描述中可以抓出几个关键点，第一hash值，第二equals方法。这就是为什么我们需要将存入HashSet的元素重写hashCode和euqals方法的根本原因。
```
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

```

#####三、HashSet遍历
我找了HashSet的源码中找了一通，然并没有找到get方法，那么我们将如何获取元素呢？我发现了iterator接口，这个接口返回的Iterator并不是HashSet自己维护的Iterator，而是通过返回HashMap的keySet().Iterator，这个迭代器遍历的是HashMap的key值。也就是HashSet中保存的value。
```
public Iterator<E> iterator() {
    return map.keySet().iterator();
}

```

















