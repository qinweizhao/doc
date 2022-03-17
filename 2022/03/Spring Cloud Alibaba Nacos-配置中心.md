# Spring Cloud Alibaba Nacos-配置中心

> 在微服务架构中，每个服务都可能会多机部署，当需要更改配置时，则每一台服务器上的配置都要被更改，这无疑是繁琐的。使用配置中心可以将配置从各应用中剥离出来，对配置进行统一管理，应用自身不需要自己去管理配置。
>
> 配置中心的服务流程如下：
>
> 1、用户在配置中心更新配置信息。
> 2、服务A和服务B及时得到配置更新通知，从配置中心获取配置

## 一、下载

参考：[Spring Cloud Alibaba Nacos-注册中心](https://www.qinweizhao.com/?p=81)

## 二、启动

省略：[同上](https://www.qinweizhao.com/?p=81)

## 三、使用

### 1、引入依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

### 2、编写配置

在`bootstrap.yml`添加 Nacos 配置，配置文件加载的优先级（由高到低）`bootstrap.properties ->bootstrap.yml -> application.properties -> application.yml`  。

```yaml
# Spring
spring: 
  application:
    # 应用名称
    name: test
  cloud:
    nacos:
      config:
        # 配置中心地址
        server-addr: 127.0.0.1:8848
        # 配置文件格式
        file-extension: yml
```

**注意**：

**spring-cloud-dependencies 2020.0.0 版本不在默认加载 bootstrap 文件，如果需要加载 bootstrap 文件需要手动添加依赖：**

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

### 3、添加配置

打开控制台，在 Nacos 中添加 test.yml 配置文件：

```yml
test:
    user:
        name: weizhao
        age: 20
```

在 Nacos Spring Cloud 中，dataId 的完整格式如下:

```text
${prefix}-${spring.profile.active}.${file-extension}
```

- prefix：

   默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix 来配置。

- spring.profile.active：

  即为当前环境对应的 profile。

- file-exetension ：

  为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

![2022-03-17_230322](https://img.qinweizhao.com/2022/03/2022-03-17_230322.png)

### 4、测试代码

```java
/**
 * @author qinweizhao
 */
@RestController
//动态刷新
@RefreshScope
public class ConfigController {

    @Value("${test.user.name}")
    private String name;

    @Value("${test.user.age}")
    private String age;

    @GetMapping("info")
    public String get() {
        return name + age;
    }
}
```

### 5、启动测试

#### 1. 基础测试

访问：[localhost:8080/info](http://localhost:8080/info)

结果：weizhao20

#### 2. 动态测试

修改 Nacos 中的 test.yml

```yaml
test:
    user:
        name: wz
        age: 22
```

访问：[localhost:8080/info](http://localhost:8080/info)

结果：wz22

## 四、进阶

### 1、概念

- **命名空间：** 

  用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 **Group** 或 **Data ID** 的配置。**Namespace** 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

- **配置集：**

  一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。

- **配置集** **ID**：

  Nacos 中的某个配置集的 ID。配置集 ID 是组织划分配置的维度之一。**Data ID** 通常用于组织划分系统的配置集。一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。Data ID 通常采用类 Java 包（com.taobao.tc.refund.log.level 的命名规则保证全局唯一性。此命名规则非强制。

- **配置分组：** 

  Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 database_url 配置和 MQ_topic 配置。

### 2、实践

- 配置分组

  在新增配置的时候可以输入 Grop ，此时新增一个配置在 TEST 组。配置内容为：

  ```yaml
  test:
      user:
          name: weizhao-a
          age: 21
  ```

  ![2022-03-17_231233](https://img.qinweizhao.com/2022/03/2022-03-17_231233.png)

  在 `bootstrap.yml`中添加配置：

  ```yaml
  spring:
    application:
      # 应用名称
      name: test
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
        config:
          # 配置中心地址
          server-addr: 127.0.0.1:8848
          # 配置文件格式
          file-extension: yml
          # 添加
          group: TEST
  ```

  **启动程序**

  访问：[localhost:8080/info](http://localhost:8080/info)

  结果：weizhao-a21

- 命名空间

  新增一个命名空间，并在当前命名空间添加一个配置并分配到 TEST 分组，配置内容为：

  ```yaml
  test:
      user:
          name: weizhao-b
          age: 22
  ```

  ![2022-03-17_231932](https://img.qinweizhao.com/2022/03/2022-03-17_231932.png)

  ![2022-03-17_232541](https://img.qinweizhao.com/2022/03/2022-03-17_232541.png)

  在 `bootstrap.yml`中添加配置：

  ```yaml
  spring:
    application:
      # 应用名称
      name: test
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
        config:
          # 配置中心地址
          server-addr: 127.0.0.1:8848
          # 配置文件格式
          file-extension: yml
          group: TEST
          ## 添加 名称空间 ID
          namespace: cbb3183b-c9ef-4604-8643-4f5da6033510
  ```

  **启动程序**

  访问：[localhost:8080/info](http://localhost:8080/info)

  结果：weizhao-a21

- 配置集

  配置集的实践，即加载多个配置文件，为了更好的说明，此时**将 test.yml 删除**，如果不删除的话还是会加载 test.yml 并且优先级要高于配置的其他配置文件。

  新增 a.yml：

  ```yaml
  test:
      user:
          name: weizhao-c
  ```

  新增 b.yml：

  ```yaml
  test:
      user:
          age: 23
  ```

  在 `bootstrap.yml`中新增配置：

  ```yaml
  # Spring
  spring:
    application:
      # 应用名称
      name: test
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
        config:
          # 配置中心地址
          server-addr: 127.0.0.1:8848
          # 配置文件格式
          file-extension: yml
          group: TEST
          namespace: cbb3183b-c9ef-4604-8643-4f5da6033510
          # 新增
          extension-configs:
                - data-id: a.yml
                  group: TEST
                  refresh: true
                - data-id: b.yml
                  group: TEST
                  refresh: true
  ```

  补充：写法说明

  ![2022-03-17_235946](https://img.qinweizhao.com/2022/03/2022-03-17_235946.png)

  extension-configs 参数为一个 List。

  **启动程序**

  访问：[localhost:8080/info](http://localhost:8080/info)

  结果：weizhao-c23

### 3、原理

- **自动注入：** 

  NacosConfigStarter 实现了 org.springframework.cloud.bootstrap.config.PropertySourceLocator 接口，并将优先级设置成了最高。 在 Spring Cloud 应用启动阶段，会主动从 Nacos Server 端获取对应的数据，并将获取到的 数据转换成 PropertySource 且入到 Environment 的 PropertySources 属性中，所以使用 @Value 注解也能直接获取 Nacos Server 端配置的内容。

- **动态刷新：** 

  Nacos Config Starter 默认为所有获取数据成功的 Nacos 的配置项添加了监听功能，在监听 到服务端配置发生变化时会实时触发 org.springframework.cloud.context.refresh.ContextRefresher 的 refresh 方法 。 如果需要对 Bean 进行动态刷新，请参照 Spring 和 Spring Cloud 规范。**推荐给类添加** **@RefreshScope** **或** **@ConfigurationProperties** **注解**。

### 4、建议

- **namespace** **与** **group** **最佳实践**

  每个微服务创建自己的 namespace 进行隔离，group 来区分 dev，beta，prod 等环境

### 5、MySQL 支持

在单机模式时 Nacos 使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。可以配置 MySQL 数据库（版本要求：5.6.5+），可视化的查看数据的存储。

- 使用 *conf/nacos-mysql.sql* 文件初始化数据库
- 修改conf/application.properties文件增加 MySQL 支持

```properties
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos-mysql?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password=root
```

### 6、集群部署

集群模式适用于生产环境需要依赖 MySQL，单机可不必，三个及以上 Nacos 节点才能构成集群。

1、在 Nacos 的解压目录`nacos/conf`目录下，修改配置文件`cluster.conf`

```sh
192.168.79.65:8848
192.168.79.66:8848
192.168.79.67:8848
```

2、修改`bootstrap.yml`中的`server-addr`属性，添加对应集群地址。

```properties
server-addr: 192.168.79.65:8848,192.168.79.66:8848,192.168.79.67:8848
```

启动运行成功后查看**集群管理/节点列表**来判断是否成功。

##  

>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/distributed/d-nacos/dn-test

