# Docker 安装常用应用

## 一、MySQL

### 1、创建目录

```sh
mkdir -p /Users/weizhao/Docker/mysql/data /Users/weizhao/Docker/mysql/logs /Users/weizhao/Docker/mysql/mysql/conf
```

创建三个文件夹：data、logs、conf。具体位置可自行调整。

### 2、拉取镜像

```sh
docker pull mysql:8.0.30
```

也可以不指定版本号拉取最新版本。

### 3、创建容器

```sh
docker run -p 3316:3306 --name mysql -v /Users/weizhao/Docker/mysql/conf:/etc/mysql/conf.d -v /Users/weizhao/Docker/mysql/logs:/logs -v /Users/weizhao/Docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=Qwz#1201 -d mysql:8.0.30
```

## 二、Redis

### 1、拉取镜像

```sh
docker pull redis
```

也可以不指定版本号拉取最新版本。

### 2、创建容器

```sh
docker run -di --name redis -p 6379:6379 redis
```

连接容器中的`Redis`时，只需要连接宿主机的`IP + 指定的映射端口`即可。
