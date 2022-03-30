# Spring Boot 解决跨域

## 一、实现 WebMvcConfigurer 接口

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                // .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

>当 allowCredentials 为 true 时，allowingOrigins 不能包含特殊值 “ *” ，因为无法在 “Access-Control-Allow-Origin” 响应标头上设置。要允许凭据具有一组来源，请明确列出它们或考虑改用“ allowedOriginPatterns”。


## 二、过滤器

```java
@Configuration
public class GlobalCorsConfiguration {
    private CorsConfiguration buildConfig() {
		CorsConfiguration corsConfiguration = new CorsConfiguration();
		corsConfiguration.addAllowedOrigin("*");
		corsConfiguration.addAllowedHeader("*");
		corsConfiguration.addAllowedMethod("*");
		corsConfiguration.addExposedHeader("Authorization");
		return corsConfiguration;
	}

	@Bean
	public CorsFilter corsFilter() {
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", buildConfig());
		return new CorsFilter(source);
	}

}
```

## 三、@CrossOrigin 注解

```java
public class GoodsController {
@CrossOrigin(origins = "http://localhost:8080")
@GetMapping("goods-url")
public Response queryGoodsWithGoodsUrl(@RequestParam String goodsUrl) throws Exception {}
}  
```

## 补充：

注：当项目整合了 Spring Security 后，配置就会失效，因为请求会被 Spring Security 拦截。

解决方案：

在这种配置的基础上添加 Spring Security 对 CORS 的支持。

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                、、、
                .and()
                .cors()
                、、、
    }
}
```

一个 `.cors` 就开启了 Spring Security 对 CORS 的支持。
