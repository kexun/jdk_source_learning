#TreeSet

#####前言：
Set接口有三个实现类，HashSet，LinkedHashSet和TreeSet。HashSet前文已经介绍过了，他内部通过维护一个HashMap存储数据，LinkedHashSet继承了HashSet，他与HashSet的唯一区别就是他维护的是一个LinkedHashMap，实现了双向链表的功能，维护数据插入的顺序，这部分内容我将在介绍Map类的时候详细介绍。TreeSet也就是本文要介绍的这个实现类，顾名思义他内部维护的是一个TreeMap，底层是红黑二叉树，他使得集合内都是有序的序列。写Set这部分的时候让我很纠结，set有点像一个精简版的map，他核心的功能都是用了map的实现，但是功能又远没有map强大。关于这点让我很困惑，或许作者设计这个接口的用意就是希望用户能快速简单的操作集合元素，而不需要像map那么复杂。  
TreeSet实现的部分和HashSet非常类似，我不太希望重复描述哪些相似的地方，而把精力放在重点部分，读者可以结合HashSet一起阅读。  

#####一、Comparator和TreeSet的创建
TreeSet的创建和HashSet类似，创建的是一个TreeMap实例。TreeMap是一个什么存在呢？简单的说，他是基于红黑树结构实现的，会对元素进行排序。下面来看一看他的构造过程。  
```
public TreeSet() {
    this(new TreeMap<E,Object>());
}
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```
上面的四个构造函数中我着重要介绍第二个，他通过Comparator实例来创建TreeMap，那么Comparator到底是何方神圣呢？通过阅读Comparator的源码发现，这是一个用于集合类排序的辅助接口，用户需要实现compare方法。如果用户用了这种方式创建TreeSet，那么集合元素就不需要做额外处理，否则集合元素都需要实现Comparable接口，因为Tree在排序的时候会调用compare或者compareTo方法（介绍TreeMap的时候会具体讲解）。下面来看看我写的一个样例代码。  

```
public class MyComparator implements Comparator<Person> {
		@Override
		public int compare(Person o1, Person o2) {
			return o1.age - o2.age;
		}
	}
    
public class Person {
    public Integer age;
    public Person(Integer value) {
        this.age = value;
    }
}

public static void TreeSetTest() {

	// TreeMap在底层put元素的时候会判断是否存在Comparator实例，如果存在，则每次添加元素排序比较的时候会调用compare接口。
    TreeSet<Person> set = new TreeSet<Person>(new MyComparator());

    Person p1 = new Person(1);
    Person p2 = new Person(1);
    Person p3 = new Person(5);
    Person p4 = new Person(9);
    Person p5 = new Person(10);

    set.add(p1);
    set.add(p2);
    set.add(p3);
    set.add(p4);
    set.add(p5);

    Iterator<Person> i = set.iterator();
    while (i.hasNext()) {
        Person p = (Person) i.next();
        System.out.println(p.age);
    }
}

打印结果：
1
5
9
10
```

#####二、NavigableSet接口介绍

```
// 返回比当前元素小的最近的一个元素
public E lower(E e) {
    return m.lowerKey(e);
}

// 返回小于等于当前元素的最近一个元素
public E floor(E e) {
    return m.floorKey(e);
}

//　返回大于等于当前元素的最近一个元素
public E ceiling(E e) {
    return m.ceilingKey(e);
}

// 返回大于当前元素的最近一个元素
public E higher(E e) {
    return m.higherKey(e);
}
```































