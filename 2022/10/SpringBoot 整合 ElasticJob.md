# SpringBoot 整合 ElasticJob

## 一、说明

**版本信息**：

| 框架 | SpringBoot | ElasticJob |
| :--: | :--------: | :--------: |
| 版本 |   2.7.4    |   3.0.1    |

**依赖坐标**：

```pom
		<dependency>
			<groupId>org.apache.shardingsphere.elasticjob</groupId>
			<artifactId>elasticjob-lite-spring-boot-starter</artifactId>
			<version>3.0.1</version>
		</dependency>
```

## 二、基本使用

### 1、作业

**简单作业**

```java
/**
 * @author qinweizhao
 * @since 2022/10/17
 */
@Component
public class MySimpleJob implements SimpleJob {

    /**
     * @param shardingContext 作业配置、分片和运行时信息
     */
    @Override
    public void execute(ShardingContext shardingContext) {
        int shardingItem = shardingContext.getShardingItem();
        System.out.println("简单任务执行，当前分片项为" + shardingItem);
    }

}
```

**数据流作业**

```java
/**
 * @author qinweizhao
 * @since 2022/10/17
 */
@Slf4j
@Component
public class MyDataflowJob implements DataflowJob<User> {

    /**
     * 抓取数据
     * @param shardingContext shardingContext
     * @return List
     */
    @Override
    public List<User> fetchData(ShardingContext shardingContext) {
        log.debug("获取数据,分片项为"+shardingContext.getShardingItem());
        User user = new User(1, "qwz", "123", "china");
        return Collections.singletonList(user);
    }

    /**
     * 处理数据
     * @param shardingContext shardingContext
     * @param list list
     */
    @Override
    public void processData(ShardingContext shardingContext, List<User> list) {
        log.debug("开始处理数据");
        list.forEach(System.out::println);
    }

}
```

**脚本作业**

支持 shell，python，perl 等所有类型脚本。可通过属性配置 script.command.line 配置待执行脚本，无需编码。执行脚本路径可包含参数，参数传递完毕后，作业框架会自动追加最后一个参数为作业运行时信息。

```sh
echo sharding execution context is $*
```

作业运行时将输出：

```txt
sharding execution context is {"jobName":"scriptElasticDemoJob","shardingTotalCount":10,"jobParameter":"","shardingItem":0,"shardingParameter":"A"}
```

**HTTP 作业**

可通过属性配置 `http.url`，`http.method`，`http.data` 等配置待请求的 http 信息。分片信息以 Header形式传递，key 为 shardingContext，值为 json 格式。

```java
    private static void oneOff() {
        OneOffJobBootstrap jobBootstrap = new OneOffJobBootstrap(createRegistryCenter(), "HTTP", JobConfiguration.newBuilder(
                        "javaHttpJob", 1)
                .setProperty(HttpJobProperties.URI_KEY, "http://www.qinweizhao.com")
                .setProperty(HttpJobProperties.METHOD_KEY, "GET")
                .setProperty(HttpJobProperties.DATA_KEY, "source=ejob")
                .shardingItemParameters("0=Beijing").
                build());

        // 可多次调用一次性调度
        jobBootstrap.execute();
        jobBootstrap.execute();
        jobBootstrap.execute();
    }
```

注：（3.0.0-beta）提供。

### 2、配置

```yaml
elasticjob:
  reg-center:
    server-lists: 127.0.0.1:2181
    namespace: springboot-elasticjob-lite
  jobs:
    simpleJob:
      elastic-job-class: org.apache.shardingsphere.elasticjob.simple.job.SimpleJob
      cron: 0/10 * * * * ?
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
    dataflowJob:
      elasticJobClass: org.apache.shardingsphere.elasticjob.dataflow.job.DataflowJob
      cron: 0/10 * * * * ?
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
    scriptJob:
      elasticJobType: SCRIPT
      cron: 0/10 * * * * ?
      shardingTotalCount: 3
      props:
        script.command.line: "java -version"
```

主要配置协调服务和作业（三个）。

### 3、启动

**定时调度**：

定时调度作业在 Spring Boot 应用程序启动完成后会自动启动，无需其他额外操作。

**一次性调度**：

一次性调度作业，需要通 jobBootstrapBeanName 指定 OneOffJobBootstrap Bean 的名称， 在触发点注入 OneOffJobBootstrap 的实例并手动调用 execute() 方法。

```yaml
elasticjob:
  jobs:
    myOneOffJob:
      jobBootstrapBeanName: myOneOffJobBean
      ....
```

