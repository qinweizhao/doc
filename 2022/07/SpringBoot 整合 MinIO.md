# SpringBoot 整合 MinIO

## 一、准备

打开 MinIO 可视化管理平台创建一个 Bucket ，并将访问策略调整为 public 。

![2022-07-31_213413](https://img.qinweizhao.com/2022/07/2022-07-31_213413.png)

选择 Service Accounts ，点击 Create service account 获取 accessKey 和 secretKey 。

![2022-07-31_213330](https://img.qinweizhao.com/2022/07/2022-07-31_213330.png)

## 二、导包

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>8.2.1</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.6.5</version>
        </dependency>
```

## 三、配置

```yaml
minio:
  # 访问的url
  endpoint: http://127.0.0.1
  # API的端口
  port: 9000
  # 秘钥
  accessKey: jhaws0hyaXAoJkUd
  secretKey: XisfiplLISXnIseNKV6ZFrO1Qry4nZQD
  secure: false
  # 桶名
  bucket-name: qwz-test
  # 图片文件的最大大小
  image-size: 10485760
  # 文件的最大大小
  file-size: 1073741824
```

```java
package com.qinweizhao.minio.config.properties;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;


/**
 * @author qinweizhao
 * @since 2022/7/31
 */
@Data
@ConfigurationProperties(prefix = "minio")
public class MinioProperties {

    /**
     * 是一个URL，域名，IPv4或者IPv6地址
     */
    private String endpoint;

    /**
     * "TCP/IP端口号"
     */
    private Integer port;

    /**
     * "accessKey类似于用户ID，用于唯一标识你的账户"
     */
    private String accessKey;

    /**
     * "secretKey是你账户的密码"
     */
    private String secretKey;

    /**
     * "如果是true，则用的是https而不是http,默认值是true"
     */
    private boolean secure;

    /**
     * "默认存储桶"
     */
    private String bucketName;

    /**
     * 图片的最大大小
     */
    private long imageSize;

    /**
     * 其他文件的最大大小
     */
    private long fileSize;


}
```

```java
package com.qinweizhao.minio.config;

import com.qinweizhao.minio.config.properties.MinioProperties;
import io.minio.MinioClient;
import lombok.AllArgsConstructor;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author qinweizhao
 * @since 2022/7/31
 */
@EnableConfigurationProperties(MinioProperties.class)
@AllArgsConstructor
@Configuration
public class MinioConfig {


    private final MinioProperties minioProperties;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .credentials(minioProperties.getAccessKey(), minioProperties.getSecretKey())
                .endpoint(minioProperties.getEndpoint(), minioProperties.getPort(), minioProperties.isSecure())
                .build();
    }
}
```

## 四、使用

**说明：**由于做了封装，此处只贴出核心代码。

### 1、上传

controller：

```java
    @Override
    public String upload(MultipartFile file, String bucketName) {
        String fileType = FileTypeUtils.getFileType(file);
        if (fileType != null) {
            return this.putObject(file, bucketName, fileType);
        }
        return "不支持的文件格式。请确认格式,重新上传！！！";
    }

================================================================================================================

    private String putObject(MultipartFile file, String bucketName,String fileType) {
        try {
            bucketName = StringUtils.isNotBlank(bucketName) ? bucketName : minioProperties.getBucketName();
            if (!this.bucketExists(bucketName)) {
                this.makeBucket(bucketName);
            }
            String fileName = file.getOriginalFilename();

            assert fileName != null;
            String objectName = UUID.randomUUID().toString().replaceAll("-", "")
                    + fileName.substring(fileName.lastIndexOf("."));
            minioUtil.putObject(bucketName, file, objectName,fileType);
            return minioProperties.getEndpoint()+":"+minioProperties.getPort()+"/"+bucketName+"/"+objectName;
        } catch (Exception e) {
            e.printStackTrace();
            return "上传失败";
        }
    }
```

测试：

![2022-07-31_221701](https://img.qinweizhao.com/2022/07/2022-07-31_221701.png)

## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/tree/master/spring-boot-minio
