# 配置 Maven 仓库和 JDK

## 一、本地仓库

修改 **Maven** 安装目录的 **conf** 文件夹中的 **settings** 文件。

![2021-08-14_185834](https://img.qinweizhao.com/2021/08/2021-08-14_185834.png)

在 **settings** 下添加 **localRepository** 标签，内容为本地仓库目录。

![2021-08-14_190217](https://img.qinweizhao.com/2021/08/2021-08-14_190217.png)

## 二、中央仓库

在 **mirrors** 标签下增加以下内容

```xml
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

## 三、配置 JDK

```xml
      <profile>
          <id>JDK-1.8</id>
          <activation>
              <activeByDefault>true</activeByDefault>
              <jdk>1.8</jdk>
          </activation>
          <properties>
              <maven.compiler.source>1.8</maven.compiler.source>
              <maven.compiler.target>1.8</maven.compiler.target>
              <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
          </properties>
      </profile>
```

![2021-08-14_205141](https://img.qinweizhao.com/2021/08/2021-08-14_205141.png)
