# Gateway-熔断降级

##  一、添加依赖。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

## 二、配置服务

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
            - Path=/service/**
          filters:
            - StripPrefix=1
            # 降级配置
            - name: Hystrix
              args:
                name: default
                # 降级接口的地址
                fallbackUri: 'forward:/fallback'
```

上面配置包含了一个 Hystrix 过滤器，该过滤器会应用 Hystrix 熔断与降级，会将请求包装成名为 **fallback** 的路由指令 **RouteHystrixCommand** ，RouteHystrixCommand 继承于 HystrixObservableCommand，其内包含了 Hystrix 的断路、资源隔离、降级等诸多断路器核心功能，当网关转发的请求出现问题时，网关能对其进行快速失败，执行特定的失败逻辑，保护网关安全。配置中有一个可选参数 fallbackUri ，当前只支持 forward 模式的 URI。如果服务被降级，请求会被转发到该 URI 对应的控制器。控制器可以是自定义的 fallback 接口；也可以使自定义的 Handler，需要实现接口`org.springframework.web.reactive.function.server.HandlerFunction<T extends ServerResponse>`。

## 三、配置返回

添加熔断降级处理返回信息

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

## 四、配置路由

路由配置信息加一个控制器方法用于处理重定向的`/fallback`请求

```java
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

## 五、测试服务

启动网关服务 `GatewayApplication.java`，访问`/service/**`在进行测试，会发现返回服务已被降级熔断，表示降级成功。


>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/distributed/d-gateway
