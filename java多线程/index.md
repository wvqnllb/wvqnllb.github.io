# Java多线程

Java多线程基础

## 线程 进程 多线程
* 进程指的是占用一定内存的程序 资源分配的的单位
* 线程是进程中的一个执行单元 负责当前进程中的程序的执行
* java的多线程 指的是 java调用系统资源创建多线程 start0();
* java默认两线程 main线程 GC线程
* 并发编程 充分利用CPU资源

## 线程基本方法和状态
* 线程状态
  * 新建 就绪 运行 阻塞 死亡
* static yield() 线程礼让
  * 暂停但是不阻塞 给一个重新竞争的机会
* 线程停止
  * 建议使用一个标志位进行终止变量 在runnable里搞一个flag成员变量
* 线程休眠 sleep()
  * 阻塞线程 可以指定时间
* join() 线程强制执行 而且要执行完 其他线程都阻塞
* 守护线程daemon setDaemon();
  * 虚拟机必须确保用户线程执行完毕 不用太管守护线程
  * 后才记录操作日志 监控内存 垃圾回收
* wait和sleep的区别
  * Object#wait() 释放锁 只能在同步代码块中
  * Thread#sleep() 抱着锁睡觉 可以在任何地方睡

## 创建方式

```java
//继承Thread类 重写run()方法
//匿名类
public class ThreadDemo1 extends Thread {
    @Override
    public void run() {
        sout("Thread is running");
    }
}

//实现Runnalbe接口 实现run方法
public class ThreadDemo2 implements Runnable {
    public void run(){
        sout("xxx");
    }
}

//线程池+Callable
ExecutorService executorService = Executors.newSingleThreadExecutor();
Future<String> submit = executorService.submit(()-> {
    sout(Thread.currentThread().getName() + "------>正在执行");
    Thread.sleep(3*1000L);
    return "success"; // 可以看到call是有返回值的 泛型接口
});
String result = submit.get();
sout("result------->" + result);
executorService.shutdown();


ThreadDemo1 t1 = new ThreadDemo1();
t.start();

ThreadDemo2 target = new ThreadDemo2();
Thread t2 = new Thread(target);
t2.start();
```
* 区别和本质
    * 之所以有Runnable接口
        * 限定Thread构造方法的形参类型
        * 将run()向上抽取 让实现类去重写
    * 好处？
        * 有点解耦的味道 把原本线程类中的"待执行代码"挪到了Runnable实现类中
        * 执行者和被执行者分离了
        * 继承Thread的话 线程和待运行的代码在同一个类中 无法做到资源独立 也就无法共享

## 线程池
* 如何确认线程数量
  * cpu密集型 cpu内核数 可以保证cpu效率最高 
  * IO密集型 15个大型任务 IO十分占用资源 应该采用大于IO数量的线程

* 管理线程 
    * 避免增加创建线程和销毁线程的资源损耗 线程也是对象
    * 提高响应速度
    * 方便管理
    * 最终目的还是去分配不同线程执行任务
* 工具类Executors 线程池的工厂类 用于创建并返回不同类型的线程池
    * newSingleThreadExecutor 返回一个ExecutorService线程池
        * #execute(()->())
        * 用完要关闭 finally #shutdown()
    * newFixedThreadPool
    * newCachedThreadPool 可伸缩的

* 祖宗Executor接口 解耦执行器Executor#execute(Runnable runnable)和具体方法
* ThreadPoolExecutor (实现ExecutorService接口) 最好直接用这个创建(Ali规范)
    * corePoolSize 线程池中维持的线程数量 当线程池中的线程数量超过corePoolSize数量时 闲置线程将被终止
    * maximumPoolSize 线程池中最大允许的线程数量 队列满了之后会都打开
    * keepAliveTime 当线程池中的线程数量大于线程核心池（corePoolSize）允许的数量时，在终止空闲线程前允许等待新任务的时间。
    * Unit 这个就是keepAliveTime的单位，秒、毫秒、纳秒等
    * workQueue 阻塞队列该队列保存未被执行的任务。其中的任务类型是Runnable的
    * ThreadFactory 创建线程的工厂类 比如给线程设置名字
    * handler 拒绝策略类，当线程池数量达到上线并且workQueue队列长度达到上限时就需要对到来的任务做拒绝处理。默认的情况下是当以上情况发生时，抛出一个RejectedExecutionException异常
      * 四种拒绝策略
      * 最大承载 线程数量+队列长度
      * AbortPolicy 就不处理这个人 并且抛出异常
      * CallerRunsPolicy 哪来的去哪里 比如是main线程来的 那就让main线程来执行
      * DiscardPolicy 丢了 不抛出异常
      * DiscardOldPolicy 尝试和最早的竞争 不会抛出异常

