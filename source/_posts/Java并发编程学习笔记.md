---
layout: post
title: Java并发编程学习笔记
comments: true
toc: true
date: 2018-02-01 20:05:05
updated: 2018-02-01 20:05:05
categories:
tags:
---
目录

    1. [X] 线程安全
    2. [X] synchronized & volatile
    3. [X] 同步类容器、并发类容器“Concurrent”、“CopyOnWrite”
    4. [X] Queue
    5. [X] 生产者消费者模式
    6. [X] Executors线程池


学习目的

    1. 面试
    2. 提高技术
    3. 发现类似并非/分布式/并行处理问题


第一节 线程安全
线程安全


    1. 概念：当多个线程方位某一个类（对象或方法）时，这个类始终都能表现出正确的行为，那么这个类（对象/方法）是线程安全的
    2. synchronized: 可以在任意对象方法上加锁，而枷锁的这段代码被称为“互斥区”或“临界区”
    3. 示例：MyThread: 当多个线程访问maThread的run方法时，以排队的方式进行处理（这里排队时按照CPU分配的先后顺序而定的），一个线程想要执行synchronized修饰的方法里的代码，首先尝试获得锁，如果拿到锁，执行更改代码体，拿不到锁，这个线程就会不断的尝试获得这把锁，知道拿到为止，而且是多线程去竞争这把锁（也就是会有锁竞争问题）。


public class MyThread extends Thread{
      
      private int count = 5;
        
      //synchronized加锁
      public synchronized void run(){
            count --;
            System.out.println(this.currentThread().getName() + " count = " + count);
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            MyThread myThread = new MyThread();
            Thread t1 = new Thread(myThread, "t1");
            Thread t2 = new Thread(myThread, "t2");
            Thread t3 = new Thread(myThread, "t3");
            Thread t4 = new Thread(myThread, "t4");
            Thread t5 = new Thread(myThread, "t5");
            t1.start();
            t2.start();
            t3.start();
            t4.start();
            t5.start(); 
      }
}

第二节 多个线程多个锁
synchronized只能对一个对象加锁，如果要对整个类加锁，需要将类声明为static对象，使所有实例都使用类对象
关键字synchronized取得的锁都是对象锁，而不是把一段代码（方法）当作锁，两个对象，线程获得的就是两个不同的锁，它们互不影响；
有一种情况则是相同的锁，即在静态方法上加synchronized关键字，表示锁定.class类，类一级别的锁（独占.class类）
package com.mymuti.sample02;
public class MutiThread {
      
      private static int num = 0;
      
      /** static **/
      public static synchronized void printNum(String tag){
            try {
                  if(tag.equals("a")){
                        num = 100;
                        System.out.println("tag a, set num over!");
                        Thread.sleep(1000);
                  }else {
                        num = 200;
                        System.out.println("tag b, set num over!");
                  }
                  System.out.println("tag " + tag + ", num = " + num);
            } catch (InterruptedException e){
                  e.printStackTrace();
            }
      }
      
      //注意观察run方法的输出
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            
            //两个不同的对象
            final MutiThread m1 = new MutiThread();
            final MutiThread m2 = new MutiThread();
            
            Thread t1 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        m1.printNum("a");
                  }
            });
            
            Thread t2 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        m2.printNum("b");
                  }
            });
            
            t1.start();
            t2.start();
      }
}

同步和异步

synchronized

第三节 脏读
当读取线程较慢，写入线程教快时容易产生脏读---解决方法---读写同一个对象，用synchronized分别对读方法和写方法加锁


第四节synchronize其他概念

    1. 可以嵌套调用
    2. 支持继承
    3. 支持锁重入

package com.mymuti.lesson04_synchronized_02;
/**
 * synchronized的重入
 * @author Administrator
 *
 */
public class SyncDubbo1 {
      
      public synchronized void method1(){
            System.out.println("method1..");
            method2();
      }
      public synchronized void method2(){
            System.out.println("method2..");
            method3();
      }
      public synchronized void method3(){
            System.out.println("method3..");
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            final SyncDubbo1 sd = new SyncDubbo1();
            Thread t1 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        sd.method1();
                  }
            });
            t1.start();
      }
}

package com.mymuti.lesson04_synchronized_02;
/**
 * synchronized的重入
 * @author Administrator
 *
 */
public class SyncDubbo2 {
      static class Main{
            public int i = 10;
            public synchronized void operationSup(){
                  try {
                        i--;
                        System.out.println("Main print i = " + i);
                        Thread.sleep(100);
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
                  
            }
      }
      
      static class Sub extends Main {
            public synchronized void operationSub(){
                  try {
                        while(i > 0){
                              i--;
                              System.out.println("Sub print i = " + i);
                              Thread.sleep(100);
                              this.operationSup();
                        }
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            Thread t1 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        Sub sub = new Sub();
                        sub.operationSub();
                  }
            });
            t1.start();
      }
}

package com.mymuti.lesson04_synchronized_02;
/**
 * synchronized异常
 * @author Administrator
 *
 */
