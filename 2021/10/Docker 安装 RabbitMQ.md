# Docker 安装 RabbitMQ

## 一、查询镜像

**查询获取 management 版本的，不要获取 last 版本的，management版本的才带有管理界面。**

```bash
 docker search rabbitmq:management
```

## 二、拉取镜像

```bash
docker pull rabbitmq:management
```

## 三、运行

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

