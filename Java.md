### ThreadLocal

> This class provides thread-local variables.  These variables differ from their normal counterparts in that each thread that accesses one (via its {@code get} or {@code set} method) has its own, independently initialized copy of the variable.  {@code ThreadLocal} instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).
> <p>For example, the class below generates unique identifiers local to each thread. A thread's id is assigned the first time it invokes {@code ThreadId.get()} and remains unchanged on subsequent calls.
>
> ThreadLocal 提供了线程的局部变量，每个线程可通过set() 和 get()方法来对这个局部变量进行操作，但不会和其他线程的局部变量冲突，实现了线程数据隔离。

#### 理解：

同一个ThreadLocal所包含的对象，在不同的thread中有不同的副本（实际是不同的实例）注意：

* 因为每个thread内有自己的实例副本，且该副本只能由当前thread使用，固：threadLocal
* 因每个thread有自己的副本，且其他Thread不可访问，固：无多线程共享问题
* 因无共享问题 ，所不没有同步问题 

#### 用法

```java
import java.util.concurrent.CountDownLatch;

public class ThreadLocalDemo {
    public static void main(String[] args) throws InterruptedException {
        int threads = 3;
        CountDownLatch countDownLatch = new CountDownLatch(threads);
        InnerClass innerClass = new InnerClass();
        for(int i = 1; i <= threads; i++) {
            new Thread(() -> {
                for(int j = 0; j < 4; j++) {
                    innerClass.add(String.valueOf(j));
                    innerClass.print();
                }
                innerClass.set("hello world");
                countDownLatch.countDown();
            }, "thread - " + i).start();
        }
        countDownLatch.await();



    }

    private static class InnerClass{
        public void add(String newStr){
            StringBuilder str = Counter.counter.get();
            Counter.counter.set(str.append(newStr));
        }
        public void print(){
            System.out.printf("Thread name:%s ,ThreadLocal hashCode:%s,Instance hashCode:%s,Value:%s\n",
            Thread.currentThread().getName(),Counter.counter.hashCode(),Counter.counter.get().hashCode(),Counter.counter.get().toString()
            );
        }
        public void set(String words){
            Counter.counter.set(new StringBuilder(words));
            System.out.printf("Set, Thread name:%s , ThreadLocal hashcode:%s,  Instance hashcode:%s, Value:%s\n",
                    Thread.currentThread().getName(),
                    Counter.counter.hashCode(),
                    Counter.counter.get().hashCode(),
                    Counter.counter.get().toString());

        }
    }

    private static class Counter{
        private static ThreadLocal<StringBuilder> counter = new ThreadLocal<StringBuilder>(){
            @Override
            protected StringBuilder initialValue() {
                return new StringBuilder();
            }
        };
    }
}

```

#### 适用场景

如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

对于第一点，每个线程拥有自己实例，实现它的方式很多。例如可以在线程内部构建一个单独的实例。ThreadLocal 可以以非常方便的形式满足该需求。

对于第二点，可以在满足第一点（每个线程有自己的实例）的条件下，通过方法间引用传递的形式实现。ThreadLocal 使得代码耦合度更低，且实现更优雅。

#### 案例

对于 Java Web 应用而言，Session 保存了很多信息。很多时候需要通过 Session 获取信息，有些时候又需要修改 Session 的信息。一方面，需要保证每个线程有自己单独的 Session 实例。另一方面，由于很多地方都需要操作 Session，存在多方法共享 Session 的需求。如果不使用 ThreadLocal，可以在每个线程内构建一个 Session实例，并将该实例在多个方法间传递，如下所示。

```java
public class SessionHandler {

  @Data
  public static class Session {
    private String id;
    private String user;
    private String status;
  }

  public Session createSession() {
    return new Session();
  }

  public String getUser(Session session) {
    return session.getUser();
  }

  public String getStatus(Session session) {
    return session.getStatus();
  }

  public void setStatus(Session session, String status) {
    session.setStatus(status);
  }

  public static void main(String[] args) {
    new Thread(() -> {
      SessionHandler handler = new SessionHandler();
      Session session = handler.createSession();
      handler.getStatus(session);
      handler.getUser(session);
      handler.setStatus(session, "close");
      handler.getStatus(session);
    }).start();
  }
}
```

该方法是可以实现需求的。但是每个需要使用 Session 的地方，都需要显式传递 Session 对象，方法间耦合度较高。

这里使用 ThreadLocal 重新实现该功能如下所示。

```java
public class SessionHandler {

  public static ThreadLocal<Session> session = ThreadLocal.<Session>withInitial(() -> new Session());

  @Data
  public static class Session {
    private String id;
    private String user;
    private String status;
  }

  public String getUser() {
    return session.get().getUser();
  }

  public String getStatus() {
    return session.get().getStatus();
  }

  public void setStatus(String status) {
    session.get().setStatus(status);
  }

  public static void main(String[] args) {
    new Thread(() -> {
      SessionHandler handler = new SessionHandler();
      handler.getStatus();
      handler.getUser();
      handler.setStatus("close");
      handler.getStatus();
    }).start();
  }
}
```