public class SyncException {
      private int i = 0;
      public synchronized void operation(){
            while(true){
                  try {
                        i++;
                        Thread.sleep(200);
                        System.out.println(Thread.currentThread().getName() + " , i = " + i);
                        if(i==10){
                              Integer.parseInt("a");
                              //throw new RuntimeException();
                        }
                  } catch (Exception e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                        System.out.println(" log info i =" + i);
                        //throw new RuntimeException();
                        //continue;
                  }
            }
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            final SyncException se = new SyncException();
            Thread t1 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        se.operation();
                  }
            },"t1");
            t1.start();
      }
}

第五节synchronize代码块

    1. Object lock--使用任意的Object进行枷锁，用法比较灵活
    2. String lock---不要使用String的常量加锁，会出现死循环问题
    3. 锁对象改变---对一个对象进行加锁的时候要注意对象本身是否发生改变，改变，那么持有的锁不同
    4. 死锁问题

package com.mymuti.lesson05_synchronized_03;
public class ObjectLock {
      public void method1(){
            synchronized (this){ //对象锁,this代表ObjectLock
                  try {
                        System.out.println("do method1..");
                        Thread.sleep(2000);
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
      }
      public void method2(){
            synchronized (ObjectLock.class) {
                  try {
                        Thread.sleep(2000);
                        System.out.println("do method2..");
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
      }
      
      public Object lock = new Object();
      public void method3(){ //任何对象锁
            synchronized (lock){
                  try {
                        System.out.println("do method3..");
                        Thread.sleep(2000);
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            ObjectLock ol = new ObjectLock();
            Thread t1 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        ol.method3();
                        ol.method1();
                        ol.method2();
                        
                  }
            });
            t1.start();
      }
}

package com.mymuti.lesson05_synchronized_03;
public class StringLock {
      public void method(){
            //new String("字符串常量");
            //synchronized("字符串常量"){
            synchronized(new String("字符串常量")){
                  while(true){
                        try {
                              System.out.println("当前线程： " + Thread.currentThread().getName() + "开始");
                              Thread.sleep(1000);
                              System.out.println("当前线程： " + Thread.currentThread().getName() + "结束");
                        } catch (InterruptedException e) {
                              // TODO Auto-generated catch block
                              e.printStackTrace();
                        }
                        
                  }
            }
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            final StringLock stringLock = new StringLock();
            Thread t1 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        stringLock.method();
                  }
            },"t1");
            
            Thread t2 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        stringLock.method();
                  }
            },"t2");
            t1.start();
            t2.start();
            
      }
}
package com.mymuti.lesson05_synchronized_03;
public class ModifyLock {
      private String name;
      private int age;
      
      public String getName() {
            return name;
      }
      public void setName(String name){
            this.name = name;
      }
      public int getAge() {
            return age;
      }
      public void setAge (int age){
            this.age = age;
      }
      
      public synchronized void changeAttributte(String name, int age){
            try {
                  System.out.println("当前线程： " + Thread.currentThread().getName() + "开始");
                  this.setName(name);
                  this.setAge(age);
                  System.out.println("当前线程： " + Thread.currentThread().getName() + "修改对象内容为：" + this.getName() + "," + this.getAge());
                  Thread.sleep(2000);
                  System.out.println("当前线程： " + Thread.currentThread().getName() + "结束");
            } catch (InterruptedException e) {
                  // TODO Auto-generated catch block
                  e.printStackTrace();
            }
      }
      
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            final ModifyLock modifyLock = new ModifyLock();
            Thread t1 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        modifyLock.changeAttributte("张三",20);
                  }
            },"t1");
            
            Thread t2 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        modifyLock.changeAttributte("李四",21);
                  }
            },"t2");
            t1.start();
            t2.start();
      }
}

