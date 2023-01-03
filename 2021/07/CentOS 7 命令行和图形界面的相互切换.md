# CentOS 7 命令行和图形界面的相互切换

## 一、快捷键切换

图形界面——>命令行：

Ctrl+Alt+F1

图形界面<——命令行：

Ctrl+Alt+F2

## 二、命令切换

图形界面——>命令行：

```sh
init 3
```

图形界面<——命令行：

```sh
init 5
```

## 三、修改默认启动模式

### 1、查看系统的启动模式

```sh
vi /etc/inittab
```

![2021-07-26_165209](https://img.qinweizhao.com/2021/07/2021-07-26_165209.png)

图中1所示：系统的2种启动模式：

- multi-user.target: analogous to runlevel 3 #命令行模式

- graphical.target: analogous to runlevel 5  #图形模式

### 2.设置默认启动模式

上图中2所示：使用命令

```sh
systemctl get-default multi-user.target
#or
systemctl get-default graphical.target
```

#### 3.重启

```sh
reboot
```
