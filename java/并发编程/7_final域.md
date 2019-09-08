# final域

## final域重排序规则

对于final域，编译器和处理器要遵守两个重排序规则：
1、在构造函数对final域的写入，与随后把这个构造的对象的引用赋值给一个引用变量，这两个操作不能重排序。
2、初次读final域的对象的引用，与随后初次读final域，这两个操作不能重排序。

```java
public class FinalExample {
    int i;                            //普通变量
    final int j;                      //final变量
    static FinalExample obj;
 
    public void FinalExample () {     //构造函数
        i = 1;                        //写普通域
        j = 2;                        //写final域
    }
 
    public static void writer () {    //写线程A执行
        obj = new FinalExample ();
    }
 
    public static void reader () {       //读线程B执行
        FinalExample object = obj;       //读对象引用
        int a = object.i;                //读普通域
        int b = object.j;                //读final域
    }
}
```

这里假设一个线程A执行writer ()方法，随后另一个线程B执行reader ()方法。

对于final域，编译器和处理器要遵守两个重排序规则：
1、在构造函数对final域的写入，与随后把这个构造的对象的引用赋值给一个引用变量，这两个操作不能重排序。
2、初次读final域的对象的引用，与随后初次读final域，这两个操作不能重排序。

写final域的重排序规则
写final域的重排序规则禁止把final域的写重排序到构造函数之外。这个规则的实现包含下面2个方面：
1、JMM禁止编译器把final域的写重排序到构造函数之外。
2、编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外。

读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。在这个示例程序中，如果该引用不为null，那么引用对象的final域一定已经被A线程初始化过了。

## 总结：

为什么final域有这样的语义呢？如果在多线程开发中对一个final变量的读。比如，一个线程当前看到一个整形final域的值为0（还未初始化之前的默认值），过一段时间之后这个线程再去读这个final域的值时，却发现值变为了1（被某个线程初始化之后的值）。

final域对于编译器和处理器来说有两个重排序规则:

1. 写: final域写入和对象引用赋值这两个操作不能重排序 – 保证创建的顺序性
2. 读: 初次读对象的引用和里面的final域不能重排序 – 保证读的顺序性