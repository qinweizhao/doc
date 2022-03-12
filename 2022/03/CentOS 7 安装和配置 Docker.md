# CentOS 7 安装和配置 Docker

## 一、文档

官方文档：https://docs.docker.com/get-docker 。

## 二、安装

### 1、卸载

Docker 的旧版本被称为 Docker 或 Docker Engine 。如果安装了Docker或Docker Engine，需要先卸载：

```sh
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2、安装软件包

安装所需的软件包：

```sh
yum install -y yum-utils
```

### 3、设置存储库

官方地址：

```sh
# 官方地址（比较慢）
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

或（建议）：

```sh
# 阿里云地址（国内地址，相对更快）
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 4、安装 Docker 引擎

```sh
yum install docker-ce docker-ce-cli containerd.io
```

### 5、验证

```sha
docker --version
```

### 6、启动

```sh
systemctl start docker
```

## 三、配置

Docker 默认拉取镜像是从这里拉取(https://hub.docker.com)，国外地址拉取的速度比较慢。

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

- 科大镜像：**https://docker.mirrors.ustc.edu.cn/**
- 网易：**https://hub-mirror.c.163.com/**
- 阿里云：**https://<你的ID>.mirror.aliyuncs.com**
- 七牛云加速器：**https://reg-mirror.qiniu.com**

当配置某一个加速器地址之后，若发现拉取不到镜像，请切换到另一个加速器地址。国内各大云服务商均提供了 Docker 镜像加速服务，建议根据运行 Docker 的云平台选择对应的镜像加速服务。

阿里云镜像获取地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors，并有具体的操作方法：

![2022-03-12_223214](https://img.qinweizhao.com/2022/03/2022-03-12_223214.png)



