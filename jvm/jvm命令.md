# JVM命令

## jps  

jps 命令类似与 linux 的 ps 命令，但是它只列出系统中所有的 Java 应用程序。

如果在 linux 中想查看 java 的进程，一般我们都需要 ps -ef | grep java 来获取进程 ID。
如果只想获取 Java 程序的进程，可以直接使用 jps 命令来直接查看。

#### 参数说明

-q：只输出进程 ID
 -m：输出传入 main 方法的参数
 -l：输出完全的包名，应用主类名，jar的完全路径名
 -v：输出jvm参数
 -V：输出通过flag文件传递到JVM中的参数

## jstack

jstack是jdk自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息。

通过使用 jps 命令获取需要监控的进程的pid，然后使用 jstack pid 命令查看线程的堆栈信息。

## jmap 查看内存使用详情

jmap -heap <pid>　　打印堆的使用情况

jmap -dump:live,format=b,file=/tmp/data-cloud-jmapfile.txt pid

  拿到进程的 pid 后，我们就可以用 jmap 来 dump 出堆转储文件了，执行命令如下

```less
jmap -dump:format=b,file=heapdump.phrof pid
```

拿到 dump 文件后我们就可以用 MAT 工具来分析了。
但这个命令在生产上一定要慎用！因为JVM 会将整个 heap 的信息 dump 写入到一个文件，heap 比较大的话会导致这个过程比较耗时，并且执行过程中为了保证 dump 的信息是可靠的，会暂停应用！