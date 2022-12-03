并发编程领域抽象成三个核心问题：分工，同步和互斥。

**分工**：把一个功能分给不同的线程共同实现。核心是编程模式。

**同步**：当某个条件不满足时，线程需要等待，当某个条件满足时，线程需要被唤醒执行。管程是解决并发问题的万能钥匙。

**互斥**：保证线程安全，同一时刻，只允许一个线程访问共享变量。核心技术是锁。

  
![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663986664903-72e64cb0-c08d-4f94-a063-60977f278b8c.png)

## 1. 并发编程Bug的源头 - 可见性、原子性和有序性问题

为了平衡CPU、内存和I/O设备三者的速度差异，计算机体系机构、操作系统、编译程序都做了优化，主要为：

1.  CPU增加了缓存，以均衡与内存的速度差异；
2.  操作系统增加了进程、线程，已分时复用CPU，进而均衡CPU与I/O设备的速度差异；
3.  编译程序优化指令执行次序，使得缓存能够得到更加合理地利用；
∂
### 1.1 缓存导致的可见性问题

一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为**可见性**。

单核时代，不同线程操作同一个CPU里面的缓存，不存在可见性问题。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987264067-09886bf4-aab7-4691-9bb1-fcca044fc440.png)

多核时代，每颗CPU都有自己的缓存，不同的线程操作不同CPU的缓存时，对彼此之间就不具备可见性了

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987279237-c741838a-2dd6-4b93-88f3-70edbacb9e17.png)

### 1.2 线程切换带来的原子性问题

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987348748-fc24e38b-281e-420c-9a3b-a61e1279d65c.png)

任务切换的时机大多数是在时间片结束的时候。我们现在使用的高级程序语言一条语句往往对应多条CPU指令，例如语句：count += 1，至少需要三条CPU指令。

-   指令1：首先，需要把变量 count 从内存加载到CPU的寄存器；
-   指令2：之后，在寄存器中执行 +1 操作；
-   指令3：最后，将结果写入内存（缓存机制导致可能写入的是CPU缓存而不是内存）。

操作系统做任务切换，可以发生在任何一条CPU指令执行完。这就可能得到意想不到的结果

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987410123-98bfb71d-20c4-4543-86e5-db239c0537d1.png)

### 1.3 编译优化带来的有序性问题

**有序性**指的是程序按照代码的先后顺序执行。编译器为了优化性能，有时候会改变程序中语句的先后顺序。

双重检查创建单例对象：

```java
public class Singleton {
    static Singleton instance;
    static Singleton getInstance(){
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}

```
instance = new Singleton() 语句经过编译优化重排序后的CPU执行过程可能是：

1.  分配一块内存M；
2.  将M的地址赋值给instance实例；
3.  最后再内存M上初始化Singleton对象。

当线程A执行完指令2时，线程切换B线程调用getInstance方法，获得未初始化的Singleton对象，如果此时访问对象成员变量，那么就可能触发空指针异常。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987614376-ab2eb3a6-aa2d-4c12-88fd-f2d999741f66.png)

## 2. Java内存模型：看Java如何解决可见性和有序性问题

此处为语雀内容卡片，点击链接查看：[https://www.yuque.com/anner-dhcrb/pafw88/1661904379980](https://www.yuque.com/anner-dhcrb/pafw88/1661904379980)

## 3. 互斥锁 ：解决原子性问题

**同一时刻只有一个线程执行**，我们称之为**互斥**。如果能够保证对共享变量的修改是互斥的，那么，无论是单核CPU还是多核CPU，就都能保证原子性了

### 3.1 synchronized

锁是一种通用的技术方案，Java语言提供的synchronized关键字，就是锁的一种实现。synchronized 关键字既可以用来修饰方法，也可以用来修饰代码块

class X {
  // 修饰非静态方法
  synchronized void foo() {
	// 临界区
  }
  // 修饰静态方法
  synchronized static void bar() {
	// 临界区
  }
  // 修饰代码块
  Object obj = new Object()；
  void baz() {
	synchronized(obj) {
  		// 临界区
	}
  }
}  

-   当synchronized 修饰静态方法的时候，锁定的是当前类的class 对象， 在上面的例子中就是 class X；
-   当synchronized 修饰非静态方法的时候，锁定的是当前实例对象this

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663989080790-62b07b85-a027-4bc6-8069-003311d83181.png)

