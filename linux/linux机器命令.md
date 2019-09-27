# Linux机器命令

### CENTER OS7防火墙

```shell
// 这些命令立即生效，机器重启失效
firewall-cmd --state    //查看防火墙是否关闭
systemctl start firewalld   //开启防火墙
systemctl stop firewalld   // 关闭防火墙
```

### read hat 防火墙

```sh
// 这些命令立即生效，机器重启失效
service  iptables status  // 查看防火墙是否关闭
service  iptables stop		//关闭防火墙
service iptables start   // 开启防火墙
```

