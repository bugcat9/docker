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

#### 后台运行

前面都是在前景运行，一般来说我们都是让容器在后台中运行，而这也比较简单直接加`-d`即可。

**但是后台运行并不是一直运行**，这和`docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker container ls` 命令来查看容器信息。

比如：

```
[root@VM-0-12-centos ~]# docker run -d  ubuntu:18.04 /bin/echo 'Hello world'
1d5842df536283ab8fa632bf092d95df997e4c11290b66209ed4addb57264921
[root@VM-0-12-centos ~]# docker container logs 1d5842df536283ab8fa632bf092d95df997e4c11290b66209ed4addb57264921
Hello world
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS                          PORTS     NAMES
1d5842df5362   ubuntu:18.04   "/bin/echo 'Hello wo…"   About a minute ago   Exited (0) About a minute ago             naughty_bardeen
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"              24 hours ago         Exited (0) 24 hours ago                   boring_saha
4335143fda62   ubuntu:18.04   "/bin/echo 'Hello wo…"   24 hours ago         Exited (0) 24 hours ago                   peaceful_franklin
d6f6eb64d97c   hello-world    "/hello"                 2 weeks ago          Exited (0) 2 weeks ago                    zen_almeida
```

我们看到我们使用` docker container logs [container ID or NAMES]`查看到后台输出的hello world，并且可以看到输出完容器就自动停止了

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

## 停止容器

终止容器比较简单，可以使用 `docker container stop` 来终止一个运行中的容器。

**比如：**

我们使用`-d`让后台运行一个容器并且查看一下输出情况

```
[root@VM-0-12-centos ~]# docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
fdacd170a371e98f6d81bc5210c7ec889e3170722daba9bfa5f600f06ac43cc0
[root@VM-0-12-centos ~]# docker container ls
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS     NAMES
fdacd170a371   ubuntu:18.04   "/bin/sh -c 'while t…"   About a minute ago   Up About a minute             tender_feynman
[root@VM-0-12-centos ~]# docker container logs fdacd170a371
hello world
hello world
hello world
hello world
hello world
```

然后我们使用 `docker container stop` 

```
[root@VM-0-12-centos ~]# docker container stop  fdacd170a371
fdacd170a371
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                        PORTS     NAMES
fdacd170a371   ubuntu:18.04   "/bin/sh -c 'while t…"   6 minutes ago    Exited (137) 17 seconds ago             tender_feynman
1d5842df5362   ubuntu:18.04   "/bin/echo 'Hello wo…"   12 minutes ago   Exited (0) 12 minutes ago               naughty_bardeen
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"              24 hours ago     Exited (0) 24 hours ago                 boring_saha
4335143fda62   ubuntu:18.04   "/bin/echo 'Hello wo…"   24 hours ago     Exited (0) 24 hours ago                 peaceful_franklin
d6f6eb64d97c   hello-world    "/hello"                 2 weeks ago      Exited (0) 2 weeks ago                  zen_almeida
```

可以看到我们刚刚停下 的容器

## 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

### `attac`命令

下面示例如何使用 `docker attach` 命令。

