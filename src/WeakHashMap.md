#WeakHashMap

前言：  
WeakHashMap这个类我看了好久，一直不知道怎么写，有两点原因。第一：我怕把他写简单了，光从WeakHashMap的功能实现是描述，他和HashMap等非常的相似，无非也是用来hash表+单向链表的结构作为底层数据存储，再写一遍没太大意思。 第二：WeakHashMap的特点是以一种弱引用的关系存储数据，存储对象长期不用，可以被垃圾回收。讲这部分内容非常有意思，但是关于jvm部分了解不深，又怕讲不好。因此托了很长时间没写。  
权衡以后，决定先讲解WeakHashMap弱引用的实现原理，垃圾回收部分先缓一缓。  

#####一、与HashMap异同
WeakHashMap的实现和HashMap非常类似，他们有相同的数据结构，类似的存储机制。数据底层都是通过hash表+单向链表的结构存储数据。  
1.get  
获取元素的流程非常类似，首先对key进行hash处理，通过hash值寻找到对应的元素位置。数组元素保存的是单向链表，这是hash冲突导致的结果。然后元每个entry对比key值是否相同，最终找到对应的元素。  

2.put  
添加元素的流程也和HashMap类似，首先对key值hash处理，通过hash值求出对应的元素位置，然后通过对比单向链表中的key值，将元素添加到链表头部。最后会更加当前元素的数量与threshold对比，进行动态扩容。扩容部分的原理也和HashMap类似。

3.getTable  
getTable方法是WeakHashMap特有的，这个方法是干什么用的呢？因为我们知道WeakHashMap的元素是通过弱引用的关系存储的。在容器中，有部分元素长时间未用，会被垃圾回收，getTable的作用就是清除被垃圾回收的元素。源码如下：  
```
private Entry<K,V>[] getTable() {
    expungeStaleEntries();
    return table;
}
// 清除所有被垃圾回收的元素
private void expungeStaleEntries() {
	// queue队列中保存的是已经被垃圾回收的元素key值。遍历队列
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    e.value = null; // 有助于垃圾回收
                    size--;
                    break;
                }
                prev = p;
                p = next;
            }
        }
    }
}
```
从源码中可以看出，首先会遍历queue队列，队列中保存的元素是已经被垃圾回收的元素的key值。然后将该key从table中删除。这步方法在getTable，resize，size方法中使用到，并且在垃圾回收过程中也会使用，因此他是在多线程环境下调用的，所以该方法使用了同步方法，确保线程安全。  
那么是谁将垃圾回收的后的元素放入queue中的呢？这就要从Entry的结构说起。  

4.Entry结构介绍  
WeakHashMap的Entry是Reference的子类。Entry实例化时会引用当前的queue，如果当前Entry被垃圾回收后，会将key注册的queue中。在后文中我会详细介绍Reference类。  

Entry结构  
```
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
    V value;
    final int hash;
    Entry<K,V> next;

    Entry(Object key, V value,
          ReferenceQueue<Object> queue,
          int hash, Entry<K,V> next) {
        super(key, queue);
        this.value = value;
        this.hash  = hash;
        this.next  = next;
    }

    ......
}
```
##### 二、弱键引用
上文简单介绍了WeakHashMap和HashMap的异同点，大同小异，不想过多重复。我认为WeakHashMap的重点是对元素进行垃圾回收的部分。下面将结合源码进行讲解。  

```
public abstract class Reference<T> {

 	private T referent;

	// 将回收的元素添加到队列中。
    volatile ReferenceQueue<? super T> queue;

    @SuppressWarnings("rawtypes")
    Reference next;

    transient private Reference<T> discovered;

    static private class Lock { };
    private static Lock lock = new Lock();

	// 垃圾收集器将回收的引用加入队列
    private static Reference<Object> pending = null;

    private static class ReferenceHandler extends Thread {

        ReferenceHandler(ThreadGroup g, String name) {
            super(g, name);
        }

        public void run() {
            for (;;) {
                Reference<Object> r;
                synchronized (lock) {
                    if (pending != null) {
                        r = pending;
                        pending = r.discovered;
                        r.discovered = null;
                    } else {
                        try {
                            try {
                                lock.wait();
                            } catch (OutOfMemoryError x) { }
                        } catch (InterruptedException x) { }
                        continue;
                    }
                }

                // Fast path for cleaners
                if (r instanceof Cleaner) {
                    ((Cleaner)r).clean();
                    continue;
                }

                ReferenceQueue<Object> q = r.queue;
                if (q != ReferenceQueue.NULL) q.enqueue(r);
            }
        }
    }

    static {
        ThreadGroup tg = Thread.currentThread().getThreadGroup();
        for (ThreadGroup tgn = tg;
             tgn != null;
             tg = tgn, tgn = tg.getParent());
        Thread handler = new ReferenceHandler(tg, "Reference Handler");
        handler.setPriority(Thread.MAX_PRIORITY);
        handler.setDaemon(true);
        handler.start();
    }

    。。。省略。。。
}
```
Reference的实现原理：  