package com.mymuti.lesson05_synchronized_03;
public class ChangeLock {
      private String lock = "lock";
      private void method(){
            synchronized (lock){
                  try {
                        System.out.println("当前线程： " + Thread.currentThread().getName() + "开始");
                        lock = "change lock";  //尽量不要修改lock的内容
                        Thread.sleep(6000);
                        System.out.println("当前线程： " + Thread.currentThread().getName() + "结束");
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
      }
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            final ChangeLock changeLock = new ChangeLock();
            Thread t1 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        changeLock.method();
                  }
            },"t1");
            
            Thread t2 = new Thread(new Runnable() {
                  @Override
                  public void run() {
                        changeLock.method();
                  }
            },"t2");
            t1.start();
            t2.start();
      }
}
第六节 volatile关键字

    1. volatile概念：
     volatile关键字的主要作用是使变量在多个线程减可见
    在java中，每一个线程都会又一块工作内存区，其中存放着所有线程共享的主内存中的变量值的拷贝，当线程执行时，他在自己的工作内存区中才做这些变量。为了存取一个公想的变量，一个线程通常先获取锁并区清除它的内存工作区，把这些共享变量从所有的线程的共享内存区中正确的装入到他自己所在的工作内存区中，当线程解锁时保证该工作内存中变量的值写回到共享内存中。
    一个线程可以执行的操作又use,assign(赋值），load,store,lock,unlock,而主内存可以执行的操作又read,write,lock,unlock，每个操作都是原子的。
    volatile的作用就是强制线程到主内存（共享内）里去读取变量，而不去线程工作内存区里去读取，从而实现了多个线程间的变量可见。也就满足线程安全的可见性。
    2. volatile 没有原子性

package com.mymuti.lesson06_volatile;
import java.util.concurrent.atomic.AtomicInteger;
/**
 * volatile关键字不具备synchronized关键字的原子性（同步）
 */
public class VolatileNoAtomic extends Thread {
      private static volatile int count;
      // static AtomicInteger count = new AtomicInteger(0);
      private static void addCount(){
            for(int i =0; i<1000; i++){
                  count ++;
                  //count.incrementAndGet();
            }
            System.out.println(count);
      }
      
      public void run() {
            addCount();
      }
      
      public static void main(String[] args) {
            // TODO Auto-generated method stub
            VolatileNoAtomic[] arr = new VolatileNoAtomic[10];
            for(int i = 0; i < 10; i++){
                  arr[i] = new VolatileNoAtomic();
            }
            
            for(int i = 0;i < 10; i++){
                  arr[i].start();
            }
      }
}

第七节 wait和notify

    1. wait/notify方法实现线程间的通信（注意这两个方法都是object的类的方法，所有的对象都可以使用着两个方法）
    2. wait 和 notify必须配合synchronized关键字使用
    3. wait方法释放锁，notify方法不释放锁


package com.mymuti.wait_notify;
import java.util.ArrayList;
import java.util.List;
/**
* wait notify 方法， wait释放锁，notify不释放锁
* @author Administrator
*
*/
public class ListAdd2 {
    private volatile static List list = new ArrayList();
    
    public void add(){
        list.add("hello");
    }
    
    public int size(){
        return list.size();
    }
    public static void main(String[] args) {
        final ListAdd2 list2 = new ListAdd2();
        
        //1 实例化一个lock
        //当使用wait和notify的时候，一i的那个要配合着synchronized关键字去使用
        final Object lock = new Object();
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run(){
                synchronized (lock) {
                    for(int i = 0; i< 10; i++){
                        list2.add();
                        System.out.println("当前线程： " + Thread.currentThread().getName() + "添加了一个元素..");
                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            // TODO Auto-generated catch block
                            e.printStackTrace();
                        }
                        if(list2.size() == 5){
                            System.out.println("已经发出通知..");
                            lock.notify();
                        }
                    }
                }
            }
        },"t1");
        
        Thread t2 = new Thread(new Runnable(){
            public void run(){
                synchronized(lock){
                    if(list.size() != 5){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            // TODO Auto-generated catch block
                            e.printStackTrace();
                        }
                    }
                    System.out.println("当前线程：" + Thread.currentThread().getName() + "收到通知线程停止");
                    throw new RuntimeException();
                }
            }
        },"t2");
        
        t2.start();
        t1.start();
    }
}


第八节 使用wait/notify模拟Queue
BlockingQueue:顾名思义，阻塞队列，阻塞的放入和取出数据，我们要实现LinkedBlokingQueue下面的两个简单方法put和take
put 把object加入queue,如果queque没有空闲，则阻塞
take,取出，若队列为空则阻塞
package com.mymuti.lesson08_queue_simulate;
import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
public class MyQueue {
    
    //1. 需要一个承装元素的集合
    private final LinkedList<Object> list = new LinkedList<Object>();
    
    //2. 需要一个计数器
    private AtomicInteger count = new AtomicInteger(0);
    
    //3. 需要制定上限和下限
    private int miniSize = 0;
    private int maxSize;
    
    //4. 构造方法
    public MyQueue(int size){
        this.maxSize = size;
    }
    
    //5. 初始化一个对象用于加锁
    private final Object lock = new Object();
    
    //6. put 方法
    public void put(Object obj){
        synchronized (lock) {
            while(count.get() == this.maxSize){
                try {
                    lock.wait();    //当长度满时，使用wait阻塞该线程
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }  
            }
            //1. 加入元素
            list.add(obj);
            //2.计数器增加
            count.incrementAndGet();
            //3.通知另一个线程
            System.out.println("新加入的元素为：" + obj);
            lock.notify();     //加入元素后广播通知
        }
    }
    
    //7. take方法
    public Object take(){
        Object ret = null;
        synchronized (lock){
            while(count.get() == this.miniSize){
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }
        
            //1. 移除元素
            ret = list.removeFirst();
            
            //2. 计数器递减
            count.decrementAndGet();
            
            //3. 唤醒另一个线程
            lock.notify();
        }
        return ret;
    }
    
    public int getSize(){
        return this.count.get();
    }
    
    
    public static void main(String[] args) {
        MyQueue mq = new MyQueue(5);
        mq.put("a");
        mq.put("b");
        mq.put("c");
        mq.put("d");
        mq.put("e");
        
        System.out.println("当前容器长度： " + mq.getSize());
        
        Thread t1 = new Thread(new Runnable(){
            @Override
            public void run(){
                mq.put("f");
                mq.put("g");
            }
        },"t1");
        
        Thread t2 = new Thread(new Runnable(){
            @Override
            public void run(){
                Object o1 = mq.take();
                System.out.println("移除的元素为： " + o1);
                Object o2 = mq.take();
                System.out.println("移除的元素为：" + o2);
            }
        },"t2");
        
        t1.start();
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        t2.start();
    }
}