### 3.2 锁和受保护资源的关系

受保护资源和锁之间的关联关系是 N:1 的关系。一个锁可以锁多个资源，但是多个锁不能锁一个资源

class SafeCalc {
  static long value = 0L;
  synchronized long get() {
	return value;
  }
  synchronized static void addOne() {
	value += 1;
  }
}

上面的例子中，get() 方法加的是 this 锁， addOne() 方法加的是 SafeCalc.class 锁，它们都保护 value 资源。但是由于是不同的锁，因此两个临界区就没有互斥关系，就导致了addOne() 方法对value的操作对临界区 get() 没有可见性保证，从而导致并发问题

### 3.3 如何用一把锁保护多个资源？

一个锁来保护多个资源的关键是：锁能覆盖所有受保护资源

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663991690869-5a2e62a7-1312-4598-87b0-cefb54730bb7.png)

当要保护多个资源时，关键要分析多个资源之间的关系。如果资源之间没有关系，那么就每个资源都有一把锁。如果资源之间有关联关系，就要选择一个粒度更大的锁，这个锁应该能够覆盖所有相关的资源。

原子性的本质是多个资源间有一致性的要求，操作的中间状态对外不可见。解决原子性问题，是要保证中间状态对外不可见

## 4. 死锁问题

当下面这四个条件都发生时才会出现死锁：

1.  互斥，共享资源 X 和 Y 只能被一个线程占用；
2.  占有且等待，线程 T1 已经取得共享资源 X，在等待共享资源 Y 的时候，不释放共享资源 X；
3.  不可抢占，其他线程不能强行抢占线程 T1 占有的资源；
4.  循环等待，线程 T1 等待线程 T2占有的资源，线程T2 等待线程 T1 占有的资源，就是循环等待。

只要破坏其中一个条件，就可以避免死锁的发生

#### 破坏占用且等待条件

要破坏这个条件，可以一次性申请所有资源。

#### 破坏不可抢占条件

synchronized如果申请不到资源就会进入阻塞状态，同时线程已经占有的资源也不会释放。但是在 java.util.concurrent包下面提供的Lock是可以轻松解决这个问题。

#### 破坏循环等待条件

破坏这个条件，需要对资源进行排序，然后按序申请资源

## 5. 管程：协作

管程模型有：Hasen 模型、Hoare 模型和 MESA 模型。Java 管程实现用的是 MESA 模型

并发领域的两大核心问题：互斥，即同一时刻只允许一个线程访问共享资源；另一个是同步，即线程之间如何通信、协作。

管程解决互斥问题的思路，就是将共享变量及其对共享变量的操作统一封装起来。同一时刻的操作只允许一个线程进入管程

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663992281615-ec801d66-338e-4012-91d5-e445e6a505db.png)

当一个线程 T1 从入口等待队列中出列，从入口处进入管程中。只有满足某一条件时才能执行操作，这个条件就是条件变量。条件不满足时，就会进入这个条件变量的等待队列等待（通过调用 wait() 方法实现），此时管程是允许其他线程进入的。其他线程 T2 进入了管程，执行了某些操作，导致 T1 线程的条件满足，T2 要通知 T1 条件已经满足（通过调用 notify()/notifyAll() 方法实现）。T1 得到通知后，会从条件变量等待队列中出来，然后进入入口等待队列，等待从新进入管程

### 5.1 wait() 的正确姿势

MESA 管程特有的一个编程范式，就是需要在一个 while 循环里面调用 wait()

while(条件不满足) {
  wait();
}

管程要求同一时刻只允许一个线程执行，当线程 T2 的操作使线程 T1 等待的条件满足时，究竟是哪个线程执行呢。不同的模型有不同的执行策略：