## Synchronized
* [link](https://zhuanlan.zhihu.com/p/57482990)初级
* [link](https://zhuanlan.zhihu.com/p/29866981)中级
* 同步关键字
* 真正的锁是某一个对象 对象头里包含了锁信息
* 静态方法的锁是所在类的字节码对象 普通方法的锁是this对象！~~
* synchronized是可重入的
    * 同一个线程在已经持有该锁的情况下 可以再次获取锁 并会在某个状态量上做+1的操作
    * 子类同步方法 调用父类的同步方法嘛 可以 用的都是this锁对象
    * 静态同步和非静态同步方法不互斥
    * 看八锁的例子就好了
```java
//写一个固定容量的同步容器，拥有put和get方法，以及getCount方法，能够支持2个生产者线程以及10个消费者线程的阻塞调用
```
* object的方法 notify notifyall 和 reetrantlock的await和singalAll()区别
* 1.6之后的优化：
  * 1.6之前都是重量锁 mutex 操作系统切换内核态 阻塞和恢复浪费资源量大
  * 1.6之后
    * 无锁
    * 偏向锁
      * 单一线程执行同步代码块 提高性能
      * 获取锁：cas获锁之后在对象头和栈帧记录中存储锁偏向的进程ID， 之后线程在进入和退出块时不需要进行cas操作来加锁解锁，只简单测试下对象头的markword中偏向锁的id字段
      * 竞争出现：检测当前对象是不是可偏向的，再确认线程id，如果通过CAS竞争成功，换对象偏向锁字段的线程id。如果竞争失败，说明出现了竞争，到全局安全点的时候，首先暂定拥有偏向锁的线程。检查该线程是否还活着，如果已经执行完毕了，由于不会主动释放锁，将对象头设置成无锁，然后重新偏向。如果还活着，撤销偏向锁之后 升级到轻量锁的状态。
      * 释放：采用了一种等到竞争出现才释放偏向锁的机制。
      * -XX:-UseBiasedLocking=false 用于关闭
    * 轻量锁
      * 和偏向锁一样都是乐观锁，其实是牺牲了一部分自旋的资源来交换阻塞和激活所用的资源。
    * 重量锁
    * 其他优化：
      * 锁消除 如果堆上的数据不会逃逸出当前线程 可以认为是安全的 不加锁
      * 锁粗化 连续对同一对象反复加锁解锁 可以统一加一次
      * -XX:PreBlcokSpin 默认的自旋次数


## ReetrantLock手撕AQS
* [link](https://zhuanlan.zhihu.com/p/54297968)
* AQS AbstractQueuedSynchronizer JDK为线程同步提供的一套基础工具类
* 理解主要方法
```java
//加锁
public void lock() {
    sync.lock();
}
//解锁
public void unlock(){
    sync.release(1);
}
```
* sync
    * ReetrantLock内部类 均继承于抽象类Sync
    * Sync继承于AQS
      * AQS中定义了内部类Node
    * NonFairSync
        * 直接抢 线程饥饿
    * FairSync
        * FairSync在tryAquire方法中，当判断到锁状态字段state == 0 时，不会立马将当前线程设置为该锁的占用线程，而是去判断是在此线程之前是否有其他线程在等待这个锁（执行hasQueuedPredecessors()方法），如果是的话，则该线程会加入到等待队列中，进行排队（FIFO，先进先出的排队形式）。这也就是为什么FairSync可以让线程之间公平获得该锁。
    * 线程排队
        * tryAcquire为false 获锁失败的时候 执行acquireQueued方法
        * 队列对线程进行排队和管理
            * 线程 队列中线程状态 前驱和后继线程
            * Node的数据结构 内部类
                * 两种等待方式 SHARED 线程以共享的模式等待锁 ReadLock EXCLUSIVE 以互斥的模式等待锁 ReentrantLock
                * 状态 
                    * CANCELLED 1 获锁请求取消 
                    * SIGNAL -1 线程早准备好的一切等待锁 
                    * CONDITION -2 线程等在某一个条件被满足
                    * PROPAGATE -3 当处SHARED模式的时候才会被用上
                * 成员变量
                    * waitStatus 该int变量表示线程在队列中的状态
                    * prev 该变量类型为Node对象，表示该节点的前一个Node节点
                    * next 该变量类型为Node对象，表示该节点的后一个Node节点
                    * thread 该变量类型为Thread对象，表示该节点的代表的线程
                    * nextWaiter 该变量类型为Node对象，表示等待condition条件的Node节点
    * 等待中的线程如何感知到锁空闲并且获得锁
    * 下面这些都是AQS中的方法
        * acquiredQueued(addWaiter(Node.EXCLUSIVE),arg)中的外层方法
        * 一个线程获取锁失败了 被放到了线程的等待队列中 把放入队列中的这个线程不断的进行获锁 直到成功获锁 或者 不再需要
        * 一个while循环 跳出条件 p是head节点，并且当前线程获锁成功
        * 如果就是那么不恰巧,就是不符合这个唯一跳出循环的条件”,那就一直在循环里面空跑了吗!那CPU使用率不就会飙升?!
            * 阻塞线程 第一次循环 线程状态被设置成了signal 第二次就会被阻塞
* 解锁
  * release(1);
  * unparkSuccessor(h)就是唤醒操作
  * 尝试释放当前线程持有的锁 如果成功释放，那么去唤醒头结点的后继节点 头节点不保存线程信息
* 可重入
  * 这里需要注意到AQS中很多内部变量的修饰符都是采用的volitale,然后配合CAS操作来保证AQS本身的线程安全(因为AQS自己线程安全,基于它的衍生类才能更好地保证线程安全),这里的state字段就是AQS类中的一个用volitale修饰的int变量
  * state字段初始化时,值为0。表示目前没有任何线程持有该锁。当一个线程每次获得该锁时，值就会在原来的基础上加1，多次获锁就会多次加1（指同一个线程），这里就是可重入。因为可以同一个线程多次获锁，只是对这个字段的值在原来基础上加1; 相反unlock操作也就是解锁操作，实际是是调用AQS的release操作，而每执行一次这个操作，就会对state字段在原来的基础上减1，当state==0的时候就表示当前线程已经完全释放了该锁。



## Synchronized 和 Lock的区别

* 内置的java关键字 和 java类
* 无法判断获取锁的状态 可以获得并判断是否获取到了锁
* 自动释放 手动释放
* 如果当前方法阻塞了 其他没法获得锁 Lock锁可以tryLock 
* 不可以中断 非公平  可以中断锁 非公平/公平
* 对象上重入         线程上重入
* 适合锁少量的代码同步 lock大量
* Condition的优势 精准的通知和唤醒线程

## FutureTask = 任务 + 结果

### 继承体系
* -->RunnableFuture--->Future & Runnable
* 能包装Runnable和Callable(构造器传入) 但本身又实现了Runnable接口 本质是Runnable
* 能作为任被Thread执行，但诡异的是FutureTask#get()可以获取结果
* 一个场景
    * 调用时必须能够立即返回result(非阻塞)
    * 拿到result之后 可以在随后任意时刻通过result.get()获取或是任务的最终结果(异步结果)
### Callable
```java
public interface Callable<V> {
　　V call() throws Exception;
}
//对比
interface Runnable {
　　public abstract void run();
}
```
* Callable能接受一个泛型，然后在call方法中返回一个这个类型的值。而Runnable的run方法没有返回值
* Callable的call方法可以抛出异常，而Runnable的run方法不会抛出异常。
### 简单例子
```java
public class AsyncAndWaitTest {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        // FutureTask实现了Runnable，可以看做是一个任务
        FutureTask<String> futureTask = new FutureTask<>(new Callable<String>() {
            @Override
            public String call() throws Exception {
                System.out.println(Thread.currentThread().getName() + "========>正在执行");
                try {
                    Thread.sleep(3 * 1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "success";
            }
        });
        
        System.out.println(Thread.currentThread().getName() + "========>启动任务");

        // 传入futureTask，启动线程执行任务
        new Thread(futureTask).start();

        // 获取执行结果（会阻塞3秒）
        String result = futureTask.get();
        System.out.println("任务执行结束，result====>" + result);
    }

}
```

### 如何包装Runnable Callable
* 里面只有一个callable的成员 如果传入runnable的话 先通过内部将Runnable包装成callable
* Runnable --> Executors.callable() --> RunnableAdapter implements Callable --> FutureTask.callable 适配器模式 这里搞了个假的输出参数 反正就大概是原封不动的过一遍流程

### 如何被Thread执行
* 由于本身就是一个Runnable 所以进去的流程基本一样 然后运行target.run();
* FutureTask#run()方法中
    * 取出之前包装的任务 即成员callable
    * 调用call方法 返回result(形式上的)或者实际的执行结果
    * 把结果设置到某个地方 FutureTask#set() 存到一个成员变量当中
    * 所以FutureTask = 任务 + 结果
### 为什么get是阻塞的
```java
// 获取执行结果（会阻塞3秒）
String result = futureTask.get();
```
* FutureTask中定义了很多任务状态
    * 刚创建
    * 即将完成
    * 完成
    * 抛异常
    * 任务取消
    * 任务将被打断
    * 任务被打断
* 阻塞的意义
    * 如果获取结果的这个线程不阻塞的话 很有可能尚未执行完耗时任务 比如io 因此会获得空的result
* 如何做到
    * get里面有一个awaitdone 挺麻烦的
    * forloop + LockSupport 

## 集合类的安全问题

并发修改异常

* CopyOnWriteArrayList<>
  * 并发修改异常
  * 适用于多读取少添加的时候
  * 写入时复制  优势 写入时可以读 全写入还是要加锁 全用关键字方法的话就效率更低
  * 多个线程调用的时候 读取的是固定的 写入的时候会覆盖
  * 在写入的时候避免覆盖造成数据问题
  * 读写分离

* CopyOnWriteArraySet<>
* ConcurrentHashMap



## 常用的辅助类

* CountDownLatch
  * 允许一个或多个线程等待 知道在其他线程中执行的一组操作完成的同步辅助
  * 必须要执行任务的时候再使用
  * 一个减法计数器
* CyclicBarrier
  * 构造参数 int Runnable
  * 允许一组线程全部等待彼此到达共同屏障点的同步辅助
  * 加法计数器
  * 集齐七颗龙珠召唤神龙
  * await()
* Semaphore
  * 一个计数信号量。在概念上信号量维持一组许可证
  * acquire() 得到
  * release() 释放
  * 同一时间只能有指定数量个得到线程

## 阻塞队列

![image-20201101120707180](C:\Users\wvqnllb\AppData\Roaming\Typora\typora-user-images\image-20201101120707180.png)

* 如果队列满了 就必须阻塞等待
* 如果队列是空的 必须阻塞等待生产
* 实现类 同步队列 数组阻塞队列 链表阻塞队列
* 四组API：
  * 抛出异常
    * add 	 remove	 element
  * 不会抛出异常
    * offer	poll 	peek
  * 阻塞等待 一直阻塞
    * put	take 	
  * 超时等待 
    * offer("a", 2, TimeUnit.Seconds); 超过两秒就退出
    * poll(2, TimeUnit.Seconds);
* 同步队列 SynchronousQueue 一种实现类
  * 不存储元素 
  * put了一个元素 就必须先取出来

## 各种锁

### 读写锁

* Interface ReadWriteLock 和Lock同级
* 实现ReentrantReadWriteLock()
  * 维护一堆关联的Locks 一个用于只读操作 一个用于写入
* 独占锁 写锁  一次只能被一个线程占有 共享锁 读锁 多个线程可以同时占有
* 读-读 共存 读-写 不能共存 写-写 不能共存
* 了解底层实现 尤其是共享锁

## JMM

Java内存模型 是一种概念和约定

* 主内存和工作内存

* 线程解锁前 必须把共享变量立即刷回主存
* 线程加锁前必须主存中最新值到工作内存中
* 加锁和解锁是同一把锁

8种操作

* read + load
* use + assign
* store+write
* lock + unlock

## Volatile解析

* 和可见性与有序性有关
* 当多线程环境同时保证了对共享资源的互斥和可见性时 就能保证线程安全
* 原子性 一个步骤或多个步骤 不可被拆分 流程不可被中断 可以使用原子类
* 可见性 一个线程对共享资源的修改 对其他线程来说是可见的
* 互斥访问 同一时间只允许同一个线程访问
* 互斥访问的代码块 称为临界区
* volatile保证可见性
  * 当线程A和线程B读取了主存上的值v，同时存一个副本在线程内部的本地内存MA和本地内存MB，当线程A对MA内的副本进行修改后，则置本地副本的变量所在的内存为M（modify）标记，同时因为本地内存MB也有该副本，这个时候本地内存MB的该变量的所在内存被标记为I（Invalid）标记，当线程B对本地内存MB内的共享数据进行读取时，发现标记无效，则使本地内存MA上modify的数据写回主存，然后线程B再从内存中读取该值进行操作（也可能是直接到MA中读取）。为什么保证可见性仍然不能保证形成安全就在于，当线程A和线程B同时对本地内存进行操作时，因为这个时候两个线程内的数据都是原来的数据，双方已经一致，因此也可以同时进行操作，操作后同时提交到总线上进行cache之间的同步，这个时候就该协议就会失效。
* volatile保证有序性
  * 禁止重排序

* 应用
  * 状态量标记
  * 单例模式的实现

## 单例模式

* 饿汉式单例

  * 浪费内存 static对象

* 懒汉式单例

  * 就是在getInstance里面构建

  * 单线程下单例ok

  * 并发下

  * 双重检测锁模式

    * DCL懒汉式

      ```java
      public static LazyMan getInstance() {
          if (lazyMan == null) {
              synchronized (LazyMan.class) {
                  if (lazyMan == null) {
                      lazyMan = new LazyMan();
                     	/** 不是原子性操作
                     	* 1 分配内存空间
                     	* 2 执行构造方法
                     	* 3 把这个对象指向这个空间
                     	* 如果线程A 132
                     	* 线程B会判断不等于null 然后就获得不到单例对象
                     	*/
                  }
              }
          }
          return lazyMan;
      }
      //所以一定要加volatile
      ```

      

    * 还是可以用反射破坏单例

      * 构造器上再加锁
      * 信号灯
      * 枚举

## 深入理解CAS

* CAS: 比较并交换 本省只有成功和失败两种状态

* getAndIncrement:不会陷入无限循环 因为会一直从内存中读出当前的值

[link](https://www.cnblogs.com/keiyoumi520/p/13410253.html)

* 内存位置的内容和给定值进行比较 只有在相同的情况下 将给内存位置的内容修改为新给定的值,这是作为单个原子操作完成的.

* Java无法操作内存  unsafe 可以调用C++ native
  * 可以通过这个类操作内存

* 比较当前工作内存的值和主内存中的值 如果这个值是期望的 那么则执行操作 如果不是一直循环

  * 循环会耗时
  * 一次性只能保证一个共享变量的原子性
  * ABA问题

* 自旋锁

  ```java
  public final int getAndAddInt(Object var1, long var2, int var4) {
      int var5;
      do {
          var5 = this.getIntVolatile(var1, var2);
      } while (!this.compareAndSwapInt(var1, var2, var5, var5+var4))
  }
  ```

* ABA问题

  * 另一个线程换过但是不知道

* 原子引用

  * AutomicStampedReference
  * 初始化参数 初始值 + 时间戳
  * 解决ABA问题
  * 注意使用Integer时 如果超过-128 - 127的情况下 会重新构建对象
    * Integer是在Integer Cache.cache产生 会复用
    * 这个区间外的所有数据 都不会复用
  * 乐观锁

## 各种锁的理解

* 公平锁 非公平锁
  * 先来后到 和 竞争
  * 默认都是非公平
  * Lock 
* 可重入锁
  * 递归锁
  * synchronized 和 ReentrantLock的区别
    * 锁一次
    * 锁两次 锁必须配对 不然就会死在里面

* 自旋锁 SpinLock

  * getAndIncrement

  * 
  ```java
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    ```

  * 
  ```java
    AutomicReference<Thread> automicReference = new AutomicReference();
    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread.getName()+"myLock");
        //自旋锁
        while (automicReference.compareAndSet(null, thread)) {
        }
    }
    public void myUnLock() {
        Thread = Thread.currentThread();
        System.out.printlin(Thread.currentThread.getName()+"UnLock");
        atomicReference.compareAndSet(thread, null);
    }
    ```

* 死锁

  * 是什么？ 
    * ![image-20201102104844792](C:\Users\wvqnllb\AppData\Roaming\Typora\typora-user-images\image-20201102104844792.png)
  * 怎么排除死锁
    * 使用jps-l定位进程号
    * jstack进程号 找到死锁
  * 四要素
    * 互斥条件
      * 一个资源每次只能被一个进程使用
    * 请求和保持条件
      * 一个进程因请求资源而阻塞时 对已获得的资源保持不放
    * 不剥夺条件
      * 进程已经获得的资源 未使用完之前 不能强行剥夺
    * 循环等待条件
      * 若干进程之间形成一种头尾相接的循环等待资源关系
























