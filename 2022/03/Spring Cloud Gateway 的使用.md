# Spring Cloud Gateway 的使用

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

##  五、限流配置

顾名思义，限流就是限制流量。通过限流，我们可以很好地控制系统的 QPS，从而达到保护系统的目的。
常见的限流算法有：**计数器**算法，**漏桶(Leaky Bucket)**算法，**令牌桶(Token Bucket)**算法。

Spring Cloud Gateway 官方提供了RequestRateLimiterGatewayFilterFactory 过滤器工厂，使用 Redis 和 Lua 脚本实现了令牌桶的方式。

1、添加依赖

```xml
<!-- spring data redis reactive 依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

2、限流规则，根据 URI 限流

```yml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
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
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1 # 令牌桶每秒填充速率
                redis-rate-limiter.burstCapacity: 2 # 令牌桶总容量
                key-resolver: "#{@pathKeyResolver}" # 使用 SpEL 表达式按名称引用 bean
```

提示

`StripPrefix=1`配置，表示网关转发到业务模块时候会自动截取前缀。

3、编写`URI`限流规则配置类

```java
package com.ruoyi.gateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

/**
 * 限流规则配置类
 */
@Configuration
public class KeyResolverConfiguration
{
    @Bean
    public KeyResolver pathKeyResolver()
    {
        return exchange -> Mono.just(exchange.getRequest().getURI().getPath());
    }
}
```

4、测试服务验证限流
启动网关服务`RuoYiGatewayApplication.java`和系统服务`RuoYiSystemApplication.java`。
因为网关服务有认证鉴权，可以设置一下白名单`/system/**`在进行测试，多次请求会发现返回`HTTP ERROR 429`，同时在`redis`中会操作两个`key`，表示限流成功。

```text
request_rate_limiter.{xxx}.timestamp
request_rate_limiter.{xxx}.tokens
```

其他限流规则

参数限流：`key-resolver: "#{@parameterKeyResolver}"`

```java
@Bean
public KeyResolver parameterKeyResolver()
{
	return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
}
```

IP限流：`key-resolver: "#{@ipKeyResolver}"`

```java
@Bean
public KeyResolver ipKeyResolver()
{
	return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}
```

##  六、熔断降级

1、添加依赖。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

2、配置需要熔断降级服务

```yml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
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
            # 降级配置
            - name: Hystrix
              args:
                name: default
                # 降级接口的地址
                fallbackUri: 'forward:/fallback'
```

提示

上面配置包含了一个`Hystrix`过滤器，该过滤器会应用`Hystrix`熔断与降级，会将请求包装成名为`fallback`的路由指令`RouteHystrixCommand`，`RouteHystrixCommand`继承于`HystrixObservableCommand`，其内包含了`Hystrix`的断路、资源隔离、降级等诸多断路器核心功能，当网关转发的请求出现问题时，网关能对其进行快速失败，执行特定的失败逻辑，保护网关安全。

配置中有一个可选参数`fallbackUri`，当前只支持`forward`模式的`URI`。如果服务被降级，请求会被转发到该`URI`对应的控制器。控制器可以是自定义的`fallback`接口；也可以使自定义的`Handler`，需要实现接口`org.springframework.web.reactive.function.server.HandlerFunction<T extends ServerResponse>`。

3、实现添加熔断降级处理返回信息

```java
/**
 * 熔断降级处理
 * 
 * @author qinweizhao
 */
@Component
public class HystrixFallbackHandler implements HandlerFunction<ServerResponse>
{
    private static final Logger log = LoggerFactory.getLogger(HystrixFallbackHandler.class);

    @Override
    public Mono<ServerResponse> handle(ServerRequest serverRequest)
    {
        Optional<Object> originalUris = serverRequest.attribute(GATEWAY_ORIGINAL_REQUEST_URL_ATTR);
        originalUris.ifPresent(originalUri -> log.error("网关执行请求:{}失败,hystrix服务降级处理", originalUri));
        return ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR.value()).contentType(MediaType.APPLICATION_JSON)
                .body(BodyInserters.fromValue(JSON.toJSONString(R.fail("服务已被降级熔断"))));
    }
}
```

4、路由配置信息加一个控制器方法用于处理重定向的`/fallback`请求

```java
package com.ruoyi.gateway.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.RequestPredicates;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.RouterFunctions;
import com.ruoyi.gateway.handler.HystrixFallbackHandler;
import com.ruoyi.gateway.handler.ValidateCodeHandler;

/**
 * 路由配置信息
 * 
 * @author qinweizhao
 */
@Configuration
public class RouterFunctionConfiguration
{
    @Autowired
    private HystrixFallbackHandler hystrixFallbackHandler;

    @Autowired
    private ValidateCodeHandler validateCodeHandler;

