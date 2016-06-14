#Iterator

Iterator模式用于遍历所有集合类的元素，他的设计是为了把所有的集合遍历逻辑抽出来，从而避免向客户暴露集合的内部结构。

##### 1. Iterator
boolean hasNext()：这个方法是在遍历的时候，判断是否还有更多的元素  
E next()： 返回下一个元素  
default void remove()：这里涉及到了jdk8的特性，在接口定义中，将方法描述为default-虚拟扩展方法，就可以在接口中进行默认实现，从而提高接口的扩展性，避免在接口扩展的时候，破坏原有的实现。  
default void forEachRemaining(Consumer<? super E> action)：这个方面一般都用不到，不做具体描述。

##### 2. ListIterator
ListIterator 是对Iterator的扩展，他增加了  
boolean hasPrevious()  
E previous()  
int previousIndex()  
void set(E e)  
void add(E e) 5个方法。通过前3个方法可以进行向前遍历元素，后面两个set和add 可以插入元素，但是set是将元素插入到链表的最后位置，add是插入到当前返回的元素之前。

以上两个接口的具体实现会在后面的子类中有所描述。