1.  **Hasen 模型里面，要求 notify() 放在代码的最后，这样 T2 通知完 T1 后，T2 就结束了，然后 T1 再执行，这样就能保证同一时刻只有一个线程执行。**
2.  **Hoare 模型里面，T2 通知完 T1 后，T2 阻塞，T1 马上执行；等 T1 执行完，再唤醒 T2 ，也能保证同一时刻只有一个线程执行。但是 T2 多了一次阻塞唤醒操作。**
3.  **MESA 模型里面，T2 通知完 T1 后，T2 还是会继续执行，T1 并不立即执行，仅仅是从条件变量的等待队列进入入口等待队列里面。这样做不好的地方是，当 T1 再次执行时，可能曾经满足的条件，现在已经不满足了。所以需要以循环的方式检验条件变量。**

### 5.2 notify() 何时可以使用

一般情况下尽量使用 notifyAll() 方法，只有满足下面三个条件时，才使用 notify():

1.  所有等待线程有相同的等待条件；
2.  所有等待线程被唤醒后，执行相同的操作；
3.  只需要唤醒一个线程。

## 6. Java线程的生命周期

线程生命周期内的 5 种状态：初始状态、可运行状态、运行状态、休眠状态和终止状态

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663992527676-2545bc1c-9a4c-4651-8c90-c27060268943.png)

1、初始状态，指的是线程已经被创建，但还不允许分配 CPU 执行。这是在编程语言层面被创建，而操作系统层面，真正的线程还没被创建。

2、可运行状态，指的是线程可以分配 CPU 执行。这时真正的操作系统线程已经被成功创建，所以可以分配 CPU 执行。

3、运行状态，指的是有空闲的 CPU 时，操作系统会将其分配给一个可运行状态的线程，这时线程就是运行状态。

4、休眠状态，指的是运行状态的线程调用一个阻塞的 API 或等待某个事件，那么状态就会转为休眠，同时释放 CPU 使用权。当线程被唤醒时进入可运行状态。

5、终止状态，指的是线程执行完或者出现异常，意味着线程生命周期结束。

  
Java 语言中线程共有六种状态：

1.  NEW（初始化状态）
2.  RUNNABLE（可运行/运行状态）
3.  BLOCKED（阻塞状态）
4.  WAITING（无时限等待）
5.  TIMED_WAITING（有时限等待）
6.  TERMINATED（终止状态）

#### 1、 RUNNABLE 与 BLOCKED 的状态转换

当线程等待 synchronized 的隐式锁时，会触发 RUNNABLE 转换成 BLOCKED 状态，当等待线程获得 synchronized 隐式锁时，就又会从 BLOCKED 状态转换为 RUNNABLE 状态。

线程调用阻塞式 API 时，操作系统层面线程会转换到休眠状态。而在 jvm 层面并不关心操作系统调度相关的状态，其状态还是 RUNNABLE。

#### 2、RUNNABLE 与 WAITING 的状态转换

第一种，获得 synchronized 隐式锁的线程，调用无参数的 Object.wait() 方法。

第二种，调用无参的 Thread.join() 方法。join() 是一种线程同步方法， B 线程中调用 A.join() ，那么 B 线程将会进入 WAITING 状态，当 A 线程执行完后，B 线程又转为 RUNNABLE 状态。

第三种，调用 LockSupport.park() 方法。Java 并发包中的锁，都是基于它实现的。调用 LockSupport.park() 方法，当前线程会进入 WAITING 状态；调用 LockSupport.unpark(Thread thread) 方法，会唤醒目标线程。

#### 3、RUNNABLE 与 TIMED_WAITING 的状态转换

1.  调用 Thread.sleep(long millis) 方法；
2.  获得 synchronized 隐式锁的线程，调用 Object.wait(long timeout) 方法；
3.  调用 Thread.join(long millis) 方法；
4.  调用 LockSupport.parkNanos(Object blocker, long deadline) 方法；
5.  调用 LockSupprot.parkUntil(long deadline) 方法；

#### 4、从 NEW 到 RUNNABLE 状态

Java 新创建的 Thread 对象是处于 NEW 状态，线程对象调用 start() 方法就会转到 RUNNABLE 状态。

创建和启动多线程（thread.start()）

-   扩展Thread类
-   实现Runable接口
-   实现Callable接口

#### 5、从 RUNNABLE 到 TERMINATED 状态

线程执行完 run() 方法后，会自动转换到 TERMINATED 状态。如果执行过程中异常抛出，也会导致线程终止。强制线程终止调用 interrupt() 方法。

  

