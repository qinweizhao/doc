# Calf 项目部署

## 一、工具准备

操作系统：CentOS 7

终        端：Tabby

应用容器：Docker

## 二、项目准备

### 1、前端

修改 calf-frontend 目录下的 .env.production 文件：

```env
# 生产环境配置
ENV = 'production'

# Calf 管理系统/生产环境
VUE_APP_BASE_API = '/prod-api'

# 根据服务器或域名修改
PUBLIC_PATH = 'http://www.qinweizhao.com:8080/'

# 二级部署路径
VUE_APP_APP_NAME =''
```

执行打包命令：

```sh
npm run build
```

执行结束后生成一个 dist 文件夹，其中的文件即为需要的前端资源。

### 2、后端

修改配置文件 application-prod.yml：

```yml
spring:
  datasource:
    name: druidDataSource
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      # 更改为 ip 更改为容器名字 calf-mysql
      url: jdbc:mysql://calf-mysql:3316/calf?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&allowMultiQueries=true&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai&zeroDateTimeBehavior=convertToNull
      username: root
      password: 112121
      web-stat-filter:
        enabled: true
        url-pattern: /druid/*
      stat-view-servlet:
        allow:
        enabled: true
```

application.yml ：

```yml
spring:
  profiles:
    # 环境切换为 prod 
    active: prod
```

执行打包命令（跳过测试）：

```sh
mvn clean package -Dmaven.test.skip=true
```

执行结束后 calf-start 模块中 target 下的 calf.jar 即为后端资源。

## 三、环境搭建

### 1、安装 Docker

```sh
yum install docker
```

验证：

```sha
docker --version
```

启动：

```sh
systemctl start docker
```

### 2、安装 Docker Compose

下载当前稳定版本：

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

因为Docker Compose存放在GitHub，可能不太稳定。可以通过DaoCloud加速下载

```sh
curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

授权：

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

测试：

```sh
docker-compose --version
```

#### 3、Dockerfile 文件

Dockerfile

```file
FROM java:8
EXPOSE 8081
ADD vueblog-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

命令解释：

依赖jdk8环境，对外暴露8080，复制 calf.jar 到 docker 容器内并命名为 app.jar，最后是执行命令 java -jar /app.jar 。

### 4、docker-compose.yml 文件

需要用到的有nginx、mysql 和后端项目，文件内容：

```yml
version: "3"
services:
nginx: # 服务名称，用户自定义
  image: nginx:latest  # 镜像版本
  ports:
    - 81:80  # 暴露端口
  volumes: # 挂载
    - /home/share/calf/nginx/html:/usr/share/nginx/html
    - /home/share/calf/nginx/nginx.conf:/etc/nginx/nginx.conf
  privileged: true # 这个必须要，解决nginx的文件调用的权限问题
caf-mysql:
  image: mysql:5.7.27
  ports:
    - 3316:3306
  environment: # 指定用户root的密码
    - MYSQL_ROOT_PASSWORD=112121
calf:
  image: calf:latest
  build: . # 表示以当前目录下的Dockerfile开始构建镜像
  ports:
    - 8081:8081
  depends_on: # 依赖与mysql，可以不填，默认已经表示可以
    - mysql
```





因为需要用到 Nginx 并且要做文件挂载，所以可以先创建一个 Nginx 容器，将里面的文件拷贝出来，然后再构建我们最终需要的。

拉取官方 Nginx 镜像：

```sh
docker pull nginx
```

创建Nginx容器：

```sh
docker run -di --name nginx -p 88:80 nginx
```

将容器内的配置文件拷贝到指定目录（提前创建好目录）。

```sh
# 创建目录
mkdir -p /home/share/calf/nginx

# 将容器内的配置文件拷贝到指定目录
docker cp nginx:/etc/nginx /home/share/calf/nginx/conf
```

终止并删除容器

```sh
docker stop nginx
docker rm nginx
```







将前端项目打包得到的 dist 文件夹上传到 /home/share/calf/nginx/html 文件夹中







将后端项目(calf.jar)和docker-compose.yml、Dockerfile上传到同一目录下。



访问正常

点击任意一个链接或者按钮或者刷新界面，这时候出现了404：

刷新之后nginx就找不到路由了，因为vue项目的入口是index.html文件，nginx路由的时候都必须要先经过这个文件，所以我们得给nginx定义一下规则，让它匹配不到资源路径的时候，先去读取index.html，然后再路由。所以我们配置一下nginx.conf文件。具体操作就是找到**location /**,添加上一行代码**try_files $uri $uri/ /index.html last**;如下：

nginx-1.18.0/conf/nginx.conf

```plain
location / {
  root   html;
  try_files $uri $uri/ /index.html last;
  index  index.html index.htm;
}
```

- 这一行代码是什么意思呢？
  try_files的语法规则： 格式1：try_files file … uri，表示按指定的file顺序查找存在的文件，并使用第一个找到的文件进行请求处理，last表示匹配不到就内部直接匹配最后一个。

重启nginx之后，链接再刷新都正常啦。接着部署一下后端。