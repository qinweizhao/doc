# SpringSecurity 配置

## 一、流程

- 客户端发起一个请求，进入 Security 过滤器链。

- 当到 LogoutFilter 的时候判断是否是登出路径，如果是登出路径则到 logoutHandler ，如果登出成功则到 logoutSuccessHandler 登出成功处理，如果登出失败则由 ExceptionTranslationFilter ；如果不是登出路径则直接进入下一个过滤器。

- 当到 UsernamePasswordAuthenticationFilter 的时候判断是否为登录路径，如果是，则进入该过滤器进行登录操作，如果登录失败则到 AuthenticationFailureHandler 登录失败处理器处理，如果登录成功则到 AuthenticationSuccessHandler 登录成功处理器处理，如果不是登录请求则不进入该过滤器。

- 当到 FilterSecurityInterceptor 的时候会拿到 uri ，根据 uri 去找对应的鉴权管理器，鉴权管理器做鉴权工作，鉴权成功则到 Controller 层否则到 AccessDeniedHandler 鉴权失败处理器处理。

## 二、配置类代码

```java
/**
 * @author qinweizhao
 * @since 2021/11/10
 */
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 放行的资源
     */
    private static final String[] URL_WHITELIST = {
            "/css/**",
            "/js/**",
            "/favicon.ico"
    };


    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private MyAccessDeniedHandler myAccessDeniedHandler;

    @Autowired
    private MyAuthenticationEntryPoint myAuthenticationEntryPoint;

    @Autowired
    private MyLogoutSuccessHandler myLogoutSuccessHandler;

    /**
     * 密码编码器
     *
     * @return BCryptPasswordEncoder
     */
    @Bean
    BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 认证管理器
     *
     * @return AuthenticationManager
     */
    @Override
    public AuthenticationManager authenticationManagerBean() {
        return new ProviderManager(Arrays.asList(daoAuthenticationProvider()
                , myAuthenticationProvider()
        ));
    }

    /**
     * 认证提供者
     *
     * @return AuthenticationProvider
     */
    private AuthenticationProvider daoAuthenticationProvider() {
        return new DaoAuthenticationProvider();
    }

    /**
     * 自定义认证提供者
     *
     * @return AuthenticationProvider
     */
    private AuthenticationProvider myAuthenticationProvider() {
        return new MyAuthenticationProvider();
    }

    /**
     * (授权)
     *
     * @return 投票管理器
     */
    private AccessDecisionManager accessDecisionManager() {
        List<AccessDecisionVoter<?>> decisionVoters
                = Arrays.asList(
                new MyExpressionVoter(),
                new WebExpressionVoter(),
                new RoleVoter(),
                new AuthenticatedVoter());
        return new UnanimousBased(decisionVoters);
    }

    /**
     * 配置 AuthenticationManagerBuilder 会让 Security 自动构建一个 AuthenticationManager；
     * 如果想要使用该功能你需要配置一个 UserDetailService 和 PasswordEncoder。UserDetailsService 用于在认证器中根据用户传过来的用户名查找一个用户
     * PasswordEncoder 用于密码的加密与比对，我们存储用户密码的时候用PasswordEncoder.encode() 加密存储，在认证器里会调用 PasswordEncoder.matches() 方法进行密码比对。
     * 如果重写了该方法，Security 会启用 DaoAuthenticationProvider 这个认证器，该认证就是先调用 UserDetailsService.loadUserByUsername
     * 然后使用 PasswordEncoder.matches() 进行密码比对，如果认证成功成功则返回一个 Authentication 对象
     *
     * @param auth auth
     * @throws Exception exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }

    /**
     * 配置静态资源的处理方式，可使用 Ant 匹配规则
     *
     * @param web web
     */
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers(URL_WHITELIST);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http

                .csrf().disable()


                // 登录
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/login")
                .usernameParameter("username")
                .passwordParameter("password")
                // 如果不放行则会出现无限重定向
                .permitAll()


                // 注销
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .logoutSuccessHandler(myLogoutSuccessHandler)


                // 记住我
                .and()
                .rememberMe()
                .rememberMeParameter("remember")


                .and()
                .authorizeRequests()
                .antMatchers("/")
                .permitAll()
                .anyRequest()
                .authenticated()
                // 指定投票器
                .accessDecisionManager(accessDecisionManager())


                .and()
                .exceptionHandling()
                .authenticationEntryPoint(myAuthenticationEntryPoint)
                .accessDeniedHandler(myAccessDeniedHandler)

                .and()
                .addFilterAfter(new MyFilter(), LogoutFilter.class);
    }
}
```

## 三、配置简介

### 1、configure(AuthenticationManagerBuilder auth)

AuthenticationManager 的建造器，配置 AuthenticationManagerBuilder 会让Security 自动构建一个 AuthenticationManager（该类的功能参考流程图）；如果想要使用该功能你需要配置一个 UserDetailService 和 PasswordEncoder。UserDetailsService 用于在认证器中根据用户传过来的用户名查找一个用户， PasswordEncoder 用于密码的加密与比对，我们存储用户密码的时候用PasswordEncoder.encode() 加密存储，在认证器里会调用 PasswordEncoder.matches() 方法进行密码比对。如果重写了该方法，Security 会启用 DaoAuthenticationProvider 这个认证器，该认证就是先调用 UserDetailsService.loadUserByUsername 然后使用 PasswordEncoder.matches() 进行密码比对，如果认证成功成功则返回一个 Authentication 对象。

### 2、configure(WebSecurity web)

这个配置方法用于配置静态资源的处理方式，可使用 Ant 匹配规则。

### 3、configure(HttpSecurity http)

关键的方法

