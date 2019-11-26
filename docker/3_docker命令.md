# docker命令

### 1、docker容器命令

使用  ubuntu 镜像启动一个容器：

```bash
$ docker run -it ubuntu /bin/bash
```

参数说明：

-  **-i**: 交互式操作。
-  **-t**: 终端。
-  **ubuntu**: ubuntu 镜像。
-  **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

要退出终端，直接输入 **exit**:

查看所有的容器命令

```bash
$ docker ps -a
```

使用 docker start 启动一个已停止得容器：

```bash
$ docker start b750bbbcfd88 
```

后台运行通过 -d指定容器的运行模式

```bash
$ docker run -itd --name ubuntu-test ubuntu /bin/bash
```

停止容器

```bash
docker stop <容器 ID>
```

停止的容器可以通过 docker restart 重启：

```bash
$ docker restart <容器 ID>
```

进入容器 推荐使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

```bash
$ docker attach <容器 ID> 
```

```bash
$ docker exec <容器 ID> 
```

如果要导出本地某个容器，可以使用 **docker export** 命令。

```bash
$ docker export 1e560fca3906 > ubuntu.tar
```

可以使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 ubuntu.tar 导入到镜像 test/ubuntu:v1:

```bash
$ cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```

也可以通过指定 URL 或者某个目录来导入，例如：

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

删除容器

```bash
$ docker rm -f 1e560fca3906
```

清理掉所有处于终止状态的容器。

```bash
$ docker container prune 
```

### 2、docker镜像命令

列出本地镜像列表

```bash
$ docker images
```

使用镜像运行容器

```bash
$ docker run -t -i ubuntu:15.10 /bin/bash 
```

获取一个新镜像，默认是从Docker Hup公共镜像源下载

```bash
$ docker pull ubuntu:13.10
```

查找镜像，可以在**https://hub.docker.com/**网址搜索，也可以用命令

```bash
$  docker search httpd
```