第九节 单例和线程安全
TreadLocal概念：线程局部变量，是一种多线程间并发访问变量的解决方案。与其synchronized等加锁的方式不同，ThreadLocal完全不提供锁，而使用以空间换时间的手段，为每个线程提供变量的独立副本，以保障线程安全
从性能上说，ThreadLocal不具有绝对的优势，在并发不是很高的时候，枷锁的性能会更好，但作为一套与锁完全无关的线程安全解决方案，在高并发量或着竞争激烈的场景，使用ThreadLocal可以在一定程度上减少锁竞争
package com.mymuti.lesson09_thread_local;
      
public class ConnThreadLocal {
      public static ThreadLocal<String> th = new ThreadLocal<String>();
      
      public void setTh(String value){
            th.set(value);
      }
      
      public void getTh(){
            System.out.println(Thread.currentThread().getName() + ":" + this.th.get());
      }
      
      public static void main(String [] args){
            final ConnThreadLocal ct = new ConnThreadLocal();
            Thread t1 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        ct.setTh("张三");
                        ct.getTh();
                  }
            },"t1");
            
            Thread t2 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        try {
                              Thread.sleep(1000);
                        } catch (InterruptedException e) {
                              // TODO Auto-generated catch block
                              e.printStackTrace();
                        }
                        ct.getTh();
                  }
            },"t2");
            t1.start();
            t2.start();
      }
}
单例模式：最常见的就是饥饿模式和懒汉模式，一个直接实例化对象，一个在调用方法时进行实例化对象，在多线程模式中，考虑到性能和线程安全问题，我们一般选择下面两种比较经典的单例模式，在性能提高的同时，又保证了线程安全。
dubble check instance
static inner class--静态内部类
package com.mymuti.lesson09_thread_local;
/**
 * 静态内部类
 * @author Administrator
 *
 */
public class InnerSingleton {
      
      //静态内部类
      private static class Singletion {
            private static Singletion single = new Singletion();
      }
      
      
      //外部接口
      public static Singletion getInstance(){
            return Singletion.single;
      }
      
}

package com.mymuti.lesson09_thread_local;
public class DubbleSingleton {
      private static DubbleSingleton ds;  //懒汉模式，懒加载模式
      
      public static DubbleSingleton getDs(){
            if(ds == null){
                  //模拟初始化的准备时间
                  try {
                        Thread.sleep(1000);
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
                  synchronized (DubbleSingleton.class){
                        if(ds == null){    // Double Check 双重检查
                              ds = new DubbleSingleton();
                        }
                  }
            }
            return ds;
      }
      
      
      public static void main(String [] args){
            Thread t1 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        System.out.println(DubbleSingleton.getDs().hashCode());
                  }
            },"t1");
            
            Thread t2 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        System.out.println(DubbleSingleton.getDs().hashCode());
                  }
            },"t2");
            
            Thread t3 = new Thread(new Runnable(){
                  @Override
                  public void run(){
                        System.out.println(DubbleSingleton.getDs().hashCode());
                  }
            },"t3");
            
            t1.start();
            t2.start();
            t3.start();
      }
}

第十节 同步类容器和并发类容器
同步类容器
同步类容器都是都是线程安全的，单在某些场景下可能需要加锁来保护符合操作。复合操作如：迭代（反复访问元素，遍历玩容器中所有的元素）、跳转（根据指定的顺序找到当前元素的下一个元素）、以及条件运算。这些符合操作在多线程并发地修改容器时，可能会表现出意外的行尾，最经典的便是ConcurrentModificationException,愿意时当容器迭代的过程中，被并发的修改了内容，这是由于早期迭代器设计的时候没有考虑并发修改的问题。

同步类容器：如古老的Vector、HashTable.这些容器的同步功能其实都是由JDK的Collections.synchronized...等工厂方法去创建实现的，使得每次只能有一个线程访问容器的状态，这明显不满足我们今天互联网时代高并发的需求，在保证线程安全的同时，也必须要有足够好的性能。

并发类容器
jdk5.0以后提供了多种并发类容器来替代同步类容器从而改善性能。同步类容器的状态都是串行化的。他们虽然实现了线程安全，但是严重降低了并发性，在多线程环境时，严重降低了应用程序的吞吐量。
并发类容器时专门针对并发设计的，使用ConcurrentHashMap来代替给与散列的传统的HashTable,而且在ConcurrentHashMap中，添加了一些常见复合操作的支持。以及使用了CopyonWriteArrayList代替Voctor,并发的CopyonWriteArraySet,以及并发的Queue,ConcurrentLinkedQueue和LinkedBlockingQueue,前者是高性能的队列，后者是阻塞形式的队列，具体实现Queue还有很多，例如ArrayBlockingQueue、PriorityBlockingQueue、SynchronousQueue等

类型
同步类容器
并发类容器
Array
Vector
CopyonWriteArrayList

HashMap
HashTable
ConcurrentHashMap
ConcurrentSkipListMap---支持并发排序
Queue

