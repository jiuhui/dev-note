# CentOS 安装docker

- ### 卸载旧版本

  ```less
  $ sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-engine
  ```

  

- ### 安装 Docker Engine-Community

  #### 使用 Docker 仓库进行安装

  在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker。

   **设置仓库**

  安装所需的软件包。yum-utils 提供了 yum-config-manager  ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

  ```less
  $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
  ```

  使用以下命令来设置稳定的仓库。

  ```less
  $ sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
  ```

  #### 安装 Docker Engine-Community

  安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

  ```less
  $ sudo yum install docker-ce docker-ce-cli containerd.io
  ```

  1、列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序。

  ```less
  $ yum list docker-ce --showduplicates | sort -r
  
  docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
  docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
  docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
  docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
  ```

  2、通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。

  ```less
  $ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
  ```

  

- ### 启动 Docker

  ```less
  $ sudo systemctl start docker
  ```

  通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。

  ```less
  $ sudo docker run hello-world
  ```

  