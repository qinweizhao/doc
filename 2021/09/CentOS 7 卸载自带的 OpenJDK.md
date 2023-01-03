# CentOS 7 卸载自带的 OpenJDK

## 一、查询系统是否已经安装 JDK

```sh
rpm -qa|grep java
# 或
rpm -qa|grep jdk
# 或
rpm -qa|grep gcj 
```

![2021-09-15_145835](https://img.qinweizhao.com/2021/09/2021-09-15_145835.png)

## 二、卸载已安装的 JDK

除去以下三个可以不删除，剩余的全部删除。

![2021-09-15_150553](https://img.qinweizhao.com/2021/09/2021-09-15_150553.png)

删除命令：

```sh
rpm -e --nodeps xxx
```

>参数介绍：
>
> -e ：清除 (卸载) 软件包
>
> --nodeps：不验证软件包依赖

验证是否卸载：

```sh
java -version
```

![2021-09-15_151037](https://img.qinweizhao.com/2021/09/2021-09-15_151037.png)