ConcurrentLinkedQueue---高性能队列
LinkedBlockingQueue---阻塞队列
ArrayBlockingQueue
PriorityBlockingQueue
SynchronousQueue
ConcurrentMap
ConcurrentMap接口有以下两个重要的实现：
ConcurrentHashMap
ConcurrentSkipListMap(支持并发排序功能，弥补ConcurrentHashMap)
ConcurrentHashMap内部使用段（Segment)啦表示这些不同的部分，每个段其实就是一个小的HashTable,它们有自己的锁。只要多个修改操作发生在不同的段上，他们就可以并发的进行。吧一个整体分成了16个段（segment)。也就是最高只是16个线程的并发修改操作。这也是多线程场景时减小锁的粒度从而降低锁竞争的一种方案。并且代码中大多共享变量使用volatile关键字声明，目的时第一时间获取修改的内容，性能非常好。

第十一节 Concurrent与CopyOnWrite
Copy-On-Write简称COW，是一种用于程序设计中的优化策略。
JDK里的COW容器有两种：CopyonWriteArrayList和CopyOnWriteArraySet,COW容器非常有用，可以在非常多的并发场景中使用到。
什么是CopyOnWrite容器？
CopyOnWrite容器即写时复制的容器。通俗的理解时当我们往一个容器添加元素的时候，不直接往当前容器添加，而实先将当前容器进行Copy,复制出一个新的容器，然后新的容器里添加验收，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处时我们可以对CopyOnWrite容器进行并发的都，而不需要锁，因为当前容器不会添加任何元素。所有CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。
第十二节 Queue讲解
ConcurrentLinkedQueue:
是一个适用于高并发场景下的队列，通过无锁的方式，实现了高变更发状态下的高性能，通常ConcurrentLinkedQueue性能好余BlockingQueue,它是一个基于链接节点的无界线程安全队列。该队列的元素遵循先进先出的原则。头是最先加入的，尾是最近加入的，该队列不允许null元素。
ConcurrentLinkedQueue重要方法：
add()和offer()都是加入元素的方法（在ConcurrentLinkedQueue中，这两个方法没有任何区别）
poll()和peek()都是去头元素节点，区别在于前者会删元素，后者不会。

在并发队列上JDK提供了两套实现，ConcurrentLinkedQueue和BlockingQueue都继承自Queue

BlockingQueue接口

    * ArrayBlockingQueue: 基于数组的阻塞队列实现，在ArrayBlockingQueue内部，维护了一个定常数组，以便缓存队列中的数据对象，其内部没有实现读写分离，页就是以为着生成和消费不能完全并行，长度是需要定义的，可以指定现金显出或着先进后出，也叫有界队列，在很多场合非常适合使用
    * LinkedBlockingQueue：基于链表的阻塞队列，同ArrayBlockingQueue类似，期内部页维持着一个数据缓存队列（该队列由一个链表构成），LinkedBlockingQueue之所有能够搞笑的处理并发数据，是因为其内部实现采用分离锁（读写分离两个锁），从而实现生产者和消费者操作的完全并行允许，它是一个无界队列
    * PriorityBlockingQueue： 基于优先级的阻塞对垒（优先级的判断通过构造函数传入的Compator对象来决定，也就是说传入队列必须实现Comparable接口），在实现PriorityBlockingQueue时，内部控制线程同步的锁采用的是公平锁，他也是一个无界的队列
    * DelayQueue： 带有言辞的Queue,其中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue中的元素必须实现Delayed接口，DelayQueue是一个没有大小限制的队列，应用场景很多，比如对缓存超时的数据进行移除，任务超时处理，空闲链接的关闭等等
    * SynchronousQueue： 一种没有缓冲的队列，生产者产生的数据直接会呗消费着获取并消费



第十三节 Future模式
多线程的设计模式
并行设计模式俗语设计优化的一部分，它时对一些常用的多线程结构的总结和抽象。与串行程序相比，并行的结构通常更复杂。因此合理的使用并行模式在多线程开发中更具有意义，在这里主要介绍Future,Master-Worker和生产者-消费者模式

Future模式有点类似与商品的订单。比如在网购时，当看中一款商品时，就可以提交订单，当订单处理完成后，在家里等待订单送货上门即可。或者更形象的比如我们发送Ajax请求的时候，页面是异步的进行后台处理，用户无须一直等待请求的结果，可以继续浏览或操作其他内容。

package com.mymuti.lesson13_future;
public class FutureClient {
      public Data request(final String queryStr) {
            //1 我想要一个代理对象（Data接口的实现类）先返回给请求的客户段，告诉它已收到请求
            final FutureData futureData = new FutureData();
            //2 启动一个新的线程，去加载真实的数据，传递给这个代理对象
          new Thread(new Runnable() {
            @Override
            public void run() {
                  //3 这个新的线程可以去慢慢的加载真实对象，然后传递给代理
                  RealData realData = new RealData(queryStr);
                  futureData.setRealData(realData);
            }
          }).start();
         
          return futureData;
      }
}
package com.mymuti.lesson13_future;
public class FutureData implements Data{
      private RealData realData;
      private boolean isReady = false;
      
      public synchronized void setRealData(RealData realData){
            //如果已经加载完毕
            if(isReady){
                  return;
            }
            //如果没有装载，进行装载真实对象
            this.realData = realData;
            isReady = true;
            //进行通知
            notify();
      }
      
