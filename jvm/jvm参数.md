# jvm参数

jvm参数固定以*-XX:*开始

-XX:+<option>   表示开启option选项

-XX:-<option>    表示关闭option选项

-XX:-<option>=<value>   表示将option选项赋值为value

-XX:+TraceClassUnloading 打印类的卸载信息





-XX:+PrintGCDetails -XX:+PrintGCDateStamps   打印gc信息，看有没有full gc