```java
@RestController
public class OneOffJobController {

    // 通过 "@Resource" 注入
    @Resource(name = "myOneOffJobBean")
    private OneOffJobBootstrap myOneOffJob;
    
    @GetMapping("/execute")
    public String executeOneOffJob() {
        myOneOffJob.execute();
        return "{\"msg\":\"OK\"}";
    }

    // 通过 "@Autowired" 注入
    @Autowired
    @Qualifier(name = "myOneOffJobBean")
    private OneOffJobBootstrap myOneOffJob2;

    @GetMapping("/execute2")
    public String executeOneOffJob2() {
        myOneOffJob2.execute();
        return "{\"msg\":\"OK\"}";
    }
}
```

补充：

- 判断是那种类型取决于是否配置了 cron 属性则为定时调度作业，反之则为一次性调度。

  核心源码：

  ```java
          if (Strings.isNullOrEmpty(jobConfig.getCron())) {
              Preconditions.checkArgument(!Strings.isNullOrEmpty(jobBootstrapBeanName), "The property [jobBootstrapBeanName] is required for One-off job.");
              singletonBeanRegistry.registerSingleton(jobBootstrapBeanName, new OneOffJobBootstrap(registryCenter, elasticJob, jobConfig));
          } else {
              String beanName = !Strings.isNullOrEmpty(jobBootstrapBeanName) ? jobBootstrapBeanName : jobConfig.getJobName() + "ScheduleJobBootstrap";
              singletonBeanRegistry.registerSingleton(beanName, new ScheduleJobBootstrap(registryCenter, elasticJob, jobConfig));
          }
  ```

- 经过测试，**3.0.0** 可以正常使用，**3.0.1 启动报错**（TODO：待补充原因及解决方案）。


### 4、错误处理

使用 ElasticJob‐Lite 过程中当作业发生异常后，可采用以下错误处理策略。

| 错误处理策略名称 |                  说明                  | 是否内置 | 是否默认 | 是否需要额外配置 |
| :--------------: | :------------------------------------: | :------: | :------: | :--------------: |
|   记录日志策略   |   记录作业异常日志，但不中断作业执行   |    是    |    是    |                  |
|   抛出异常策略   |       抛出系统异常并中断作业执行       |    是    |          |                  |
|   忽略异常策略   |      忽略系统异常且不中断作业执行      |    是    |          |                  |
|   邮件通知策略   |   发送邮件消息通知，但不中断作业执行   |          |          |        是        |
| 企业微信通知策略 | 发送企业微信消息通知，但不中断作业执行 |          |          |        是        |
|   钉钉通知策略   |   发送钉钉消息通知，但不中断作业执行   |          |          |        是        |

**记录日志策略**

```yaml
elasticjob:
  jobs:
    jobErrorHandlerType: LOG
```

**抛出异常策略**

```yaml
elasticjob:
  jobs:
    jobErrorHandlerType: THROW
```

**忽略异常策略**

```yaml
elasticjob:
  jobs:
    jobErrorHandlerType: IGNORE
```

**邮件通知策略**

```xml
		<dependency>
			<groupId>org.apache.shardingsphere.elasticjob</groupId>
			<artifactId>elasticjob-error-handler-email</artifactId>
			<version>${e-job.version}</version>
		</dependency>
```

```yaml
elasticjob:
  jobs:
    occurErrorNoticeEmailJob:
      elasticJobClass: com.qinweizhao.elasticjob.lite.job.OccurErrorNoticeEmailJob
      overwrite: true
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
      jobErrorHandlerType: EMAIL
      jobBootstrapBeanName: occurErrorNoticeEmailBean
      props:
        email:
          host: host
          port: 465
          username: username
          password: password
          useSsl: true
          subject: ElasticJob error message
          from: from@xxx.xx
          to: to1@xxx.xx,to2@xxx.xx
          cc: cc@xxx.xx
          bcc: bcc@xxx.xx
          debug: false
```

**企业微信通知策略**

```yaml
		<dependency>
			<groupId>org.apache.shardingsphere.elasticjob</groupId>
			<artifactId>elasticjob-error-handler-wechat</artifactId>
			<version>${e-job.version}</version>
		</dependency>
```

```yaml
elasticjob:
  jobs:
    occurErrorNoticeWechatJob:
      elasticJobClass: com.qinweizhao.elasticjob.lite.job.OccurErrorNoticeWechatJob
      overwrite: true
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
      jobErrorHandlerType: WECHAT
      jobBootstrapBeanName: occurErrorNoticeWechatBean
      props:
        wechat:
          webhook: you_webhook
          connectTimeout: 3000
          readTimeout: 5000
```