      @Override
      public synchronized String getRequest(){
            //如果没有装载好 程序一直阻塞
            while(!isReady){
                  try {
                        wait();
                  } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                  }
            }
            //装载好直接获取数据即可
            return this.realData.getRequest();
      }
}
package com.mymuti.lesson13_future;
public class RealData implements Data{
      private String result;
      
      public RealData (String queryStr){
            System.out.println("根据"+queryStr+"进行查询，这是一个很耗时的操作");
            try {
                  Thread.sleep(5000);
            } catch (InterruptedException e) {
                  // TODO Auto-generated catch block
                  e.printStackTrace();
            }
            System.out.println("操作完毕，获取结果");
            result = "查询结果：张珊";
      }
      
      @Override
      public String getRequest() {
            // TODO Auto-generated method stub
            return result;
      }
}
package com.mymuti.lesson13_future;
public interface Data {
      String getRequest();
      
}
package com.mymuti.lesson13_future;
public class Main {
      public static void main(String[] args) {
            FutureClient fc = new FutureClient();
            Data data = fc.request("请求参数");
            System.out.println("请求发送成功!");
            System.out.println("请做其他的事情...");
            
            String result = data.getRequest();
            System.out.println(result);
      }
}
第十四节 MasterWorker模式
Master-Worker模式是最常用的并行计算模式。它的核心思想是系统由两类进程协作工作：Master进程负责接收和分配任务，Worker负责处理子任务。当哥哥Worker子进程处理完成后，会将结果返回给Master,由Master做归纳和总结。其好处是将一个大任务分解成若干个小任务，并行执行，从而提高系统的吞吐量。
package com.mymuti.lesson14_master_worker;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedQueue;
public class Master {
    
    //1.应该有个承装任务的集合
    private ConcurrentLinkedQueue<Task> workQueue = new ConcurrentLinkedQueue<Task>();
    
    //2.是你用HashMap去承装所有worker对象
    private HashMap<String, Thread> workers = new HashMap<String,Thread>();
    
    //3.使用一个容器承装每一个worker执行任务的结果集合
    private ConcurrentHashMap<String, Object> resultMap = new ConcurrentHashMap<String, Object>();
    
    //4.构造方法
    public Master(Worker worker, int workerCount){
        //每一个worker对象都选哟由Master的引用workQueue用于任务的领取
        worker.setWorkerQueue(this.workQueue);
        worker.setResultMap(this.resultMap);
        
        for(int i = 0; i< workerCount; i++){
            //key表示每一个worker的名字，value表示线程执行对象
            workers.put("子节点" + Integer.toString(i), new Thread(worker));
        }
    }
    //5. 提交方法
    public void submit(Task task){
        this.workQueue.add(task);
    }
    //6.需要一个执行的方法
    public void execute(){
        for(Map.Entry<String, Thread>me: workers.entrySet()){
            me.getValue().start();
        }
    }
    
    //7.判断线程是否执行完毕
    public boolean isComplete() {
        for(Map.Entry<String, Thread>me: workers.entrySet()){
            if(me.getValue().getState() != Thread.State.TERMINATED){
                return false;
            }
            return true;
        }
        return false;
    }
    //8.返回结果集
    public int getResult(){
        int ret = 0;
        for(Map.Entry<String, Object>me: resultMap.entrySet()){
            ret +=(Integer)me.getValue();
        }
        return ret;
    }
    
    
}

package com.mymuti.lesson14_master_worker;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedQueue;
public class Worker implements Runnable{
    
    private ConcurrentLinkedQueue<Task> workQueue;
    private ConcurrentHashMap<String, Object> resultMap;
    public void setWorkerQueue(ConcurrentLinkedQueue<Task> workQueue) {
        this.workQueue = workQueue;
        
    }
    public void setResultMap(ConcurrentHashMap<String, Object> resultMap) {
        this.resultMap = resultMap;
        
    }
    
    @Override
    public void run(){
        while(true){
            Task input = this.workQueue.poll();
            if(input == null) break;
            //真正去做业务处理
            Object output = handle(input);
            this.resultMap.put(Integer.toString(input.getId()), output);
        }
    }
    
