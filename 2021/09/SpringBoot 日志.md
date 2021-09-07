# SpringBoot 日志

## 一、现有的日志框架

- JUL（Java util logging）、Logback、 Log4J、Log4J2

- JCL（Jakarta Commons Logging）、SLF4J（ Simple Logging Facade for Java）

| 日志门面  （日志的抽象层）                                   | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| JCL（Jakarta  Commons Logging）    SLF4J（Simple  Logging Facade for Java）    **jboss-logging** | Log4J  JUL（java.util.logging）  Log4J2  **Logback** |

左边选一个门面（抽象层）、右边来选一个实现；

日志门面：  SLF4J；

日志实现：Logback；

>SpringBoot：底层是 Spring 框架，Spring框架默认是用 JCL ；
>
>**SpringBoot 选用 SLF4j 和 Logback；**；

## 二、SLF4J 使用

官网地址：https://www.slf4j.org

### 1、如何在系统中使用

1. 导包

   ``` xml
   <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
           <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-api</artifactId>
               <version>2.0.0-alpha5</version>
           </dependency>
           <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
           <dependency>
               <groupId>ch.qos.logback</groupId>
               <artifactId>logback-classic</artifactId>
               <version>1.3.0-alpha10</version>
               <scope>compile</scope>
           </dependency>
   ```

2. 使用

   ``` java
   public class TestMain {
       public static void main(String[] args) {
           Logger log = LoggerFactory.getLogger(UtilMain.class);
           log.info("Hello World");
       }
   }
   ```

3. 图示

   ![2021-09-06_171032](https://img.qinweizhao.com/2021/09/2021-09-06_171032.png)

### 2、遗留问题

>Spring（commons-logging）、Hibernate（jboss-logging）

如何让系统中所有的日志都统一到 slf4j?

1. 将系统中其他日志框架先排除出去。
2. 用中间包来替换原有的日志框架。
3. 我们导入 slf4j 其他的实现。
4. 图示：

![2021-09-06_171526](https://img.qinweizhao.com/2021/09/2021-09-06_171526.png)

## 三、依赖关系

1. pom依赖

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
			||
			\/
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</dependency>
```

2. 图示

![2021-09-07_150213](../../../img/2021/09/2021-09-07_150213.png)

## 四、使用

1. 默认配置

   ```java
   @SpringBootTest
   class LevelTest {
   
       Logger logger = LoggerFactory.getLogger(getClass());
   
       /**
        * 日志的级别 由高到低 OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL
        * SpringBoot 默认使用的是 info 级别，日志输出只会输出 info 级别之后的
        */
       @Test
       void testLevel() {
           logger.trace("这是trace日志...");
           logger.debug("这是debug日志...");
           logger.info("这是info日志...");
           logger.warn("这是warn日志...");
           logger.error("这是error日志...");
       }
   
   }
   ```

   ![2021-09-07_162233](../../../img/2021/09/2021-09-07_162233.png)

2. 修改默认配置

   ```properties
   # 修改日志输出级别
   logging.level.com.qinweizhao=trace
   # 在src/main/resources文件夹下生成 spring.log 作为默认文件
   logging.file.path=src/main/resources
   # 在控制台输出的日志的格式
   logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
   # 指定文件中日志输出的格式
   logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
   ```

   ![2021-09-07_170744](../../../img/2021/09/2021-09-07_170744.png)

   >补充：
   >日志输出格式：
   >	%d表示日期时间，
   >	%thread表示线程名，
   >	%-5level：级别从左显示5个字符宽度
   >	%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
   >	%msg：日志消息，
   >	%n是换行符
   >-->
   >%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n

3. 指定配置

   给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

   | Logging System          | Customization                                                |
   | ----------------------- | ------------------------------------------------------------ |
   | Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
   | Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
   | JDK (Java Util Logging) | `logging.properties`                                         |

   logback.xml：直接就被日志框架识别了；

   **logback-spring.xml**：日志框架就不直接加载日志的配置项，由 SpringBoot 解析日志配置，可以使用 SpringBoot 的高级Profile功能

   ```xml
   <springProfile name="staging">
       <!-- configuration to be enabled when the "staging" profile is active -->
     	可以指定某段配置只在某个环境下生效
   </springProfile>
   ```

   如：

   ```xml
   <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
           <!--
           日志输出格式：
   			%d表示日期时间，
   			%thread表示线程名，
   			%-5level：级别从左显示5个字符宽度
   			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
   			%msg：日志消息，
   			%n是换行符
           -->
           <layout class="ch.qos.logback.classic.PatternLayout">
               <springProfile name="dev">
                   <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
               </springProfile>
               <springProfile name="!dev">
                   <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
               </springProfile>
           </layout>
       </appender>
   ```

   

   如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误

   > no applicable action for [springProfile]

   如果 logback.xml 和logback-spring.xml 同时存在，那么 logback.xml 优先生效。