1. 登录

   ```java
                   .formLogin()
                   .loginPage("/login")
                   .loginProcessingUrl("/login")
                   .usernameParameter("username")
                   .passwordParameter("password")
                   // 如果不放行则会出现无限重定向
                   .permitAll()
   ```

   这是配置登录相关的操作从方法名可知，配置了登录页请求路径，密码属性名，用户名属性名，和登录请求路径，permitAll()代表任意用户可访问。

2. 权限

   ```java
   				.and()
                   .authorizeRequests()
                   .antMatchers("/")
                   .permitAll()
                   .anyRequest()
                   .authenticated()
                   // 指定投票器
                   .accessDecisionManager(accessDecisionManager())
   ```

   以上配置是权限相关的配置，配置了一个 `/` url 放行， anyRequest() 表示所有请求，authenticated() 表示已登录用户才能访问， accessDecisionManager() 表示绑定在 url 上的鉴权管理器。

3. 注销

   ```java
   				.and()
                   .logout()
                   .logoutSuccessUrl("/")
                   .logoutSuccessHandler(myLogoutSuccessHandler)
   ```

   登出相关配置，这里配置了登出 url 和登出成功处理器:

4. 记住我

   ```java
                   .and()
                   .rememberMe()
                   .rememberMeParameter("remember")
   ```

   配置记住我功能，并指定参数名。

5. 异常

   ```java
   				.and()
                   .exceptionHandling()
                   .authenticationEntryPoint(myAuthenticationEntryPoint)
                   .accessDeniedHandler(myAccessDeniedHandler)
   ```

   配置认证失败和鉴权失败的处理器

6. 配置自定义过滤器

   ```java
                   .and()
                   .addFilterAfter(new MyFilter(), LogoutFilter.class);
   ```

   在过滤器链中插入自己的过滤器，addFilterBefore 加在对应的过滤器之前，addFilterAfter 加在对应的过滤器之后，addFilterAt 加在过滤器同一位置，事实上框架原有的 Filter 在启动 HttpSecurity 配置的过程中，都由框架完成了其一定程度上固定的配置，是不允许更改替换的。根据测试结果来看，调用 addFilterAt 方法插入的 Filter ，会在这个位置上的原有 Filter 之前执行。

## 四、拓展

Security 可扩展的有

1. 鉴权失败处理器
2. 验证器
3. 登录成功处理器
4. 投票器
5. 自定义token处理过滤器
6. 登出成功处理器
7. 登录失败处理器
8. 自定义 UsernamePasswordAuthenticationFilter

- 鉴权失败处理器

  Security 鉴权失败默认跳转登录页面，我们可以实现 AccessDeniedHandler 接口，重写 handle() 方法来自定义处理逻辑；然后参考配置类说明将处理器加入到配置当中。

- 验证器

  实现 AuthenticationProvider 接口来实现自己验证逻辑。需要注意的是在这个类里面就算你抛出异常，也不会中断验证流程，而是算你验证失败，我们由流程图知道，只要有一个验证器验证成功，就算验证成功，所以你需要留意这一点。

- 登录成功处理器

  在 Security 中验证成功默认跳转到上一次请求页面或者路径为 "/" 的页面，我们同样可以自定义：继承 SimpleUrlAuthenticationSuccessHandler 这个类或者实现 AuthenticationSuccessHandler 接口。我这里建议采用继承的方式,SimpleUrlAuthenticationSuccessHandler 是默认的处理器，采用继承可以契合里氏替换原则，提高代码的复用性和避免不必要的错误。

- 投票器

  投票器可继承 WebExpressionVoter 或者实现 AccessDecisionVoter接口；WebExpressionVoter 是 Security 默认的投票器；我这里同样建议采用继承的方式；添加到配置的方式参考 上文；

  **注意：投票器 vote 方法返回一个int值；-1代表反对，0代表弃权，1代表赞成；投票管理器收集投票结果，如果最终结果大于等于0则放行该请求。**

- 自定义token处理过滤器

  自定义 token 处理器继承自 OncePerRequestFilter 或者 GenericFilterBean 或者 Filter 都可以，在这个处理器里面需要完成的逻辑是：获取请求里的 token，验证 token 是否合法然后填充 SecurityContextHolder ，虽然说过滤器只要添加在投票器之前就可以，但我这里还是建议添加在 http.addFilterAfter(new MyFittler(), LogoutFilter.class);

- 登出成功处理器

  实现LogoutSuccessHandler接口，添加到配置的方式参考上文。

- 登录失败处理器

  登录失败默认跳转到登录页，我们同样可以自定义。继承 SimpleUrlAuthenticationFailureHandler 或者实现 AuthenticationFailureHandler，建议采用继承。

- 自定义UsernamePasswordAuthenticationFilter

  继承 UsernamePasswordAuthenticationFilter ，然后在配置类中初始化这个过滤器，给这个过滤器添加登录失败处理器，登录成功处理器，登录管理器，登录请求 url 。

## 五、补充

**SecurityContextHolder**

> 用户在完成登录后 Security 会将用户信息存储到这个类中，之后其他流程需要得到用户信息时都是从这个类中获得，用户信息被封装成 SecurityContext ，而实际存储的类是 SecurityContextHolderStrategy ，默认的SecurityContextHolderStrategy 实现类是 ThreadLocalSecurityContextHolderStrategy 它使用了ThreadLocal来存储了用户信息。

手动填充 SecurityContextHolder 示例：

```java
UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken("test","test",list);	
SecurityContextHolder.getContext().setAuthentication(token);
```

对于使用 token 鉴权的系统，我们就可以验证token后手动填充SecurityContextHolder，填充时机只要在执行投票器之前即可，或者干脆可以在投票器中填充，然后在登出操作中清空SecurityContextHolder。

