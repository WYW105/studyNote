# Java并发补充

1. 临界区Critical Section = 访问共享资源的代码区域。

2. 锁只保护临界区，锁范围越小越好，非必要的东西不要写在锁里（比如写log日志）。

## Java中的锁的特性

### 互斥性 Mutual Exclusion

1. 同一时刻只能有一个线程进入临界区进行操作，其他线程必须等待。因为**锁的本质是锁资源**。

### 可重入 Reentrant

1. 同一个线程已经拿到了某个锁，再次请求的时候不需要重新竞争，可以直接获得。

2. synchronized就是可重入锁

   ```java
    public synchronized void a() {

        System.out.println("进入A");

        b();

    }

    public synchronized void b() {
        // 要进入b()就要再次申请同一把锁，完全没问题。
        System.out.println("进入B");

    }
   ```

3. JVM会记录线程重入的次数，重入次数为0时就是释放了锁。

4. **原始Redis分布式锁（SET order:1 abc123 NX EX
   30）不可重入**，因为redis不会判断再次申请的是不是同一个线程。
5. Redisson的RLock是可重入锁，结构可以简化如下：

   ```
    lock:order:1

    {
      // UUID是客户端id + 线程idThreadId + 锁的重入次数
       UUID:ThreadId : count
    }
   ```

### 公平锁 Fair Lock 和 非公平锁

1. 公平锁 = 按照请求锁的顺序获得锁，先请求先获得。为了实现公平锁，通常会维护等待队列。
2. 公平锁不一定比非公平锁好，因为**非公平锁通常性能更高、吞吐量更高**（**比如排在第三位的线程C此时刚好运行在CPU上，那他就可以直接占有锁、少一次线程切换**）
3. synchronized是非公平锁
4. new ReentrantLock()和new
   ReentrantLock(false) 即ReentrantLock默认是非公平锁，new
   ReentrantLock(true)是公平锁。

5. **Redis
   Lua脚本实现库存扣减时**，多个请求按照到达Redis服务器的顺序串行执行（到达顺序不一定和请求发出顺序一样，因为有网络延迟），因此**不属于公平锁或非公平锁的范畴**，因为它就**不是锁机制**，是**单线程事件循环模型**。

6. **Redis的分布式锁（SET lock token NX EX
   30）抢锁的过程也是redis单线程事件循环模型，是非公平锁**，因为不能保证是按照请求锁的顺序获得锁，哪个命令先到达Redis服务器哪个就先执行。

7. Redisson的 的RLock是非公平锁，RFairLock是公平锁

   ```java
   RFairLock lock = redissonClient.getFairLock("order");
   ```

### 可中断 （lock Interruptibly）

1. 可中断就是线程A在等待锁的时候，锁如果一直被其他线程占有，线程A抢不到锁，可以中断等待，而不是必须等到锁才行。
2. synchronized不是可中断锁；ReentrantLock有维护等待队列，因此是可中断锁；Redis锁（SET
   lock token NX EX
   30）失败就会立刻放弃不会等待，所以谈不上中不中断；Redisson支持lockInterruptibly()，可中断并抛出InterruptedException。

