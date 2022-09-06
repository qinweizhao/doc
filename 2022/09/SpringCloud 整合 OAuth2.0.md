# SpringCloud 整合 OAuth2.0

微服务认证方案：

1. 网关只负责转发请求，认证鉴权交给每个微服务控制。
2. 统一在网关层面认证鉴权，微服务只负责业务。

第一种方案代码耦合严重，每个微服务都要维护一套认证鉴权，无法做到统一认证鉴权，开发难度太大。而第二种方案实现了统一的认证鉴权，微服务只需要各司其职，专注于自身的业务。代码耦合性低，方便后续的扩展。

![2022-09-06_223135](/Users/weizhao/Note/img/2022/09/2022-09-06_223135.png)

**Spring Security OAuth** 虽然已经**停止维护**，但学习一下还是很有必要的。

## 一、环境搭建

| 名称            | 功能     |
| :-------------- | :------- |
| oauth2-auth     | 认证服务 |
| Oauth2-common   | 公共依赖 |
| oauth2-gateway  | 网关服务 |
| oauth2-resource | 资源服务 |

## 二、认证服务

### 1、添加依赖

```xml
   <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
```

### 2、Spring Security 配置

新建 `SecurityConfig` 配置类继承 `WebSecurityConfigurerAdapter` 。

#### 1. 加密方式

```java
    /**
     * 加密算法
     */
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
```

#### 2. 配置用户

```java
    @Resource
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(new BCryptPasswordEncoder());
    }
```

为了方便并没有使用数据库，在类加载时添加两个用户。`UserDetailsService`：

```java
/**
 * @author qinweizhao
 * @since 2022/6/8
 */
@Service
public class MyUserDetailsServiceImpl implements UserDetailsService {

    static List<SecurityUser> users = new ArrayList<>();

    static {
        SecurityUser admin = SecurityUser.builder().userId(UUID.randomUUID().toString().replaceAll("-", "")).username("admin").password(new BCryptPasswordEncoder().encode("123456")).authorities(AuthorityUtils.createAuthorityList("ROLE_user", "ROLE_admin")).build();
        SecurityUser user = SecurityUser.builder().userId(UUID.randomUUID().toString().replaceAll("-", "")).username("user").password(new BCryptPasswordEncoder().encode("123456")).authorities(AuthorityUtils.createAuthorityList("ROLE_user")).build();

        users.add(admin);
        users.add(user);
    }


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //从数据库中查询
        List<SecurityUser> list = users.stream().filter(p -> username.equals(p.getUsername())).limit(1).collect(Collectors.toList());
        
        //用户不存在直接抛出 UsernameNotFoundException，security 会捕获抛出 BadCredentialsException
        
        if (Objects.isNull(list.get(0))) {
            throw new UsernameNotFoundException("用户不存在！");
        }
        return list.get(0);
    }

}
```

#### 3. 注入认证管理器

```java
    /**
     * AuthenticationManager对象在OAuth2认证服务中要使用，提前放入IOC容器中
     * Oauth的密码模式需要
     */
    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
```

注入认证管理器是因为**密码模式**需要用到，并**不是必加**。

#### 4. 配置安全拦截策略

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated().and().formLogin().loginProcessingUrl("/login").permitAll().and().csrf().disable();
    }
```

由于需要验证授权码模式，所以开启表单提交模式。

### 3、OAuth 配置

认证中心的配置类，需要满足两点：继承 **AuthorizationServerConfigurerAdapter **和标注 **@EnableAuthorizationServer** 注解。

>**AuthorizationServerConfigurerAdapter** 类定义了三个方法：
>
>```java
>public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
>    public AuthorizationServerConfigurerAdapter() {
>    }
>		// 令牌端点安全约束配置
>    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
>    }
>    
>		// 客户端配置
>    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
>    }
>
>		// 令牌访问端点的配置
>    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
>    }
>}
>```

#### 1. 令牌访问安全约束配置

```java
    @Resource
    private AuthServerAuthenticationEntryPoint authenticationEntryPoint;

    /**
     * 认证服务器安全配置（令牌访问的安全约束）
     *
     * @param security security
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        // 自定义 ClientCredentialsTokenEndpointFilter，用于处理客户端id，密码错误的异常
        OAuthServerClientCredentialsTokenEndpointFilter endpointFilter = new OAuthServerClientCredentialsTokenEndpointFilter(security, authenticationEntryPoint);
        endpointFilter.afterPropertiesSet();

        security.addTokenEndpointAuthenticationFilter(endpointFilter);

        security
                // 开启 /oauth/token_key 验证端口权限访问
                .tokenKeyAccess("permitAll()")
                // 开启 /oauth/check_token 验证端口权限访问
                .checkTokenAccess("permitAll()");
    }
```

`AuthServerAuthenticationEntryPoint`：

```java
/**
 * 用于处理客户端想认证出错，包括客户端id、密码错误
 *
 * @author qinweizhao
 * @since 2022/6/7
 */
@Component
@Slf4j
public class AuthServerAuthenticationEntryPoint implements AuthenticationEntryPoint {

