# SpringDoc

SpringDoc 是一款可以结合 SpringBoot 使用的 API 文档生成工具，基于OpenAPI 3，底层使用的是 Swagger 3。

## 一、基本使用

### 1、依赖

```xml
<!-- springdoc 基础依赖，必须有它 -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.14</version>
</dependency>
<!-- 如果使用了 spring security，需要额外加上依赖 -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-security</artifactId>
    <version>1.6.14</version>
</dependency>
```

### 2、用法

#### 1. 给 Controller 加注解

主要就是下面 4 个注解：
1. `@Tag` 用来设置 Controller 的名称和描述，类似于给 Postman 的 Collections 命名；1. `@ApiResponses` 和 `@ApiResponse` 用来配置响应；1. `@Operation` 用来设置接口名称和描述；1. `@Parameter` 用来设置请求参数的描述、是否必填和示例。
```
@RestController
// 响应的 MediaType 都是 application/json
@RequestMapping(path = "/process-definition", produces = "application/json")
// Tag 注解, 给整个接口起了个名字 "流程定义", 描述是 "流程定义 API"
@Tag(name = "流程定义", description = "流程定义 API")
// ApiResponses 给每个接口提供一个默认的响应, 状态码是 200, 描述是 "接口请求成功"
@ApiResponses(@ApiResponse(responseCode = "200", description = "接口请求成功"))
public class ProcessDefinitionController {<!-- -->

    // Operation 注解设置了接口名称, 接口描述
    @Operation(summary = "上传 BPMN xml 字符串 并部署", description = "此接口处理的是 xml 字符串")
    @PostMapping("/upload-and-deploy/bpmn-xml-str")
    public JsonResult&lt;?&gt; uploadAndDeployBpmnXmlStr(@RequestBody BpmnXmlReq req) {<!-- -->
        return JsonResult.of(CommonCodeEnum.OK);
    }

    @Operation(summary = "查询单个 bpmn xml 数据")
    @GetMapping("/bpmn-xml")
    public JsonResult&lt;BpmnXmlResp&gt; findBpmnXml(
            // Parameter 注解设置了请求参数的描述, 是否必填 以及示例
            @Parameter(description = "流程部署ID", required = true, example = "1234") String deployId,
            @Parameter(description = "流程资源名称", required = true, example = "xxx.bpmn") String resourceName) {<!-- -->
        return JsonResult.of(CommonCodeEnum.OK, new BpmnXmlResp());
    }
}

```

启动项目后，效果如下图：

