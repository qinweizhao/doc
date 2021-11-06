# Linux 防火墙 Firewall 的使用

```bash
# 开启防火墙
systemctl start firewalld
# 关闭防火墙
systemctl stop firewalld
# 查看防火墙状态
systemctl status firewalld
# 设置开机启动
systemctl enable firewalld
# 禁用开机启动
systemctl disable firewalld
# 重启防火墙
firewall-cmd --reload
# 开放端口（修改后需要重启防火墙方可生效）
firewall-cmd --zone=public --add-port=8080/tcp --permanent
# 查看开放的端口
firewall-cmd --list-ports
# 关闭端口
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```

>```
># 参数解释
>1、firwall-cmd：是Linux提供的操作 firewall 的一个工具；
>2、--permanent：表示设置为持久；
>3、--add-port：标识添加的端口；
>```

