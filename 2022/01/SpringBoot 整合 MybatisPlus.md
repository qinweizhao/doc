# SpringBoot 整合 MybatisPlus

> 官方地址：https://baomidou.com/

## 一、导包

### 1、MybatisPlus

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>
```

### 2、MySQL 驱动

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

兼容性说明：https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-versions.html

### 3、测试

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter-test</artifactId>
    <version>3.5.0</version>
</dependency>
```

## 二、配置

1、配置数据源；

在 application.properties 配置数据源相关信息

```properties
# 应用名称
spring.application.name=spring-boot-mybatis-plus
#************H2  Begin****************
#创建表的MySql语句位置
spring.datasource.schema=classpath:schema.sql
#插入数据的MySql语句的位置
spring.datasource.data=classpath:data.sql
#remote visit
spring.h2.console.settings.web-allow-others=true
#console url。Spring启动后，可以访问 http://127.0.0.1:8080/h2-console 查看数据库
spring.h2.console.path=/h2-console
#default true。也可以用命令行访问好数据库
spring.h2.console.enabled=true
spring.h2.console.settings.trace=true
#指定数据库的种类，这里 file意思是文件型数据库
spring.datasource.url=jdbc:h2:file:~/test
#用户名密码不需要改，都是临时值
spring.datasource.username=san
spring.datasource.password=
#指定Driver，有了Driver才能访问数据库
spring.datasource.driver-class-name=org.h2.Driver
```

2、配置MyBatis-Plus

```java
@SpringBootApplication
@MapperScan("com.qinweizhao.mybatis.plus.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
}
```

## 三、准备

entity:

```java
package com.qinweizhao.mybatis.plus.entity;

import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;

/**
 * @author qinweizhao
 * @since 2022/1/6
 */
@TableName("USERINFO")
public class UserInfo {

    /**
     * 姓名
     */
    private String id;

    /**
     * 年龄
     */
    private String age;

    /**
     * 身高
     */
    private String height;

    /**
     * 体重
     */
    private String weight;


    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getHeight() {
        return height;
    }

    public void setHeight(String height) {
        this.height = height;
    }

    public String getWeight() {
        return weight;
    }

    public void setWeight(String weight) {
        this.weight = weight;
    }


    @Override
    public String toString() {
        return "UserInfo{" +
                "id='" + id + '\'' +
                ", age='" + age + '\'' +
                ", height='" + height + '\'' +
                ", weight='" + weight + '\'' +
                '}';
    }
}
```

mapper:

```java
package com.qinweizhao.mybatis.plus.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.qinweizhao.mybatis.plus.entity.UserInfo;

/**
 * @author qinweizhao
 * @since 2022/1/6
 */
public interface UserInfoMapper extends BaseMapper<UserInfo> {
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.qinweizhao.mybatis.plus.mapper.UserInfoMapper">

</mapper>
```

service:

```java
package com.qinweizhao.mybatis.plus.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.qinweizhao.mybatis.plus.entity.UserInfo;

/**
 * @author qinweizhao
 * @since 2022/1/6
 */
public interface UserInfoService extends IService<UserInfo> {
    
}
```

```java
package com.qinweizhao.mybatis.plus.service;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.qinweizhao.mybatis.plus.entity.UserInfo;
import com.qinweizhao.mybatis.plus.mapper.UserInfoMapper;

/**
 * @author qinweizhao
 * @since 2022/1/6
 */
public class UserInfoServiceImpl extends ServiceImpl<UserInfoMapper, UserInfo> implements UserInfoService{
    
}
```

项目整体结构：

![2022-01-06_164929](https://img.qinweizhao.com/2022/01/2022-01-06_164929.png)

## 四、测试

```java
package com.qinweizhao.mybatis.plus;

import com.baomidou.mybatisplus.test.autoconfigure.MybatisPlusTest;
import com.qinweizhao.mybatis.plus.entity.UserInfo;
import com.qinweizhao.mybatis.plus.mapper.UserInfoMapper;
import org.junit.jupiter.api.Test;

import javax.annotation.Resource;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * @author qinweizhao
 * @since 2022/1/6
 */
@MybatisPlusTest
class MybatisPlusApplicationTests {

    @Resource
    private UserInfoMapper userInfoMapper;

    @Test
    void testList() {
        List<UserInfo> userInfos = userInfoMapper.selectList(null);
        assertThat(userInfos).isNotNull();
        System.out.println(userInfos);
    }
}
```

输出：

![2022-01-06_164425](https://img.qinweizhao.com/2022/01/2022-01-06_164425.png)

