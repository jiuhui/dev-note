#### 拉取镜像

docker pull 拉取镜像
Docker 镜像的拉取使用docker pull命令， 命令格式一般为 docker pull [Registry]/[Repository]/[Image]:[Tag]。

Registry 为注册服务器，Docker 默认会从 docker.io 拉取镜像，如果你有自己的镜像仓库，可以把 Registry 替换为自己的注册服务器。

Repository 为镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，library为 Docker 默认的镜像仓库。

Image 为镜像名称。

Tag 为镜像的标签，如果你不指定拉取镜像的标签，默认为latest。

> 例如，我们需要获取一个 busybox 镜像，可以执行以下命令：
>
> busybox 是一个集成了数百个 Linux 命令（例如 curl、grep、mount、telnet 等）的精简工具箱，只有几兆大小，被誉为 Linux 系统的瑞士军刀。我经常会使用 busybox 做调试来查找生产环境中遇到的问题。

```shell
docker pull busybox
```

实际上执行docker pull busybox命令，都是先从本地搜索，如果本地搜索不到busybox镜像则从 Docker Hub 下载镜像。

#### 重命名镜像

docker tag 重命名镜像

docker tag的命令格式为 docker tag [SOURCE_IMAGE][:TAG] [TARGET_IMAGE][:TAG]。

```shell
$ docker tag busybox:latest mybusybox:latest

docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
mybusybox           latest              018c9d7b792b        3 weeks ago         1.22MB
```

`busybox`和`mybusybox`这两个镜像的 IMAGE ID 是完全一样的。为什么呢？实际上它们指向了同一个镜像文件，只是别名不同而已。

#### 查看镜像

docker image ls 或者docker images 查看本地已存在镜像

```shell
docker images
docker image ls busybox
odcker images | grep busybox
```

#### 删除镜像

docker rmi 命令删除无用镜像

```shell
$ docker rmi mybusybox
Untagged: mybusybox:latest
```

#### 构建镜像

docker build 命令基于Dockerfile构建镜像 推荐的方式    docker commit 命令基于已经运行的容器提交为镜像



