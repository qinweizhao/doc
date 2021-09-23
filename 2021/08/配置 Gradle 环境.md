# 配置 Gradle 环境

## 一、 Windows

### 1、配置环境变量

- 变量名： **GRADLE_HOME**

  变量值： **D:\Gradle\gradle-7.1.1**

- 变量名： **GRADLE_USER_HOME**（可选）

  变量值： **D:\Maven\repository**

![2021-08-14_224053](https://img.qinweizhao.com/2021/08/2021-08-14_224053.png)

- 变量名： **Path**

  变量值： **%GRADLE_HOME%\bin**

  ![2021-08-14_224303](https://img.qinweizhao.com/2021/08/2021-08-14_224303.png)

### 2、测试

1. "开始”->"运行"，键入"cmd"；
2. 键入命令： **gradle -v** 命令，出现以下信息，说明环境变量配置成功；

![2021-08-14_224418](https://img.qinweizhao.com/2021/08/2021-08-14_224418.png)

### 3、配置 Gradle 仓库源

　在Gradle安装目录下的 init.d 文件夹下，新建一个 init.gradle 文件，里面填写以下配置。

```gradle
allprojects {
    repositories {
        maven { url 'file:///D:/Maven/repository'}
        mavenLocal()
        maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public" }
        maven { name "Bstek" ; url "http://nexus.bsdn.org/content/groups/public/" }
        mavenCentral()
    }

    buildscript { 
        repositories { 
            maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public' }
            maven { name "Bstek" ; url 'http://nexus.bsdn.org/content/groups/public/' }
            maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }
        }
    }
}
```

​repositories 中写的是获取 jar 包的顺序。先是本地的 Maven 仓库路径；接着的 mavenLocal() 是获取 Maven 本地仓库的路径，应该是和第一条一样，但是不冲突；第三条和第四条是从国内和国外的网络上仓库获取；最后的 mavenCentral() 是从Apache提供的中央仓库获取 jar 包。
