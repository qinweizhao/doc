# 配置 Maven 开发环境

## 一、 Windows

### 1、配置环境变量

- 变量名： **MAVEN_HOME**

  变量值： **D:\Maven\apache-maven-3.8.1**

![2021-08-14_184007](https://img.qinweizhao.com/2021/08/2021-08-14_184007.png)

- 变量名： **Path**

  变量值： **%MAVEN_HOME%\bin**

  ![2021-08-14_184207](https://img.qinweizhao.com/2021/08/2021-08-14_184207.png)

### 2、测试

1. "开始”->"运行"，键入"cmd"；
2. 键入命令： **mvn -v**命令，出现以下信息，说明环境变量配置成功；

![2021-08-14_184606](https://img.qinweizhao.com/2021/08/2021-08-14_184606.png)

## 二、CentOS 7

### 1、编辑 profile 文件，配置环境变量

```sh
vim /etc/profile
```

将下面这两行代码拷贝到文件末尾并保存

```profile
MAVEN_HOME=/usr/local/maven
export PATH=${MAVEN_HOME}/bin:${PATH}
```

![2021-09-15_142613](https://img.qinweizhao.com/2021/09/2021-09-15_142613.png)

重载环境变量

```sh
source /etc/profile
```

### 2、在任意路径下测试

```sh
mvn –v
```

![2021-09-15_142920](https://img.qinweizhao.com/2021/09/2021-09-15_142920.png)
