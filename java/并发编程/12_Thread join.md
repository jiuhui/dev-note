# Thread join()方法的作用

```java
package com.unisound.dcs.project.concurrent;

/**
 * @author: fanjiuhui
 * @Date: 2019/8/29
 * @Description:
 */
public class ThreadJoinTest {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Thread1());
        thread1.start();
    }
}

class Thread1 implements Runnable{

    @Override
    public void run() {
        Thread thread2 = new Thread(new Thread2());
        thread2.start();
        System.out.println("Thread1 execute");
    }
}

class Thread2 implements Runnable{

    @Override
    public void run() {
        System.out.println("thread2 execute");
    }
}

```

上面代码的执行结果是：

```less
Thread1 execute
thread2 execute
```

给thread2.start()后面加上join()方法：

```java
package com.unisound.dcs.project.concurrent;

/**
 * @author: fanjiuhui
 * @Date: 2019/8/29
 * @Description:
 */
public class ThreadJoinTest {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Thread1());
        thread1.start();
    }
}

class Thread1 implements Runnable{

    @Override
    public void run() {
        Thread thread2 = new Thread(new Thread2());
        thread2.start();
        try {
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread1 execute");
    }
}

class Thread2 implements Runnable{

    @Override
    public void run() {
        System.out.println("thread2 execute");
    }
}

```

执行结果是：

```less
thread2 execute
Thread1 execute
```

说明：1、Thread2没加join()方法：首先 thread1.start()执行之后 cpu是由thread1占有的，thread1执行到 Thread2.start(),Thread2到就绪状态Runable，但是还没有获得cpu，只有当Thread1执行完了释放了cpu thread2才获取cpu到运行状态 Running.所以第一个程序的结果是先打印thread1 execute 最后打印thread2 execute.

2、Thread2加上了join()方法：首先 thread1.start()执行之后 cpu是由thread1占有的，thread1执行到 Thread2.start(),Thread2到就绪状态Runable，但是还没有获得cpu，接下来执行thread2.join(); thread1就会挂起，thread2获得cpu，thread2执行完了 thread1 才能再次获得cpu继续执行（执行System.out.println("Thread1 execute");）所以结果就是先打印thread2 execute 再打印 thread1 execute