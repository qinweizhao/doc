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

![2022-03-06_203544](https://img.qinweizhao.com/2022/03/2022-03-06_203544.png)

### 3、启动已停止运行的容器

查看所有的容器命令

```bash
docker ps -a
```

![2022-03-06_203610](https://img.qinweizhao.com/2022/03/2022-03-06_203610.png)

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

![2022-03-06_210511](https://img.qinweizhao.com/2022/03/2022-03-06_210511.png)

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

![2022-03-06_211459](https://img.qinweizhao.com/2022/03/2022-03-06_211459.png)

### 10、删除容器

删除容器使用 **docker rm** 命令：

```
$ docker rm -f 1e560fca390
```

下面的命令可以清理掉所有处于终止状态的容器。

```bash
docker container prune
```

## 11、其他

更多命令直接输入 docker 命令来查看

可以通过命令 **docker command --help** 更深入的了解指定的 Docker 命令使用方法。

例如我们要查看 **docker stats** 指令的具体使用方法：

```bash
docker stats --help
```

![2022-03-06_212339](https://img.qinweizhao.com/2022/03/2022-03-06_212339.png)

## 二、镜像

> 当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。

### 1、列出镜像列表

使用 **docker images** 列出本地主机上的镜像

![2022-03-06_212718](https://img.qinweizhao.com/2022/03/2022-03-06_212718.png)

各个选项说明

- **REPOSITORY：**表示镜像的仓库源
- **TAG：**镜像的标签
- **IMAGE ID：**镜像ID
- **CREATED：**镜像创建时间
- **SIZE：**镜像大小

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本。当有多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。

如果不指定一个镜像的版本标签，将默认使用 xxx:latest 镜像。

### 2、获取一个新的镜像

当本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果想预先下载这个镜像，可以使用 docker pull 命令来下载它。

### 3、查找镜像

可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： **https://hub.docker.com/**

我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个 centos 镜像。我们可以通过 docker search 命令搜索 centos 来寻找适合我们的镜像。

```bash
docker search centos
```

![2022-03-06_213408](https://img.qinweizhao.com/2022/03/2022-03-06_213408.png)

**NAME:** 镜像仓库源的名称

**DESCRIPTION:** 镜像的描述

**OFFICIAL:** 是否 docker 官方发布

**stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思

**AUTOMATED:** 自动构建

### 4、删除镜像

镜像删除使用 **docker rmi** 命令

```bash
docker rmi fab
```

### 5、创建镜像

当从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

- 1、从已经创建的容器中更新镜像，并且提交这个镜像
- 2、使用 Dockerfile 指令来创建一个新的镜像

#### 1. 更新镜像

通过命令 docker commit 来提交容器副本

#### 2. 构建镜像

使用命令 **docker build** ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

#### 3. 设置镜像标签

使用 docker tag 命令，为镜像添加一个新的标签。
