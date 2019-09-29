# cpu使用率高排查

1.使用top 定位到占用CPU高的进程PID

2.获取线程信息，并找到占用CPU高的线程

ps -mp pid -o THREAD,tid,time | sort -rn 

3.将需要的线程ID转换为16进制格式

printf "%x\n" tid

4.打印线程的堆栈信息

jstack pid |grep tid -A 30

一个应用占用CPU很高，除了确实是计算密集型应用之外，通常原因都是出现了死循环。或者队列排队计算。