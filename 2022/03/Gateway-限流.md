# Gateway-限流

限流即限制流量。通过限流，我们可以很好地控制系统的 QPS，从而达到保护系统的目的。常见的限流算法有：计数器算法，漏桶(Leaky Bucket)算法，令牌桶(Token Bucket)算法。

## 一、令牌桶

Spring Cloud Gateway 官方提供了 RequestRateLimiterGatewayFilterFactory 过滤器工厂，使用 Redis 和 Lua 脚本实现了令牌桶的方式。

### 1、添加依赖

```xml
<!-- spring data redis reactive 依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

### 2、限流规则，根据 URI 限流

```yml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
  cloud:
    gateway:
      routes:
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

**StripPrefix=1** 配置，表示网关转发到业务模块时候会自动截取前缀。

### 3、编写 URI 限流规则配置类

```java
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

### 4、测试服务验证限流
启动网关服务`GatewayApplication.java`和业务服务`ServiceApplication.java`。多次请求会发现返回`HTTP ERROR 429`，同时在 Redis 中会操作两个 key，表示限流成功。

```text
request_rate_limiter.{xxx}.timestamp
request_rate_limiter.{xxx}.tokens
```

### 5、其他限流规则

#### 1. 参数限流

`key-resolver: "#{@parameterKeyResolver}"`

```java
@Bean
public KeyResolver parameterKeyResolver()
{
	return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
}
```

#### 2. IP 限流

`key-resolver: "#{@ipKeyResolver}"`

```java
@Bean
public KeyResolver ipKeyResolver()
{
	return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}
```

##  二、Sentinel

Sentinel 支持对 Spring Cloud Gateway、Netflix Zuul 等主流的 API Gateway 进行限流。

### 1、添加依赖

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

### 2、限流规则配置类

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

### 3、测试验证

一分钟内访问三次系统服务出现异常提示表示限流成功。

### 4、分组限流

对 service、service-support分组限流配置

#### 1.  配置文件

application.yml

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: service
          uri: lb://service
          predicates:
            - Path=/service/**
          filters:
            - StripPrefix=1

        - id: ruoyi-gen
          uri: lb://service-support
          predicates:
            - Path=/support/**
          filters:
            - StripPrefix=1
```

#### 2. 配置类

```java
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
        rules.add(new GatewayFlowRule("service-api")
                .setCount(3) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        rules.add(new GatewayFlowRule("support-api")
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
        ApiDefinition api1 = new ApiDefinition("service-api").setPredicateItems(new HashSet<ApiPredicateItem>()
        {
            private static final long serialVersionUID = 1L;
            {
                // 匹配 /user 以及其子路径的所有请求
                add(new ApiPathPredicateItem().setPattern("/service/user/**")
                        .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
            }
        });
        // support 组
        ApiDefinition api2 = new ApiDefinition("support-api").setPredicateItems(new HashSet<ApiPredicateItem>()
        {
            private static final long serialVersionUID = 1L;
            {
                // 只匹配 
                add(new ApiPathPredicateItem().setPattern("/support/gen/list"));
            }
        });
        definitions.add(api1);
        definitions.add(api2);
        // 加载限流分组
        GatewayApiDefinitionManager.loadApiDefinitions(definitions);
    }
}
```

访问：`http://localhost:8080/service/user/list` （触发限流 ）
访问：`http://localhost:8080/service/role/list` （不会触发限流）
访问：`http://localhost:8080/support/gen/list` （触发限流）
访问：`http://localhost:8080/support/gen/xxxx` （不会触发限流）

### 5、自定义异常

为了展示更加友好的限流提示， Sentinel支持自定义异常处理。

#### 1. 配置文件

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

#### 2. 注册 Bean

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

##  

>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/distributed/d-gateway