![2023-03-15_153142](https://img.qinweizhao.com/img/2023/03/2023-03-15_153142.png)

![2023-03-15_153225](https://img.qinweizhao.com/img/2023/03/2023-03-15_153225.png)

![2023-03-15_153310](https://img.qinweizhao.com/img/2023/03/2023-03-15_153310.png)

#### 2. 给 Model 加注解

```
@Data
// Schema 注解设置这个类的描述
@Schema(description = "bpmn xml 请求参数")
public class BpmnXmlReq {<!-- -->
    // Schema 注解设置每个属性的描述和示例
    @Schema(description = "bpmn文件的内容, 字符串格式", example = "&lt;?xml version=\"1.0\" encoding=\"UTF-8\"?&gt;")
    private String xml;
    @Schema(description = "流程部署名称", example = "请假流程")
    private String deployName;
}


@Schema(description = "json结构的响应")
public class JsonResult&lt;T&gt; {<!-- -->
    @Schema(description = "状态码", example = "200")
    private Integer code;
    @Schema(description = "状态码对应的信息", example = "请求成功")
    private String message;
    @Schema(description = "给前端返回的 json 格式的内容")
    private T content;
    // 省略部分内容
}

```

效果如下图：

![2023-03-15_153405](https://img.qinweizhao.com/img/2023/03/2023-03-15_153405.png)

![2023-03-15_153451](https://img.qinweizhao.com/img/2023/03/2023-03-15_153451.png)

滑到页面的最下方，可以看到：

![2023-03-15_153601](https://img.qinweizhao.com/img/2023/03/2023-03-15_153601.png)

### 3. 上传附件

#### 3.1 错误写法

先看下错误写法😁：

```
@Operation(summary = "上传 BPMN 文件并部署", description = "此接口处理的是文件流")
@PostMapping(path = "/upload-and-deploy/bpmn-file")
public JsonResult&lt;DeploymentResp&gt; uploadAndDeployBpmnFile(
        @Parameter(description = "要上传的 BPMN 文件", required = true) MultipartFile file,
        @Parameter(description = "流程部署的名称", required = true) @RequestParam String deployName) {<!-- -->
    return processDefinitionService.uploadAndDeployBpmnFile(file, deployName);
}

```

效果如下：

![2023-03-15_153749](https://img.qinweizhao.com/img/2023/03/2023-03-15_153749.png)

这表明，springdoc 并不能根据接口的请求参数类型是 MultipartFile ，来自动识别出我们要的是上传附件。所以解决办法就是指明此接口需要媒体类型的是附件。

#### 3.2 正确写法

代码如下，我们指明此接口消费的是 `multipart/form-data` 这种媒体类型。

```
@Operation(summary = "上传 BPMN 文件并部署", description = "此接口处理的是文件流")
@PostMapping(path = "/upload-and-deploy/bpmn-file", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public JsonResult&lt;DeploymentResp&gt; uploadAndDeployBpmnFile(
        @Parameter(description = "要上传的 BPMN 文件", required = true) MultipartFile file,
        @Parameter(description = "流程部署的名称", required = true) @RequestParam String deployName) {<!-- -->
    return processDefinitionService.uploadAndDeployBpmnFile(file, deployName);
}

```

效果如下：

![2023-03-15_153859](https://img.qinweizhao.com/img/2023/03/2023-03-15_153859.png)

### 4. 排序

参考官方文档  的 5.2 swagger-ui properties 可知，有以下2个配置项可供我们给 API 和 HTTP 方法排序：
- `springdoc.swagger-ui.tagsSorter` 给 API 排序， 如果其值为 `alpha` 就表示按字母顺序排序。默认情况下（也就是不配置此项），API 的顺序由 swagger 自己决定（也就是没什么顺序）；- `springdoc.swagger-ui.operationsSorter` 给 HTTP 方法排序，其值为 `alpha` 同样表示按字母顺序排序，值为 `method` 表示根据 HTTP 请求的类型（顺序如下：DELETE &gt; GET &gt; POST &gt; PUT）排序。默认情况下，Controller 代码里面，你写的是什么顺序，swagger 就给你展示什么顺序。
#### 4.1 API 排序示例

Java 的 Controller 代码如下：

```
@Tag(name = "01 登录", description = "登录相关API")
public class AuthController {<!-- -->}

@Tag(name = "05 历史", description = "历史 API")
public class HistoryController {<!-- -->}

@Tag(name = "02 流程定义", description = "流程定义 API")
public class ProcessDefinitionController {<!-- -->}

@Tag(name = "03 流程实例", description = "流程实例 API")
public class ProcessInstanceController {<!-- -->}

@Tag(name = "04 任务", description = "任务 API")
public class TaskController {<!-- -->}

```

`application.yml` 部分内容如下：

```
springdoc:
  swagger-ui:
    # API 排序
    tags-sorter: alpha

```

最终效果如下图所示：

![2023-03-15_153956](https://img.qinweizhao.com/img/2023/03/2023-03-15_153956.png)

#### 4.2 HTTP 方法排序示例

`application.yml` 部分内容如下：

```
springdoc:
  swagger-ui:
    # API 排序
    tags-sorter: alpha
    # HTTP 方法排序
    operations-sorter: method

```

最终效果如下图所示：

![2023-03-15_154150](https://img.qinweizhao.com/img/2023/03/2023-03-15_154150.png)

## 二、高级用法

SpringDoc 与 SpringSecurity 配合_使用访问令牌来认证。

### 1、依赖

```
<!-- security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- security jwt -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>${jjwt.version}</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>${jjwt.version}</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>${jjwt.version}</version>
    <scope>runtime</scope>
</dependency>

<!-- spring doc -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>${springdoc.version}</version>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-security</artifactId>
    <version>${springdoc.version}</version>
</dependency>

```

### 2、配置

SpringDoc：

```
import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.info.License;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySpringDocConfig {<!-- -->
    /**
     * 一个自定义的 OpenAPI 对象
     *
     * @return 一个自定义的 OpenAPI 对象
     */
    @Bean
    public OpenAPI customOpenApi() {<!-- -->
        return new OpenAPI()
                .components(new Components()
                        // 设置 spring security jwt accessToken 认证的请求头 Authorization: Bearer xxx.xxx.xxx
                        .addSecuritySchemes("authScheme", new SecurityScheme()
                                .type(SecurityScheme.Type.HTTP)
                                .bearerFormat("JWT")
                                .scheme("bearer")))
                // 设置一些标题什么的
                .info(new Info()
                        .title("学习 Activiti 7 的 API")
                        .version("1.0.0")
                        .description("加油↖(^ω^)↗")
                        .license(new License()
                                .name("Apache 2.0")
                                .url("https://www.apache.org/licenses/LICENSE-2.0")));
    }
}

```

效果如下图：

![2023-03-15_154330](https://img.qinweizhao.com/img/2023/03/2023-03-15_154330.png)

SpringSecurity：

```
import com.example.activiti7learn.security.filter.JwtFilter;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.security.servlet.PathRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityCustomizer;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.DelegatingPasswordEncoder;
import org.springframework.security.crypto.password.MessageDigestPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@Configuration
@RequiredArgsConstructor
public class MySecurityConfig {<!-- -->
    private final JwtFilter jwtFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {<!-- -->
        return http
                .sessionManagement(session -&gt; session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authorizeHttpRequests(auth -&gt; auth
                        .antMatchers("/auth/**").permitAll()
                        .anyRequest().authenticated())
                .csrf(AbstractHttpConfigurer::disable)
                .formLogin(AbstractHttpConfigurer::disable)
                .httpBasic(AbstractHttpConfigurer::disable)
                // 在普通的用户名密码验证之前, 把 jwt 的过滤器加上
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
                .build();
    }

    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {<!-- -->
        // 忽略 /error 页面
        return web -&gt; web.ignoring().antMatchers("/error",
                        // 忽略 swagger 接口路径
                        "/swagger-ui/**", "/v3/api-docs/**", "/swagger-ui.html")
                // 忽略常见的静态资源路径
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }
}

```

在第 4 部分，已经有一个 springdoc 的简单配置，其中有和 JWT 相关的配置，代码片断如下（这个是从 springdoc 官网上学到的）：

```
// 设置 spring security jwt accessToken 认证的请求头 Authorization: Bearer xxx.xxx.xxx
.addSecuritySchemes("authScheme", new SecurityScheme()
        .type(SecurityScheme.Type.HTTP)
        .bearerFormat("JWT")
        .scheme("bearer")))

```

有了上面的配置，算是成功第一步了。接下来我们还要给 Controller 的方法加注解，如下：

```
import com.example.activiti7learn.common.JsonResult;
import com.example.activiti7learn.domain.common.PageInfo;
import com.example.activiti7learn.domain.common.PageReq;
import com.example.activiti7learn.domain.req.def.BpmnXmlReq;
import com.example.activiti7learn.domain.req.def.ProcessDefReq;
import com.example.activiti7learn.domain.resp.def.BpmnXmlResp;
import com.example.activiti7learn.service.ProcessDefinitionService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import io.swagger.v3.oas.annotations.security.SecurityRequirement;
import io.swagger.v3.oas.annotations.tags.Tag;
import lombok.RequiredArgsConstructor;
import org.activiti.api.process.model.ProcessDefinition;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequiredArgsConstructor
@Tag(name = "流程定义", description = "流程定义 API")
@ApiResponses(@ApiResponse(responseCode = "200", description = "接口请求成功"))
@RequestMapping(path = "/process-definition", produces = MediaType.APPLICATION_JSON_VALUE)
public class ProcessDefinitionController {<!-- -->
    private final ProcessDefinitionService processDefinitionService;

    // 重点在下面的 @SecurityRequirement, 其 name 就是前面 MySpringDocConfig addSecuritySchemes() 配置的 key
    @Operation(summary = "分页查询流程定义数据", security = @SecurityRequirement(name = "authScheme"))
    @PostMapping("/def-page-data")
    public JsonResult&lt;PageInfo&lt;ProcessDefinition&gt;&gt; findProcessDefinitionPage(@RequestBody PageReq&lt;ProcessDefReq&gt; req) {<!-- -->
        return processDefinitionService.findProcessDefinitionPage(req);
    }
}

```

上面最重要的一句是 `@Operation(summary = "分页查询流程定义数据", security = @SecurityRequirement(name = "authScheme"))` ，authScheme 就是前面 MySecurityConfig `.addSecuritySchemes("authScheme"` 的 authScheme。

效果如下图：

![2023-03-15_154454](https://img.qinweizhao.com/img/2023/03/2023-03-15_154454.png)

### 3、验证

#### 3.1 登录一个用户

首先在登录API里面登录一个用户，如下图：

![2023-03-15_154620](https://img.qinweizhao.com/img/2023/03/2023-03-15_154620.png)

由图可知 accessToken 是 `eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJzbWl0aCIsImF1dGhvcml0aWVzIjpbIlJPTEVfQUNUSVZJVElfVVNFUiJdLCJpYXQiOjE2NzU4MzM2NjIsImV4cCI6MTY3NTg1MTY2Mn0.aW4ui_kEzrvxxSQbx4XZAfR6UGU-5Q7avyOM4oDnf2IdKB4nvA8Vt0DLo1JwKMtzyvZe_4Zwi6v5s_9HUvTpiQ`

#### 3.2 给 springdoc 设置 accessToken

把 3.1 的 accessToken 设置到 springdoc 中，如下图，粘贴好 accessToken 之后，点 Authorize 按钮就行。

![2023-03-15_154725](https://img.qinweizhao.com/img/2023/03/2023-03-15_154725.png)

#### 3.3 请求一个需要认证的接口

这个时候，再请求一个需要认证的接口，就会发现，它自动加上了请求头 `Authorization: Bearer xxx.xxx.xxx`，如下 curl 代码所示：

```
curl -X 'POST' \
  'http://localhost:8081/process-definition/def-page-data' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJzbWl0aCIsImF1dGhvcml0aWVzIjpbIlJPTEVfQUNUSVZJVElfVVNFUiJdLCJpYXQiOjE2NzU4MzM2NjIsImV4cCI6MTY3NTg1MTY2Mn0.aW4ui_kEzrvxxSQbx4XZAfR6UGU-5Q7avyOM4oDnf2IdKB4nvA8Vt0DLo1JwKMtzyvZe_4Zwi6v5s_9HUvTpiQ' \
  -H 'Content-Type: application/json' \
  -d '{
  "currentPage": 1,
  "pageSize": 20,
  "req": {
    "processDefinitionId": null,
    "processDefinitionKey": "ProcessRuntimeDefKey",
    "processDefinitionKeys": null
  }
}'

```

![2023-03-15_154808](https://img.qinweizhao.com/img/2023/03/2023-03-15_154808.png)

其实费了大半天劲，要的就是自动加上请求头 `Authorization: Bearer xxx.xxx.xxx` 的效果。

