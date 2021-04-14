# docker容器导出和导入

## 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。

```
[root@VM-0-12-centos ~]# docker container ls -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                     PORTS     NAMES
8a61ea2578cb   ubuntu    "/bin/bash"   20 minutes ago   Exited (0) 3 seconds ago             practical_solomon
[root@VM-0-12-centos ~]# docker export 8a61 > ubuntu.tar
[root@VM-0-12-centos ~]# ll
total 73524
-rw-r--r-- 1 root root 75287552 Apr 14 21:26 ubuntu.tar
```

这样将导出容器快照到本地文件。

## 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```
[root@VM-0-12-centos ~]# cat ubuntu.tar | docker import - test/ubuntu:v1.0
sha256:088b6718e34671a4b16883dc8a688d3eef1788111a2be8ba7a77ffabd76130e3
[root@VM-0-12-centos ~]# docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
test/ubuntu   v1.0      088b6718e346   7 seconds ago   72.9MB
ubuntu        latest    26b77e58432b   11 days ago     72.9MB
ubuntu        18.04     3339fde08fc3   2 weeks ago     63.3MB
hello-world   latest    d1165f221234   5 weeks ago     13.3kB
[root@VM-0-12-centos ~]# 
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```
docker import http://example.com/exampleimage.tgz example/imagerepo
```

*注：用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*