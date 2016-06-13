# jdk_source_learning
jdk源码学习笔记，人话翻译

前言： 前一段时间开始学习了一些基本的数据结构和算法，算是弥补了这方面的知识短板，但是仅仅是对一些算法的了解，目前工作当中也并没有应用到这些，因此希望通过结合实际例子来学习，巩固之前学到的内容，思前想后觉得jdk源码其实非常适合学习，首先jdk的一些类在工作中使用频率非常高，并且他的底层实现结合了不少的设计模式，和算法。如：java集合类中的LinkedHashMap通过维护hash表和双向链表，可以实现读取数据O(1)的时间复杂度，并可以用于实现LRU算法。 jdk中的绝大部分代码都是经过千锤百炼的，代码质量非常之高，在了解其底层实现的过程中，也可以帮助我们提高编码规范，养成良好的习惯。
####一. java 集合类
![](/img/1.gif)
上图为java集合类的集合框架图，图中非常清楚的展示了java集合类中的各种依赖继承关系。所有的元素都实现了Iterator接口，用于遍历集合元素。集合分两大类，Collection和Map，Collection中又分List和Set，Map接口下有HashMap，Hashtable,TreeMap等。顾名思义这些不同的子类都有对应不同的含义，本文要详细讲述的就是不同子类的具体实现，以及子类之间的异同点。


1. Iterator LinkIterator
2. Collection List Set Queue
3. Map AbstractMap SortedMap