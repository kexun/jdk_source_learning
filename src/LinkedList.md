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
2.FIFO(队列)实现原理：  
队列的原理就是每次都从链表尾部添加元素，从链表头部获取元素，就像生活中的排队叫号，总是有个先来后到。  
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

3.LIFO(栈)实现原理：  
栈的原理是每次从头部添加元素，也从头部获取元素，那么后进入的元素反而最先出来。就像我们平时叠盘子，洗好了就一个一个往上放，然后要用了就从上往下一个一个拿。  
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
以上就是LinkedList实现的两种功能，这里包含了大部分关于链表操作的方法，但不仅限于这几种。不管是栈也好，队列也好，元素都是从头部删除的unlinkFirst方法。但是用户在使用的过程中并不只用到上面两张方式，我们也可以从链表尾部删除元素如removeLast，peekLast，pollLast，unlinkLast等方法。

#####二、存取操作
上文讲到的功能，其实是实现了Deque接口，而现在要讲述的是实现与List的部分功能。那么最典型的操作就是直接对容器元素的读取，因为List容器的一大特点就是顺序存储，元素在容器中的位置和存入时是保持一致的，那么用户在读取元素的时候理所当然就可以通过元素下标来获取，下面就具体介绍这几种方法。  

1.将元素插入容器的指定位置  
```
// 将元素插入指定位置
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
     	// 如果位于末尾，则直接插入链表尾部
        linkLast(element);
    else
    	// 如果不是位于末尾，那么就将元素插入指定位置的元素之前，
        linkBefore(element, node(index));
}

// 将元素插入指定元素前，链表插入元素是数据结构中最基础的知识，这里就不再赘述了。
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}

```

2.获取指定位置元素  
```
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}

// 获取指定位置的元素，使用的方式就是对链表进行顺序遍历，直到指定位置位置，不过他的处理越有一些小技巧值得学习，这个技巧也是利用了双向链表的特性。
Node<E> node(int index) {
    // assert isElementIndex(index);

	// 在遍历之前先判断元素位置是位于链表前半部，还是位于后半部，如果位于后半部，那么从尾部向前遍历则可以大大提高效率。
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}


```
#####三、迭代器实现
LinkedList的迭代器实现有两个，一个是实现了Iterator接口的DescendingIterator，另一个则是实现了ListIterator接口的ListItr。  

1.ListItr  
ListItr遍历需要指定一个起始值  
```
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
```
ListItr会创建一个以index为起始值的迭代器，然后用户便可以以这个位置为起点，实现向前或者向后遍历。  
```
ListItr(int index) {
    // 实例化的时候，将next指针指向指定位置的元素
    next = (index == size) ? null : node(index);
    nextIndex = index;
}

// 向后遍历
public E next() {
    checkForComodification();
    if (!hasNext())
        throw new NoSuchElementException();

    lastReturned = next;
    next = next.next;
    nextIndex++;
    return lastReturned.item;
}

// 向前遍历
public E previous() {
    checkForComodification();
    if (!hasPrevious())
        throw new NoSuchElementException();

    lastReturned = next = (next == null) ? last : next.prev;
    nextIndex--;
    return lastReturned.item;
}

```

2.DescendingIterator  
DescendingIterator迭代器实现的是对链表从尾部向头部遍历的功能，他复用里ListItr中的previous方法，将当前位置指向链表尾部，然后逐个向前遍历。  
```
private class DescendingIterator implements Iterator<E> {
    private final ListItr itr = new ListItr(size());
    public boolean hasNext() {
        return itr.hasPrevious();
    }
    public E next() {
        return itr.previous();
    }
    public void remove() {
        itr.remove();
    }
}
```