3. 可中断锁 是线程在等待锁的时候，受到了其他线程（通常是管理线程，比如线程池发送信号给具体的执行线程）发出的中断interrupt信号，线程被设置了中断标记但是不会停止，**如果线程阻塞在支持被中断的方法上如线程的sleep()、wait()、join()方法 或者 ReentrantLock的lockInterruptibly()方法 Redisson的lockInterruptibly()方法 ，就会中断然后抛出InterruptedException**。

   ```java
    import java.util.concurrent.locks.ReentrantLock;

    public class InterruptibleLockDemo {

        private static final ReentrantLock lock = new ReentrantLock();

        public static void main(String[] args) throws InterruptedException {

            Thread threadA = new Thread(() -> {
                lock.lock();
                try {
                    System.out.println("线程A获得锁，开始长时间处理");

                    Thread.sleep(10000);

                } catch (InterruptedException e) {
                    System.out.println("线程A被中断");
                } finally {
                    lock.unlock();
                    System.out.println("线程A释放锁");
                }
            }, "thread-A");

            Thread threadB = new Thread(() -> {
                try {
                    System.out.println("线程B尝试获取锁");

                    lock.lockInterruptibly(); // 阻塞在lockInterruptibly方法上

                    try {
                        System.out.println("线程B获得锁，开始处理业务");
                    } finally {
                        lock.unlock();
                        System.out.println("线程B释放锁");
                    }

                } catch (InterruptedException e) {
                    System.out.println("线程B等待锁期间被中断，不再继续等待");
                }
            }, "thread-B");

            threadA.start();

            Thread.sleep(1000);

            threadB.start();

            Thread.sleep(3000);

            System.out.println("主线程发出中断信号给线程B");

            threadB.interrupt(); // 主线程发出中断信号给线程B
        }
    }
   ```

   ```java
    public class SleepInterruptDemo {

        public static void main(String[] args) throws Exception {

            Thread worker = new Thread(() -> {

                try {

                    System.out.println("开始执行任务");

                    System.out.println("准备睡眠10秒");

                    Thread.sleep(10000); // 线程阻塞在sleep方法上

                    System.out.println("睡眠结束");

                } catch (InterruptedException e) {

                    System.out.println("sleep期间收到中断信号");

                    System.out.println("抛出InterruptedException");

                }

                System.out.println("线程结束");

            });

            worker.start();

            Thread.sleep(3000);

            System.out.println("主线程发送interrupt");

            worker.interrupt(); // 主线程发送interrupt

        }
    }
   ```

### 超时获取锁 tryLock

1. 在指定时间内等待，超时未获得锁就自己放弃等待并返回失败。==>客户很不喜欢无限等待，因此业务会有超时时间
2. ReentrantLock有tryLock(timeout, TimeUnit) 方法。
3. Redisson的RLock也有该方法

   ```java
    RLock lock = redissonClient.getLock("order");

    boolean success =lock.tryLock(
                                     3,  // 超时等待时间
                                     30, // 获得锁之后30秒释放
                                     TimeUnit.SECONDS
                                 );
   ```

### 条件等待 Condition 和 synchronized + wait/notify 对比

1. synchronized + wait/notify。 **生产者和消费者在同一个等待队列中**。
2. 进入synchronized代码块就是持有锁，退出时自动释放锁。
3. **lock.wait()方法会让调用该方法的线程由RUNNING状态变成WAITING状态，释放锁，并进入等待队列**；
4. **lock.notify()方法会随机唤醒一个等待线程，被唤醒的线程从WAITING状态到BLOCKED状态**，此时被唤醒的线程还不会持有锁，锁还在被调用lock.notify()方法的线程持有，退出synchronized代码块时才会释放锁。

   ```java
   import java.util.LinkedList;
   import java.util.Queue;

   public class SyncWarehouse {

       /**
        * 作为 synchronized 的锁对象。
        * wait() / notify() 也必须基于这个对象调用。
        */
       private final Object lock = new Object();

       /**
        * 仓库队列
        */
       private final Queue<String> warehouse = new LinkedList<>();

       /**
        * 仓库最大容量。
        */
       private final int capacity = 1;

       /**
        * 生产者。
        */
       public void produce(String product) throws InterruptedException {
            // 0.假设一开始仓库是满的，里面有一个货物
           synchronized (lock) { // 1. 生产者进入synchronized代码块，持有了lock，是RUNNING状态

               while (warehouse.size() == capacity) {
                   System.out.println("仓库满了，生产者等待");

                   lock.wait();
                   // 2. 执行 wait() 后发生三件事：
                   //    ① 生产者释放 lock 锁
                   //    ② 生产者进入 lock 的等待队列
                   //    ③ 生产者线程状态：RUNNING -> WAITING
                   // 生产者不会继续执行后面的warehouse.add(product)了，直到被notify()唤醒
               }

               // 7.再次进来的时候仓库是空的，因此执行放入商品的业务操作
               warehouse.add(product);
               System.out.println("生产者放入商品：" + product);

               lock.notify();
               //8. notify()方法随机唤醒一个等待线程，如果有WAIT的消费者，则从WAITING到BLOCKED，生产者还持有lock

           }// 9. 方法返回前，退出synchronized代码块的同时，JVM释放锁！
       }

       /**
        * 消费商品。
        */
       public String consume() throws InterruptedException {

           synchronized (lock) { // 3. 消费者进入synchronized代码块，持有了lock，是RUNNING状态

               while (warehouse.isEmpty()) {
                   System.out.println("仓库空了，消费者等待");

                   lock.wait();
                    // 10. 仓库空的，wait()方法之后：
                    // ① 消费者释放 lock
                    // ② 消费者进入 lock 的等待队列
                    // ③ 消费者 RUNNING -> WAITING
               }

               // 4. 执行取走商品的业务操作
               String product = warehouse.poll();
               System.out.println("消费者取走商品：" + product);

               lock.notify();
               // 5. notify()会随机唤醒一个等待线程，这里往往会唤醒生产者
               // 生产者状态 WAITING 变为 BLOCKED，因为此时lock还是消费者持有

               return product;

           } //6. 方法返回前，退出synchronized代码块的同时，JVM释放锁！
       }
   }
   ```

   真正运行时消费者和生产者是两个线程

   ```java
    Thread producer = new Thread(() -> {

        try {

            warehouse.produce("苹果");

        } catch (Exception e) {
            e.printStackTrace();
        }

    });
   ```

   ```java
    Thread consumer = new Thread(() -> {

        try {

            warehouse.consume();

        } catch (Exception e) {
            e.printStackTrace();
        }

    });
   ```

