# Spring Security 的执行流程

## 一、Spring Security执行流程图

![2021-08-03_151308](https://img.qinweizhao.com/2021/08/2021-08-03_151308.png)

## 二、Spring Security 采用的是责任链的设计模式，它有一条很长的过滤器链。过滤器链的各个过滤器说明

![2021-08-03_152037](https://img.qinweizhao.com/2021/08/2021-08-03_152037.png)

1. WebAsyncManagerIntegrationFilter：将 Security 上下文与 Spring Web 中用于处理异步请求映射的 WebAsyncManager 进行集成。

2. SecurityContextPersistenceFilter：在每次请求处理之前将该请求相关的安全上下文信息加载到 SecurityContextHolder 中，然后在该次请求处理完成之后，将 SecurityContextHolder 中关于这次请求的信息存储到一个“仓储”中，然后将 SecurityContextHolder 中的信息清除，例如在Session中维护一个用户的安全信息就是这个过滤器处理的。

3. HeaderWriterFilter：用于将头信息加入响应中。

4. CsrfFilter：用于处理跨站请求伪造。

5. LogoutFilter：用于处理退出登录。

6. UsernamePasswordAuthenticationFilter：用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自 /login 的请求。从表单中获取用户名和密码时，默认使用的表单 name 值为 username 和 password，这两个值可以通过设置这个过滤器的usernameParameter 和 passwordParameter 两个参数的值进行修改。

7. DefaultLoginPageGeneratingFilter：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。

8. BasicAuthenticationFilter：检测和处理 http basic 认证。

9. RequestCacheAwareFilter：用来处理请求的缓存。

10. SecurityContextHolderAwareRequestFilter：主要是包装请求对象request。

11. AnonymousAuthenticationFilter：检测 SecurityContextHolder 中是否存在 Authentication 对象，如果不存在为其提供一个匿名 Authentication。

12. SessionManagementFilter：管理 session 的过滤器。

13. ExceptionTranslationFilter：处理 AccessDeniedException 和 AuthenticationException 异常。

14. FilterSecurityInterceptor：可以看做过滤器链的出口。

15. RememberMeAuthenticationFilter：当用户没有登录而直接访问资源时, 从 cookie 里找出用户的信息, 如果 Spring Security 能够识别出用户提供的remember me cookie, 用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。
