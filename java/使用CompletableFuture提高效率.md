# 使用CompletableFuture提高效率

### 异步计算

- 异步调用其实就是实现一个可无需等待被调用函数的返回值而让操作继续运行的方法。在 Java 语言中，简单的讲就是另启一个线程来完成调用中的部分计算，使调用继续运行或返回，而不需要等待计算结果。但调用者仍需要取线程的计算结果。
- JDK5新增了Future接口，用于描述一个异步计算的结果。虽然Future 以及相关使用方法提供了异步执行任务的能力，但是对于结果的获取却是很不方便，只能通过阻塞或者轮询的方式得到任务的结果。阻塞的方式显然和我们的异步编程的初衷相违背，轮询的方式又会耗费无谓的CPU 资源，而且也不能及时地得到计算结果。

```java
/**
 * 用Future获取一个异步任务的结果
 * @author: fanjiuhui
 * @Date: 2019/8/8
 * @Description:
 */
public class FutureAsync {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        ExecutorService executor = Executors.newCachedThreadPool();
        Future<String> future = executor.submit(() -> {
            Thread.sleep(5000);
            return "future";
        });

        System.out.println(future.get());
        System.out.println("finshed");

    }
}
```

### Function、Predicate、Consumer、Supplier接口

这些接口都有一个@FunctionalInterface注解，表明这个接口将是一个函数式接口，里面只能有一个抽象方法

```java
package com.unisound.dcs.project.async;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * @author: fanjiuhui
 * @Date: 2019/8/8
 * @Description:
 */
public class Test {
    public static void main(String[] args) {

       // 这些接口都有一个@FunctionalInterface注解，表明这个接口将是一个函数式接口，里面只能有一个抽象方法
       //  Function 接受一个输入参数，返回一个结果
        Function<Integer,String> function = (x) -> "result"+x;
        String apply = function.apply(1);
        System.out.println(apply);

        // Predicate 接受一个输入参数，返回布尔值
        Predicate<Integer> predicate = (x) -> 1 == x;
        boolean test = predicate.test(1);
        System.out.println(test);

        // Consumer  接受一个输入参数，无返回值
        Consumer<String> consumer = (str) -> System.out.println("str is " +str);
        consumer.accept("123");

        // Supplier  无输入参数，返回一个结果
        Supplier<String> supplier = () -> "supplier";
        String s = supplier.get();
        System.out.println(s);


    }
}

====================输出结果===============================================================
result1
true
str is 123
supplier

Process finished with exit code 0
```

### CompletableFuture

```java
 *
 * @author Doug Lea
 * @since 1.8
 */
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
```

可以看到实现了Future和CompletionStage接口

#### 创建CompletableFuture

```java
    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the {@link ForkJoinPool#commonPool()} with
     * the value obtained by calling the given Supplier.
     *
     * @param supplier a function returning the value to be used
     * to complete the returned CompletableFuture
     * @param <U> the function's return type
     * @return the new CompletableFuture
     */
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
        return asyncSupplyStage(asyncPool, supplier);
    }

    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the given executor with the value obtained
     * by calling the given Supplier.
     *
     * @param supplier a function returning the value to be used
     * to complete the returned CompletableFuture
     * @param executor the executor to use for asynchronous execution
     * @param <U> the function's return type
     * @return the new CompletableFuture
     */
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                       Executor executor) {
        return asyncSupplyStage(screenExecutor(executor), supplier);
    }

    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the {@link ForkJoinPool#commonPool()} after
     * it runs the given action.
     *
     * @param runnable the action to run before completing the
     * returned CompletableFuture
     * @return the new CompletableFuture
     */
    public static CompletableFuture<Void> runAsync(Runnable runnable) {
        return asyncRunStage(asyncPool, runnable);
    }

    /**
     * Returns a new CompletableFuture that is asynchronously completed
     * by a task running in the given executor after it runs the given
     * action.
     *
     * @param runnable the action to run before completing the
     * returned CompletableFuture
     * @param executor the executor to use for asynchronous execution
     * @return the new CompletableFuture
     */
    public static CompletableFuture<Void> runAsync(Runnable runnable,
                                                   Executor executor) {
        return asyncRunStage(screenExecutor(executor), runnable);
    }

```



thenApply 当前阶段正常完成以后执行，而且当前阶段的执行的结果会作为下一阶段的输入参数。thenApplyAsync默认是异步执行的。这里所谓的异步指的是不在当前线程内执行。

如下：

```java

/**
 * @author: fanjiuhui
 * @Date: 2019/8/8
 * @Description:
 */
public class CompletableFutureAsync {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> integerCompletableFuture = CompletableFuture.supplyAsync(() -> 1)
                // thenApply 相当于回调函数callback
                .thenApply((i) ->i+1)
                // thenApplyAsync 异步执行的callback
                .thenApplyAsync((i) -> i*2);
        System.out.println(integerCompletableFuture.get());


        // String 的例子
        String s = CompletableFuture.supplyAsync(() -> "hello")
                .thenApply((x) -> x + " word")
                .thenApply(String::toUpperCase)
                .get();
        System.out.println(s);
    }
}
=====================================输出结果===========================================
    4
HELLO WORD

Process finished with exit code 0
```

#### thenAcccept和thenRun

- 可以看到，thenAccept和thenRun都是无返回值的。如果说thenApply是不停的输入输出的进行生产，那么thenAccept和thenRun就是在进行消耗。它们是整个计算的最后两个阶段。
- 同样是执行指定的动作，同样是消耗，二者也有区别：
  - thenAccept接收上一阶段的输出作为本阶段的输入 　　
  - thenRun根本不关心前一阶段的输出，根本不不关心前一阶段的计算结果，因为它不需要输入参数

```java
 CompletableFuture.supplyAsync(() -> "hello")
                .thenApply((x) -> x + " word")
                .thenApply(String::toUpperCase)
                .thenAccept(str -> System.out.println("accept" + str))
                .thenRun(() -> System.out.println("finished"));
```

#### thenCombine整合两个计算结果

```java
 CompletableFuture.supplyAsync(() -> "hello")
                .thenApply((x) -> x + " word")
                .thenApply(String::toUpperCase)
                .thenCombine(CompletableFuture.supplyAsync(() -> " java"),(s1,s2) -> s1 +s2)
                .thenApply(String::toUpperCase)
                .thenAccept(System.out::println);
```