5. ```java
   import java.util.LinkedList;
   import java.util.Queue;
   import java.util.concurrent.locks.Condition;
   import java.util.concurrent.locks.ReentrantLock;

   public class ConditionWarehouse {

       /**
        * ReentrantLock 代替 synchronized
        * 基于AQS 里面有一个AQS同步队列
        */
       private final ReentrantLock lock = new ReentrantLock();

       /**
        * 仓库满时
        * 生产者进入这个等待队列
        */
       private final Condition notFull = lock.newCondition();

       /**
        * 仓库空时
        * 消费者进入这个等待队列
        */
       private final Condition notEmpty = lock.newCondition();

       private final Queue<String> warehouse = new LinkedList<>();

       private final int capacity = 1;

       public ConditionWarehouse() {

           // 0. 假设一开始仓库是满的
           warehouse.add("old-product");
       }

       /**
        * 生产者线程执行
        */
       public void produce(String product)
               throws InterruptedException {

           lock.lock();
           // 1. 生产者获得 lock
           //    状态：RUNNING

           try {

               while (warehouse.size() == capacity) {

                   System.out.println("仓库满了，生产者等待");

                   notFull.await();
                   // 2. await() 后发生三件事：
                   // ① 释放 lock
                   // ② 进入 notFull 等待队列
                   // ③ RUNNING -> WAITING
                   //
                   // 队列层面：
                   // ① 当前生产者线程被封装成 Node
                   // ② Node.waitStatus = CONDITION
                   // ③ 生产者进入 notFull 条件队列
                   // 注意：
                   // 代码暂停在这里,不会继续执行 warehouse.add(product)
               }

               // 7. 被消费者 signal()并重新获得 lock
               warehouse.add(product);
               System.out.println("生产者放入商品：" + product);

               notEmpty.signal();

               // 8. 精准通知：
               //    唤醒 notEmpty 等待队列中的线程，也就是消费者线程
               //    消费者线程从notEmpty条件队列中进入AQS同步队列中
               //    消费者：WAITING 变成 BLOCKED
               //
               // 注意：signal()不释放 lock，当前 lock仍然属于生产者

           } finally {
               lock.unlock();

               // 9. finally执行，释放 lock，被唤醒的消费者才有机会重新竞争 lock
           }
       }

       /**
        * 消费者线程执行
        */
       public String consume()
               throws InterruptedException {

           lock.lock();

           // 3. 消费者获得 lock
           //
           //    状态：RUNNING
           try {

               while (warehouse.isEmpty()) {

                   System.out.println("仓库空了，消费者等待");

                   notEmpty.await();
                   // 10. await() 后：
                   // ① 释放 lock
                   // ② 进入 notEmpty 等待队列
                   // ③ RUNNING -> WAITING
               }

               // 4. 当前仓库不空,执行消费业务
               String product = warehouse.poll();
               System.out.println(
                       "消费者取走商品：" + product);

               notFull.signal();
               // 5. 精准通知：
               //    唤醒 notFull 等待队列中的线程
               //    也就是生产者线程，生产者从notFull队列中出来进入AQS同步队列中
               //    生产者：WAITING 变成 BLOCKED
               //
               // 当前 lock仍然由消费者持有
               return product;

           } finally {

               lock.unlock();
               // 6. return之前finally先执行，消费者释放 lock
           }
       }
   }

   ```

