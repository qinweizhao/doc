# CentOS 7 设置静态IP

## 一、查看当前网卡名称

```bash
ip addr
```

![2021-08-10_152459](https://img.qinweizhao.com/2021/08/2021-08-10_152459.png)

## 二、修改配置文件

ens33网卡对应的配置文件为**ifcfg-ens33**

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

![2021-08-10_153428](https://img.qinweizhao.com/2021/08/2021-08-10_153428.png)

```file
# 网络类型为以太网
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
# 手动分配ip
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
# 网卡设备名，设备名一定要跟文件名一致
NAME=ens33
UUID=db6ec3a4-f327-4b4b-95cf-203cf9c67c6b
# 网卡设备名，设备名一定要跟文件名一致
DEVICE=ens33
# 该网卡是否随网络服务启动
ONBOOT=yes
# 该网卡ip地址就是你要配置的固定IP，如果你要用xshell等工具连接，220这个网段最好和你自己的电脑网段一致，否则有可能用xshell连接失败
IPADDR=192.168.79.79
# 子网掩码
NETMASK=255.255.255.0
# 网关
GATEWAY=192.168.79.2
PREFIX=24
# DNS
DNS1=114.114.114.114
```

### 三、重启网卡

```bash
service network restart
```

![2021-08-10_153732](https://img.qinweizhao.com/2021/08/2021-08-10_153732.png)
