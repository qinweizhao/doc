# Maven-依赖的范围

## 一、compile

- main 目录下的 Java 代码**可以**访问这个范围的依赖
- test 目录下的 Java 代码**可以**访问这个范围的依赖
- 部署到 Tomcat 服务器上运行时**要**放在 WEB-INF 的 lib 目录下

## 二、test

- main 目录下的 Java 代码**不能**访问这个范围的依赖

- test 目录下的 Java 代码**可以**访问这个范围的依赖

- 部署到 Tomcat 服务器上运行时**不会**放在 WEB-INF 的 lib 目录下。

  例如：对junit的依赖。仅仅是测试程序部分需要。

## 三、provided

- main 目录下的 Java 代码**可以**访问这个范围的依赖

- test 目录下的 Java 代码**可以**访问这个范围的依赖

- 部署到 Tomcat 服务器上运行时**不会**放在 WEB-INF 的 lib 目录下

  例如：servlet-api在服务器上运行时，Servlet 容器会提供相关 API，所以部署的时候不需要。

## 四、runtime[了解]

- main 目录下的 Java 代码**不能**访问这个范围的依赖

- test 目录下的 Java 代码**可以**访问这个范围的依赖

- 部署到 Tomcat 服务器上运行时**会**放在 WEB-INF 的 lib 目录下

  例如：JDBC 驱动。只有在测试运行和在服务器运行的时候才决定使用什么样的数据库连接。

## 五、其他：import、system等。

各个依赖范围的作用可以概括为下图：

![2022-04-18_140847](https://img.qinweizhao.com/2022/04/2022-04-18_140847.png)
