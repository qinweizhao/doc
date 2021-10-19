# CentOS 7 安装 RabbitMQ

## 一、安装依赖

```bash
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
```

## 二、下载并解压 erlang

```bash
# 下载
wget https://erlang.org/download/otp_src_24.0.tar.gz
# 解压
tar -zxvf otp_src_24.0.tar.gz
```

下载地址：

![2021-10-19_182219](https://img.qinweizhao.com/2021/10/2021-10-19_182219.png)

## 三、编译并安装

```bash
# 安装位置分别为 /usr/local/erlang
# 配置安装路径
./configure --prefix=/usr/local/erlang
# 编译
make
# 安装
make install
```

编译时出现的错误，可以忽略。

![2021-10-19_180453](https://img.qinweizhao.com/2021/10/2021-10-19_180453.png)

## 四、配置环境变量并检验

```bash
# 添加 erlang 环境变量
echo 'export PATH=$PATH:/usr/local/erlang/bin' >> /etc/profile
# 刷新环境变量
source /etc/profile
# 检验 erlang
erl
# 退出
halt().
```

## 五、下载并解压 RabbitMQ

```bash
# 下载
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.9.8/rabbitmq-server-generic-unix-3.9.8.tar.xz
# 解压 RabbitMQ 
# 安装位置 /usr/local/rabbitmq
# 需要 xz ,如果没有先安装 xz
yum install -y xz
# 1.
/bin/xz -d rabbitmq-server-generic-unix-3.7.15.tar.xz
# 2.
tar -xvf rabbitmq-server-3.9.8.tar -C /usr/local/rabbitmq/
```

下载地址：

![2021-10-19_214856](https://img.qinweizhao.com/2021/10/2021-10-19_214856.png)

## 六、配置 RabbitMQ 环境变量

```bash
# 添加 rabbitmq 环境变量
echo 'export PATH=$PATH:/usr/local/rabbitmq/rabbitmq-server-3.9.8/sbin' >> /etc/profile
# 刷新环境变量
source /etc/profile
```

## 七、开启 web 插件

```bash
rabbitmq-plugins enable rabbitmq_management
```

## 八、防火墙放行端口

省略

## 八、常用命令

```bash
# 启动
rabbitmq-server -detached
# 停止
rabbitmqctl stop
# 状态
rabbitmqctl status
# 查看所有用户
rabbitmqctl list_users
# 添加一个用户
rabbitmqctl add_user zhaobl 123456
# 配置权限
rabbitmqctl set_permissions -p "/" zhaobl ".*" ".*" ".*"
# 查看用户权限
rabbitmqctl list_user_permissions zhaobl
# 设置tag
rabbitmqctl set_user_tags zhaobl administrator
# 删除用户（安全起见，删除默认用户）
rabbitmqctl delete_user guest
```

