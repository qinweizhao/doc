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

## 五、MinIO

### 1、拉取镜像

```sh
docker pull minio/minio
```

### 2、建立目录

为了方便管理讲容器中的配置和数据挂载到本地，所以要创建两个文件夹，根据个人情况来选择位置。

```sh
mkdir /minio/config /minio/data
```

我的安装目录为：**/Users/weizhao/Docker/**。

### 3、创建容器

```sh
docker run -p 9000:9000 -p 9090:9090 \
 -d \
 --name minio \
 -e "MINIO_ACCESS_KEY=minioadmin" \
 -e "MINIO_SECRET_KEY=minioadmin" \
 -v /Users/weizhao/Docker/minio/data:/data \
 -v /Users/weizhao/Docker/minio/config:/root/.minio \
 minio/minio server /data \
 --console-address ":9090"
```

命令解释：

```txt
docker run :docker 启动容器命令
-d :后台启动
-p :端口映射
-name :为这个容器取一个名字
-e :设置环境变量
-v :文件挂载位置
minio/minio server /data :minio启动命令 （minio/minio 是镜像名字、/data:数据存储位置）
--console-address ":9090" :选择静态端口号，这里注意下控制台端口号不能和静态端口号一样
```

注意：

如果不设置静态端口，启动后会出现警告信息。

```
WARNING: Console endpoint is listening on a dynamic port (42369), please use --console-address ":PORT" to choose a static port.
```

### 4、测试访问

可以先查看下容器状态：

```sh
docker ps
docker logs minio
```

为了方便记忆，我启动时使用了默认的`access Key`和`Secret Key`。所以启动成功以后提示修改它们。

![2022-07-03_224048](https://img.qinweizhao.com/2022/07/2022-07-03_224048.png)

浏览器输入：http://127.0.0.1:9090 或 http://127.0.0.1:9000 访问可视化的管理控制平台。不过还是会跳转到设置的静态端口（9090）。

![2022-07-03_224117](https://img.qinweizhao.com/2022/07/2022-07-03_224117.png)