    /**
     * 认证失败处理器会调用这个方法返回提示信息
     */
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException {
        ResponseUtils.result(response, new Result(ResultCode.CLIENT_AUTHENTICATION_FAILED.getCode(), ResultCode.CLIENT_AUTHENTICATION_FAILED.getMsg(), null));
    }
    
}
```

#### 2. 客户端配置

并不是所有的客户端都有权限向认证中心申请令牌的，认证服务需要配置客户端唯一Id**、**秘钥**、**权限。

```java
    /**
     * 客户端详情配置
     *
     * @param clients clients
     * @throws Exception e
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 内存模式
        clients.inMemory()
                // 客户端 id
                .withClient("c1")
                // 客户端密钥
                .secret(new BCryptPasswordEncoder().encode("123"))
                // 资源 id ， 唯一，一个服务即一个资源，可以设置多个
                .resourceIds("resource1")
                // 授权类型  总共四种，1. authorization_code（授权码模式）、password（密码模式）、client_credentials（客户端模式）、implicit（简化模式）
                // refresh_token并不是授权模式，
                .authorizedGrantTypes("authorization_code", "password", "client_credentials", "implicit", "refresh_token")
                // 允许的授权范围，客户端的权限，这里的 all 是一种标识，可以自定义，为了后续的资源服务进行权限控制
                .scopes("all")
                // false 则跳转到授权页面
                .autoApprove(false)
                // 回掉的地址
                .redirectUris("https://www.qinweizhao.com");
    }
```

使用内存模式，实际项目中应使用数据库来存储。

#### 3. 端点配置

```java
    /**
     * 认证服务器端点配置（令牌访问的端点）
     *
     * @param endpoints endpoints
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                // 1、设置异常 WebResponseExceptionTranslator，用于处理用户名，密码错误、授权类型不正确的异常
                .exceptionTranslator(new AuthServerWebResponseExceptionTranslator())
                // 2、授权码模式所需要的 service
                .authorizationCodeServices(authorizationCodeServices())
                // 3、密码模式需要的认证管理器
                .authenticationManager(authenticationManager)
                // 4、令牌管理服务，必须存在
                .tokenServices(tokenServices())
                // 5、提交访问令牌（/oauth/token）只允许 POST 请求
                .allowedTokenEndpointRequestMethods(HttpMethod.POST);
    }
```

框架默认的访问端点有如下**六**个：

- **/oauth/authorize**：获取授权码的端点
- **/oauth/token**：获取令牌端点。
- **/oauth/confifirm_access**：用户确认授权提交端点。
- **/oauth/error**：授权服务错误信息端点。
- **/oauth/check_token**：用于资源服务访问的令牌解析端点。
- **/oauth/token_key**：提供公有密匙的端点，如果你使用 JWT 令牌的话。

1. `AuthServerWebResponseExceptionTranslator`

   ```java
   /**
    * @author qinweizhao
    * @since 2022/6/7
    */
   public class AuthServerWebResponseExceptionTranslator implements WebResponseExceptionTranslator {
   
       /**
        * 业务处理方法，重写这个方法返回客户端信息
        */
       @Override
       public ResponseEntity<Result> translate(Exception e) {
           Result resultMsg = doTranslateHandler(e);
           return new ResponseEntity<>(resultMsg, HttpStatus.UNAUTHORIZED);
       }
   
       /**
        * 根据异常定制返回信息
        */
       private Result doTranslateHandler(Exception e) {
           //初始值，系统错误，
           ResultCode resultCode = ResultCode.UNAUTHORIZED;
           //判断异常，不支持的认证方式
           if (e instanceof UnsupportedGrantTypeException) {
               //不支持的授权类型异常
               resultCode = ResultCode.UNSUPPORTED_GRANT_TYPE;
           } else if (e instanceof InvalidGrantException) {
               //用户名或密码异常
               resultCode = ResultCode.USERNAME_OR_PASSWORD_ERROR;
           }
           return new Result(resultCode.getCode(), resultCode.getMsg(), null);
       }
   }
   ```

2. `authorizationCodeServices()`

   ```java
       /**
        * 授权码模式的service，使用授权码模式必须注入
        */
       @Bean
       public AuthorizationCodeServices authorizationCodeServices() {
           return new InMemoryAuthorizationCodeServices();
       }
   
   ```

3. `authenticationManager`

   ```java
       /**
        * Security配置类注入
        */
       @Resource
       private AuthenticationManager authenticationManager;
   ```

4. `tokenServices()`

   ```java
       /**
        * 客户端存储策略，这里使用内存方式，后续可以存储在数据库
        */
       @Resource
       private ClientDetailsService clientDetailsService;
   
       /**
        * 令牌存储策略
        */
       @Resource
       private TokenStore tokenStore;
   
       /**
        * JWT 编码的令牌值和 OAuth 身份验证信息之间进行转换
        */
       @Resource
       private JwtAccessTokenConverter jwtAccessTokenConverter;
   
   
       /**
        * 令牌管理服务的配置
        */
       @Bean
       public AuthorizationServerTokenServices tokenServices() {
           DefaultTokenServices services = new DefaultTokenServices();
           //客户端端配置策略
           services.setClientDetailsService(clientDetailsService);
           //支持令牌的刷新
           services.setSupportRefreshToken(true);
           //令牌服务
           services.setTokenStore(tokenStore);
           //access_token的过期时间
           services.setAccessTokenValiditySeconds(60 * 60 * 2);
           //refresh_token的过期时间
           services.setRefreshTokenValiditySeconds(60 * 60 * 24 * 3);
   
           // * 令牌增强
           services.setTokenEnhancer(jwtAccessTokenConverter);
           return services;
       }
   ```

   省略部分代码，此处主要是对令牌的配置。

## 三、网关服务

### 1、添加依赖