**stop() 和 interrupt() 方法的区别**

stop() 方法会立即杀死线程，如果线程持有 ReentrantLock 锁，是不会调用 ReentrantLock 的 unlock() 去释放锁的，因此其他线程也就没有机会获得 ReentrantLock 锁，这是很危险的。类似的还有 suspend() 方法和 resume() 方法。

interrupt() 方法仅仅是通知线程中断，被通知的线程有机会执行一些后续的操作，同时也可以无视这个通知。收到通知的方式有两种：一种是异常，另一种是主动检测。

当线程 A 处于 WAITING、TIMED_WAITING 状态时，其他线程调用 A.interrupt() 方法，会是线程 A 转到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常。

当线程 A 处于 RUNNABLE 状态时，并且阻塞在 Java.nio.channels.InterruptibleChannel 上时，如果其他线程调用 A.interrupt() ，A 会触发 ClosedByInterruptException 异常；如果阻塞在 Selector 上，会立即返回。

如果线程处于 RUNNABLE 状态，并且没有阻塞在某个 I/O 操作上，中断就要依赖线程主动检测中断状态了，通过调用 isInterrupted() 方法。

  

为什么局部变量是线程安全的？

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993102797-982b7ce7-d280-452a-b586-45a960da679a.png)

每个方法在调用栈里都有自己的独立空间，称为栈帧。每个栈帧里都有对应方法需要的参数和返回地址。当调用方法时，会创建新的栈帧，并压入调用栈；当方法返回时，对应的栈帧就会被自动弹出。也就是说栈帧和方法是同生共死的。方法的调用是利用栈结构解决的

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993146186-e404a399-6783-4411-a05c-8b53f37bfd97.png)

局部变量放到了调用栈里

## 7. 创建多少线程才是合适的

程序大致分为 CPU 密集型计算和 I/O 密集型计算

对于 CPU 密集型的计算场景，理论上“线程的数量=CPU 核数”就是最合适的。

在工程上，一般会设置为“CPU 核数 + 1”。这样的话，当线程因为偶尔的内存页失效或者其他原因导致阻塞时，额外的线程可以用上，保证CPU的利用率。如果在增加线程也只是增加线程切换的成本

对应 I/O 密集型操作，最佳的线程数与程序中 CPU 计算和 I/O 操作的耗时比相关

-   单核 CPU 公式： 最佳线程数 = 1 + (I/O 耗时 / CPU 耗时)。
-   多核 CPU 公式： 最佳线程数 = CPU 核数 * [1 + (I/O 耗时 / CPU 耗时)]

[https://www.zhihu.com/question/27994350](https://www.zhihu.com/question/27994350)

  

## 8.同步容器及其注意事项

ava 中的容器主要分为四大类：List、Map、Set 和 Queue。这里面并不是所有容器都是线程安全的，我们可以通过把非线程安全的容器封装在对象内部，然后控制访问路径就能达到线程安全的目的

Java 在 1.5 之前的线程安全的容器，主要指的是同步容器，所有方法使用 synchronized 来保证互斥，性能差。1.5 之后提供了性能更改的容器，称之为并发容器

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993482302-e0340f89-f49b-4359-af21-01ff28f49323.png)

### 一、List

CopyOnWriteArrayList 就是写的时候将共享变量新复制一份出来，这样就可以读操作完全无锁。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993552836-c91619c0-c2a0-4ac7-94a6-03e3308e7104.png)

CopyOnWriteArrayList 仅适用于写操作非常少的场景，而且能容忍读写的短暂不一致，新写的元素不能立刻被遍历到。CopyOnWriteArrayList 迭代器是只读的，不支持增删改。

### 二、Map

ConcurrentHashMap 的 key是无序的，而 ConcurrentSkipListMap 的 key 是有序的。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993584532-81b6099f-15f4-40e5-8179-9ae6a42c9199.png)

ConcurrentSkipListMap 里面的 SkipList 数据结构称之为“跳表”，跳表插入、删除、查询操作平均的时间复杂度是 O(log n)。理论上和并发线程数无关，对性能要求更高的场景下，可以尝试使用 ConcurrentSkipListMap