使用 ThreadLocal 改造后的代码，不再需要在各个方法间传递 Session 对象，并且也非常轻松的保证了每个线程拥有自己独立的实例。

如果单看其中某一点，替代方法很多。比如可通过在线程内创建局部变量可实现每个线程有自己的实例，使用静态变量可实现变量在方法间的共享。但如果要同时满足变量在线程间的隔离与方法间的共享，ThreadLocal再合适不过。

#### 总结：

- ThreadLocal 并不解决线程间共享数据的问题
- ThreadLocal 通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题
- 每个线程持有一个 Map 并维护了 ThreadLocal 对象与具体实例的映射，该 Map 由于只被持有它的线程访问，故不存在线程安全以及锁的问题
- ThreadLocalMap 的 Entry 对 ThreadLocal 的引用为弱引用，避免了 ThreadLocal 对象无法被回收的问题
- ThreadLocalMap 的 set 方法通过调用 replaceStaleEntry 方法回收键为 null 的 Entry 对象的值（即为具体实例）以及 Entry 对象本身从而防止内存泄漏
- ThreadLocal 适用于变量在线程间隔离且在方法间共享的场景

#### 参考：

[ThreadLocal](http://www.jasongj.com/java/threadlocal/)

### hashCode equals()

hashCode 和equals是Object对象的非final方法，可被 ovrride 在程序设计中需要处理这两个方法

#### hashCode

hashCode用于返回对象的hash值，主要用于查找的快捷性，因为hashCode也是在Object对象中就有的，所以所有Java对象都有hashCode，在HashTable和HashMap这一类的散列结构中，都是通过hashCode来查找在散列表中的位置的。

**如果两个对象equals，那么它们的hashCode必然相等， 但是hashCode相等，equals不一定相等。**

一个好的hashCode的方法的目标：**为不相等的对象产生不相等的散列码**，同样的，相等的对象必须拥有相等的散列码。



#### equals

Object中的equals方法

```java
public boolean equals(Object obj){
    return this == obj;
}
// 只看地址是否一样，是否是同一个对象
// 而我们在重写时，有时需要判断是否是逻辑上相等（值 相等，内容相等）
```

覆写equals时的准则：

1. 自反性：x.equals(x) 返回true
2. 对称性：x.equals(y) == y.equals(x)
3. 传递性：x.equals(y) 返回true y.equals(z)返回true,则 x.equals(z)返回true
4. 一致性：多次调用返回相同的结果
5. 非空性：任何非空引用值 x, x.equals(null)都应返回false

有个准则，一般在覆写equals**只兼容同类型的变量**。
**覆盖equals时一定要覆盖hashCode**
equals函数里面一定要是Object类型作为参数
equals方法本身不要过于智能，只要判断一些值相等即可。

### 多线程：

1. 原子性：一个操作要么全部执行，要么全部不执行。

2. 可见性：当多个线程并发访问共享变量时，一个线程对共享变量的修改，其他线程能够立刻看到。

3. 顺序性：程序执行的顺序按照代码的先后顺序执行

   下面的4行代码应该依次执行，但Jvm真正执行时，并不保证它们一定按此顺序

   ```java
   boolean started = false;
   long counter = 0L;
   counter = 1;
   started = true;
   ```

#### java 如何保证原子性

常用的方法：锁和同步方法（同步代码块），使用锁，可以保证同一时间只有一个线程能拿到锁，也就保证了同一时间只有一个线程能执行申请锁和释放锁之间的代码

```java
public void testLock(){
    lock.lock();
    try{
        int j = i;
        i = j+1;
    }finally{
        lock.unlock();
    }
}
```

同步方法或同步代码块：

1. 使用非静态同步方法时，锁住的是当前实例

2. 使用静态同步方法时，锁住的是该类的Class对象

3. 使用静态代码块时，锁住的是synchronized关键字后面括号内的对象

   ```java
   public void testLock(){
       synchronized(anyObject){
           int j = i;
           i = j+1;
       }
   }
   ```

CAS:(compare and swap)

基础类型变量自增（i++） 是一种常被误以为是原子操作，其实不是原子操作。java中提供了对应的原子操作类来实现操作，并保证原子性，其本质是利用了CPU级别的CAS指令，由于是CPU级别的指令，开销比OS参与的锁的开销小。

```java
AtomicInteger atomicInteger = new AtomicInteger();
for(int b = 0; b<numThreads ; b++){
    new Thread(()=>{
        for(int a = 0; a<iteration;a++){
            atomicInteger.incrementAndGet();
        }
    })
}
```

#### 可见性：

java提供了 volatile 关键字来保证可见性，当使用volatile修饰某个变量时，它会保证对该变量的修改会立即被更新到内存中，并且将缓存设置成无效，因此其他线程读取该值时，从主存中读取，从而保证得到最新值

#### 顺序性：

volatile一定程序中保证了顺序性。

能过synchronized 和 锁保证顺序性。

jvm通过happens-before 隐式的保证顺序性

- 传递规则：如果操作1在操作2前面，而操作2在操作3前面，则操作1肯定会在操作3前发生。该规则说明了happens-before原则具有传递性
- 锁定规则：一个unlock操作肯定会在后面对同一个锁的lock操作前发生。这个很好理解，锁只有被释放了才会被再次获取
- volatile变量规则：对一个被volatile修饰的写操作先发生于后面对该变量的读操作
- 程序次序规则：一个线程内，按照代码顺序执行
- 线程启动规则：Thread对象的start()方法先发生于此线程的其它动作
- 线程终结原则：线程的终止检测后发生于线程中其它的所有操作
- 线程中断规则： 对线程interrupt()方法的调用先发生于对该中断异常的获取
- 对象终结规则：一个对象构造先于它的finalize发生

CountDownLatch

CyclicBarrier

从使用场景上来说，CyclicBarrier是让多个线程互相等待某一事件的发生，然后同时被唤醒。而上文讲的CountDownLatch是让某一线程等待多个线程的状态，然后该线程被唤醒。

Phaser

Phaser顾名思义，与阶段相关。Phaser比较适合这样一种场景，一种任务可以分为多个阶段，现希望多个线程去处理该批任务，对于每个阶段，多个线程可以并发进行，但是希望保证只有前面一个阶段的任务完成之后才能开始后面的任务。这种场景可以使用多个CyclicBarrier来实现，每个CyclicBarrier负责等待一个阶段的任务全部完成。但是使用CyclicBarrier的缺点在于，需要明确知道总共有多少个阶段，同时并行的任务数需要提前预定义好，且无法动态修改。而Phaser可同时解决这两个问题。

```java
public class PhaserDemo {

  public static void main(String[] args) throws IOException {
    int parties = 3;
    int phases = 4;
    final Phaser phaser = new Phaser(parties) {
      @Override  
      protected boolean onAdvance(int phase, int registeredParties) {  
          System.out.println("====== Phase : " + phase + " ======");  
          return registeredParties == 0;  
      }  
    };
    
    for(int i = 0; i < parties; i++) {
      int threadId = i;
      Thread thread = new Thread(() -> {
        for(int phase = 0; phase < phases; phase++) {
          System.out.println(String.format("Thread %s, phase %s", threadId, phase));
          phaser.arriveAndAwaitAdvance();
        }
      });
      thread.start();
    }
  }
}
/*执行结果如下：
Thread 0, phase 0
Thread 1, phase 0
Thread 2, phase 0
====== Phase : 0 ======
Thread 2, phase 1
Thread 0, phase 1
Thread 1, phase 1
====== Phase : 1 ======
Thread 1, phase 2
Thread 2, phase 2
Thread 0, phase 2
====== Phase : 2 ======
Thread 0, phase 3
Thread 1, phase 3
Thread 2, phase 3
====== Phase : 3 ======
*/

```

Phaser主要接口如下

- **arriveAndAwaitAdvance()** 当前线程当前阶段执行完毕，等待其它线程完成当前阶段。如果当前线程是该阶段最后一个未到达的，则该方法直接返回下一个阶段的序号（阶段序号从0开始），同时其它线程的该方法也返回下一个阶段的序号。
- **arriveAndDeregister()** 该方法立即返回下一阶段的序号，并且其它线程需要等待的个数减一，并且把当前线程从之后需要等待的成员中移除。如果该Phaser是另外一个Phaser的子Phaser（层次化Phaser会在后文中讲到），并且该操作导致当前Phaser的成员数为0，则该操作也会将当前Phaser从其父Phaser中移除。
- **arrive()** 该方法不作任何等待，直接返回下一阶段的序号。
- **awaitAdvance(int phase)** 该方法等待某一阶段执行完毕。如果当前阶段不等于指定的阶段或者该Phaser已经被终止，则立即返回。该阶段数一般由*arrive()*方法或者*arriveAndDeregister()*方法返回。返回下一阶段的序号，或者返回参数指定的值（如果该参数为负数），或者直接返回当前阶段序号（如果当前Phaser已经被终止）。
- **awaitAdvanceInterruptibly(int phase)** 效果与*awaitAdvance(int phase)*相当，唯一的不同在于若该线程在该方法等待时被中断，则该方法抛出**InterruptedException**。
- **awaitAdvanceInterruptibly(int phase, long timeout, TimeUnit unit)** 效果与*awaitAdvanceInterruptibly(int phase)*相当，区别在于如果超时则抛出**TimeoutException**。
- **bulkRegister(int parties)** 注册多个party。如果当前phaser已经被终止，则该方法无效，并返回负数。如果调用该方法时，*onAdvance*方法正在执行，则该方法等待其执行完毕。如果该Phaser有父Phaser则指定的party数大于0，且之前该Phaser的party数为0，那么该Phaser会被注册到其父Phaser中。
- **forceTermination()** 强制让该Phaser进入终止状态。已经注册的party数不受影响。如果该Phaser有子Phaser，则其所有的子Phaser均进入终止状态。如果该Phaser已经处于终止状态，该方法调用不造成任何影响。