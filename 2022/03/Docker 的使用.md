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

> 当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。

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

使用命令 **docker build** ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

如何创建 Dockerfile 下文详细描述......

```sh
docker build -t bcentos:b .
```

参数说明：

- **-t** ：指定要创建的目标镜像名。
- **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径。

### 6、 设置镜像标签

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

#### 3.配置 DNS

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

结果：![2022-03-07_211913](https://img.qinweizhao.com/2022/03/2022-03-07_211913.png)

## 五、Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

### 1、指令详解

#### 1. COPY

复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

格式：

```
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

**[--chown=<user>:<group>]**：可选参数，用户改变复制到容器内文件的拥有者和属组。

**<源路径>**：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：

```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

**<目标路径>**：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

#### 2. ADD

ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：

- ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
- ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

#### 3. CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式：

```
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

#### 4. ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是，如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序。

**优点**：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意**：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：

```
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参。

#### 5. ENV

设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：

```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：

```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

#### 6. ARG

构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：

```
ARG <参数名>[=<默认值>]
```

#### 7. VOLUME

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

格式：

```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

#### 8. EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```
EXPOSE <端口1> [<端口2>...]
```

#### 9. WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：

```
WORKDIR <工作目录路径>
```

#### 10. USER

用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：

```
USER <用户名>[:<用户组>]
```

#### 11. HEALTHCHECK

用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：

```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

#### 12. ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：

```
ONBUILD <其它指令>
```

#### 13. LABEL

LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：

```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

比如我们可以添加镜像的作者：

```
LABEL org.opencontainers.image.authors="runoob"
```

### 2、 构建镜像

`Dockerfile`文件编写好以后，真正构建镜像时需要通过`docker build`命令。

`docker build`命令用于使用`Dockerfile`创建镜像。

```sh
# 使用当前目录的 Dockerfile 创建镜像
docker build -t mycentos:7 .

# 通过 -f Dockerfile 文件的位置创建镜像
docker build -f /home/ruoyi/docker/Dockerfile -t mycentos:7 .
```

- -f：指定要使用的 Dockerfile 路径；
- --tag, -t：镜像的名字及标签，可以在一次构建中为一个镜像设置多个标签。

### 3、实践

接下来我们通过基础镜像`centos:7`，在该镜像中安装`jdk`和`tomcat`以后将其制作为一个新的镜像`mycentos:7`

创建目录，编写`Dockerfile`文件

```sh
mkdir -p /usr/local/`dockerfile`
```

执行命令：`vi Dockerfile`，写入信息。

```sh
# 指明构建的新镜像是来自于`centos:7`基础镜像
FROM centos:7
# 通过镜像标签声明了作者信息
LABEL maintainer="wz"

# 设置工作目录
WORKDIR /usr/local
# 新镜像构建成功以后创建指定目录
RUN mkdir -p /usr/local/java && mkdir -p /usr/local/tomcat
# 拷贝文件到镜像中并解压
ADD jdk-8u111-linux-x64.tar.gz /usr/local/java
ADD apache-tomcat-8.5.27.tar.gz /usr/local/tomcat
# 暴露容器运行时的 8080 监听端口给外部
EXPOSE 8080
# 设置容器内 JAVA_HOME 环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_111
ENV PATH $PATH:$JAVA_HOME/bin
# 启动容器时启动 tomcat
CMD ["/usr/local/tomcat/apache-tomcat-8.5.27/bin/catalina.sh", "run"]
```

构建镜像

```sh
docker build -f /home/ruoyi/docker/Dockerfile -t mycentos:test .
```

启动镜像

```sh
docker run -di --name mycentos -p 8080:8080 mycentos:test
```

进入容器

```sh
docker exec -it mycentos7 /bin/bash
```

### 4、镜像构建历史

```
docker history 镜像名称:标签|ID
docker history mycentos:7
```

### 5、附：指令说明简洁版

>- FROM
>
>构建镜像基于哪个镜像
>
>- MAINTAINER
>
>镜像维护者姓名或邮箱地址
>
>- RUN
>
>构建镜像时运行的指令
>
>- CMD
>
>运行容器时执行的shell环境
>
>- VOLUME
>
>指定容器挂载点到宿主机自动生成的目录或其他容器
>
>- USER
>
>为RUN、CMD、和 ENTRYPOINT 执行命令指定运行用户
>
>- WORKDIR
>
>为 RUN、CMD、ENTRYPOINT、COPY 和 ADD 设置工作目录，就是切换目录
>
>- HEALTHCHECH
>
>健康检查
>
>- ARG
>
>构建时指定的一些参数
>
>- EXPOSE
>
>声明容器的服务端口（仅仅是声明）
>
>- ENV
>
>设置容器环境变量
>
>- ADD
>
>拷贝文件或目录到容器中，如果是URL或压缩包便会自动下载或自动解压
>
>- COPY
>
>拷贝文件或目录到容器中，跟ADD类似，但不具备自动下载或解压的功能
>
>- ENTRYPOINT
>
>运行容器时执行的shell命令