6. Condition await/signal + lock 和 synchronized +
   wait/notify 区别是 条件等待Condition生产者和消费者各有自己的等待队列，能使用不同的condition精准唤醒，而synchronized +
   wait/notify消费者和生产者都在一个等待队列中，是notify是随机唤醒一个WAITING状态的线程。

7. **Condition的await()方法，会让线程进入对应的条件队列**；
8. **Condition的signal()方法，会让线程从对应的条件队列中被唤醒、转移到AQS同步队列中**；
9. **lock.unlock() 释放锁之后，AQS机制会唤醒下一个节点，让他重新竞争lock**。
10. **实际生产者都不要用，用阻塞队列等封装好的东西。**

## AQS AbstractQueuedSynchronizere 抽象队列同步器

1. AQS（AbstractQueuedSynchronizer）是JDK并发包中的同步框架，在java.util.concurrent.locks包下。通过一个 volatile
   state 状态变量和一个 FIFO 双向等待队列实现线程同步。ReentrantLock、Semaphore、CountDownLatch、ReentrantReadWriteLock 等并发工具都基于 AQS 实现。

### AQS为什么存在

1. 因为 ReentrantLock、Semaphore、CountDownLatch、ReentrantReadWriteLock 等 都需要
   **等待队列、维护线程挂起和线程唤醒**，所以抽象出AQS同步框架，不然每一套锁机制都要写一套这个。

### AQS核心思想

1. AQS = volatile状态(state)变量 + CAS原子修改state + FIFO等待队列(Node
   Queue) + 线程挂起/唤醒(park/unpark)；

#### AQS的 state变量

1. AQS类中的 private volatile int state;
   volatile字段负责可见性和有序性，真正修改state字段时，使用Unsafe类的CAS操作来更新state，保证原子性。

   ```java
   // compareAndSwapInt：原子地判断当前 state 是否等于 expect，如果是，则更新为 update；否则失败。
    protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
   ```

2. ReentrantLock里用state记录锁的重入次数；
3. Semaphore（信号量）允许多个线程同时访问特定资源，state就是初始化的时候许可证的数量。

   ```java
    // 初始共享资源数量
    final Semaphore semaphore = new Semaphore(5);
    // 获取1个许可
    semaphore.acquire();
    // 释放1个许可
    semaphore.release();

   ```

   Semaphore默认非公平模式，Semaphore semaphore = new Semaphore(3,
   true);才会显式的创建公平模式。

4. CountDownLatch （倒计时器）允许count个线程阻塞在同一个资方，直至所有线程的任务都执行完毕；AQS中的state就是count；countDown() 方法通过 CAS 原子地减少 state；当 state为0 时，await() 的线程被唤醒。**CountDownLatch是一次性的门，count等于0的时候门就永久打开，不能被重置。**

   ```java
   // 所有依赖服务都启动好了之后 微服务再启动
   @Component
   public class ServiceHealthChecker {

       @PostConstruct
       public void checkDependencies() throws InterruptedException {
           String[] services = {"Redis", "MySQL", "MongoDB", "Kafka"};
           CountDownLatch latch = new CountDownLatch(services.length);

           for (String service : services) {
               executor.submit(() -> {
                   try {
                       checkService(service);
                   } finally {
                       latch.countDown();
                   }
               });
           }

           if (!latch.await(10, TimeUnit.SECONDS)) {
               throw new RuntimeException("服务依赖检查超时");
           }
           System.out.println("所有依赖服务正常");
       }
   }

   ```

#### AQS 的 Node 就是等待中的线程

1. AQS不是要维护等待队列吗，等待队列是FIFO的双线队列，队列中的元素是Node类型，用来保存具体的线程。

   ```
    static final class Node {
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        volatile int waitStatus;
    }
   ```

   其中waitStatus表示节点状态

   ```
    CANCELLED = 1   // 节点取消等待
    SIGNAL    = -1  // 需要唤醒后继节点
    CONDITION = -2  // 在条件等待队列中
    PROPAGATE = -3  // 用于共享模式
    0         = 进入队列后的初始状态
   ```

