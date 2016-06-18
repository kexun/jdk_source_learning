#LinkedList

#####前言：
本文将介绍List家族中的另一个重要的实现类，他的实现有别与ArrayList和Vector，虽说List的所有类都是线性存储结构，但前者是通过数组存储的顺序结构，而本次要讲的LinkedList是通过链表的形式存放数据。当然，LinkedList也是非线程安全的，如果要在多线程环境下使用他，需要做一些额外的努力。  
LinkedList分别实现了List接口和Deque接口，通过这种杂交方式，使他具有了独特的有别于ArrayList的特异功能。那么这也是本文着重要讲解的地方，而与ArrayList类似的部分就弱化讲解了。

#####一、双向链表
ArrayList是通过数组实现存储，而LinkedList则是通过链表来存储数据，而且他实现的是一个双向链表，简单的说一下什么是双向链表。双向链表是数据结构的一种形式，他的每个节点维护两个指针，prev指向上一个节点，next指向下一个节点。这种结构有什么特点呢？他可以实现双向遍历，这使得在链表中的数据读取变得非常灵活自由。同时，LinkedList中维护了两个指针，一个指向头部，一个指向尾部。维护这两个指针后，可以使得元素从头部插入，也可以使元素从尾部插入。基于方式，用户很容易就能实现FIFO(队列)，LIFO(栈)等效果。那么下面我们来看一下源码中的具体实现。  
1.Node节点定义：
```
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
2.FIFO(队列)实现原理：队列的原理就是每次都从链表尾部添加元素，从链表头部获取元素，就像生活中的排队叫号，总是有个先来后到。  
```

// 队列尾部添加一个元素，建议使用这个，约定俗成吧。
public boolean offer(E e) {
    return add(e);
}
// 队列尾部添加一个元素
public boolean offerLast(E e) {
    addLast(e);
    return true;
}

// offer和offerLast底层调用的都是linkLast这个方法，顾名思义就是将元素添加到链表尾部。
void linkLast(E e) {
    final Node<E> l = last;
    // 创建一个节点，将prev指针指向链表的尾节点。
    final Node<E> newNode = new Node<>(l, e, null);
    // 将last指针指向新创建的这个节点。
    last = newNode;
    if (l == null)
    	// 如果当前链表为空，那么将头指针也指向这个节点。
        first = newNode;
    else
    	// 将链表的尾节点的next指针指向新建的节点，这样就完整的实现了在链表尾部添加一个元素的功能。
        l.next = newNode;
    size++;
    modCount++;
}

// 在链表头部删除一个元素，建议用这个，别问我为什么，我也不知道
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// 在链表头部删除一个元素
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
// poll和pollFirst底层调用的就是这个方法，将链表的头元素删除。
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}

// 获取头元素，但是不会删除他。
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

3.LIFO(栈)实现原理：栈的原理是每次从头部添加元素，也从头部获取元素，那么后进入的元素反而最先出来。就像我们平时叠盘子，洗好了就一个一个往上放，然后要用了就从上往下一个一个拿。  
```
// 在链表的头部添加一个元素
public void push(E e) {
    addFirst(e);
}

// addFirst调用的就是linkFirst，这段代码就是实现将元素添加的链表头部。
private void linkFirst(E e) {
    final Node<E> f = first;
    // 创建一个新元素，将元素的next指针指向当前的头结点
    final Node<E> newNode = new Node<>(null, e, f);
    // 将头指针指向这个节点。
    first = newNode;
    if (f == null)
    	// 如果当前节点为空，则把尾指针指向这个节点。
        last = newNode;
    else
    	// 将当前头结点的prev指针指向此结点。
        f.prev = newNode;
    size++;
    modCount++;
}

// 弹出顶部结点。
public E pop() {
    return removeFirst();
}

// removeFirst调用的就是unlinkFirst，unlinkFirst实现将链表顶部元素删除
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}

// 获取顶部结点，但是不删除
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```






















