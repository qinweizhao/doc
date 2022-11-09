# Nacos-注册中心


> 注册中心类似于**通讯录**，它记录了服务和服务地址的映射关系。在分布式架构中，服务会注册到这里，当服务需要调用其它服务时，就到这里找到服务的地址，进行调用。注册中心解决了**服务发现**的问题。在没有注册中心时候，服务间调用需要知道被调方的地址或者代理地址。当服务更换部署地址，就不得不修改调用当中指定的地址或者修改代理配置。而有了注册中心之后，服务之间调用只需要记住服务名即可。
>
> Nacos 是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。Nacos 使用 Java 编写，所以需要依赖 Java 环境 
>
> Nacos 文档地址： https://nacos.io/zh-cn/docs/quick-start.html

## 一、下载

官方地址：https://github.com/alibaba/nacos/releases

### 1、安装包

从 GitHub 上下载对应平台的压缩包

### 2、源码

```sh
git clone https://github.com/alibaba/nacos.git

cd nacos/

mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  

ls -al distribution/target/

// 更改 $version
cd distribution/target/nacos-server-$version/nacos/bin

// 执行 bin 目录下载 startup.xx
./startup.xx -m standalone
```

### 3、Docker

（本次使用）

```sh
docker pull nacos/nacos-server
```

## 二、启动

- 压缩包
- 源码

bin 目录的 startup.xx 即为启动脚本，根据相应的环境，选择不同的文件。

```sh
./startup.xx -m standalone
```

注意：xx

- Docker

```sh
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server
```

注意：Nacos默认是集群模式 cluster。可以修改启动脚本将模式更改为 standalone。当 Nacos 从1.x版本升级为2.x版本后，需要暴露 9848 端口。

Nacos 提供了一个可视化的操作平台，访问  http://localhost:8848/nacos/ ，使用默认的 nacos/nacos 进行登录。

## 三、使用

### 1、引入依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

### 2、编写配置

```yaml
# Spring
spring: 
  application:
    # 应用名称
    name: service
  cloud:
    nacos:
      discovery:
        # 服务注册地址
        server-addr: 127.0.0.1:8848
```

### 3、标注注解

使用**@EnableDiscoveryClient** 开启服务注册发现功能：

```java
/**
 * @author qinweizhao
 */
@EnableDiscoveryClient
@SpringBootApplication
public class DiscoveryApplication {

    public static void main(String[] args) {
        SpringApplication.run(DiscoveryApplication.class, args);
    }

}
```

### 4、验证结果

#### 1. 观察控制台

启动应用，观察控制台的服务列表是否已经注册上服务。

![2022-03-17_203000](https://img.qinweizhao.com/2022/03/2022-03-17_203000.png)

#### 2. 服务发现

为了便于使用，NacosServerList 实现了 com.netflix.loadbalancer.ServerList 接口，并在 @ConditionOnMissingBean 的条件下进行自动注入。如果有定制化的需求，可以自己实现自己的 ServerList。

Nacos Discovery Starter 默认集成了 Ribbon ，所以对于使用了 Ribbon 做负载均衡的组件，可以直接使用 Nacos 的服务发现。

经过测试 SpringCloudAlibaba 的版本在2021.1剔除了 Ribbon，所以要使用 @LoadBalanced 注解则需要引入 loadbalancer：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



- FeignClient 已经默认集成了 Ribbon。（测试略）
- 配置 RestTemplate 添加 @LoadBlanced 注解，使得 RestTemplate 接入 Ribbon。（此次使用）

![2022-03-17_203823](https://img.qinweizhao.com/2022/03/2022-03-17_203823.png)

解释：

dn-discovery-test 模块和 dn-discovery 都注册在 Nacos。dn-discovery-test 通过 restTemplate 调用 dn-discovery。dn-discovery 做的处理为 id 加 100。

访问：http://localhost:8080/1

结果：1011

##  

>涉及模块：
>
>dn-discovery
>
>dn-discovery-test
>
>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/distributed/d-nacos

