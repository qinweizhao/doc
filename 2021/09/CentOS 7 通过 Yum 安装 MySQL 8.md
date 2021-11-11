# CentOS 7 通过 yum 安装 MySQL 8

## 一、官网查看最新的安装包

[MySQL Yum Repository]([MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/))

## 二、下载 MySQL 源安装包

### 1、获取 rpm 包

``` bash
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

### 2、安装 MySQL 源

```bash
yum -y install mysql80-community-release-el7-3.noarch.rpm
```

### 3、查看效果

```bash
yum repolist enabled | grep mysql.*
```

![2021-09-11_235114](https://img.qinweizhao.com/2021/09/2021-09-11_235114.png)

## 三、安装 MySQL 服务器

``` bash
yum install mysql-community-server
```

## 四、启动 MySQL 服务

```bash
systemctl start  mysqld.service
```

```bash
systemctl status mysqld.service
```

![2021-09-11_235229](https://img.qinweizhao.com/2021/09/2021-09-11_235229.png)

## 五、查看初始化密码

```gradle
grep "password" /var/log/mysqld.log
```

![2021-09-11_235335](https://img.qinweizhao.com/2021/09/2021-09-11_235335.png)

## 六、修改 MySQL 密码

``` sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'dcPAy5Sl';
```

注：可能如果密码过于简单则会报错（1819）：新密码不符合当前策略；

![2021-09-12_005621](https://img.qinweizhao.com/2021/09/2021-09-12_005621.png)

设置简单密码（可选）：

### 1、查看 MySQL 初始的密码策略（如果报错，直接执行第二步）

```sql
SHOW VARIABLES LIKE 'validate_password%'; 
```

![2021-09-12_011809](https://img.qinweizhao.com/2021/09/2021-09-12_011809.png)

### 2、首先需要设置密码的验证强度等级，设置 validate_password_policy 的全局参数为 LOW 即可

```sql
set global validate_password.policy=LOW; 
```

![2021-09-12_011841](https://img.qinweizhao.com/2021/09/2021-09-12_011841.png)

### 3、当前密码长度为 8 ，如果不介意的话就不用修改了，按照通用的来讲，设置为 6 位的密码，设置 validate_password_length 的全局参数为 6 即可

```sql
set global validate_password.length=6; 
```

![2021-09-12_011847](https://img.qinweizhao.com/2021/09/2021-09-12_011847.png)

### 4、现在可以为 mysql 设置简单密码了，只要满足六位的长度即可

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '112121'; 
```

![2021-09-12_011854](https://img.qinweizhao.com/2021/09/2021-09-12_011854.png)

注：在默认密码的长度最小值为 4 ，由 大/小写字母各一个 + 阿拉伯数字一个 + 特殊字符一个，
只要设置密码的长度小于 3 ，都将自动设值为 4 。

![2021-09-12_012144](https://img.qinweizhao.com/2021/09/2021-09-12_012144.png)

关于 mysql 密码策略相关参数；
1）、validate_password.length  固定密码的总长度；
2）、validate_password.dictionary_file 指定密码验证的文件路径；
3）、validate_password.mixed_case_count  整个密码中至少要包含大/小写字母的总个数；
4）、validate_password.number_count  整个密码中至少要包含阿拉伯数字的个数；
5）、validate_password.policy 指定密码的强度验证等级，默认为 MEDIUM；
关于 validate_password.policy 的取值：
0/LOW：只验证长度；
1/MEDIUM：验证长度、数字、大小写、特殊字符；
2/STRONG：验证长度、数字、大小写、特殊字符、字典文件；
6）、validate_password_special_char_count 整个密码中至少要包含特殊字符的个数；

## 六、数据库授权

```sql
USE mysql;
UPDATE user SET host='%',plugin='mysql_native_password' WHERE u='root';
```

![2021-09-12_003311](https://img.qinweizhao.com/2021/09/2021-09-12_003311.png)

## 七、授权其他用户(可选)

### 1、创建新用户

```sql
CREATE USER 'yvkg'@'%' IDENTIFIED BY '123321';
```

![2021-09-12_015218](https://img.qinweizhao.com/2021/09/2021-09-12_015218.png)

### 2、授权

```sql
GRANT ALL PRIVILEGES ON *.* TO 'yvkg'@'%';
```

![2021-09-12_015225](https://img.qinweizhao.com/2021/09/2021-09-12_015225.png)

## 八、刷新权限

```sql
FLUSH PRIVILEGES;
```

![2021-09-12_003503](https://img.qinweizhao.com/2021/09/2021-09-12_003503.png)

## 九、防火墙打开3306端口

```bash
# 执行 exit 退出 MySQL
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

![2021-09-12_004529](https://img.qinweizhao.com/2021/09/2021-09-12_004529.png)

## 十、重启防火墙

```bash
firewall-cmd --reload
```

![2021-09-12_004540](https://img.qinweizhao.com/2021/09/2021-09-12_004540.png)