    private Object handle(Task input){
        Object output = null;
        try {
            Thread.sleep(500);
            output = input.getPrice();
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        
        return output;
    }
}

package com.mymuti.lesson14_master_worker;
import java.util.Random;
public class Main {
      public static void main(String[] args) {
            Master master = new Master(new Worker(), 20);
            
            Random r = new Random();
            for(int i = 1; i <= 100; i++) {
                  Task t = new Task();
                  t.setId(i);
                  t.setName("任务"+i);
                  t.setPrice(r.nextInt(1000));
                  master.submit(t);
            }
            master.execute();
            
            long start = System.currentTimeMillis();
            
            while(true){
                  if(master.isComplete()){
                        long end = System.currentTimeMillis() - start;
                        int ret = master.getResult();
                        System.out.println("最终结果：" + ret + ", 执行耗时：" + end);
                        break;
                  }
            }
      }
}
第十五节 生产者消费者模式
package com.mymuti.lesson15_provider_consumer;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;
public class ProviderConsumer {
    public static void main(String[] args) throws InterruptedException {
        // TODO Auto-generated method stub
        //内存缓冲区
        BlockingQueue<Data> queue = new LinkedBlockingQueue<Data>(10);
        //生产者
        Provider p1 = new Provider(queue);
        Provider p2 = new Provider(queue);
        Provider p3 = new Provider(queue);
        //消费者
        Consumer c1 = new Consumer(queue);
        Consumer c2 = new Consumer(queue);
        Consumer c3 = new Consumer(queue);
        
        //创建线程池运行，这是一个缓存的线程次，可以创建无穷大的线程，没有任务时不创建线程，空闲线程存货时间为60s
        ExecutorService cachePool = Executors.newCachedThreadPool();
        cachePool.execute(p1);
        cachePool.execute(p2);
        cachePool.execute(p3);
        cachePool.execute(c1);
        cachePool.execute(c2);
        cachePool.execute(c3);
        
        Thread.sleep(3000);
        p1.stop();
        p2.stop();
        p3.stop();
        Thread.sleep(2000);
        //cachePool.shutdown();
        //cachePool.shutdownNow();
        
    }
}

package com.mymuti.lesson15_provider_consumer;
import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
public class Provider implements Runnable{
    //共享缓存区
    private BlockingQueue<Data> queue;
    //多线程间，是否启动变量，有强制且内存中刷新的功能，即时返回线程的状态
    private volatile boolean isRunning = true;
    //id生成器
    private static AtomicInteger count = new AtomicInteger();
    //随机对象
    private static Random r = new Random();
    
    //构造方法
    public Provider(BlockingQueue queue){
        this.queue = queue;
    }
    
