<div id="top"></div>

### 多线程编程

* [基本知识回顾](#mark)

* [怎么理解多线程](#lijie)

* [线程安全](#anquan)

* [出现线程安全的原因](#yuanyin)

* [解决线程安全问题](#jieju)

* [java API 中的线程安全问题](#API)

* [线程间的通信](#tongxin)

* [生产者消费者模式](#sq)

<div id="Mark"></div>

#### 基本知识回顾 

线程是比进程更小的能独立运行的基本单位,他是进程的一部分,一个进程可以有多个进程,但至少有一个线程,即主线程执行(java的 main方法).
我们既可以编写单线程应用也可以编写多线程樱应用.

一个进程中的多个线程可以并发(同步)的去执行,在一些执行时间长,需要等待的任务上(例如:文件读写和网络传输等),多线程就比较有用了.

<div id="lijie"></div>

**怎么理解多线程?**

1.进程就是一个工厂,一个线程就是工厂中的一条生产线,一个工厂至少有一条生产线,只有一条生产线就是单线程应用,拥有多条生产线就是多线程应用
多条生产线可以同时运行.

2.我们使用迅雷可以同时下载多个视频，迅雷就是进程，多个下载任务就是线程，这几个线程可以同时运行去下载视频
  
多线程可以共享内存,充分利用CPU,通过提高资源(内存和CPU)使用率从而提高程序的执行效率.CPU使用抢占式调度模式在多个线程间随机高速的切换,
对于CPU的一个核而言,某个时刻,只执行一个线程,而CPU在多个线程间的切换速度相对我们的感觉要快很多,看上去就像是多个线程或任务在同时运行

java 天生就支持多线程并提供了两种编程方式,一种是继承Thread和实现Runnable接口:

**ThreadFor1**

```java
public class ThreadFor1 extends Thread {

    public void run() {
        for (int i = 0; i < 50; i++) {
            System.out.println(this.getName() + ":" + i);
        }
    }
}
```
**ThreadFor2**

```java
public class ThreadFor2 extends Thread {

    public void run() {
        for (int i = 51; i < 100; i++) {
            System.out.println(this.getName() + ":" + i);
        }
    }
}
```
**测试类**

```
    @Test
    public void testThread() {
        ThreadFor1 for1 = new ThreadFor1();
        for1.setName("线程A");
        ThreadFor2 for2=new ThreadFor2();
        for2.setName("线程B");
        for1.start();
        for2.start();
    }
```
通过继承Thread类,ThreadFor1和ThreadFor2就是线程类,通过测试发现CPU在两个线程之间快速随机切换,也就是我们所说的同时执行


**RunnableFor1**
```java
public class RunnableFor1 implements Runnable {

    public void run() {
        for (int j = 0; j < 50; j++) {
            System.out.println(Thread.currentThread().getName() + ":" + j);
        }
    }
}
```
**RunnableFor2**
```java
public class RunnableFor2 implements Runnable {

    public void run() {
        for (int i = 51; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+":"+i);
        }
    }
}
```
**测试类**

```
@Test
    public void testRunnable() {
        Thread t1 = new Thread(new RunnableFor1());
        t1.setName("线程A");
        Thread t2 = new Thread(new RunnableFor2());
        t2.setName("线程B");
        t1.start();
        t2.start();
    }
```
RunnableFor1和RunnableFor2这个时候是通过实现Runnable类来实现的,这个时候RunnableFor1和RunnableFor2就不能叫线程类了,可以叫任务类

<div id="anquan"></div>

#### 线程安全 

**产生多线程安全问题的原因**
 
在进行多线程编程时,要注意线程安全问题,
 
**SaleWindow** 
 ```java
public class SaleWindow implements Runnable {

    private int id = 10; //表示10张火车票

    public void run() {

        for (int i = 0; i < 10; i++) {
            if (id > 0) {
                System.out.println(Thread.currentThread().getName() + "卖了编号为" + id + "的火车票");
                id--;
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
**测试类**

```
public static void main(String[] args) {
        SaleWindow saleWindow = new SaleWindow();
        Thread t1 = new Thread(saleWindow);
        Thread t2 = new Thread(saleWindow);
        t1.setName("窗口 A");
        t2.setName("窗口 B");
        t1.start();
        t2.start();
    }
```
**测试结果:**

```
窗口 A卖了编号为10的火车票
窗口 B卖了编号为9的火车票
窗口 A卖了编号为8的火车票
窗口 B卖了编号为8的火车票
窗口 A卖了编号为6的火车票
窗口 B卖了编号为5的火车票
窗口 A卖了编号为4的火车票
窗口 B卖了编号为4的火车票
窗口 A卖了编号为2的火车票
窗口 B卖了编号为1的火车票
```

我们看到，10 张火车票都卖出去了，但是出现了重复售票，这就是线程安全问题造成
的。这10 章火车票是共享资源，也就是说任何窗口都可以进行操作和销售，问题在于窗口
A 把某一张火车票卖出去之后，窗口B 并不知道，因为这是两个线程，所以窗口B 也可能会
再卖出去一张相同的火车票。

<div id="yuanyin"></div>

#### 线程安全问题出现的原因 

<img src="https://upload-images.jianshu.io/upload_images/15181329-927d3143d10abb49.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

多个线程操作的是同一个共享资源，但是线程之间是彼此独立、互相隔绝的，因此就会
出现数据（共享资源）不能同步更新的情况，这就是线程安全问题。

<div id="jieju"></div>

#### 解决线程安全问题

java 中提供了一个同步机制(锁)来解决线程安全问题,即让操作共享数据的代码在某一时间段,只被一个线程执行(锁住),
在执行过程中,其他线程不可以参与进来,这样共享数据就能同步了,简单来说,就是给某些代码加把锁.
锁是什么? 又从哪里来?锁的专业名称叫监视器 monitor,其实 java 为每个对象都自带内置了一个锁(监视器 monitor),
当某个线程执行到了某代码快时就会自动得到这个对象的锁,那么其他线程就无法执行该代码块了,一直要等到之前那个线程停止(释放锁),
需要特别注意的是:多个线程必须使用同一把锁(对象).

* 同步代码块:即给代码块上锁,变成同步代码块

* 同步方法: 即给方法上锁,变成同步方法


**方式一:**

```java
public class SaleWindow1 implements Runnable {

    private int id = 10; //表示10张火车票

    public void run() {

        for (int i = 0; i < 10; i++) {
            synchronized (this){
                if (id > 0) {
                    System.out.println(Thread.currentThread().getName() + "卖了编号为" + id + "的火车票");
                    id--;
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}

```
**测试结果**

```
窗口 A卖了编号为10的火车票
窗口 A卖了编号为9的火车票
窗口 A卖了编号为8的火车票
窗口 A卖了编号为7的火车票
窗口 A卖了编号为6的火车票
窗口 A卖了编号为5的火车票
窗口 A卖了编号为4的火车票
窗口 A卖了编号为3的火车票
窗口 A卖了编号为2的火车票
窗口 A卖了编号为1的火车票
```

**方式二:**

```java
public class SaleWindow2 implements Runnable {

    private int id = 10; //表示10张火车票

    public void run() {

        for (int i = 0; i < 10; i++) {

            saleOne();
        }

    }

    private synchronized void saleOne() {
        if (id > 0) {
            System.out.println(Thread.currentThread().getName() + "卖了编号为" + id + "的火车票");
            id--;
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```
**测试结果**

```
窗口 A卖了编号为10的火车票
窗口 A卖了编号为9的火车票
窗口 A卖了编号为8的火车票
窗口 A卖了编号为7的火车票
窗口 A卖了编号为6的火车票
窗口 A卖了编号为5的火车票
窗口 B卖了编号为4的火车票
窗口 B卖了编号为3的火车票
窗口 B卖了编号为2的火车票
窗口 B卖了编号为1的火车票
```

第二种方式是把原来同步代码块中的代码抽取出来放到一个方法中，然后给这个方法加上
synchronized 关键字修饰，锁住的代码是一样的，因此本质上和第一种方式没什么区别。

synchronized的作用就是说进入这个带有synchronized关键字修饰的方法时,这个方法必须要执行完毕你才能离开,通俗的讲就是这块代码
被锁住了,必须执行完你才能执行其他的线程.

<img src="https://upload-images.jianshu.io/upload_images/15181329-fc5b69aee4c114af.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

<div id="API"></div>

#### java API 中的线程安全问题

我们平时在使用Java API 进行编程时，经常遇到说哪个类是线程安全的，哪个类是不保
证线程安全的，例如：StringBuffer / StringBuilder 和Vector / ArrayList ，谁是线程安全的？
谁不是线程安全的？我们查一下它们的源码便可知晓。

**StringBuffer**
<img src="https://upload-images.jianshu.io/upload_images/15181329-afaa5eb8ee01ce11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

**StringBuilder**
<img src="https://upload-images.jianshu.io/upload_images/15181329-991e7be0d66cf27a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

**Vector**

<img src="https://upload-images.jianshu.io/upload_images/15181329-9239de1b638ee1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

**ArrayList**

<img src="https://upload-images.jianshu.io/upload_images/15181329-074e15b032c37788.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">

通过查看源码，我们发现StringBuffer 和Vector 类中的大部分方法都是同步方法，所以证明这两个类在使用时是保证线程安全的；而StringBuilder 和ArrayList 类中的方法都是普通方法，
没有使用synchronized 关键字进行修饰，所以证明这两个类在使用时不保证线程安全。线程安全和性能之间不可兼得，保证线程安全就会损失性能，保证性能就不能满足线程安全。
 
<div id="tongxin"></div> 
 
#### 线程间的通信

多个线程并发执行时,在默认情况下CPU是随机性的在线程之间进行切换的,但是有时候我们希望他们能有规律的执行,那么,多个线程之间就需要一些协调通信来改变或者控制CPU
的随机性,java提供了等待唤醒机制来解决这个问题,具体来说是多个线程依靠一个同步锁,然后借助 wait()和 notify()方法就可以实现线程间的协调通信.
同步锁相当于中间人的作用,多个线程必须用同一个同步锁(认识同一个中间人),只有同一个锁上的被等待的线程，才可以被持有该锁的另一个线程唤醒，使用不同锁的线程之间
不能相互唤醒，也就无法协调通信。                                     
   
<img src="https://upload-images.jianshu.io/upload_images/15181329-1888d5c432510843.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240">                                     

Java 在Object 类中提供了一些方法可以用来实现线程间的协调通:

* public final void wait(); 让当前线程释放锁
* public final native void wait(long timeout); 让当前线程释放锁，并等待xx 毫秒
* public final native void notify(); 唤醒持有同一锁的某个线程
* public final native void notifyAll(); 唤醒持有同一锁的所有线程

需要注意的是：在调用wait 和notify 方法时，当前线程必须已经持有锁，然后才可以调用，否则将会抛出IllegalMonitorStateException 异常。

**ThreadForNum1:**
```java
public class ThreadForNum1 extends Thread {

    public void run() {
        for (int i = 0; i < 11; i++) {
            synchronized (MyLock.o) {
                System.out.println("1");
                MyLock.o.notify();//唤醒另一个线程
                try {
                    MyLock.o.wait();//让自己休眠并释放锁
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
**ThreadForNum2:**
```java
public class ThreadForNum2 extends Thread {

    public void run() {
        for (int i = 0; i < 11; i++) {
            synchronized (MyLock.o) {
                System.out.println("2");
                MyLock.o.notify();//唤醒另一个线程
                try {
                    MyLock.o.wait();//让自己休眠并释放锁
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
**测试结果**
```
1
2
1
2
1
2
1
2
...
```

<div id="sq"></div>

#### 生产者消费者模式

该模式现实生活中很常见,在项目开发中也广泛使用,他是线程间通信的经典应用,生产者是一堆线程,消费者也是一堆线程,
内存缓冲区(容器)可以使用List集合,该模式的关键之处是如何处理多线程之间的协调通信,内存缓冲区为空的时候,消费者必须等待,
而内存缓冲区满的时候,生产者必须等待,一定要保持消费者和生产者的动态平衡.

下面的案例模拟实现农夫采摘水果放到筐里，小孩从筐里拿水果吃，农夫是一个线程，
小孩是一个线程，水果筐放满了，农夫停；水果筐空了，小孩停

**Kuang**

```java
public class Kuang {
//这个集合就是水果筐假设最多存10 个水果
public static ArrayList<String> kuang=new ArrayList<String>();
}
```

上述代码定义一个静态集合作为内存缓冲区用来存储数据，同时这个集合也可以作为锁去被
多个线程使用。

**Farmer**

```java
public class Farmer extends Thread {

    @Override
    public void run() {
        while (true) {
            synchronized (Kuang.kuang) {
                //筐放满了就让农夫休息
                if (Kuang.kuang.size() == 10) {
                    try {
                        Kuang.kuang.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //2 往筐里放水果
                Kuang.kuang.add("apple");
                System.out.println("农夫放了一个水果,目前框里有" + Kuang.kuang.size() + "个水果");
                //3唤醒小孩继续吃
                Kuang.kuang.notify();
            }
            //模拟控制速度
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

上述代码就是农夫线程，不断的往集合（筐）里放水果，当筐满了就停，同时释放锁。

**Child**
```java
public class Child extends Thread {

    @Override
    public void run() {
        while (true) {
            synchronized (Kuang.kuang) {
                //1 框里没有水果就让小孩休息
                if (Kuang.kuang.size() == 0) {
                    try {
                        Kuang.kuang.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                // 2 小孩吃水果
                Kuang.kuang.remove("apple");
                System.out.println("小孩吃了一个水果,目前框里有" + Kuang.kuang.size() + "个水果");
                //唤醒农夫继续放水果
                Kuang.kuang.notify();
            }
            //模拟控制速度
            try {
                Thread.sleep(400);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}
```
上述代码是小孩线程，不断的从集合（筐）里拿水果吃，当筐空了就停，同时释放锁

**测试**
1,当农夫放水果的速度大于小孩吃的速度的时候
```
农夫放了一个水果,目前框里有1个水果
农夫放了一个水果,目前框里有2个水果
农夫放了一个水果,目前框里有3个水果
小孩吃了一个水果,目前框里有2个水果
农夫放了一个水果,目前框里有3个水果
农夫放了一个水果,目前框里有4个水果
农夫放了一个水果,目前框里有5个水果
农夫放了一个水果,目前框里有6个水果
小孩吃了一个水果,目前框里有5个水果
农夫放了一个水果,目前框里有6个水果
农夫放了一个水果,目前框里有7个水果
农夫放了一个水果,目前框里有8个水果
农夫放了一个水果,目前框里有9个水果
小孩吃了一个水果,目前框里有8个水果
```
2,当农夫放的水果的速度小于小孩吃的速度的时候

```
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
小孩吃了一个水果,目前框里有0个水果
农夫放了一个水果,目前框里有1个水果
```
[回到顶部](#top)
                                                  