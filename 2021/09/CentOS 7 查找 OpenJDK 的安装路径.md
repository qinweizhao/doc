# CentOS 7 查找 OpenJDK 的安装路径

## 一、确认是否已安装 JDK

```bash
java -version
```

![2021-09-15_144425](https://img.qinweizhao.com/2021/09/2021-09-15_144425.png)

## 二、然后查找 java 命令的位置

```bash
which java
```

![2021-09-15_144503](https://img.qinweizhao.com/2021/09/2021-09-15_144503.png)

## 三、查找java命令的位置所对于的软链地址

```bash
ll /usr/bin/java
```

![2021-09-15_144519](https://img.qinweizhao.com/2021/09/2021-09-15_144519.png)

## 四、最后通过软链地址查找JDK的安装目录

```bash
ll /etc/alternatives/java
```

![2021-09-15_144605](https://img.qinweizhao.com/2021/09/2021-09-15_144605.png)