**钉钉通知策略**

```xml
		<dependency>
			<groupId>org.apache.shardingsphere.elasticjob</groupId>
			<artifactId>elasticjob-error-handler-dingtalk</artifactId>
			<version>${e-job.version}</version>
		</dependency>
```

```yaml
elasticjob:
  jobs:
    occurErrorNoticeEmailJob:
      elasticJobClass: com.qinweizhao.elasticjob.lite.job.OccurErrorNoticeEmailJob
      overwrite: true
      shardingTotalCount: 3
      shardingItemParameters: 0=Beijing,1=Shanghai,2=Guangzhou
      jobErrorHandlerType: EMAIL
      jobBootstrapBeanName: occurErrorNoticeEmailBean
      props:
        email:
          host: host
          port: 465
          username: username
          password: password
          useSsl: true
          subject: ElasticJob error message
          from: from@xxx.xx
          to: to1@xxx.xx,to2@xxx.xx
          cc: cc@xxx.xx
          bcc: bcc@xxx.xx
          debug: false
```

## 三、监听器

用于在任务执行前和执行后执行监听的方法。监听器分为每台作业节点均执行的常规监听器和分布式场景中仅单一节点执行的分布式监听器。

### 1、定义

**常规监听器**

若作业处理作业服务器的文件，处理完成后删除文件，可考虑使用每个节点均执行清理任务。此类型任务实现简单，且无需考虑全局分布式任务是否完成，应尽量使用此类型监听器。

```java
/**
 * @author qinweizhao
 * @since 2022/10/19
 */
@Slf4j
public class MyJobListener implements ElasticJobListener {

    @Override
    public void beforeJobExecuted(ShardingContexts shardingContexts) {
        log.info("常规监听器开始");
        log.info("作业执行前{}", shardingContexts);
    }

    @Override
    public void afterJobExecuted(ShardingContexts shardingContexts) {
        log.info("作业执行后{}", shardingContexts);
        log.info("常规监听器结束");
    }

    @Override
    public String getType() {
        return "myJobListener";
    }
    
}
```

**分布式监听器**

若作业处理数据库数据，处理完成后只需一个节点完成数据清理任务即可。此类型任务处理复杂，需同步分布式环境下作业的状态同步，提供了超时设置来避免作业不同步导致的死锁，应谨慎使用。

```java
/**
 * @author qinweizhao
 * @since 2022/10/19
 */
@Slf4j
public class MyDistributeOnceJobListener extends AbstractDistributeOnceElasticJobListener {

    public MyDistributeOnceJobListener(long startedTimeoutMilliseconds, long completedTimeoutMilliseconds) {
        super(startedTimeoutMilliseconds, completedTimeoutMilliseconds);
    }

    @Override
    public void doBeforeJobExecutedAtLastStarted(ShardingContexts shardingContexts) {
        log.debug("分布式监听器开始");
        log.debug("作业执行前{}", shardingContexts);
    }

    @Override
    public void doAfterJobExecutedAtLastCompleted(ShardingContexts shardingContexts) {
        log.debug("作业执行后{}", shardingContexts);
        log.debug("分布式监听器结束");
    }

    @Override
    public String getType() {
        return "distributeOnceJobListener";
    }
    
}
```

### 2、添加 SPI 实现

将 JobListener 实现类的全限定类名添加项目的 `resources/META-INF/services/org.apache.shardingsphere.elasticjob.infra.listener.ElasticJobListener` 文件中。

**监听器配置**

```yaml
elasticjob:
  jobs:
    simpleJob:
      job-listener-types: myJobListener
```

## 四、事件追踪

ElasticJob 提供了事件追踪功能，可通过事件订阅的方式处理调度过程的重要事件，用于查询、统计和监控。目前提供了基于关系型数据库的事件订阅方式记录事件，也可以通过 SPI 自行扩展。

ElasticJob‐Lite 的 Spring Boot Starter 集成了 TracingConfiguration 自动配置，开发者只需注册一个 Data‐Source 到 Spring 容器中并在配置文件指定事件追踪数据源类型，Starter 就会自动创建一个 TracingConfiguration 实例并注册到 Spring 容器中。

### 1、依赖

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
```

### 2、配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: Qwz#1201
elasticjob:
  tracing:
    type: RDB
```

### 3、启动

作业启动器会自动生成两张表并记录作业的执行记录。

![2022-10-20_222221](https://img.qinweizhao.com/2022/10/2022-10-20_222221.png)