#### AQS和CLH，AQS是CLH队列的变体

##### CLH是什么？

1. CLH 全称 Craig, Landin, and Hagersten lock，是一种基于队列的自旋锁。
2. **普通自旋锁**，线程B、C、D都想抢锁，他们都要while(locked)，一直自旋等待 ==>
   **多个线程一直访问同一个锁变量** ==> **CPU压力大、性能下降**。 ==>
   **CLH队列做了优化，增加单向等待队列**，每个节点（队列里的节点就是等待的线程B、C、D）都**自旋看他前面的那个节点是否还占有锁**，比如B前面是A，B自旋是while(A.locked), 只要发现A释放了锁，就表示B可以占有锁了，因为这个等待队列是公平的。==>
   **避免普通自旋锁多个线程竞争同一个锁变量的开销**。

   ```
    B 盯 A
    C 盯 B
    D 盯 C
   ```

##### AQS基于CLH做的优化 ==> 等待线程不自旋，前驱节点唤醒后继节点

1. AQS等待队列是双向链表，Node有prev和next属性，知道自己前面的节点和后面的节点是谁。
2. 线程A、B、C、D，线程暂时拿不到锁就加入FIFO队列的队尾，并且LockSupport.park()进入挂起状态；**前驱节点释放锁的时候LockSupport.unpark(B)唤醒自己后面的节点**B，**B就通过CAS重新抢state、抢到之后B才能获得锁**。因此**AQS中Node等待锁的时候不需要自旋**。

   ```
    LockSupport.park()
        ↓
    RUNNABLE -> WAITING

    LockSupport.unpark(thread)
        ↓
    WAITING -> RUNNABLE
   ```

#### AQS的独占模式和共享模式

| 对比点    | 独占模式                       | 共享模式                                         |
| --------- | ------------------------------ | ------------------------------------------------ |
| 含义      | 同一时刻只允许一个线程获取资源 | 同一时刻允许多个线程获取资源                     |
| 典型类    | ReentrantLock                  | Semaphore、CountDownLatch、ReadWriteLock中的读锁 |
| AQS入口   | `acquire()`                    | `acquireShared()`                                |
| 释放入口  | `release()`                    | `releaseShared()`                                |
| 获取方法  | `tryAcquire()`                 | `tryAcquireShared()`                             |
| 释放方法  | `tryRelease()`                 | `tryReleaseShared()`                             |
| 唤醒特点  | 通常唤醒一个后继线程           | **可能传播式唤醒多个线程**                       |
| state含义 | 锁状态/重入次数                | 许可证数量/计数/读锁数量                         |
| 例子      | 只有一个人进厕所               | 3个停车位可同时停3辆车                           |

##### 独占模式获取资源流程

1. AQS源码

   ```java

   public final void acquire(int arg) {
       if (!tryAcquire(arg) &&
           acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
           selfInterrupt();
       }
   }

   protected boolean tryAcquire(int arg) {
       throw new UnsupportedOperationException();
   }

   private Node addWaiter(Node mode) {
       Node node = new Node(mode);

       for (;;) {
           Node oldTail = tail;
           if (oldTail != null) {
               node.setPrevRelaxed(oldTail);
               if (compareAndSetTail(oldTail, node)) {
                   oldTail.next = node;
                   return node;
               }
           } else {
               initializeSyncQueue();
           }
       }
   }

   final boolean acquireQueued(final Node node, int arg) {
       boolean interrupted = false;
       try {
           for (;;) {
               final Node p = node.predecessor();
               if (p == head && tryAcquire(arg)) {
                    // 获取锁成功了
                   setHead(node);
                   p.next = null; // help GC
                   // 返回中断状态
                   return interrupted;
               }

               // 没拿到锁就park
               if (shouldParkAfterFailedAcquire(p, node))
                   interrupted |= parkAndCheckInterrupt();
           }
       } catch (Throwable t) {
           cancelAcquire(node);
           if (interrupted)
               selfInterrupt();
           throw t;
       }
   }
   ```

