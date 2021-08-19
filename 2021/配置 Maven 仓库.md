# 配置 Maven 仓库

## 一、本地仓库

修改 **Maven** 安装目录的 **conf** 文件夹中的 **settings** 文件。

![2021-08-14_185834](https://img.qinweizhao.com\2021\08\2021-08-14_185834.png)

在 **settings** 下添加 **localRepository** 标签，内容为本地仓库目录。

![2021-08-14_190217](https://img.qinweizhao.com/2021/08/2021-08-14_190217.png)

## 二、中央仓库

在 **mirrors** 标签下增加以下内容

```
<!-- 阿里云仓库 -->
      <mirror>
          <id>alimaven</id>
          <mirrorOf>central</mirrorOf>
          <name>aliyun maven</name>
          <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
      </mirror>

      <!-- 中央仓库1 -->
      <mirror>
          <id>repo1</id>
          <mirrorOf>central</mirrorOf>
          <name>Human Readable Name for this Mirror.</name>
          <url>http://repo1.maven.org/maven2/</url>
      </mirror>

      <!-- 中央仓库2 -->
      <mirror>
          <id>repo2</id>
          <mirrorOf>central</mirrorOf>
          <name>Human Readable Name for this Mirror.</name>
          <url>http://repo2.maven.org/maven2/</url>
      </mirror>
```

![2021-08-14_191035](https://img.qinweizhao.com/2021/08/2021-08-14_191035.png)