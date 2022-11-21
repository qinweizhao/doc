# SpringBoot 整合 Spring Authorization Server

## 概述

Spring 授权服务器是一个框架，它提供了 [OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-05) 和 [OpenID Connect 1.0](https://openid.net/specs/openid-connect-core-1_0.html) 规格等相关规范。 它建立在 [Spring Security](https://spring.io/projects/spring-security) 为构建 OpenID Connect 1.0 身份提供程序和 OAuth 2 授权服务器产品提供安全、轻量级和可自定义的基础。

版本说明

| 框架 | SpringBoot | SpringAuthorizationServer |
| :--: | :--------: | :-----------------------: |
| 版本 |   2.7.5    |           0.3.1           |


## 依赖

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>0.3.1</version>
</dependency>

```

## 代码

**SecurityConfig**

```java
package com.qinweizhao.authorization.server.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;


/**
 * 访问认证服务器的一些安全措施
 *
 * @author qinweizhao
 * @since 2022/11/19
 */
@Configuration
public class SecurityConfig {

    @Bean
    @Order(2)
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(authorize ->
                    authorize.anyRequest().authenticated()
                )
                .formLogin()
                .and()
                .logout()
                .and()
                .oauth2ResourceServer().jwt();

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails user = User.builder()
                .username("admin")
                .password("admin")
                .passwordEncoder(PasswordEncoderFactories.createDelegatingPasswordEncoder()::encode)
                .roles("USER").build();
        return new InMemoryUserDetailsManager(user);
    }

}
```

**AuthorizationServerConfig**

```java
package com.qinweizhao.authorization.server.config;

import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;
import com.qinweizhao.authorization.server.util.Jwks;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.config.annotation.web.configurers.oauth2.server.authorization.OAuth2AuthorizationServerConfigurer;
import org.springframework.security.oauth2.core.AuthorizationGrantType;
import org.springframework.security.oauth2.core.ClientAuthenticationMethod;
import org.springframework.security.oauth2.core.oidc.OidcScopes;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.server.authorization.client.InMemoryRegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.config.ClientSettings;
import org.springframework.security.oauth2.server.authorization.config.ProviderSettings;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.util.matcher.RequestMatcher;

import java.util.UUID;

/**
 * 用于授权、生成令牌；注册客户端，向数据库保存操作记录
 *
 * @author qinweizhao
 * @since 2022/11/21
 */
@Configuration
public class AuthorizationServerConfig {


    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        // 定义授权服务配置器
        OAuth2AuthorizationServerConfigurer<HttpSecurity> authorizationServerConfigurer = new OAuth2AuthorizationServerConfigurer<>();

        // 获取授权服务器相关的请求端点
        RequestMatcher authorizationServerEndpointsMatcher = authorizationServerConfigurer.getEndpointsMatcher();

        http// 拦截对 授权服务器 相关端点的请求
                .requestMatcher(authorizationServerEndpointsMatcher)
                // 拦载到的请求需要认证确认（登录）
                .authorizeRequests()
                // 其余所有请求都要认证
                .anyRequest().authenticated().and()
                // 忽略掉相关端点的csrf（跨站请求）：对授权端点的访问可以是跨站的
                .csrf(csrf -> csrf.ignoringRequestMatchers(authorizationServerEndpointsMatcher))

                // 表单登录
                .formLogin().and().logout().and()
                // 应用 授权服务器的配置
                .apply(authorizationServerConfigurer);

        return http.build();
    }

    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient registeredClient = RegisteredClient
                .withId(UUID.randomUUID().toString()).clientId("c1")
                .clientSecret("{noop}123")
                .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                .authorizationGrantType(AuthorizationGrantType.PASSWORD)
                .redirectUri("https://www.qinweizhao.com")
                .scope(OidcScopes.OPENID)
                .scope("message.read")
                .scope("message.write")
                .clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
                .build();

        return new InMemoryRegisteredClientRepository(registeredClient);
    }

    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        RSAKey rsaKey = Jwks.generateRsa();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return (jwkSelector, securityContext) -> jwkSelector.select(jwkSet);
    }

    @Bean
    public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
        return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
    }

    @Bean
    public ProviderSettings providerSettings() {
        return ProviderSettings.builder().build();
    }

}
```

注：还有两个工具类未贴出。

## 效果

### 客户端模式

![2022-11-21_182603](https://img.qinweizhao.com/2022/11/2022-11-21_182603.png)

![2022-11-21_182652](https://img.qinweizhao.com/2022/11/2022-11-21_182652.png)

### 授权码模式

```http
localhost:8080/oauth2/authorize?client_id=c1&response_type=code&scope=message.read&redirect_uri=https://www.qinweizhao.com
```

![2022-11-21_182919](https://img.qinweizhao.com/2022/11/2022-11-21_182919.png)

![2022-11-21_182603](https://img.qinweizhao.com/2022/11/2022-11-21_182603.png)

或

![2022-11-21_183543](https://img.qinweizhao.com/2022/11/2022-11-21_183543.png)

![2022-11-21_183010](https://img.qinweizhao.com/2022/11/2022-11-21_183010.png)


## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/tree/master/spring-boot-authorization-server
