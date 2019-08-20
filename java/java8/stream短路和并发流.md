# Stream短路和并发流

流的操作类似于sql，是描述性的

select name from student where age > 20 and address = "beijing" order by age desc;

students.stream().filter(student -> student.getAge > 20).filter(student -> student.getAddress().equals("beijing")).sorted(...).forEach(student ->System.out.println(name));

流操作和集合操作的不同：

集合关注的是数据和数据存储本身

流关注的是对数据的计算

list.stream().sorted().count();这是串行流

list.parallelStream().sorted().count();这是并行流 



Stream短路

List<String> list  = Arrays.asList("hello","word","hello word");

//打印出长度是5的第一个单词

```java
List<String> list = Arrays.asList("hello","world","hello world");
// 打印出长度是五的第一个单词
list.stream().mapToInt(item -> item.length()).filter(len -> len ==5).findFirst().ifPresent(System.out::println);
```

程序没有问题输出的是5

改造一下成为如下代码

```java
List<String> list = Arrays.asList("hello","world","hello world");
        // 打印出长度是五的第一个单词
        //list.stream().mapToInt(item -> item.length()).filter(len -> len ==5).findFirst().ifPresent(System.out::println);

        list.stream().mapToInt(item ->{
            int len = item.length();
            System.out.println(item);
            return len;
        }).filter(len -> len == 5).findFirst().ifPresent(System.out::println);
```

程序的输出结果是 hello   5 

并不是集合循环时输出的 hello world   hello world 为什么呢。

如果把程序修改一下呢 第一个元素不是hello 我们改为 hello1

```java
List<String> list = Arrays.asList("hello1","world","hello world");
        // 打印出长度是五的第一个单词
        //list.stream().mapToInt(item -> item.length()).filter(len -> len ==5).findFirst().ifPresent(System.out::println);

        list.stream().mapToInt(item ->{
            int len = item.length();
            System.out.println(item);
            return len;
        }).filter(len -> len == 5).findFirst().ifPresent(System.out::println);
```

输出的结果是 hello1 world 5

因为流里面只放的是操作计算，对第一个元素先mapInt,再对第一个元素 filter，这些操作都完了才会对第二个元素操作，但是我们的程序里面第一个元素操作完就是我们要的结果就不会对第二个操作了，第一个不符合，第二个符合就不会对第三个操作了，这就是stream的短路。



一个flatMap的应用

```java
List<String> list1 = Arrays.asList("hello welcome","world hello","hello word hello");
        // 把集合中的单词去重然后输出 这里用到flatMap
        list1.stream().map(item -> item.split(" ")).flatMap(Arrays::stream)
                .distinct().collect(Collectors.toList()).forEach(System.out::println);
```