2. tryAcquire(arg)：尝试获取锁，AQS提供模板，子类需要改写,下面以ReentrantLock源码的非公平锁为例：

   ```java
    // ReentrantLock
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        // 1、获取 AQS 中的 state 状态
        int c = getState();
        // 2、如果 state 为 0，证明锁没有被其他线程占用
        if (c == 0) {
            // 2.1、通过 CAS 对 state 进行更新
            if (compareAndSetState(0, acquires)) {
                // 2.2、如果 CAS 更新成功，就将锁的持有者设置为当前线程
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 3、如果当前线程和锁的持有线程相同，说明发生了「锁的重入」
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 3.1、将锁的重入次数加 1
            setState(nextc);
            return true;
        }
        // 4、如果锁被其他线程占用，就返回 false，表示获取锁失败
        return false;
    }

    protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
   ```

   (1). 通过CAS的 compareAndSetState方法对state进行更新；

   (2). setExclusiveOwnerThread方法把锁的持有者改为当前线程；

   ```java
    // 【核心作用】一个"身份证"存储器，只记一件事：排他锁现在是谁的。
    // 不参与锁的争抢，只负责结合AQS框架记录一下当前谁拿到了锁，跟底层JVM没什么关系
    public abstract class AbstractOwnableSynchronizer {

        // 【身份证】存当前持锁线程，null 表示锁没人拿
        private transient Thread exclusiveOwnerThread;

        // ==================== 两个方法，就是存取身份证 ====================

        // 【登记】把线程名字写上去
        protected final void setExclusiveOwnerThread(Thread thread) {
            exclusiveOwnerThread = thread;  // 就是普通赋值，没特殊操作
        }

        // 【查名】看看现在锁在谁手里
        protected final Thread getExclusiveOwnerThread() {
            return exclusiveOwnerThread;    // 就是普通返回，没特殊操作
        }
    }
   ```

3. addWaiter(Node.EXCLUSIVE)：获取失败就执行addQaiter，把当前线程包装成Node加入AQS同步队列尾部；

4. acquireQueued(...)：写线程在队列中等待的逻辑，先判断当前节点的前驱是不是head，如果是的话就尝试获取锁，获取成功了就把当前节点设置为head，如果失败了就park挂起。

##### 独占模式释放资源

1. 源码

   ```java
       // AQS
       public final boolean release(int arg) {
           // 1、尝试释放锁
           if (tryRelease(arg)) {
               Node h = head;
               // 2、唤醒后继节点
               if (h != null && h.waitStatus != 0)
                   unparkSuccessor(h);
               return true;
           }
           return false;
       }
   ```

2. 流程

   ```
   线程B调用 lock()

           ↓

   tryAcquire()
   尝试 CAS(state, 0, 1)

           ↓

   成功？
   │
   ├─ 是
   │    ↓
   │  获得锁
   │
   └─ 否
        ↓
      创建 Node(B)

        ↓
      加入 AQS 同步队列尾部

        ↓
      判断前驱是否为 head

        ↓
      如果轮到自己，再尝试获取锁

        ↓
      仍失败

        ↓
      LockSupport.park()
      挂起线程B

   ================================

   线程A unlock()

        ↓
      tryRelease()

        ↓
      state = 0

        ↓
      unpark 后继节点线程B

   ================================

   线程B被唤醒

        ↓
      再次 tryAcquire()

        ↓
      CAS(state, 0, 1)

        ↓
      成功

        ↓
      获得锁
   ```

##### 共享模式

1. 共享模式就是允许多个线程共享资源。

   ```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0) {
            doAcquireShared(arg);
        }
    }
   ```

   (3).doAcquireShared(arg); 和当独占模式acquireQueued()的区别就是，抢到资源自己称为head之后不只唤醒一个节点，会继续往后唤醒。

#### 公平锁和非公平锁，以ReentrantLock为例

1. 非公平锁一上来就抢锁，公平锁要用 !hasQueuedPredecessors() 方法判断一下有没有前驱节点，没有的话才CAS抢锁。

## 从锁的角度对比分析各个锁的特性

