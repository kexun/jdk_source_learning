#WeakHashMap

前言：  
WeakHashMap这个类我看了好久，一直不知道怎么写，有两点原因。第一：我怕把他写简单了，光从WeakHashMap的功能实现是描述，他和HashMap等非常的相似，无非也是用来hash表+单向链表的结构作为底层数据存储，再写一遍没太大意思。 第二：WeakHashMap的特点是以一种弱引用的关系存储数据，存储对象长期不用，可以被垃圾回收。讲这部分内容非常有意思，但是关于jvm部分了解不深，又怕讲不好。因此托了很长时间没写。  
权衡以后，决定先讲解WeakHashMap弱引用的实现原理，垃圾回收部分先缓一缓。 


```
package com.daimazhimei;



public class Day03 {

	private int index = 0;
	
	private static int size = 0;
	static private class Lock { };
    private static Lock lock = new Lock();
	
	private static Entry head = null;
	
	public static void main(String[] args) {
		
		new Thread(new Day03().new Consumer("aaa")).start();
		new Thread(new Day03().new Consumer("bbb")).start();
		new Thread(new Day03().new Consumer("ccc")).start();
		new Thread(new Day03().new Consumer("ddd")).start();
		new Thread(new Day03().new Producer("张三")).start();
//		new Thread(new Day03().new Producer("李四")).start();
		
	}
	
	
	public synchronized int getSize() {
		return size;
	}

	public synchronized void addSize() {
		this.size++;
	}
	
	public synchronized void manSize() {
		this.size--;
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
							Thread.sleep(getSleepTime());
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						temp = head;
						head = temp.next;
						temp.next = null;
						manSize();
						System.out.println("消费"+getSize()+"--"+name+":"+ temp.value);
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
							Thread.sleep(getSleepTime());
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						index++;
						Entry temp = new Entry(index);
						temp.next = head;
						head = temp;
						addSize();
						System.out.println("生产"+getSize()+"--" + name + ":" + index);
						lock.notify();
					} else {

						 try {
						 lock.wait();
						 } catch (InterruptedException e) {
						 // TODO Auto-generated catch block
						 e.printStackTrace();
						 }
					}

				}

			}

		}
		
	}
		
	public int getSleepTime() {
		java.util.Random random=new java.util.Random();// 定义随机类
		int result=random.nextInt(1000);// 返回[0,10)集合中的整数，注意不包括10
		return result+1;              // +1后，[0,10)集合变为[1,11)集合，满足要求
	}
	
}
```
