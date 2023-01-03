# CentOS 7 安装 Maven

## 一、下载

 ```sh
 wget https://mirrors.sonic.net/apache/maven/maven-3/3.8.1/binaries/apache-maven-3.8.1-bin.tar.gz
 ```

## 二、解压

 ```sh
 tar -zxvf apache-maven-3.8.1-bin.tar.gz
 ```

## 三、配置环境变量

```sh
vim /etc/profile
```

```config
export MAVEN_HOME=maven的路径
export PATH=$MAVEN_HOME/bin:$PATH
```

![2021-07-23_183436](https://img.qinweizhao.com/2021/07/2021-07-23_183436.png)

## 四、刷新环境变量

 ```sh
 source /etc/profile
 ```

## 五、检查版本

 ```sh
 mvn -v
 ```

![2021-07-23_183611](https://img.qinweizhao.com/2021/07/2021-07-23_183611.png)
