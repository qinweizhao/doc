# SpringSecurity-认证原理

SpringSecurity 默认的认证过滤器为 UsernamePasswordAuthenticationFilter，下面针对此过滤器进行研究。

## 一、分析

1. 获取到页面的用户名和密码。

2. 将 username 和 password 包装成 UsernamePasswordAuthenticationToken。

3. 获取系统的认证管理器（AuthenticationManager）来调用 authenticate 方法完成认证。

   3.1）AuthenticationManager 获取 ProviderManager 来调用 ProviderManager.authenticate()。

   3.2）ProviderManager 获取到所有的 AuthenticationProvider 判断当前的提供者能否支持，如果支持则调用 provider.authenticate(authentication)。

   ​	3.2.1）retrieveUser(username,(UsernamePasswordAuthenticationToken)authentication)。

​				3.2.1.1）loadedUser = this.getUserDetailsService().loadUserByUsername(username)；（调用我们自己的UserDetailsService来去数据库查用户，按照用户名查出来的用户的详细信息）封装成 UserDetails。

​			3.2.2）preAuthenticationChecks.check(user)；（前置检查，账号是否被锁定等…）。

​			3.2.3）additionalAuthenticationChecks（附加的认证检查）。

​				3.2.3.1）使用 passwordEncoder. matches 检查页面的密码和数据库的密码是否一致。

​			3.2.4）postAuthenticationChecks.check(user) ；（后置检查，检查密码是否过期）。

​			3.2.5）createSuccessAuthentication：将认证成功信息重新封装成UsernamePasswordAuthenticationToken。

​		3.3）3.2又返回了一个新的UsernamePasswordAuthenticationToken，然后擦掉密码。

4. eventPublisher.publishAuthenticationSuccess(result);告诉所有监听器认证成功了。

