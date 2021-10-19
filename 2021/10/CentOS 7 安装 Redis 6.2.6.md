# CentOS 7 安装 Redis 6.2.6

## 一、获取压缩包

![2021-10-18_131030](https://img.qinweizhao.com/2021/10/2021-10-18_131030.png)

```bash
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
```

解压到指定目录

```bash
tar -zxvf redis-6.2.6.tar.gz -C /usr/local/redis/
```

## 二、安装 gcc 和 make

检查是否安装有 gcc 和 make

```bash
whereis gcc make
```

![2021-10-18_131515](https://img.qinweizhao.com/2021/10/2021-10-18_131515.png)

如果未安装，执行安装

```bash
yum install -y gcc make
```

## 三、编译并安装

```bash
# 编译
make
```

![2021-10-18_132240](https://img.qinweizhao.com/2021/10/2021-10-18_132240.png)

```bash
# 安装，将 redis 的命令安装到 /usr/local/bin/ 目录
make install
```

![2021-10-18_132425](https://img.qinweizhao.com/2021/10/2021-10-18_132425.png)

## 四、修改配置

```bash
# 进入 redis 安装目录
cd xxx
# 编辑 redis.conf 配置文件
vim redis.conf
```

修改内容：

```conf
# 绑定ip：如果需要远程访问，可将此行注释，或绑定一个真实ip
bind 127.0.0.1
# 端口号
port 6379
# 关闭保护模式，不然远程还是连接不了
protected-mode no
# 密码
#requirepass 123456
# 进程文件保存位置，redis运行后会在此位置自动生成
pidfile /var/run/redis_6379.pid
# redis位置
dir /usr/local/redis/redis-6.2.6
```

##  五、防火墙开放 6379 端口

```bash
# 添加6379端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
# 重启防火墙
firewall-cmd --reload
# 查看所有开放端口号
firewall-cmd --list-port
# 查看指定端口是否开放
firewall-cmd --query-port=6379/tcp
```

## 六、启动

```bash
# 使用指定配置启动
redis-server /usr/local/redis/redis-single/redis.conf
```

设置开机启动：

```bash
vim /etc/systemd/system/redis.service
```

添加内容

```service
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/redis-single/src/redis-server /usr/local/redis/redis-single/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

![2021-10-18_150144](https://img.qinweizhao.com/2021/10/2021-10-18_150144.png)

>Description:描述服务
>After:描述服务类别
>　　 [Service]服务运行参数的设置
>　　 Type=forking是后台运行的形式
>　　 ExecStart为服务的具体运行命令
>　　 ExecReload为重启命令
>　　 ExecStop为停止命令
>　　 PrivateTmp=True表示给服务分配独立的临时空间
>　　 注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
>　　 [Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

```bash
# 设置开机自启动
systemctl enable redis.service
```

## 七、常用命令

调试相关命令

```bash
# 此命令用于重新加载修改后的启动脚本
systemctl daemon-reload
# 显示概要
systemctl status redis.service
# 查看启动详情
journalctl -xe
# 显示实时日志
journalctl -f
# 查看本机监听端口
netstat -tunlp|grep redis
```

systemctl 常用命令

```bash
# 启动redis服务
systemctl start redis.service
# 设置开机自启动
systemctl enable redis.service
# 停止开机自启动
systemctl disable redis.service
# 查看服务当前状态
systemctl status redis.service
# 重新启动服务
systemctl restart redis.service
# 查看所有已启动的服务
systemctl list-units --type=service
```

