# [Tomcat假死的原因及解决方案](https://www.cnblogs.com/Good-Life/p/8980985.html)



在参与搜人项目时，遇到tomcat假死的问题。

当时情况：

1、ps tomcat正在运行

2、用netstat 查看8080连接情况，有大量的close-wait，还有一些等待连接的状态

3、查看服务器的使用情况，没有过多的消耗内存和CPU

4、重新加载界面，没有报错，只是显示加载失败

5、加载时看到tomcat 日志报错 out of memary

在网上查看资料，问题得到解决

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
服务器配置:linux+tomcat

现象：Linux服务器没有崩，有浏览器中访问页面，出现无法访问的情况,没有报4xx或5xx错误(假死),并且重启tomcat后，恢复正常。

原因:tomcat默认最大连接数(线程数)200个,默认每一个连接的生命周期2小时(7200秒),tomcat使用http 1.1协议，而http1.1默认是长连接。tomcat接受处理完请求后，socket没有主动关闭，因此如果在2小时内，请求数超过200个，服务器就会出现上述假死现象。

解决方案1：及时断开socket

解决方案2：修改tomcat配置文件，修改最大连接数(增大)
```

修改server.xml配置文件，Connector节点中增加acceptCount和maxThreads这两个属性的值，并且使acceptCount大于等于maxThreads：

```
`protocol=``"org.apache.coyote.http11.Http11NioProtocol"`
```

解决方案3：修改linux的TCP超时时间(socket生命周期)限制

```
`vi` `/etc/sysctl``.conf``# Decrease the time default value for tcp_fin_timeout connection``net.ipv4.tcp_fin_timeout = 30``# Decrease the time default value for tcp_keepalive_time connection``net.ipv4.tcp_keepalive_time = 1800``# 探测次数``net.ipv4.tcp_keepalive_probes=2``# 探测间隔秒数``net.ipv4.tcp_keepalive_intvl=2` `编辑完 ``/etc/sysctl``.conf,要重启network 才会生效``[root@temp /]``# /etc/rc.d/init.d/network restart`
```