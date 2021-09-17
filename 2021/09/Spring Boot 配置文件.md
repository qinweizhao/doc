# Spring Boot 配置文件

## 一、配置文件

### 1、格式

SpringBoot 配置文件名有两种不同的格式，一个是 properties ，另一个是 yaml 。yaml 还有另外一个特点，就是 yaml 中的数据是有序的，properties 中的数据是无序的。

### 2、文件名

默认的配置文件名为 application（可更改）

		- application.properties
		- application.yml

## 二、YAML语法：

>YAML：**以数据为中心**，比json、xml等更适合做配置文件；

### 1、基本语法

k:(空格)v：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
    port: 8081
```

属性和值也是大小写敏感；

### 2、值的写法

1. 字面量：普通的值（数字，字符串，布尔）

   ​	k: v：字面直接来写；

   ​		字符串默认不用加上单引号或者双引号；

   ​		""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思

   ​				name:   "tom\n jerry"：输出： tom换行  jerry

   ​		''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

   ​				name:   ‘tom\n jerry’：输出： tom\n jerry’i

2. 对象、Map（属性和值）（键值对）：

   k: v：在下一行来写对象的属性和值的关系；对象还是 k: v 的方式

   ```yaml
   user:
     name: weizhao
     age: 24
   ```

   行内写法：

   ```yaml
   user2: {name: weizhao,age: 24}
   ```

3. 数组（List、Set）：

   用 - 值表示数组中的一个元素

   ```yaml
   pets:
     - cat
     - dog
     - pig
   ```

   行内写法

   ```yaml
   pets: [cat,dog,pig]
   ```

### 3、Profile

1. 多Profile文件

   我们在主配置文件编写的时候，文件名可以是   application-{profile}.properties/yml

   默认使用application.properties的配置；

2. yml支持多文档块方式

   ```yaml
   server:
     port: 8081
   spring:
     profiles:
       active: prod
   
   ---
   server:
     port: 8083
   spring:
     profiles: dev
   
   
   ---
   
   server:
     port: 8084
   spring:
     profiles: prod  #指定属于哪个环境
   ```

3. 激活指定profile

​	1、在配置文件中指定  spring.profiles.active=dev

​	2、命令行：java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；可以直接在测试的时候，配置传入命令行参数

​	3、虚拟机参数；-Dspring.profiles.active=dev

## 三、配置文件值注入

### 1、Spring 方式

配置文件：

```properties
user.name="qinweizhao"
user.age=24
```

entity:

```java
package com.qinweizhao.entity;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 *
 *  * <bean class="User">
 *  *      <property name="name" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
 *  *      <property name="age" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
 *  * <bean/>
 *
 * @author qinweizhao
 * @since 2021/9/17
 */
@Component
public class User {


    /**
     * 名字
     */
    @Value("${user.name}")
    private String name;

    /**
     * 年龄
     */
    @Value("${user.age}")
    private Integer age;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

运行结果：

```json
{
    "name": "qinweizhao",
    "age": 24
}
```

### 2、SpringBoot 方式

配置文件

```yaml
person:
  name: qinweizhao
  age: 24
  birth: 1997/12/01
  pet:
      tom:
        name: 小狗
  friend:
    - one
    - two
```

entity：

```java
Person:

package com.qinweizhao.entity;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;
import org.springframework.validation.annotation.Validated;


import javax.validation.constraints.Email;
import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 *
 *  * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *  * prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * @author qinweizhao
 * @since 2021/9/17
 */
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 生日
     */
    private Date birth;

    /**
     * 邮箱e
     */
    @Email
    private String email;


    /**
     * 宠物
     */
    private Map<String,Pet> pet;

    /**
     * 朋友
     */
    private List<Object> friend;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Map<String, Pet> getPet() {
        return pet;
    }

    public void setPet(Map<String, Pet> pet) {
        this.pet = pet;
    }

    public List<Object> getFriend() {
        return friend;
    }

    public void setFriend(List<Object> friend) {
        this.friend = friend;
    }

    @Override
    public String toString() {
        return super.toString();
    }
}



Pet:

package com.qinweizhao.entity;


/**
 * @author qinweizhao
 * @since 2021/9/17
 */

public class Pet {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```

运行结果：

```json
{
    "name": "qinweizhao",
    "age": 24,
    "birth": "1997-11-30T16:00:00.000+00:00",
    "email": null,
    "pet": {
        "tom": {
            "name": "小狗"
        }
    },
    "friend": [
        "one",
        "two"
    ]
}
```

**补充**：

我们可以导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

### 3、@Value 获取值和 @ConfigurationProperties 获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件无论 yml 还是 properties 他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用 @Value；

如果说，我们专门编写了一个 javaBean 来和配置文件进行映射，我们就直接使用 @ConfigurationProperties；

### 4、注入值数据校验

1. 导包

