# Docker 安装 MinIO

## 一、拉取镜像

```sh
docker pull minio/minio
```

## 二、建立目录

为了方便管理讲容器中的配置和数据挂载到本地，所以要创建两个文件夹，根据个人情况来选择位置。

```sh
mkdir /minio/config /minio/data
```

我的安装目录为：**/Users/weizhao/Docker/**。

## 三、创建容器

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

如果不设置静态端口，启动后会出现警告信息。（经过我的测试，无法访问可视化控制台）

```
WARNING: Console endpoint is listening on a dynamic port (42369), please use --console-address ":PORT" to choose a static port.
```

## 四、测试访问

可以先查看下容器状态：

```sh
docker ps
docker logs minio
```

为了方便记忆，我启动时使用了默认的`access Key`和`Secret Key`。所以启动成功以后提示修改它们。

![2022-07-03_224048](https://img.qinweizhao.com/2022/07/2022-07-03_224048.png)

浏览器输入：http://127.0.0.1:9090 或 http://127.0.0.1:9000 访问可视化的管理控制平台。不过还是会跳转到设置的静态端口（9090）。

![2022-07-03_224117](https://img.qinweizhao.com/2022/07/2022-07-03_224117.png)
