# Calf 项目部署

## 一、部署说明

Calf 快速开发平台是一个前后分离的项目，将使用 Docker 部署在 CentOS 7 上。本次部署涉及 Nginx 和 Docker Compose。

## 二、工具安装

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

## 三、项目打包

### 1、前端

修改 calf-frontend 目录下的 .env.production 文件：

```env
# 生产环境配置
ENV = 'production'

# Calf 管理系统/生产环境
# 设置为 /api 注意后面的 nginx 配置一致（代理到后端接口）
VUE_APP_BASE_API = '/api'

# 公共路径
PUBLIC_PATH = ''
```

执行打包命令：

```sh
npm run build
```

执行结束后生成一个 dist 文件夹，其中的文件即为需要的前端资源。

### 2、后端

修改配置文件 application-prod.yml：

```yml
# 项目启动端口
server:
  servlet:
    context-path: /
  port: 8088
  tomcat:
    uri-encoding: utf-8
spring:
  datasource:
    name: druidDataSource
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://mysql:3306/calf?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
      username: root
      password: Qwz#1201
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

## 四、编写文件

### 1、Dockerfile 文件

Dockerfile

```file
FROM java:8
EXPOSE 8088
ADD calf.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

命令解释：

依赖 jdk8 环境，对外暴露8088，复制 calf.jar 到 docker 容器内并命名为 app.jar，最后是执行命令 java -jar /app.jar 。

### 2、docker-compose.yml 文件

需要用到的有nginx、mysql 和后端项目，文件内容：

```yml
version: "3"
services:
  nginx: # 服务名称，用户自定义
    image: nginx:latest  # 镜像版本
    ports:
      - "88:80"  # 暴露端口
    volumes: # 挂载
      - /home/share/calf/nginx/html:/usr/share/nginx/html
      - /home/share/calf/nginx/conf:/etc/nginx
    privileged: true # 这个必须要，解决nginx的文件调用的权限问题
  mysql:
    image: mysql:8.0.22
    ports:
      - "3316:3306"
    environment: # 指定用户root的密码
      - MYSQL_ROOT_PASSWORD=Qwz#1201
  calf:
    image: calf:latest
    build: . # 表示以当前目录下的Dockerfile开始构建镜像
    ports:
      - "8088:8088"
    depends_on:
      - mysql
```

## 五、前置配置

因为需要用到 Nginx 并且要做文件挂载，所以可以先创建一个 Nginx 容器，将里面的文件拷贝出来，然后再构建我们最终需要的。

拉取官方 Nginx 镜像：

```sh
docker pull nginx
```

创建Nginx容器（因为服务器上已经有一个 Nginx 在运行了，所以用了 88 端口）：

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

## 六、上传构建

### 1、前端

将前端项目打包得到的 dist 文件夹上传到 /home/share/calf/nginx/html 文件夹中。并修改 Nginx 配置文件（/home/share/calf/nginx/conf/conf.d/default.conf）:

```conf
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    # 此处 需要与前端配置的一样才能正确代理到后端接口
    location /api/ {
        proxy_pass http://www.qinweizhao.com:8088/;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 2、后端

将 docker-compose.yml、Dockerfile 和 后端项目 (calf.jar) 上传到同一目录下。执行命令：

```sh
docker-compose up -d
```

-d 表示后台服务形式启动。

等待构建完成后连接上数据库，手动创建一下数据库并导入 sql 文件。

## 七、问题解决

访问

www.qinweizhao.com:88

页面正常显示，但点击任意一个链接或者按钮或者刷新界面，这时候出现了404：

刷新之后nginx就找不到路由了，因为 Vue 项目的入口是 index.html 文件，Nginx路由的时候都必须要先经过这个文件，所以我们得给nginx定义一下规则，让它匹配不到资源路径的时候，先去读取index.html，然后再路由。所以修改 Nginx 配置文件。具体操作就是找到 **location /**，添加上一行代码 **try_files $uri $uri/ /index.html last** 。如下：

```plain
location / {
  root   html;
  try_files $uri $uri/ /index.html last;
  index  index.html index.htm;
}

```

try_files的语法规则： 格式1：try_files file … uri，表示按指定的file顺序查找存在的文件，并使用第一个找到的文件进行请求处理，last表示匹配不到就内部直接匹配最后一个。

重启 Nginx 之后，再次刷新访问正常。