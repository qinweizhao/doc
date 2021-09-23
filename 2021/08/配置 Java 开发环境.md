# 配置 Java 开发环境

## 一、 Windows

### 1、安装 JDK

- [官方](https://www.oracle.com/java/technologies/javase-downloads.html)
- [百度网盘](https://pan.baidu.com/s/10zvgg1q02AKrtYkE3XbjzQ)(提取码：jx66)

### 2、配置环境变量

- 变量名： **JAVA_HOME**

  变量值： **D:\Java\jdk1.8.0_291**

- 变量名： **CLASSPATH**

  变量值： **.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;**

![2021-08-14_163546](https://img.qinweizhao.com/2021/08/2021-08-14_163546.png)

- 变量名： **Path**

  变量值： **%JAVA_HOME%\bin** / **%JAVA_HOME%\jre\bin**

  ![2021-08-14_165606](https://img.qinweizhao.com/2021/08/2021-08-14_165606.png)

***注：* 根据实际安装目录修改 JAVA_HOME 的变量值。**

### 3、测试

1. "开始"->"运行"，键入"cmd"；
2. 键入命令： **java -version**、**java**、**javac** 几个命令，出现以下信息，说明环境变量配置成功；

![2021-08-14_165927](https://img.qinweizhao.com/2021/08/2021-08-14_165927.png)

![2021-08-14_165850](https://img.qinweizhao.com/2021/08/2021-08-14_165850.png)

![2021-08-14_165950](https://img.qinweizhao.com/2021/08/2021-08-14_165950.png)

## 二、CentOS

### 1、下载 JDK 并解压到指定目录

```bash
tar -zxvf jdk-11.0.10_linux-x64_bin.tar.gz -C ../java
```

​  注：压缩包路径：**/usr/local/install**

​    安装路径：**/usr/local/java**

### 2、编辑 profile 文件，配置环境变量

```bash
vi /etc/profile
```

增加：

```profile
export JAVA_HOME=/usr/local/java/jdk-11.0.10
export CLASSPATH=.:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar
export PATH=$JAVA_HOME/bin:$PATH
```

### 3、刷新环境变量

```bash
source /etc/profile
```

### 3、在任意路径下测试

```bash
java -version
```

运行效果与 Windows 一致