    @Override
    public void run() {
        while(isRunning){
            //随机休眠0-1000ms，表示获取数据(产生数据耗时)
            try {
                Thread.sleep(r.nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //获取的数据进行累计
            int id = count.incrementAndGet();
            //比如通过有一个getData方法获取了
            Data data = new Data(Integer.toString(id), "数据" + id);
            System.out.println("当前线程:" + Thread.currentThread().getName()+", 获取了数据,id为:" + id + ", 进行装载到公共缓存中...");
            try {
                if(!this.queue.offer(data, 3, TimeUnit.SECONDS)){
                    System.out.println("提交缓冲区数据失败...");
                    //do something ... 比如重新提交
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
    }
    public void stop() {
        this.isRunning = false;
        
    }
}

package com.mymuti.lesson15_provider_consumer;
import java.util.Random;
import java.util.concurrent.BlockingQueue;
public class Consumer implements Runnable{
    private BlockingQueue<Data> queue;
    
    public Consumer(BlockingQueue queue){
        this.queue = queue;
    }
    //随机对象
    private static Random r = new Random();
    
    @Override
    public void run() {
        while(true){
            try {
                //获取数据
                Data data = this.queue.take();
                Thread.sleep(r.nextInt(1000));
                System.out.println("当前消费线程: " + Thread.currentThread().getName() + ", 消费成功，消费数据" + data.toString());
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        
    }
}

package com.mymuti.lesson15_provider_consumer;
public final class Data {
      private String id;
      private String name;
      
      public Data(String id, String name){
            this.id = id;
            this.name = name;
      }
      
      public String getId(){
            return id;
      }
      public void setId(String id){
            this.id = id;
      }
      public String getName(){
            return name;
      }
      public void setName(String name){
            this.name = name;
      }
      
      @Override
      public String toString(){
            return "{id: " + id + ", name: " + name + "}";
            
      }
}
第十六节 线程池

为了更好的更好的控制多线程，JDK提供了一套线程框架Executor,帮助开发人员有效地进行线程控制。他们都在java.util.concurrent包中，是JDK并发包的核心。其中又一个比较重要的类：Executors,他扮演着线程工厂的角色，我们通过Executors可以创建特定功能的线程池。
Executers创建线程池方法：

    * newFixedThreadPool()，该方法返回一个固定数量的线程次，该方法的线程数始终不变，当有一个任务提交时，若线程池中空闲，则理解执行，若没有，则会配暂缓在一个任务队列中等待有空闲的线程去执行。
    * newSingleThreadExecutor()方法，创建一个线程的线程次，若空闲则执行，若没有空闲线程则暂缓到任务队列中
    * newCachedThreadPool()方法，返回一个可以根据实际情况调整线程个数的线程次，不限制最大线程数量，若有空闲的线程则执行任务，若无任务则不创建线程，并且每一个空闲线程会再60s后自动回收
    * newScheduledThreadPool()方法，该方法返回一个SchededExecutorService对象，但该线程池可以指定线程的数量。

package com.mymuti.lesson16_excutors;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class UseExecutors {
      ExecutorService pool = Executors.newFixedThreadPool(10);
}
package com.mymuti.lesson16_excutors;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledFuture;
import java.util.concurrent.TimeUnit;
class Temp extends Thread {
    public void run(){
        System.out.println("run");
    }
}
public class ScheduledJob {
    public static void main(String[] args) throws Exception{
        Temp command = new Temp();
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        
        ScheduledFuture<?> scheduleTask = scheduler.scheduleWithFixedDelay(command, 1, 3, TimeUnit.SECONDS);
        
        
        
    }
}
自定义线程池
若Executors工厂类无法满足我们的需求，可以自己去创建自定义的线程池，其实Executors工厂类里面的创建线程的方法其内部实现是用了ThreadPollExecutor这个类，这个类可以自定义线程。构造方法如下

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler){...}

自定义线程池使用详细

ThreadPoolExecutor这个构造方法对于队列是什么类型的比较关键：

    * 在使用有界队列时，若有新的任务需要执行，如果线程池时间线程数小于corePoolSize,则有限创建线程， 若大于corePoolSize，则会将任务加入回列，若队列已满，则在总线程数不大于maxiumumPoolSize的前提下，创建新的线程，若线程数大于maximumPoolSize, 则执行拒绝策略。或其他自定义方式。
    * 无界的任务队列时： LinkedBlockingQueue。与邮件队列相比，除非系统资源耗尽，否则无界的任务队列不存在任务入队失败的情况。当有新入伍到来，系统的线程数小于corePoolSize时， 则新建线程执行任务。当达到corePoolSize后，就不会继续增加。若后续仍有新的任务加入，而没有空闲的线程资源， 则任务直接进入队列等待。若任务创建和处理的熟读差异很大，无界队列会保持快速增长，直到耗尽内存。
    * JDK拒绝策略：

        * AbortPolicy: 直接抛出异常组织系统正常工作
        * CallerRunsPolicy: 只要线程池为关闭，该策略直接在调用者线程中，运行当前呗丢弃的任务。
        * DiscardOldestPolicy:丢弃最老的一个请求， 尝试再次提交当前任务。
        * DiscardPolicy: 丢弃无法处理的任务，不给予任何处理。
        * 如果需要自定拒绝策略可以实现RejectedExecutionHandler接口。


package com.mymuti.lesson16_excutors;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.RejectedExecutionHandler;
public class UseThreadPoolExecutor1 {
    public static void main(String[] args) {
        /**
         * 在使用有界队列时， 若有新的任务需要执行，如果线程池实际线程数小于corePoolSize, 则优先创建线程。
         * 若大于corePoolSize, 则会将任务加入队列。
         * 若队列已满， 则在总线程数不大于maxiumuPoolSize的前提下， 创建新的线程。
         * 若线程数大于maximumPoolSize, 则执行拒绝策略， 或其他自定义方法。
         */
        
        
        ThreadPoolExecutor pool = new ThreadPoolExecutor(
                1,            //coreSize
                2,            //MaxSize
                60,         //60
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(3)            //指定一种队列（有界队列）
                //new LinkedBlockingQueue<Runnable>()
                , new MyRejected()
//                , new DiscardOldestPolicy()
                );
        MyTask mt1 = new MyTask(1, "任务1");
        MyTask mt2 = new MyTask(2, "任务2");
        MyTask mt3 = new MyTask(3, "任务3");
        MyTask mt4 = new MyTask(4, "任务4");
        MyTask mt5 = new MyTask(5, "任务5");
        MyTask mt6 = new MyTask(6, "任务6");
        
        pool.execute(mt1);
        pool.execute(mt2);
        pool.execute(mt3);
        pool.execute(mt4);
        pool.execute(mt5);
        pool.execute(mt6);
        
        pool.shutdown();
    }
}


package com.mymuti.lesson16_excutors;
public class MyTask implements Runnable {
      private int taskId;
      private String taskName;
      public int getTaskId() {
            return taskId;
      }
      public void setTaskId(int taskId) {
            this.taskId = taskId;
      }
      public String getTaskName() {
            return taskName;
      }
      public void setTaskName(String taskName) {
            this.taskName = taskName;
      }
      
      public MyTask(int taskId, String taskName) {
            this.taskId = taskId;
            this.taskName = taskName;
      }
      
      
      @Override
      public void run() {
            System.out.println("run taskId = " + this.taskId);
            try {
                  Thread.sleep(5000);
            } catch (InterruptedException e) {
                  // TODO Auto-generated catch block
                  e.printStackTrace();
            }
      }
      
      public String toString() {
            return Integer.toString(this.taskId);
      }
}

package com.mymuti.lesson16_excutors;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
public class MyRejected implements RejectedExecutionHandler {
    
    public MyRejected(){
        
    }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("自定义处理..");
        System.out.println("当前被拒绝的任务为： " + r.toString());
    }
}


package com.mymuti.lesson16_excutors;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
public class UseThreadPoolExecutor2 implements Runnable{
    
    private static AtomicInteger count = new AtomicInteger(0);
    
    @Override
    public void run() {
        int temp = count.incrementAndGet();
        System.out.println("任务" + temp);
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
    
    public static void main(String args[]) throws Exception {
//        System.out.println(Runtime.getRuntime().availableProcessors());
//        BlockingQueue<Runnable> queue = new LinkedBlockingQueue<Runnable>();
        BlockingQueue<Runnable> queue = new ArrayBlockingQueue<Runnable>(10);
        
        ExecutorService executor = new ThreadPoolExecutor(
                5,        //core
                10,     //max
                120L,   //2min
                TimeUnit.SECONDS,
                queue);
        
        for (int i = 0; i < 20; i++ ){
            executor.execute(new UseThreadPoolExecutor2());    
        }
        Thread.sleep(1000);
        System.out.println("queue size: " + queue.size());
        Thread.sleep(2000);
    }
}



