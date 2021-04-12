# docker容器操作

## 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`exited`）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

### 新建并启动

所需要的命令主要为 `docker run`。可以使用`docker run --help`进行查看

```
[root@VM-0-12-centos ~]# docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
。。。后面内容较多就不展示了
```

举例1：下面的命令输出一个 “Hello World”，之后终止容器。

```
[root@VM-0-12-centos ~]# docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world
[root@VM-0-12-centos ~]# 
```

这跟在本地直接执行 `/bin/echo 'hello world'` 几乎感觉不出任何区别。

下面的命令则启动一个 bash 终端，允许用户进行交互。

```
[root@VM-0-12-centos ~]# docker run -t -i ubuntu:18.04 /bin/bash
root@1fb5652fc3f3:/# 
```

其中，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如

```
[root@VM-0-12-centos ~]# docker run -t -i ubuntu:18.04 /bin/bash
root@1fb5652fc3f3:/# pwd
/
root@1fb5652fc3f3:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@1fb5652fc3f3:/# 
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从 [registry](https://vuepress.mirror.docker-practice.com/repository/) 下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

在容器中我们可以使用`exit`退出

```
root@1fb5652fc3f3:/# exit
exit
[root@VM-0-12-centos ~]#
```

接着可以查看一下容器

```
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS     NAMES
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"              5 minutes ago   Exited (0) 4 minutes ago             boring_saha
4335143fda62   ubuntu:18.04   "/bin/echo 'Hello wo…"   6 minutes ago   Exited (0) 6 minutes ago             peaceful_franklin
d6f6eb64d97c   hello-world    "/hello"                 2 weeks ago     Exited (0) 2 weeks ago               zen_almeida
[root@VM-0-12-centos ~]# 
```

可以看到我们刚才创建了两个容器。

查看正在运行的容器使用`docker container ls`

查看所有容器(包括已经停止的)使用`docker container ls -a`

### 启动已终止容器

可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行。

```
[root@VM-0-12-centos ~]# docker container start 1fb5652fc3f3
1fb5652fc3f3
[root@VM-0-12-centos ~]# docker container ls 
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS          PORTS     NAMES
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"   8 minutes ago   Up 11 seconds             boring_saha
```

当然其实可以直接使用`docker start 1fb5652fc3f3`,但是我感觉加一个`container`显得更加清楚

更多