# Docker 的使用

>Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。
>
>Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
>
>容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

## 一、容器

### 1、获取镜像

如果我们本地没有 centos 镜像，我们可以使用 docker pull 命令来载入 centos 镜像

```bash
docker pull centos
```

### 2、启动容器

使用 centos 镜像启动一个容器，参数为以命令行模式进入该容器

```bash
docker run -it centos /bin/bash
```

参数说明：

- **-i**: 交互式操作
- **-t**: 终端
- **centos**: centos 镜像
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash

要退出终端，直接输入 **exit**

```bash
exit
```

![](../../../img/2022/03/2022-03-06_203544.png)

### 3、启动已停止运行的容器

查看所有的容器命令

```bash
docker ps -a
```

![](../../../img/2022/03/2022-03-06_203610.png)

输出详情介绍

- **CONTAINER ID:** 容器 ID
- **IMAGE:** 使用的镜像
- **COMMAND:** 启动容器时运行的命令
- **CREATED:** 容器的创建时间
- **STATUS:** 容器状态，状态有 7 种
  + created（已创建）
  + restarting（重启中）
  + running 或 Up（运行中）
  + removing（迁移中）
  + paused（暂停）
  + exited（停止）
  + dead（死亡）
- **PORTS**: 容器的端口信息和使用的连接类型（tcp\udp）。
- **NAMES:** 自动分配的容器名称。

使用 docker start  （容器 ID 或容器名称）启动一个已停止的容器

```bash
docker start 6a74bbdf323e 
```

### 4、后台运行

在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 **-d** 指定容器的运行模式。加了 **-d** 参数默认不会进入容器

```bash
docker run -itd --name my-centos centos /bin/bash
```

### 5、停止一个容器

停止容器使用 docker stop 命令

```
docker stop fab
```

### 6、重启

容器可以通过 docker restart 重启

```bash
docker restart fab
```

![](../../../img/2022/03/2022-03-06_210511.png)

### 7、进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

### 8、导出容器快照

如果要导出本地某个容器，可以使用 **docker export** 命令

```bash
docker export fab > centos.tar
```

导出容器 fab 快照到本地文件 centos.tar。

### 9、导入容器快照

使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 centos.tar 导入到镜像 centos:v1:

```bash
cat docker/centos.tar | docker import - icentos:v1
```

此外，也可以通过指定 URL 或者某个目录来导入，例如：

```
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

![](../../../img/2022/03/2022-03-06_211459.png)

### 10、删除容器

删除容器使用 **docker rm** 命令：

```
$ docker rm -f 1e560fca390
```

下面的命令可以清理掉所有处于终止状态的容器。

```bash
docker container prune
```

