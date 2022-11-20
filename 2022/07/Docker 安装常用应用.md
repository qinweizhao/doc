# Docker 安装常用应用

## 一、MySQL 8.0.30

### 1、创建目录

```sh
mkdir -p /Users/weizhao/Docker/mysql/data /Users/weizhao/Docker/mysql/logs /Users/weizhao/Docker/mysql/mysql/conf
```

创建三个文件夹：data、logs、conf。具体位置可自行调整。

### 2、拉取镜像

```sh
docker pull mysql:8.0.30
```

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

## 四、ZooKeeper 3.8.0

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

## 六、RabbitMQ

### 1、查询镜像

**查询获取 management 版本的，不要获取 last 版本的，management版本的才带有管理界面。**

```bash
 docker search rabbitmq:management
```

### 2、拉取镜像

```bash
docker pull rabbitmq:management
```

### 3、运行

```bash
# 默认 guest 用户，密码也是 guest
# 管理页面地址为：http://[IP]:15672
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
```

>参数说明：
>
>-d 后台运行
>
>-p 端口映射，
>
>--name 别名
>
>可加参数：
>
>-v 映射目录或文件；
>
>--hostname  主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；
>
>-e 指定环境变量；（RABBITMQ_DEFAULT_VHOST：默认虚拟机名；RABBITMQ_DEFAULT_USER：默认的用户名；RABBITMQ_DEFAULT_PASS：默认用户名的密码）

## 七、Jenkins

>Jenkins 是一个是基于 Java 开发的一种持续集成工具。

### 1、拉取镜像

```bash
docker pull jenkins/jenkins
```

### 2、创建挂载目录

```bash
# 当前位置应该为 /usr/local
mkdir -p mount/jenkins/jenkins_home
# 授权
chmod 777 mount/jenkins/jenkins_home
```

### 3、创建并启动容器

```bash
docker run -d -p 8080:8080 -v /usr/local/mount/jenkins/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins
```

命令解释

>**-d** 后台运行镜像
>
>**-p 8080:8080** 将镜像的8080端口映射到服务器的10240端口。
>
>**/usr/local/mount/jenkins/jenkins_home:/var/jenkins_mount /var/jenkins_home** 目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录。
>
>**-v /etc/localtime:/etc/localtime** 让容器使用和服务器同样的时间设置。
>
>**--name jenkins** 给容器起一个别名

### 4、查看是否成功

```bash
# 查看运行中的容器
docker ps

# 或

# 查看日志
docker logs jenkins
```

![2021-11-23_163858](https://img.qinweizhao.com/2021/11/2021-11-23_163858.png)

### 5、配置镜像加速

当前位置 /usr/local/mount/jenkins/jenkins_home，修改 hudson.model.UpdateCenter.xml 文件。

![2021-11-23_164513](https://img.qinweizhao.com/2021/11/2021-11-23_164513.png)

更换 url 为 清华大学官方镜像

> https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

### 6、解锁 Jenkins

**访问 http:ip:8080** ，出现如下界面。

![2021-11-23_165124](https://img.qinweizhao.com/2021/11/2021-11-23_165124.png)

因为容器的 **/var/jenkins_home** 挂载在本地的 **/usr/local/mount/jenkins/jenkins_home** ，所以直接去查看 **/usr/local/mount/jenkins/jenkins_home/secrets/initialAdminPassword** 文件获取密码。

![2021-11-23_165555](https://img.qinweizhao.com/2021/11/2021-11-23_165555.png)

### 7、安装插件

选择安装推荐的插件即可

![2021-11-23_171023](https://img.qinweizhao.com/2021/11/2021-11-23_171023.png)

## 八、Elasticsearch 7.17.3

### 1、拉取镜像

```bash
docker pull elasticsearch:7.17.3
```

### 2、前置准备

```bash
mkdir -p /Users/weizhao/Docker/elasticsearch/config
mkdir -p /Users/weizhao/Docker/elasticsearch/data
echo "http.host: 0.0.0.0" >> /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml
```

**说明：**

/Users/weizhao/Docker/elasticsearch 为我本地的路径，具体存放位置根据自己安排创建。

如果要使用 Kibana 则需要使用创建一个网络，当然这不是唯一可行的解决方案。

```sh
docker network create elk-net
```

后续启动时指定为此网络即可。

### 3、运行

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx1024m" \
-v /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/weizhao/Docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /Users/weizhao/Docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
--network elk-net -d elasticsearch:7.17.3
```

### 4、测试

浏览器访问：[localhost:9200](http://localhost:9200/)

![2022-05-06_013326](https://img.qinweizhao.com/2022/05/2022-05-06_013326.png)

## 九、Kibana 7.17.3

### 1、拉取镜像

```bash
docker pull kibana:7.17.3
```

### 2、运行

```sh
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 -p 5601:5601 --network elk-net -d kibana:7.17.
```

**说明：**

**--network elk-net**：指定为和 ES 同一个网络环境。

**http://elasticsearch:9200**：elasticsearch 为 ES 容器名。

### 3、测试

浏览器访问：http://localhost:5601

![2022-05-06_023600](https://img.qinweizhao.com/2022/05/2022-05-06_023600.png)
