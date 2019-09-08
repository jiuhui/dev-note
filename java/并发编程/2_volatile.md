# volatile

- ## 可见性

  可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。volatile它在多处理器开发中保证了共享变量的“可见性”。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。
  
- ## volatile的定义

  Java语言规范第3版中对volatile的定义如下：Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁要更加方便。如果一个字段被声明成volatile，Java线程内存模型确保所有线程看到这个变量的值是一致的。

- ## volatile实现原理

  有volatile变量修饰的共享变量进行写操作的时候生成的汇编代码 “lock addl $0×0,” Lock前缀的指令在多核处理器下会引发了两件事情。

  1）将当前处理器缓存行的数据写回到系统内存。

  2）这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

  volatile的两条实现原则

  Lock前缀指令会引起处理器缓存回写到内存

  一个处理器的缓存回写到内存会导致其他处理器的缓存无效。

  > 为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现**缓存一致性协议**，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。 
  
- ## volatile的内存语义

  ### volatile的特性

  理解volatile特性的一个好方法是把对volatile变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步。下面通过具体的示例来说明，示例代码如下。

  ```java
public class VolatileTest {
      volatile long a = 1L;

      public void set(long l){
        a = l;
      }

      public void getAndIncrement(){
          a++;
      }
  
      public long get(){
          return a;
      }
  }
  ```
  
  假如多线程调用上面的三个方法，在语义上和下面的程序等价	：
  
  ```java
  public class VolatileTest {
      long a = 1L;
  
      public synchronized void set(long l){
          a = l;
      }
  
      public void getAndIncrement(){
          long l = get();
          l = l + 1L;
          set(l);
      }
  
      public synchronized long get(){
          return a;
      }
  }
  ```
  
  一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都是使用同一个锁来同步，它们之间的执行效果相同。
  
  锁的happens-before规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
  
  锁的语义决定了临界区代码的执行具有原子性。这意味着，即使是64位的long型和double型变量，只要它是volatile变量，对该变量的读/写就具有原子性。**如果是多个volatile操作或类似于volatile++这种复合操作，这些操作整体上不具有原子性。**
  
  简而言之，volatile变量自身具有下列特性。
  
  ·可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
  
  ·原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。
  
  ### volatile写-读的内存语义
  
  volatile写的内存语义如下。
  
     当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。
  
  volatile读的内存语义如下。
  
     当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。
  
  总结：
  
   	·线程A写一个volatile变量，实质上是线程A向接下来将要读这个volatile变量的某个线程发出了（其对共享变量所做修改的）消息。
  
  ​     ·线程B读一个volatile变量，实质上是线程B接收了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息。
  
     ·线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。
  
- ## volatile内存语义的实现

  在每个 volatile 写操作的前面插入一个 StoreStore 屏障（禁止前面的写与volatile写重排序）。
  在每个 volatile 写操作的后面插入一个 StoreLoad 屏障（禁止volatile写与后面可能有的读和写重排序）。
  在每个 volatile 读操作的后面插入一个 LoadLoad 屏障（禁止volatile读与后面的读操作重排序）。
  在每个 volatile 读操作的后面插入一个 LoadStore 屏障（禁止volatile读与后面的写操作重排序）。

  

volatile禁止重排序优化举例：这是个单例的实现

```java
public class DoubleCheckLock {

    private static DoubleCheckLock instance;

    private DoubleCheckLock(){}

    public static DoubleCheckLock getInstance(){

        //第一次检测
        if (instance==null){
            //同步
            synchronized (DoubleCheckLock.class){
                if (instance == null){
                    //多线程环境下可能会出现问题的地方
                    instance = new DoubleCheckLock();
                }
            }
        }
        return instance;
    }
}
```

`instance = new DoubleCheckLock();`可以分为以下3步完成(伪代码)

```less
memory = allocate(); //1.分配对象内存空间
instance(memory);    //2.初始化对象
instance = memory;   //3.设置instance指向刚分配的内存地址，此时instance！=null
```

由于步骤1和步骤2间可能会重排序，如下：

```less
memory = allocate(); //1.分配对象内存空间
instance = memory;   //3.设置instance指向刚分配的内存地址，此时instance！=null，但是对象还没有初始化完成！
instance(memory);    //2.初始化对象
```

由于步骤2和步骤3不存在数据依赖关系，而且无论重排前还是重排后程序的执行结果在单线程中并没有改变，因此这种重排优化是允许的。但是指令重排只会保证串行语义的执行的一致性(单线程)，但并不会关心多线程间的语义一致性。所以当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题。那么该如何解决呢，很简单，我们使用volatile禁止instance变量被执行指令重排优化即可。

```java
//禁止指令重排优化
  private volatile static DoubleCheckLock instance;
```

volatile除了保证内存可见性，还有个作用是防止指令重排。个人觉得：指令重排是不是最终的思想来源还是内存可见性呢？如果两个互不相关的思想，用到一个事物上，感觉怪怪的。我后来想了想：寄存器和主存的隔离造成了数据的不一致，volatile的初衷是保证数据的强一致性，当赋值基本简单类型的时候，这种一致性很容易实现。但是赋值对象类型的时候，这种一致性分为强一致性和弱一致性，重排是弱一致性，而有序则是强一致性，volatile的目的是强一致性，所以最终它要求指令不得重排。现在我感觉可以把可见性和有序性都统一到一致性上面了。