### 三、Set

内容同上。

### 四、Queue

可以从两个维度分类 Queue 并发容器。一个维度是阻塞与非阻塞，阻塞指的是当队列已满时，入队操作阻塞；当队列为空时，出队操作阻塞。另一个维度是单端与双端，单端指的是只能队尾入队，队首出队；双端指的是队首队尾皆可入队出队。

阻塞队列都用 Blocking 关键字标识，单端队列使用 Queue 标识，双端队列使用 Deque 标识

#### 1、单端阻塞队列：

ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、LinkedTransferQueue、PriorityBlockingQueue 和 DelayQueue

SynchronousQueue 的生产者线程的入队操作必须等待消费者线程的出队操作。LinkedTransferQueue 性能要比 LinkedBlockingQueue 好；PriorityBlockingQueue 支持按优先级出队；DelayQueue 支持延时出队。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993674758-71538967-0317-44ab-8e40-3f2599fea71d.png)

#### 2、双端阻塞队列：LinkedBlockingDeque

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993697622-e1706e94-f8be-42df-a1c1-2fc25e07dfd0.png)

#### 3、单端非阻塞队列：ConcurrentLinkedQueue

#### 4、双端非阻塞队列：ConcurrentLinkedDuque

在使用队列时也要注意容器界限问题，如果容量没有界限可能会到时 OOM

## 9. 原子类：无锁工具类的典范

### 9.1 无锁方案的实现原理

CPU 为了解决并发问题，提供了 CAS 指令（Compare And Swap）。作为一条 CPU 指令，CAS 指令本身是能够保证原子性的。

使用 CAS 解决并发问题，一般都会伴有自旋，也就是循环尝试。使用 CAS 方案时注意 ABA 问题，Java中提供了AtomicStampedReference和AtomicMarkableReference来解决ABA问题，内部有维护一个版本号，每次修改都同时修改版本号

do {
  // 获取当前值
  oldV = xxxx；
  // 根据当前值计算新值
  newV = ...oldV...
}while(!compareAndSet(oldV,newV);

### 9.2 原子类概览

Java SDK 并发包里的原子类分为五个类别：原子化的基本数据类型、原子化的对象引用类型、原子化数组、原子化对象属性更新器和原子化的累加器。

![](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993821497-8c46b344-adc0-41cd-9c1f-c27841ad6b65.png)

#### 1、原子化的基本数据类型

AtomicBoolean、AtomicInteger 和 AtomicLong

getAndIncrement() //原子化i++
getAndDecrement() //原子化的i--
incrementAndGet() //原子化的++i
decrementAndGet() //原子化的--i
//当前值+=delta，返回+=前的值
getAndAdd(delta)
//当前值+=delta，返回+=后的值
addAndGet(delta)
//CAS操作，返回是否成功
compareAndSet(expect, update)
//以下四个方法
//新值可以通过传入func函数来计算
getAndUpdate(func)
updateAndGet(func)
getAndAccumulate(x,func)
accumulateAndGet(x,func)

#### 2、原子化的对象引用类型

AtomicReference、AtomicStampedReference 和 AtomicMarkableReference，利用它们可以实现对象引用的原子化更新。注意AtomicReference 有ABA 问题，其他两个可以解决 ABA 问题。其原理是增加版本号。

#### 3、原子化数组

AtomicIntegerArray、AtomicLongArray 和 AtomicReferenceArray，利用这些原子类，可以原子化地更新数组里面的每一个元素。

#### 4、原子化对象属性更新器

AtomicIntegerFieldUpdater、AtomicLongFieldUpdater 和 AtomicReferenceFieldUpdater，利用它们可以原子化地更新对象的属性。都是通过反射机制实现的。

对象属性必须是 volatile 类型的，只有这样才能保证可见性

#### 5、原子化的累加器

DoubleAccumulator、DoubleAdder、LongAccumulator 和 LongAdder，仅仅用来执行累加操作，速度更快，但不支持 compareAndSet() 方法

### 9.3 总结

无锁方案相比较互斥锁方案，首先性能好，其次是基本不会出现死锁问题（但可能出现饥饿和活锁问题，因为自旋会反复重试）