```
[root@VM-0-12-centos ~]# docker run -dit ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
a70d879fa598: Pull complete 
c4394a92d1f8: Pull complete 
10e6159c56c0: Pull complete 
Digest: sha256:3c9c713e0979e9bd6061ed52ac1e9e1f246c9495aa063619d9d695fb8039aa1f
Status: Downloaded newer image for ubuntu:latest
e1dda0f1ba0102acaef7cc2343a10b8b0cb3376c7761a5db055e093730c3a0f1
[root@VM-0-12-centos ~]#  docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
e1dda0f1ba01   ubuntu    "/bin/bash"   22 seconds ago   Up 21 seconds             zealous_chaplygin
[root@VM-0-12-centos ~]# docker attach e1dd
root@e1dda0f1ba01:/#  
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。

```
root@e1dda0f1ba01:/# exit
exit
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS                      PORTS     NAMES
e1dda0f1ba01   ubuntu         "/bin/bash"              About a minute ago   Exited (0) 16 seconds ago             zealous_chaplygin
fdacd170a371   ubuntu:18.04   "/bin/sh -c 'while t…"   24 hours ago         Exited (137) 23 hours ago             tender_feynman
1d5842df5362   ubuntu:18.04   "/bin/echo 'Hello wo…"   24 hours ago         Exited (0) 24 hours ago               naughty_bardeen
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"              2 days ago           Exited (0) 47 hours ago               boring_saha
4335143fda62   ubuntu:18.04   "/bin/echo 'Hello wo…"   2 days ago           Exited (0) 2 days ago                 peaceful_franklin
d6f6eb64d97c   hello-world    "/hello"                 2 weeks ago          Exited (0) 2 weeks ago                zen_almeida
```

### `exec` 命令

`-i -t`命令

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```
[root@VM-0-12-centos ~]# docker run -dit ubuntu
8a61ea2578cbad7035fd175325b244b30af5daae790a804950183bfb067dcde0
[root@VM-0-12-centos ~]# docker container ls
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
8a61ea2578cb   ubuntu    "/bin/bash"   7 seconds ago   Up 5 seconds             practical_solomon
[root@VM-0-12-centos ~]# docker exec -i 8a61  bash
ls
bin
boot
dev
etc
home
lib
lib32
lib64
libx32
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
exit
[root@VM-0-12-centos ~]# docker exec -it 8a61 bash
root@8a61ea2578cb:/# 
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因。

更多参数说明请使用 `docker exec --help` 查看。

```
[root@VM-0-12-centos ~]# docker exec --help

Usage:  docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a container
  -e, --env list             Set environment variables
      --env-file list        Read in a file of environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format: <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
[root@VM-0-12-centos ~]# 
```

## 删除容器

可以使用 `docker container rm` 来删除一个处于终止状态的容器。我们可以使用`--help`进行查看

```
[root@VM-0-12-centos ~]# docker container rm  --help

Usage:  docker container rm [OPTIONS] CONTAINER [CONTAINER...]

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove anonymous volumes associated with the container
```

就举个简单的例子吧

```
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS     NAMES
8a61ea2578cb   ubuntu         "/bin/bash"              12 minutes ago   Up 12 minutes                         practical_solomon
e1dda0f1ba01   ubuntu         "/bin/bash"              16 minutes ago   Exited (0) 14 minutes ago             zealous_chaplygin
fdacd170a371   ubuntu:18.04   "/bin/sh -c 'while t…"   24 hours ago     Exited (137) 24 hours ago             tender_feynman
1d5842df5362   ubuntu:18.04   "/bin/echo 'Hello wo…"   24 hours ago     Exited (0) 24 hours ago               naughty_bardeen
1fb5652fc3f3   ubuntu:18.04   "/bin/bash"              2 days ago       Exited (0) 2 days ago                 boring_saha
4335143fda62   ubuntu:18.04   "/bin/echo 'Hello wo…"   2 days ago       Exited (0) 2 days ago                 peaceful_franklin
d6f6eb64d97c   hello-world    "/hello"                 2 weeks ago      Exited (0) 2 weeks ago                zen_almeida
[root@VM-0-12-centos ~]# docker container rm zealous_chaplygin 
zealous_chaplygin
[root@VM-0-12-centos ~]# 
```

用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

```
[root@VM-0-12-centos ~]# docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
fdacd170a371e98f6d81bc5210c7ec889e3170722daba9bfa5f600f06ac43cc0
1d5842df536283ab8fa632bf092d95df997e4c11290b66209ed4addb57264921
1fb5652fc3f314c6f929c1e0a237362176f79b3c752ce57b31f164714207f0f5
4335143fda62549f93068749d16ee54783a4d5a3350dc90088e5b4572a869c39
d6f6eb64d97cd20e380053c5ff72ca238cab9bef91ddbab31a92e0cb39d5c221

Total reclaimed space: 12B
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
8a61ea2578cb   ubuntu    "/bin/bash"   14 minutes ago   Up 14 minutes             practical_solomon
[root@VM-0-12-centos ~]# 
```