1. Java锁体系

   ```
    Java 锁体系
    │
    ├── 语言级内置锁（JVM 层面）
    │   └── synchronized        —— 互斥锁，关键字实现
    │
    └── JUC 显式锁（java.util.concurrent.locks 包）
        ├── ReentrantLock           —— 互斥锁
        ├── ReentrantReadWriteLock  —— 读写锁
        └── StampedLock             —— 读写锁 + 乐观读
   ```

   |          | synchronized | ReentrantLock | ReentrantReadWriteLock |    StampedLock    |
   | -------- | :----------: | :-----------: | :--------------------: | :---------------: |
   | 所属层级 |    语言级    |      JUC      |          JUC           |        JUC        |
   | 实现方式 | **JVM 内置** |      AQS      |          AQS           |     独立实现      |
   | 锁类型   |     互斥     |     互斥      |        读写分离        | 读写分离 + 乐观读 |

### synchronized

1. 对象监视器：

   (1). Java中每个对象都有一个内部锁（intrinsic lock），类也有一个内部锁。

   (2). **对象的监视器(object’s
   monitor)**是一个抽象概念，是**一种同步机制**，用于控制对共享资源的访问，**确保同一时间只有一个线程可以访问被监视的代码块。**
   ==> 其具体实现就是对象的内部锁，某线程获取了对象的内部锁，就是获取了该对象监视器的所有权。

   (3). 在 Java 虚拟机(HotSpot)中，对象监视器Monitor 是基于 C++实现的；**ObjectMonitor 中维护锁持有者、EntryList、WaitSet 和重入次数等信息**。

   | 写法                                       | 锁的是哪个对象        |
   | ------------------------------------------ | --------------------- |
   | `synchronized(obj)`                        | `obj` 对象            |
   | `synchronized(this)`                       | 当前实例对象          |
   | `public synchronized void method()`        | 当前实例对象 `this`   |
   | `public static synchronized void method()` | 当前类的 `Class` 对象 |

2. synchronized锁的本地：对象头Mark Word + Monitor。**Mark
   Word里面存储和锁相关的信息。**

   ```
    Java对象
    │
    ├── 对象头 Header
    │   ├── Mark Word
    │   └── Klass Pointer
    │
    ├── 实例数据
    │
    └── 对齐填充
   ```

   **JVM 会根据竞争情况对 synchronized 做锁优化，进行锁升级**。不同的锁状态Mark
   Word中存的东西不一样。

   (1). **无锁状态：没有竞争**。Mark
   Word 里存 对象的hashcode、年龄、锁标志位等；

   (2). **偏向锁：只有一个线程反复进入**。Mark
   Word 里存偏向线程ID，**同一个线程再次进入时不需要CAS**。

   (3). **轻量级锁**：多线程交替进入对象但是**竞争不激烈**。Mark
   Word 里存指向线程栈中 Lock Record 的指针，线程获取锁时**用CAS**把对象的 Mark
   Word 替换为指向 Lock Record 的指针。

   (4). **重量级锁：多个线程激烈竞争**。 **Mark
   Word 里存指向 ObjectMonitor 的指针**，涉及系统调用、线程阻塞和唤醒、可能有用户态↔内核态切换等。

3. 锁升级是单向的，只能升级不能降级。
4. synchronized 同步代码块，编译后会生成 monitorenter 和 monitorexit 字节码指令；synchronized同步方法，JVM 通过 ACC_SYNCHRONIZED 方法访问标志隐式获取和释放 monitor。==>
   **代码中写上synchronized字段之后，并不是一定会变成重量级锁。**

### 读写锁 ReadWriteLock

1. **实际上是两把锁，读锁和写锁**，读读共享、读写互斥、写写互斥，适用于读多写少的情况。
2. 但是一般操作数据库，数据库会帮我们处理好并发事务，ReadWriteLock适合内存读写，这种一般框架比如Caffine本地内存框架会封装好，所以用到的不多。public
   class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable

   ```java
    ReadWriteLock rwLock = new ReentrantReadWriteLock();

    Lock readLock = rwLock.readLock();

    Lock writeLock = rwLock.writeLock();
   ```

   ```java
    RReadWriteLock rwLock = redissonClient.getReadWriteLock("product");

    RLock readLock = rwLock.readLock(); // 读锁

    RLock writeLock = rwLock.writeLock(); // 写锁
   ```

