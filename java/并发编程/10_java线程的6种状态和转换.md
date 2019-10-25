# Java线程 Thread 的6种状态以及转变过程

Thread的类里面的State这个枚举标识线程的6种状态：

```java
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```

- ### NEW：新建状态。

  调用了Thread thread = new Thread(Runnale)方法之后还没有调用start()方法的时候

- ### RUNNABLE：可运行状态

  调用了start()方法之后，可运行状态的线程在java虚拟机中执行，但它可能等待来自操作系统的其他资源如处理器

- ### BLOCKED：阻塞状态

  线程的阻塞状态是它正在等待monitor lock。在这个状态的线程正在等待获取monitor lock进入同步代码块/方法、或在调用object.wait后重新输入同步块/方法。

- ### WAITING：等待状态

  线程调用object.wait thread.join LockSupport.park 的时候变成 WAITING

- ### TIMED_WAITING：时间等待状态

  执行线程的sleep 或者wait,join带时间参数等方法时

- ### TERMINATED：终止状态

  线程执行完成 或者被中断的时候变成TERMINATED.

  

  > 线程创建出来就是new的状态。
  > start方法调用之后状态变成runnable,可以执行，如果cpu把时间片切过来就可以执行。
  > runnable执行完成则变成terminated，或者中断的时候。
  > runnable如果遇到有锁的方法，在等待的时候则是阻塞blocked的状态
  > 如果调用wait或者join的方法，则变成waiting状态，
  > 如果调用带时间参数的wait或者join,则变成time_waiting。

  

  

  

  