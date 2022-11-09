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

### 2、创建容器

```sh
docker run -di --name redis -p 6379:6379 redis
```

连接容器中的`Redis`时，只需要连接宿主机的`IP + 指定的映射端口`即可。

## 三、Nginx

### 1、创建目录

```sh
mkdir -p /Users/weizhao/Docker/nginx
```

### 2、拉取镜像

```sh
docker pull nginx
```

### 3、创建容器

```sh
docker run -di --name nginx -p 80:80 nginx
```

### 4、拷贝配置文件

```sh
docker cp nginx:/etc/nginx /Users/weizhao/Docker/nginx/conf
```

### 5、终止并删除容器

```sh
docker stop nginx
docker rm nginx
```

### 6、重新创建容器

```sh
docker run -di --name nginx -p 80:80 -v /Users/weizhao/Docker/nginx/conf:/etc/nginx nginx
```

## 四、ZooKeeper

### 1、创建目录

```sh
mkdir -p /Users/weizhao/Docker/zookeeper
```

具体位置可自行调整。

### 2、拉取镜像

```sh
docker pull zookeeper:3.8.0
```

### 3、创建容器

```sh
docker run -d -e TZ="Asia/Shanghai" -p 2181:2181 -v /Users/weizhao/Docker/zookeeper:/data --name zookeeper --restart always zookeeper:3.8.0
```

说明：

```txt
-e TZ="Asia/Shanghai" # 指定上海时区 
-d # 表示在一直在后台运行容器
-p 2181:2181 # 对端口进行映射，将本地2181端口映射到容器内部的2181端口
--name # 设置创建的容器名称
-v # 将本地目录(文件)挂载到容器指定目录；
--restart always #始终重新启动zookeeper
```
