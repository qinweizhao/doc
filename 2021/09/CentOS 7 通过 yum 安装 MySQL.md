# CentOS 7 通过 yum 安装 MySQL

## 一、官网查看最新的安装包

[MySQL Yum Repository]([MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/))

## 二、下载 MySQL 源安装包

1. 获取 rpm 包

``` bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

2. 安装 MySQL 源

```bash
yum -y install mysql80-community-release-el7-3.noarch.rpm
```

3. 查看效果

```bash
yum repolist enabled | grep mysql.*
```

![2021-09-11_211413](https://img.qinweizhao.com/img/2021/09/2021-09-11_211413.png)

## 三、安装mysql服务器

``` bash
yum install mysql-community-server
```

## 四、启动mysql服务

```bash
systemctl start  mysqld.service
```

```bash
systemctl status mysqld.service
```

![2021-09-11_211733](https://img.qinweizhao.com/img/2021/09/2021-09-11_211733.png)

### 五、查看初始化密码

```gradle
grep "password" /var/log/mysqld.log
```

![2021-09-11_212625](https://img.qinweizhao.com/img/2021/09/2021-09-11_212625.png)