    @SuppressWarnings("rawtypes")
    @Bean
    public RouterFunction routerFunction()
    {
        return RouterFunctions
                .route(RequestPredicates.path("/fallback").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)),
                        hystrixFallbackHandler)
                .andRoute(RequestPredicates.GET("/code").and(RequestPredicates.accept(MediaType.TEXT_PLAIN)),
                        validateCodeHandler);
    }
}
```



4、测试服务熔断降级
启动网关服务`RuoYiGatewayApplication.java`，访问`/system/**`在进行测试，会发现返回`服务已被降级熔断`，表示降级成功。

##  七、跨域配置

项目采用的是前后端分离，如果页面直接调用后端的域名或IP，故存在跨域问题。

- 什么是跨域问题？

跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对`JavaScript`施加的安全限制。

- 什么是同源？

所谓同源是指，域名，协议，端口均相同

`http://www.ruoyi.vip` --> `http://admin.ruoyi.vip` 跨域
`http://www.ruoyi.vip` --> `http://www.ruoyi.vip` 非跨域
`http://www.ruoyi.vip` --> `http://www.ruoyi.vip:8080` 跨域
`http://www.ruoyi.vip` --> `https://www.ruoyi.vip` 跨域

- 使用 CORS（跨资源共享）解决跨域问题

CORS 是一个 W3C 标准，全称是"跨域资源共享"（Cross-origin resource sharing）。它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。

CORS 需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE 浏览器不能低于 IE10

整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉

因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信

**跨域调用示例**

```yml
// 登录方法
export function login(username, password, code, uuid) {
  return request({
    // 会出现跨域（通过IP或域名直接访问网关接口）
    url: 'http://192.168.31.138:8080/auth/login',
	// 不会出现跨域（开发环境默认通过proxy代理的方式，同理部署到生产也需要配置nginx代理）
	url: '/auth/login',
    headers: {
      isToken: false
    },
    method: 'post',
    data: { username, password, code, uuid }
  })
}
```

**可以在`nacos`配置中心`ruoyi-gateway-dev.yml`文件中加入以下配置解决跨域问题**

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

提示

如果可以通过部署手段（例如nginx配置代理等）解决跨域，则可以不需要添加跨域支持。

##  	八、黑名单配置

顾名思义，就是不能访问的地址。实现自定义过滤器`BlackListUrlFilter`，需要配置黑名单地址列表`blacklistUrl`，当然有其他需求也可以实现自定义规则的过滤器。

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

##  九、白名单配置

顾名思义，就是允许访问的地址。`且无需登录就能访问`。
在`ignore`中设置`whites`，表示允许匿名访问。

```yml
# 不校验白名单
ignore:
  whites:
    - /auth/logout
    - /auth/login
    - /*/v2/api-docs
    - /csrf
```



##  十、全局过滤器

全局过滤器作用于所有的路由，不需要单独配置，我们可以用它来实现很多统一化处理的业务需求，比如权限认证，IP访问限制等等。目前网关统一鉴权`AuthFilter.java`就是采用的全局过滤器。

单独定义只需要实现`GlobalFilter`, `Ordered`这两个接口就可以了。

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
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if (null == token)
        {
            ServerHttpResponse response = exchange.getResponse();
            response.getHeaders().add("Content-Type", "application/json; charset=utf-8");
            String message = "{\"message\":\"请求token信息不能为空\"}";
            DataBuffer buffer = response.bufferFactory().wrap(message.getBytes());
            return response.writeWith(Mono.just(buffer));
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder()
    {
        return 0;
    }
}
```

##  十一、实现Sentinel限流

Sentinel 支持对 Spring Cloud Gateway、Netflix Zuul 等主流的 API Gateway 进行限流。

1、添加依赖

```xml
<!-- SpringCloud Alibaba Sentinel -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
		
<!-- SpringCloud Alibaba Sentinel Gateway -->
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
```

2、限流规则配置类

```java
/**
 * 网关限流配置
 * 
 * @author qinweizhao
 */
@Configuration
public class GatewayConfig
{
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelFallbackHandler sentinelGatewayExceptionHandler()
    {
        return new SentinelFallbackHandler();
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter()
    {
        return new SentinelGatewayFilter();
    }

    @PostConstruct
    public void doInit()
    {
        // 加载网关限流规则
        initGatewayRules();
    }

    /**
     * 网关限流规则   
     */
    private void initGatewayRules()
    {
        Set<GatewayFlowRule> rules = new HashSet<>();
        rules.add(new GatewayFlowRule("service")
                .setCount(3) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        // 加载网关限流规则
        GatewayRuleManager.loadRules(rules);
    }
}
```

3、测试验证，一分钟内访问三次系统服务出现异常提示表示限流成功。

##  十二、Sentinel分组限流

对`service`、`ruoyi-gen`分组限流配置

