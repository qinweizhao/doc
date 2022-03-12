# Docker 的使用

>Docker 是一个开源的应用容器引擎，基于 [Go 语言](https://www.runoob.com/go/go-tutorial.html) 并遵从 Apache2.0 协议开源。
>
>Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
>
>容器是完全使用沙箱机制，相互之间不会有任何接口（类似手机的 app），更重要的是容器性能开销极低。

## 一、容器使用

### 1、获取镜像

如果本地没有 centos 镜像，可以使用 docker pull 命令来载入 centos 镜像：

```sh
docker pull centos
```

### 2、启动容器

使用 centos 镜像启动一个容器，参数为以命令行模式进入该容器：

```sh
docker run -it centos /bin/bash
```

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **centos**: centos 镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

要退出终端，直接输入 **exit**：

```sh
exit
```

![2022-03-06_203544](https://img.qinweizhao.com/2022/03/2022-03-06_203544.png)

### 3、启动已停止运行的容器

查看所有的容器命令：

```sh
docker ps -a
```

![2022-03-06_203610](https://img.qinweizhao.com/2022/03/2022-03-06_203610.png)

输出详情介绍：

- **CONTAINER ID:** 容器 ID。
- **IMAGE:** 使用的镜像。
- **COMMAND:** 启动容器时运行的命令。
- **CREATED:** 容器的创建时间。
- **STATUS:** 容器状态，状态有 7 种：
  + created（已创建）
  + restarting（重启中）
  + running 或 Up（运行中）
  + removing（迁移中）
  + paused（暂停）
  + exited（停止）
  + dead（死亡）
- **PORTS**: 容器的端口信息和使用的连接类型（tcp\udp）。
- **NAMES:** 自动分配的容器名称。

使用 docker start  （容器 ID 或容器名称）启动一个已停止的容器：

```sh
docker start 6a74bbdf323e 
```

### 4、后台运行

在大部分的场景下，我们希望 docker 的服务是在后台运行的，我们可以过 **-d** 指定容器的运行模式。加了 **-d** 参数默认不会进入容器：

```sh
docker run -itd --name my-centos centos /bin/bash
```

### 5、停止一个容器

停止容器使用 docker stop 命令：

```
docker stop fab
```

### 6、重启

容器可以通过 docker restart 重启：

```sh
docker restart fab
```

![2022-03-06_210511](https://img.qinweizhao.com/2022/03/2022-03-06_210511.png)

### 7、进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

### 8、导出容器快照

如果要导出本地某个容器，可以使用 **docker export** 命令：

```sh
docker export fab > centos.tar
```

导出容器 fab 快照到本地文件 centos.tar。

### 9、导入容器快照

使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 centos.tar 导入到镜像 centos:v1：

```sh
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

下面的命令可以清理掉所有处于终止状态的容器：

```sh
docker container prune
```

### 11、文件拷贝

如果我们需要将文件拷贝到容器内可以使用`cp`命令。

```sh
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

也可以将文件从容器内拷贝出来。

```sh
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

### 12、目录挂载

我们可以在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器，而且这个操作是双向绑定的，也就是说容器内的操作也会影响到宿主机，实现备份功能。

但是容器被删除的时候，宿主机的内容并不会被删除。如果多个容器挂载同一个目录，其中一个容器被删除，其他容器的内容也不会受到影响。

创建容器添加`-v`参数，格式为宿主机目录:容器目录，例如：

```sh
docker run -di -v /home/ruoyi/data:/usr/local/data --name centos7-01 centos:7

# 多目录挂载
docker run -di -v /宿主机目录:/容器目录 -v /宿主机目录2:/容器目录2 镜像名
```

#### 1. 匿名挂载

匿名挂载只需要写容器目录即可，容器外对应的目录会在`/var/lib/docker/volumes`中生成。

```sh
# 匿名挂载
docker run -di -v /usr/local/data --name centos7-02 centos:7
# 查看 volume 数据卷信息
docker volume ls
```

#### 2. 具名挂载

具名挂载就是给数据卷起了个名字，容器外对应的目录会在`/var/lib/docker/volume`中生成。

```sh
# 匿名挂载
docker run -di -v docker_centos_data:/usr/local/data --name centos7-03 centos:7
# 查看 volume 数据卷信息
docker volume ls
```

#### 3. 指定目录挂载

之前挂载方式就属于指定目录挂载，这种方式的挂载不会在`/var/lib/docker/volume`目录生成内容。

```sh
docker run -di -v /mydata/docker_centos/data:/usr/local/data --name centos7-01 centos:7
# 多目录挂载
docker run -di -v /宿主机目录:/容器目录 -v /宿主机目录2:/容器目录2 镜像名
```

#### 4. 查看目录挂载关系

通过`docker volume inspect`数据卷名称 可以查看该数据卷对应宿主机的目录地址。

执行命令：`docker volume inspect docker_centos_data`

通过`docker inspect`容器ID或名称 ，在返回的`JSON`节点中找到`Mounts`，可以查看详细的数据挂载信息。

#### 5. 只读/读写

```sh
# 只读。只能通过修改宿主机内容实现对容器的数据管理。
docker run -it -v /宿主机目录:/容器目录:ro 镜像名

# 读写，默认。宿主机和容器可以双向操作数据。
docker run -it -v /宿主机目录:/容器目录:rw 镜像名
```

### 11、其他

更多命令直接输入 docker 命令来查看。

可以通过命令 **docker command --help** 更深入的了解指定的 Docker 命令使用方法。

例如我们要查看 **docker stats** 指令的具体使用方法：

```sh
docker stats --help
```

![2022-03-06_212339](https://img.qinweizhao.com/2022/03/2022-03-06_212339.png)

## 二、镜像使用

> 当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。镜像都是存储在 Docker 宿主机的 /var/lib/docker 目录下。

### 1、列出镜像列表

使用 **docker images** 列出本地主机上的镜像：

![2022-03-06_212718](https://img.qinweizhao.com/2022/03/2022-03-06_212718.png)

各个选项说明

- **REPOSITORY:** 表示镜像的仓库源。
- **TAG:** 镜像的标签。
- **IMAGE ID:** 镜像ID。
- **CREATED:** 镜像创建时间。
- **SIZE:** 镜像大小。

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本。当有多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。

如果不指定一个镜像的版本标签，将默认使用 xxx:latest 镜像。

### 2、获取一个新的镜像

当本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果想预先下载这个镜像，可以使用 docker pull 命令来下载它。

### 3、查找镜像

可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： **https://hub.docker.com/**。

我们也可以使用 docker search 命令来搜索镜像。比如我们需要一个 centos 镜像。我们可以通过 docker search 命令搜索 centos 来寻找适合我们的镜像。

```sh
docker search centos
```

![2022-03-06_213408](https://img.qinweizhao.com/2022/03/2022-03-06_213408.png)

**NAME:** 镜像仓库源的名称。

**DESCRIPTION:** 镜像的描述。

**OFFICIAL:** 是否 docker 官方发布。

**STARS:** 类似 Github 里面的 star，表示点赞、喜欢的意思。

**AUTOMATED:** 自动构建。

### 4、删除镜像

镜像删除使用 **docker rmi** 命令：

```sh
docker rmi fab
```

### 5、创建镜像

当从 docker 镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

- 1、从已经创建的容器中更新镜像，并且提交这个镜像。
- 2、使用 Dockerfile 指令来创建一个新的镜像。

#### 1. 更新镜像

在运行的容器中新增一个 wz.txt 文件，然后退出：

```sh
touch wz.txt
```

```sh
exit
```

将定制后的容器通过命令 docker commit 来提交容器副本：

```sh
docker commit -m="add file" -a="qwz" fab wzcentos
```

参数说明：

- **-m:** 提交的描述信息。
- **-a:** 指定镜像作者。
- **fab:** 容器 ID。
- **wzcentos:** 指定要创建的目标镜像名。

#### 2. 构建镜像

使用命令 docker build ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

详细参考：Dockerfile

```sh
docker build -t bcentos:b .
```

参数说明：

- **-t** ：指定要创建的目标镜像名。
- **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径。

### 6、设置镜像标签

使用 docker tag 命令，为镜像添加一个新的标签：

```sh
docker tag 339 wzcentos:a
```

![2022-03-07_105226](https://img.qinweizhao.com/2022/03/2022-03-07_105226.png)

## 三、容器连接

### 1、网络端口映射

创建一个 web 应用的容器（镜像使用的是 halo 博客）：

```sh
docker run -it -d --name halo -p 8090:8090 halohub/halo:latest
```

使用 **-p** 标识来指定容器端口绑定到主机端口，也可以指定容器绑定的网络地址，比如绑定 127.0.0.1：

```sh
docker run -it -d --name ihalo -p 127.0.0.1:8090:8090 halohub/halo:latest
```

默认都是绑定 tcp 端口，如果要绑定 UDP 端口，可以在端口后面加上 **/udp**：

```sh
docker run -it -d --name uhalo -p 127.0.0.1:9090:8090/udp halohub/halo:latest
```

**docker port** 命令可以让我们快捷地查看端口的绑定情况：

```sh
docker port cb8
```

![2022-03-07_181506](https://img.qinweizhao.com/2022/03/2022-03-07_181506.png)

### 2、容器互联

端口映射并不是唯一把 docker 连接到另一个容器的方法。docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。

#### 1. 新建网络

```sh
docker network create -d bridge test-net
```

参数说明：

**-d**：参数指定 Docker 网络类型，有 bridge、overlay（用于 Swarm mode）。

#### 2. 连接容器

```sh
docker run -itd --name test1 --network test-net centos /bin/bash
```

再运行一个容器并加入到 test-net 网络:

```sh
docker run -itd --name test2 --network test-net centos /bin/bash
```

下面通过 ping 来证明 test1 容器和 test2 容器建立了互联关系。

![2022-03-07_184643](https://img.qinweizhao.com/2022/03/2022-03-07_184643.png)

如果有多个容器之间需要互相连接，推荐使用 Docker Compose。

#### 3. 配置 DNS

在宿主机（Linux）的 /etc/docker/daemon.json 文件中增加以下内容来设置全部容器的 DNS：

```json
{
  "dns" : [
    "114.114.114.114"
  ]
}
```

设置后，启动容器的 DNS 会自动配置为 114.114.114.114 和 8.8.8.8。

配置完，需要重启 docker 才能生效。

查看容器的 DNS 是否生效可以使用以下命令，它会输出容器的 DNS 信息：

```sh
docker run -it --rm  centos  cat etc/resolv.conf
```

如果只想在指定的容器设置 DNS，则可以使用以下命令：

```sh
docker run -it --rm -h host_centos  --dns=8.8.8.8 --dns-search=test.com centos
```

![2022-03-07_200016](https://img.qinweizhao.com/2022/03/2022-03-07_200016.png)

参数说明：

**--rm**：容器退出时自动清理容器内部的文件系统。

**-h HOSTNAME 或者 --hostname=HOSTNAME**： 设定容器的主机名，它会被写到容器内的 /etc/hostname 和 /etc/hosts。

**--dns=IP_ADDRESS**： 添加 DNS 服务器到容器的 /etc/resolv.conf 中，让容器用这个服务器来解析所有不在 /etc/hosts 中的主机名。

**--dns-search=DOMAIN**： 设定容器的搜索域，当设定搜索域为 .example.com 时，在搜索一个名为 host 的主机时，DNS 不仅搜索 host，还会搜索 host.example.com。

如果在容器启动时没有指定 **--dns** 和 **--dns-search**，Docker 会默认用宿主主机上的 /etc/resolv.conf 来配置容器的 DNS。

## 四、仓库管理

仓库（Repository）是集中存放镜像的地方，目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)。

注册地址： [https://hub.docker.com](https://hub.docker.com/)

### 1、登录和退出

登录需要输入用户名和密码，登录成功后，我们就可以从 docker hub 上拉取自己账号下的全部镜像。

```sh
docker login
```

退出 docker hub 可以使用以下命令：

```sh
docker logout
```

### 2、推送镜像

用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。

```sh
docker push qinweizhao/centos:7
```

![2022-03-07_211825](https://img.qinweizhao.com/2022/03/2022-03-07_211825.png)

结果：
![2022-03-07_211913](https://img.qinweizhao.com/2022/03/2022-03-07_211913.png)
