#HashTable

#####前言：
前文中已对HashMap实现原理做了详细介绍，HashTable与HashMap的原理相同，实现方式也几乎一致。除了以下几点不同：  
1.HashMap非线程安全，HashTable线程安全。HashMap与HashTable的实现方法几乎一致，区别是HashTable对所有的方法进行了同步操作，确保了线程安全。但是有需要注意的是，他只确保单个操作的原子性，如果需要在并发环境下执行复合操作，那用户需要自行同步，否则会出现问题。  
2.HashMap的key和value可以为null， HashTable的Key和value不能为null。  

别的就不多说了，可以参考HashMap的实现原理