1、`application.yml`配置文件

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
        # 代码生成
        - id: ruoyi-gen
          uri: lb://ruoyi-gen
          predicates:
            - Path=/code/**
          filters:
            - StripPrefix=1
```

2、限流规则配置类

```java
package com.ruoyi.gateway;

import java.util.HashSet;
import java.util.Set;
import javax.annotation.PostConstruct;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import com.alibaba.csp.sentinel.adapter.gateway.common.SentinelGatewayConstants;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.ApiDefinition;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.ApiPathPredicateItem;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.ApiPredicateItem;
import com.alibaba.csp.sentinel.adapter.gateway.common.api.GatewayApiDefinitionManager;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayFlowRule;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayRuleManager;
import com.alibaba.csp.sentinel.adapter.gateway.sc.SentinelGatewayFilter;
import com.ruoyi.gateway.handler.SentinelFallbackHandler;

/**
 * 网关限流配置
 * 
 * @author ruoyi
 */
@Configuration
public class GatewayConfig
{
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelFallbackHandler sentinelGatewayExceptionHandler()
    {
        return new SentinelFallbackHandler();
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter()
    {
        return new SentinelGatewayFilter();
    }

    @PostConstruct
    public void doInit()
    {
        // 加载网关限流规则
        initGatewayRules();
    }

    /**
     * 网关限流规则   
     */
    private void initGatewayRules()
    {
        Set<GatewayFlowRule> rules = new HashSet<>();
        rules.add(new GatewayFlowRule("system-api")
                .setCount(3) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        rules.add(new GatewayFlowRule("code-api")
                .setCount(5) // 限流阈值
                .setIntervalSec(60));
        // 加载网关限流规则
        GatewayRuleManager.loadRules(rules);
        // 加载限流分组
        initCustomizedApis();
    }

    /**
     * 限流分组   
     */
    private void initCustomizedApis()
    {
        Set<ApiDefinition> definitions = new HashSet<>();
        // service 组
        ApiDefinition api1 = new ApiDefinition("system-api").setPredicateItems(new HashSet<ApiPredicateItem>()
        {
            private static final long serialVersionUID = 1L;
            {
                // 匹配 /user 以及其子路径的所有请求
                add(new ApiPathPredicateItem().setPattern("/system/user/**")
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
            }
        });
        // ruoyi-gen 组
        ApiDefinition api2 = new ApiDefinition("code-api").setPredicateItems(new HashSet<ApiPredicateItem>()
        {
            private static final long serialVersionUID = 1L;
            {
                // 只匹配 /job/list
                add(new ApiPathPredicateItem().setPattern("/code/gen/list"));
            }
        });
        definitions.add(api1);
        definitions.add(api2);
        // 加载限流分组
        GatewayApiDefinitionManager.loadApiDefinitions(definitions);
    }
}
```

访问：`http://localhost:8080/system/user/list` （触发限流 ）
访问：`http://localhost:8080/system/role/list` （不会触发限流）
访问：`http://localhost:8080/code/gen/list` （触发限流）
访问：`http://localhost:8080/code/gen/xxxx` （不会触发限流）

##  十三、Sentinel自定义异常

为了展示更加友好的限流提示， Sentinel支持自定义异常处理。

方案一：`yml`配置

```yml
# Spring
spring: 
  cloud:
    sentinel:
      scg:
        fallback:
          mode: response
          response-body: '{"code":403,"msg":"请求超过最大数，请稍后再试"}'
```



方案二：`GatewayConfig`注入`Bean`

```java
@Bean
@Order(Ordered.HIGHEST_PRECEDENCE)
public SentinelFallbackHandler sentinelGatewayExceptionHandler()
{
	return new SentinelFallbackHandler();
}
```

**SentinelFallbackHandler.java**

```java
/**
 * 自定义限流异常处理
 *
 * @author qinweizhao
 */
public class SentinelFallbackHandler implements WebExceptionHandler
{
    private Mono<Void> writeResponse(ServerResponse response, ServerWebExchange exchange)
    {
        ServerHttpResponse serverHttpResponse = exchange.getResponse();
        serverHttpResponse.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        byte[] datas = "{\"code\":429,\"msg\":\"请求超过最大数，请稍后再试\"}".getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = serverHttpResponse.bufferFactory().wrap(datas);
        return serverHttpResponse.writeWith(Mono.just(buffer));
    }

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex)
    {
        if (exchange.getResponse().isCommitted())
        {
            return Mono.error(ex);
        }
        if (!BlockException.isBlockException(ex))
        {
            return Mono.error(ex);
        }
        return handleBlockedRequest(exchange, ex).flatMap(response -> writeResponse(response, exchange));
    }

    private Mono<ServerResponse> handleBlockedRequest(ServerWebExchange exchange, Throwable throwable)
    {
        return GatewayCallbackManager.getBlockHandler().handleRequest(exchange, throwable);
    }
}
```
