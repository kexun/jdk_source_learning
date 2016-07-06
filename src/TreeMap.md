#TreeMap

#####前言：
TreeMap和HashMap一样实现的是Map接口，但两者的实现方式天差地别。HashMap的底层是hash表+单向链表的形式存储数据，TreeMap底层是通过红黑树存储数据。HashMap因为是基于散列表的实现，所以时间开销为O(1)，TreeMap的时间开销是O(lgn)。TreeMap的优势在于他是基于key值排序的。  