3. 线程持有读锁的情况下，该线程不能取得写锁（因为获取写锁的时候会发现读锁被占用了）；
4. 线程持有写锁的情况下，该线程可以继续获得读锁。
5. 读锁不能升级为写锁，会引发线程的竞争（比如多个读锁都想升级为死锁），并且还可能死锁。

### StampedLock

1. **StampedLock是基于CLH锁实现的读写锁**。
2. 提供三种模式的锁：读锁、写锁、乐观读，并且支持三种锁在一定条件下相互转换。
3. **乐观读并不是获得了读锁，乐观读只是读取版本号，然后无锁读取数据,因此不会阻塞任何线程，乐观读期间写锁可以随时进入，所以StampedLock性能比ReadWriteLock好**。

   ```java
    // 第 1 步：拿到版本戳
    long stamp = lock.tryOptimisticRead();

    // 第 2 步：无锁读取数据（读多少都可以，完全无保护）
    int x = sharedData.x;
    String y = sharedData.y;
    long z = sharedData.z;

    // 第 3 步：校验——版本号变过吗？
    if (!lock.validate(stamp)) {
        // 版本变了！刚才读到的数据可能不一致
        // 需要重试或用悲观读
    }
   ```

| 锁                             | 互斥   | 可重入   | 公平性                          | 可中断           | 超时获取tryLock | 条件等待                       | 自动释放                     | 跨 JVM   |
| ------------------------------ | ------ | -------- | ------------------------------- | ---------------- | --------------- | ------------------------------ | ---------------------------- | -------- |
| `synchronized`                 | 支持✅ | 支持✅   | 非公平                          | 等锁时不可中断❌ | 不支持❌        | `wait/notify`                  | 支持，代码块运行完自动释放✅ | 不支持❌ |
| `ReentrantLock`                | 支持✅ | 支持✅   | 可选，默认非公平                | 支持✅           | 支持✅          | 支持 Condition✅               | 不支持，需要手动 unlock❌    | 不支持❌ |
| `ReentrantReadWriteLock`读写锁 | 支持✅ | 支持✅   | 可选，默认非公平                | 支持✅           | 支持✅          | 支持 Condition，主要写锁可用✅ | 不支持，需要手动 unlock❌    | 不支持❌ |
| `StampedLock`有乐观读的读写锁  | 支持✅ | 不支持❌ | 非公平                          | 部分支持         | 支持✅          | 不支持 Condition❌             | 不支持，需要手动 unlock❌    | 不支持❌ |
| Redis原生命令                  | 支持✅ | 不支持❌ | 非公平，按照到达Redis的顺序执行 | 不支持❌         | 不支持❌        | 不支持❌                       | EX命令支持✅                 | 支持✅   |

1. Redis原生命令 如 SET lockKey token NX EX 30。

| 同步工具          | 互斥                       | 可重入       | 公平性           | 可中断           | 超时获取       | 条件等待           | 自动释放                  | 跨 JVM   |
| ----------------- | -------------------------- | ------------ | ---------------- | ---------------- | -------------- | ------------------ | ------------------------- | -------- |
| `Semaphore`信号量 | 限制并发数，不是严格互斥锁 | 不强调可重入 | 可选，默认非公平 | 支持✅           | 支持✅         | 不支持 Condition❌ | 不支持，需要手动 unlock❌ | 不支持❌ |
| `CountDownLatch`  | 不是锁                     | 不涉及       | 不涉及           | `await()` 可中断 | 支持超时 await | 不支持❌           | 不涉及                    | 不支持❌ |

1. Redisson在Redis上封装出类似JUC的锁体系。

| JUC 锁                 | Redisson 对应实现 | 分布式能力                                  |
| ---------------------- | ----------------- | ------------------------------------------- |
| ReentrantLock          | RLock             | 可重入互斥锁                                |
| ReentrantReadWriteLock | RReadWriteLock    | 读写锁                                      |
| —                      | RFairLock         | 公平锁（JUC 是构造参数，Redisson 单独成类） |
| —                      | RedLock           | 红锁，多节点容错（JUC 无对应）              |
| Semaphore              | RSemaphore        | 分布式信号量                                |
| CountDownLatch         | RCountDownLatch   | 分布式闭锁                                  |

1. sleep()方法让当前线程休眠，但是不会释放锁，如果线程占有锁的时候sleep也不会释放锁；wait()会释放锁。
