---
title: docker命令
date: 2016-10-24 21:09:11
tags: 虚拟化
categories:
- docker
---
1. docker version

查看docker安装版本
2. docker search 

查找opentsdb相关的镜像
```
$ docker search opentsdb
```
3. docker pull

拉去镜像
```
$ docker pull **/**
```
4. docker ps/build

查看当前机器运行的docker容器

构建镜像
```
docker build -t=truman/redis:3.0.6 .
```
5. docker run
- **不带参数**

```
$ docker run ubuntu /bin/echo 'Hello world'
```

参数 |解释
---|---
docker |告诉操作系统我们要使用docker应用
docker run|组合起来意思就是运行一个docker镜像
ubuntu|镜像(image)名称
/bin/echo 'Hello world'|告诉docker我们要在容器中执行的操作
之后我们就可以看到输出结果：Hello world
- **带参**
```
$ docker run -t -i ubuntu /bin/bash
$ docker run -d -p 127.0.0.1:80:9000 --privileged -v /var/run/docker.sock:/var/run/docker.sock uifd/ui-for-docker
```

参数 |解释
---|---
-t|为这个容器分配一个虚拟的终端
-i|保持对于容器的stdin为打开的状态（输入）。
-d|让docker容器在后台中运行
-p|将docker容器内部端口映射到我们的host上面，我们可以使用 docker port CONTAINER_ID 来查询容器的端口 映射情况
一般情况下 -i 与 -t 参数都是结合在一起使用，这样交互会比较好一点。

- **镜像运行传参**

这个参数是在容器生成的时候传入的，例如：指定hosts
```
docker run -d -p 4244:4242 --name opentsdb5 --add-host sxlab19-0:10.16.238.82 --add-host sxlab19-1:10.16.238.83 --add-host sxlab19-2:10.16.238.84 truman/opentsdb  
```
都是在镜像名字之前传入的，可以写多个
6. docher start/stop/restart

该命令可以操作容器
7. docker rmi

强制删除镜像
```
$ docker rmi -f <img_id>
```
8. docker logs

在容器以守护进程运行的过程中，可以通过docker logs命令查看log日志，具体用法如下：
```
$ docker logs -ft <img_id>
```
以终端模式查看最新log。还有其他命令：docker logs --tail 10 <img_id>获取日志最后10行内容，也可以使用 docker logs --tail 0 -f <img_id>跟踪最新日志
9. 更多命令
```
Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders from a container's filesystem to the host path
    create    Create a new container
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    exec      Run a command in a running container
    export    Stream the contents of a container as a tar archive
    history   Show the history of an image
    images    List images
    import    Create a new filesystem image from the contents of a tarball
    info      Display system-wide information
    inspect   Return low-level information on a container or image
    kill      Kill a running container
    load      Load an image from a tar archive
    login     Register or log in to a Docker registry server
    logout    Log out from a Docker registry server
    logs      Fetch the logs of a container
    pause     Pause all processes within a container
    port      Lookup the public-facing port that is NAT-ed to PRIVATE_PORT
    ps        List containers
    pull      Pull an image or a repository from a Docker registry server
    push      Push an image or a repository to a Docker registry server
    rename    Rename an existing container
    restart   Restart a running container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save an image to a tar archive
    search    Search for an image on the Docker Hub
    start     Start a stopped container
    stats     Display a stream of a containers' resource usage statistics
    stop      Stop a running container
    tag       Tag an image into a repository
    top       Lookup the running processes of a container
    unpause   Unpause a paused container
    version   Show the Docker version information
    wait      Block until a container stops, then print its exit code
```