第一步通过静态代码块启动ReferenceHandler线程，所有Reference实例共享该线程。pending会被虚拟机调用，当有元素被垃圾回收以后，会添加到该队列中。Reference线程通过无线循环检查pending是否存在元素。当发现有元素被垃圾回收以后，会将该元素添加到queue中。如果当前没有发现被回收的元素，也就是pending为null时，会通过lock.wait() 阻塞线程。直到有元素被回收以后，会调用nodify唤醒线程。  
该过程涉及到多线程的知识，jvm垃圾回收的原理等，这部分比较复杂，暂时无法很好的讲解。将回收的元素加入到queue队列的部分实现用了消费者模式。为了读者更好的理解该原理，我对他进行模仿，实现了一个简单的消费者模式的demo。这个demo主要功能如下：  
1.多个生产者不断的生产某件产品，当产品数量大于10以后就停止生产，只要当产品数量小于10,就会生产。  
2.多个消费者不断的消费产品，知道消费结束。  
3.当生产者生产到10件以后，就阻塞生产线，通知消费者消费。  
4.当消费者消费完结束以后，阻塞消费，通知生产者生产。  


```
public class ProducerCustomerDemo {

	private static int index = 0;
	
	private static int size = 0;
	
	static private class Lock { };
    private static Lock lock = new Lock();
	
	private static Entry head = null;
	
	public synchronized int getSize() {
		return size;
	}

	public synchronized void addSize() {
		ProducerCustomerDemo.size++;
	}
	
	public synchronized void minusSize() {
		ProducerCustomerDemo.size--;
	}
	
	public synchronized int getIndex() {
		return index;
	}
	
	public synchronized void addIndex() {
		index++;
	}
	
	public synchronized void minusIndex() {
		index--;
	}
	
	public static void main(String[] args) {
		
		new Thread(new ProducerCustomerDemo().new Consumer("aaa")).start();
		new Thread(new ProducerCustomerDemo().new Consumer("bbb")).start();
		new Thread(new ProducerCustomerDemo().new Consumer("ccc")).start();
		new Thread(new ProducerCustomerDemo().new Consumer("ddd")).start();
		new Thread(new ProducerCustomerDemo().new Producer("张三")).start();
		new Thread(new ProducerCustomerDemo().new Producer("李四")).start();
	}
	
	class Entry {
		Entry next;
		int value;
		
		public Entry(int value) {
			this.value = value;
		}
	}
	
	class Consumer implements Runnable{

		private String name;
		
		public Consumer(String name) {
			this.name = name;
		}
		@Override
		public void run() {
			
			for (;;) {
				Entry temp = null;
				synchronized (lock) {
					if (head != null) {
						try {
							Thread.sleep(1000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						temp = head;
						head = temp.next;
						temp.next = null;
						minusSize();
						System.out.println("消费，当前产品数"+getSize()+"--"+name+":"+ temp.value);
						lock.notify();
					} else {
						try {
							lock.wait();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
				}
			}
		}
		
	}
	
	class Producer implements Runnable {

		private String name;
		
		public Producer(String name) {
			this.name = name;
		}
		
		@Override
		public void run() {

			for (;;) {
				synchronized (lock) {
					if (getSize() < 10) {
						try {
							Thread.sleep(1000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						addIndex();
						Entry temp = new Entry(getIndex());
						temp.next = head;
						head = temp;
						addSize();
						System.out.println("生产，当前产品数"+getSize()+"--" + name + ":" + temp.value);
						lock.notify();
					} else {
						 try {
						 lock.wait();
						 } catch (InterruptedException e) {
						 e.printStackTrace();
						 }
					}
				}
			}
		}
	}
		
}
```
