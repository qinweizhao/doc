# Spring Cloud Gateway 的基础使用

> Spring Cloud Gateway 是基于 Spring 生态系统之上构建的 API 网关，包括： Spring 5.x ， Spring Boot 2.x 和 Project Reactor 。 旨在提供一种简单而有效的方法来路由到API，并为它们提供跨领域的关注点，例如：安全性，监视/指标，限流等。

## 一、核心概念

### 1、路由（Route）

路由是网关最基础的部分，路由信息由 ID、目标 URI、一组断言和一组过滤器组成。如果断言 路由为真，则说明请求的 URI 和配置匹配。

### 2、断言（Predicate）

Java 8 中的断言函数。Spring Cloud Gateway 中的断言函数输入类型是 Spring 5.0 框架中的 ServerWebExchange。Spring Cloud Gateway 中的断言函数允许开发者去定义匹配来自于 Http Request 中的任何信息，比如请求头和参数等。

### 3、过滤器（Filter）

一个标准的 Spring Web Filter。Spring Cloud Gateway 中的 Filter 分为两种类型，分别是 Gateway Filter 和 Global Filter。过滤器将会对请求和响应进行处理。

## 二、项目整合

### 1、添加依赖

```xml
<!-- spring cloud gateway 依赖 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### 2、填写配置

resources/application.yml 文件：

```yml
server:
  port: 8080

spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Path=/service/**
          filters:
            - StripPrefix=1
```

### 3、启动程序

```java
@SpringBootApplication
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

## 三、路由规则

Spring Cloud Gateway 创建 Route 对象时， 使用 RoutePredicateFactory 创建 Predicate 对象， Predicate 对象可以赋值给 Route。

-  Spring Cloud Gateway 包含许多内置的 Route Predicate Factories。
- 所有这些断言都匹配 HTTP 请求的不同属性。
- 多个 Route Predicate Factories 可以通过逻辑与结合起来一起使用。

路由断言工厂 RoutePredicateFactory 包含的主要实现类包括 Datetime、请求的远端地址、路由权重、请求头、Host 地址、请求方法、请求路径和请求参数等类型的路由断言。

### 1、Datetime

匹配日期时间之后发生的请求：

```yml
server:
  port: 8080

spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - After=2022-03-15T14:20:00.000+08:00[Asia/Shanghai]
```

匹配日期时间之前发生的请求：

```yml
server:
  port: 8080

spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Before=2022-03-15T14:20:00.000+08:00[Asia/Shanghai]
```

### 2、Cookie

匹配指定名称且其值与正则表达式匹配的`cookie`

```yml
server:
  port: 8080

spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Cookie=loginname, weizhao
```

测试 `curl http://localhost:8080/service/a --cookie "loginname=weizhao"`

### 3、Header

匹配具有指定名称的请求头，`\d+`值匹配正则表达式

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Header=X-Request-Id, \d+
```

### 4、Host

匹配主机名的列表

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Host=**.qinweizhao.com,**.qinweizhao.cn
```

### 5、Method

匹配请求methods的参数，它是一个或多个参数

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Method=GET,POST
```

###  6、Path

匹配请求路径

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Path=/system/**
```

###  7、Query

匹配查询参数

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Query=username, abc.
```

###  8、RemoteAddr

匹配IP地址和子网掩码

```yml
 spring:
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - RemoteAddr=192.168.10.1/0
```

###  9、Weight

匹配权重

```yml
spring: 
  application:
    name: d-gateway
  cloud:
    gateway:
      routes:
        - id: service-a
          uri: http://localhost:9200/
          predicates:
            - Weight=group1, 8
        - id: service-b
          uri: http://localhost:9200/
          predicates:
            - Weight=group1, 2
```

##  四、路由配置

### 1、websocket配置方式


```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service
          uri: ws://localhost:9200/
          predicates:
            - Path=/api/**
```

### 2、http地址配置方式


```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service
          uri: http://localhost:9200/
          predicates:
            - Path=/api/**
```

### 3、注册中心配置方式

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service
          uri: lb://service
          predicates:
            - Path=/api/**
```

##  五、其他配置

### 1、跨域

```yml
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOriginPatterns: "*"
            allowed-methods: "*"
            allowed-headers: "*"
            allow-credentials: true
            exposedHeaders: "Content-Disposition,Content-Type,Cache-Control"
```

### 2、黑名单

即不能访问的地址。实现自定义过滤器 BlackListUrlFilter，需要配置黑名单地址列表 blacklistUrl，当然有其他需求也可以实现自定义规则的过滤器。

```yml
spring:
  cloud:
    gateway:
      routes:
        # 系统模块
        - id: service
          uri: lb://service
          predicates:
            - Path=/system/**
          filters:
            - StripPrefix=1
            - name: BlackListUrlFilter
              args:
                blacklistUrl:
                - /user/list
```

### 3、白名单配置

即允许访问的地址，且无需登录就能访问。在 ignore 中设置 whites，表示允许匿名访问。

```yml
# 不校验白名单
ignore:
  whites:
    - /auth/logout
    - /auth/login
    - /*/v2/api-docs
    - /csrf
```

### 4、全局过滤器

全局过滤器作用于所有的路由，不需要单独配置，我们可以用它来实现很多统一化处理的业务需求，比如权限认证，IP访问限制等等。

单独定义只需要实现 GlobalFilter，Ordered 这两个接口就可以了。

```java
/**
 * 全局过滤器
 * 
 * @author qinweizhao
 */
@Component
public class AuthFilter implements GlobalFilter, Ordered
{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain)
    {
        // 
      	// do something
      	//
        return chain.filter(exchange);
    }

    @Override
    public int getOrder()
    {
        return 0;
    }
}
```