   ```pom
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-validation</artifactId>
           </dependency>
   ```

2. 加注解

   ```java
   @Component
   @ConfigurationProperties(prefix = "person")
   @Validated
   public class Person {
   	、、、
       
       /**
        * 邮箱
        * 类上标注 @Validated 注解
        * 属性标注 @Email
        */
       @Email
       private String email;
           
       、、、
   }
   ```

### 5、@PropertySource&@ImportResource&@Bean

@**PropertySource**：加载指定的配置文件；

```java
@PropertySource(value = {"classpath:person.properties"})
加载 person.properties 配置文件
```

@**ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有 Spring 的配置文件，我们自己编写的配置文件，也不能自动识别；

想让 Spring 的配置文件生效，加载进来；@**ImportResource **标注在一个配置类上

```java
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```

SpringBoot 推荐给容器中添加组件的方式；推荐使用全注解的方式

1、配置类**@Configuration**------>Spring配置文件

2、使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyConfig {

    /**
     * 将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
     */
    @Bean
    public Person helloPerson(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new Person();
    }
}
```

### 6、占位符

1. 随机数

   ```java
   ${random.value}、${random.int}、${random.long}
   ${random.int(10)}、${random.int[1024,65536]}
   ```

2. 占位符获取之前配置的值，如果没有可以是用:指定默认值

   ```properties
   person:
     name: qinweizhao${random.uuid}
     age: ${random.int}
     birth: 1997/12/01
   #  email: 11111
     pet:
         tom:
           name: ${person.hello:hello}小狗
     friend:
       - one
       - two
   ```

   运行结果：

   ```json
   {
       "name": "qinweizhao 74f19257-6ae0-4b75-98e3-a1f5ae2176c9",
       "age": -1076407203,
       "birth": "1997-11-30T16:00:00.000+00:00",
       "email": null,
       "pet": {
           "tom": {
               "name": "hello小狗"
           }
       },
       "friend": [
           "one",
           "two"
       ]
   }
   ```

## 四、配置文件加载

### 1、加载位置

springboot 启动会扫描以下位置的 application.properties 或者 application.yml 文件作为 SpringBoot 的默认配置文件

```
–file:./config/

–file:./

–classpath:/config/

–classpath:/

优先级由高到底，高优先级的配置会覆盖低优先级的配置；SpringBoot 会从这四个位置全部加载主配置文件；互补配置；
```

可以通过 spring.config.location 来改变默认的配置文件位置；项目打包好以后，可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

```bash
java -jar boot-config-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
```

### 2、加载顺序

SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

1. 命令行参数

   所有的配置都可以在命令行上进行指定

   ```bash
   # 多个配置用空格分开； --配置项=值
   java -jar boot-config-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
   ```

2. 来自java:comp/env的JNDI属性

3. Java系统属性（System.getProperties()）

4. 操作系统环境变量

5. RandomValuePropertySource配置的random.*属性值

   由jar包外向jar包内进行寻找

   **优先加载带profile**

6. jar 包外部的 application-{profile}.propertie s或 application.yml (带 spring.profile) 配置文件

7. jar 包内部的 application-{profile}.properties 或 application.yml (带spring.profile) 配置文件

   **再来加载不带profile**

8. jar包外部的 application.properties 或 application.yml(不带spring.profile) 配置文件

9. jar包内部的 application.properties 或 application.yml(不带spring.profile) 配置文件

10. @Configuration 注解类上的 @PropertySource

11. 通过 SpringApplication.setDefaultProperties 指定的默认属性