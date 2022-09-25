# Blog 项目部署

## 一、部署说明

服务器采用  CentOS 7 ，项目部署使用 Docker 部署。项目部署的方式有多种，选择一种自己喜欢的即可。

## 二、工具准备

### 1、MySQL

参考：https://www.qinweizhao.com/?p=31

说明：MySQL 不推荐 Docker ，因为使用 Docker 另外需要做一些操作，虽说不麻烦，但个人并不喜欢使用 Docker 去安装数据库。

### 2、Docker

参考：https://docs.docker.com/engine/install

说明：官网介绍的已经很详细了，安装工具尽量去查看官方文档。

### 3、Nginx

参考：https://www.qinweizhao.com/?p=116

说明：可选，但是推荐使用 [Nginx](http://nginx.org/)  Web 服务器进行反向代理。

### 三、其他准备

### 1、数据库

创建数据库，并执行 sql 创建项目所需的表。sql 文件存放在 **blog-resource/sql** 目录下。

### 2、工作目录

执行命令创建工作目录：

```
mkdir ~/.blog && cd ~/.blog
```

说明：在博客运行的时候会在系统当前用户目录下产生一个 `.blog` 的文件夹，绝对路径为 `~/.blog`。由于这个工作目录是固定的，所以上面所说的 `运行包`不限制所存放的位置，里面通常包含下列目录或文件：

1. `frontend`：前端文件目录。
2. `log`：运行日志目录。
3. `image`：图片资源目录。
4. `application.yaml`：配置文件。

## 三、项目打包

### 1、前台

在 **blog-frontend-portal** 目录下执行命令

安装依赖：

```sh
npm i
```

说明：已经安装过不需要再次安装。

```sh
npm run build
```

说明：构建用于生产环境的 js 和 css 文件。

### 2、后台

编辑配置文件，配置运行环境、数据库或者端口等。

在 **qwz-blog** 目录下执行命令执行打包命令（跳过测试）：

```sh
mvn clean package -Dmaven.test.skip=true
```

说明：执行结束后后台的**前端资源也会打包进去**，**blog-backend** 模块中 target 下的 blog-backend-xx.jar 即为后端资源。

## 四、上传构建

### 1、前台

将 **blog-frontend-portal** 整个文件夹上传至 **frontend** 目录中即可。

### 2、后台

1. 将编写的 **Dockerfile** 和打包后的 **jar** 上传到服务器并放置在同一个目录下，**Dockerfile** 存放在项目的 **blog-resource/docker** 目录中。

2. 执行构建镜像

   ```
   docker build -t qinweizhao/blog .
   ```

3. 运行容器

   ```sh
   docker run -it -d --name blog -p 8090:8090 -v ~/.blog:/root/.blog --restart=unless-stopped qinweizhao/blog
   ```
   
## 五、其他配置

如果使用了 Nginx 做反向代理，则需要对 Nginx 进行配置，可参考 **blog-resource/nginx/nginx.conf** 进行配置。
