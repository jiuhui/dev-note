# docker运行postwomen

### 1、拉取镜像

```bash
docker pull liyasthomas/postwoman
```

### 2、运行postwomen

```shell
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
liyasthomas/postwoman   latest              55c513dcdb14        40 hours ago        581MB

$ docker run -d -p3000:3000 55c513dcdb14
d1537e7f7ba25632317e4546ce45512024e397148d49d9a1a0592805385c1daf

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d1537e7f7ba2        55c513dcdb14        "docker-entrypoint.s…"   23 seconds ago      Up 22 seconds       0.0.0.0:3000->3000/tcp   hopeful_galileo
```

参数说明:

- **-d:**让容器在后台运行。
- **-P:**将容器内部使用的网络端口随机映射到我们使用的主机上。

查看容器端口映射

```shell
$ docker port d1537e7f7ba2
3000/tcp -> 0.0.0.0:3000
```

查看运行日志

```shell
$ docker logs -f d1537e7f7ba2
-f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
```

查看容器的进程

```shell
$ docker top d1537e7f7ba2
```

查看docker底层信息，返回一个json串

```shell
$ docker inspect d1537e7f7ba2
```

停止、重启、移除容器

```shell
$ docker stop d1537e7f7ba2
$ docker restart d1537e7f7ba2
$ docker rm d1537e7f7ba2
```

查看容器列表

```shell
$ docker ps -a // 查看容器信息，包括运行的和停止的
$ docker ps // 只有运行的容器
```

删除镜像

```shell
$ docker rmi hello-world
```

