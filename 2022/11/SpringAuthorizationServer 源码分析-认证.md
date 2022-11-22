# SpringAuthorizationServer 源码分析-认证

## 一、说明

版本说明：

| 框架 | SpringBoot | SpringAuthorizationServer |
| :--: | :--------: | :-----------------------: |
| 版本 |   2.7.5    |           0.3.1           |

流程说明：

![2022-11-22_132432](https://img.qinweizhao.com/2022/11/2022-11-22_132432.png)

当接收到请求后将会经过一系列的过滤器，首先会先进行客户端认证，负责处理的过滤器为 `OAuth2ClientAuthenticationFilter` ，它会通过调用 RegisteredClientRepository  来判断传入的客户端是否正确，然后 `OAuth2TokenEndpointFilter` 会接收通过上文 `OAuth2ClientAuthenticationFilter` 客户端认证的请求。

**注意：为了更加清晰的描述，只会贴出核心代码并尽量会在代码中标明注释。**

## 二、解析

### 1、OAuth2ClientAuthenticationFilter

```java
	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		// Or [Ant [pattern='/oauth2/token', POST], Ant [pattern='/oauth2/introspect', POST], Ant [pattern='/oauth2/revoke', POST]]
		if (!this.requestMatcher.matches(request)) {
			filterChain.doFilter(request, response);
			return;
		}

		try {
      // 根据请求中的参数和授权类型组装成对应的授权认证对象
			Authentication authenticationRequest = this.authenticationConverter.convert(request);
			if (authenticationRequest instanceof AbstractAuthenticationToken) {
				((AbstractAuthenticationToken) authenticationRequest).setDetails(
            // remoteAddress 和 sessionld
						this.authenticationDetailsSource.buildDetails(request));
			}
			if (authenticationRequest != null) {
        // 进行认证-调用 RegisteredClientRepository 来判断传入的客户端是否正确（此处认证的逻辑和 OAuth2TokenEndpointFilter 相同）。
				Authentication authenticationResult = this.authenticationManager.authenticate(authenticationRequest);
				// 成功处理
        this.authenticationSuccessHandler.onAuthenticationSuccess(request, response, authenticationResult);
			}
			filterChain.doFilter(request, response);

		} catch (OAuth2AuthenticationException ex) {
      // 失败处理
			this.authenticationFailureHandler.onAuthenticationFailure(request, response, ex);
		}
	}
```

![2022-11-22_164201](https://img.qinweizhao.com/2022/11/2022-11-22_164201.png)

### 2、OAuth2TokenEndpointFilter

```java
	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		if (!this.tokenEndpointMatcher.matches(request)) {
			filterChain.doFilter(request, response);
			return;
		}

		try {
      // 获取请求参数的授权类型
			String[] grantTypes = request.getParameterValues(OAuth2ParameterNames.GRANT_TYPE);
      // 校验授权类型
			if (grantTypes == null || grantTypes.length != 1) {
				throwError(OAuth2ErrorCodes.INVALID_REQUEST, OAuth2ParameterNames.GRANT_TYPE);
			}
			// 组装登陆认证对象
			Authentication authorizationGrantAuthentication = this.authenticationConverter.convert(request);
			if (authorizationGrantAuthentication == null) {
				throwError(OAuth2ErrorCodes.UNSUPPORTED_GRANT_TYPE, OAuth2ParameterNames.GRANT_TYPE);
			}
			if (authorizationGrantAuthentication instanceof AbstractAuthenticationToken) {
				((AbstractAuthenticationToken) authorizationGrantAuthentication)
						.setDetails(this.authenticationDetailsSource.buildDetails(request));
			}
			// 认证管理器进行认证
			OAuth2AccessTokenAuthenticationToken accessTokenAuthentication =
					(OAuth2AccessTokenAuthenticationToken) this.authenticationManager.authenticate(authorizationGrantAuthentication);
			// 认证成功处理
      this.authenticationSuccessHandler.onAuthenticationSuccess(request, response, accessTokenAuthentication);
		} catch (OAuth2AuthenticationException ex) {
			SecurityContextHolder.clearContext();
      // 认证失败处理
			this.authenticationFailureHandler.onAuthenticationFailure(request, response, ex);
		}
	}
```

### 3、ProviderManager

上面两个过滤器在处理认证时都会调用 `authenticate(authorizationGrantAuthentication)` 方法。

```java
this.authenticationManager.authenticate(authorizationGrantAuthentication);
```

实际调用的是 `ProviderManager`  类中的方法。

```java
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    // 获取当前的 Authentication 对象
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();
    // 匹配相应的 AuthenticationProvider
		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
        // 认证
        // 客户端认证：AuthenticationProvider --实现类-> ClientSecretAuthenticationProvider
        // 授权码认证：AuthenticationProvider --实现类-> OAuth2AuthorizationCodeAuthenticationProvider
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
		if (result == null && this.parent != null) {
			// Allow the parent to try.
			try {
				parentResult = this.parent.authenticate(authentication);
				result = parentResult;
			}
			catch (ProviderNotFoundException ex) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException ex) {
				parentException = ex;
				lastException = ex;
			}
		}
		if (result != null) {
			if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}
			// If the parent AuthenticationManager was attempted and successful then it
			// will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent
			// AuthenticationManager already published it
			if (parentResult == null) {
				this.eventPublisher.publishAuthenticationSuccess(result);
			}

			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).
		if (lastException == null) {
			lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
					new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
		}
		// If the parent AuthenticationManager was attempted and failed then it will
		// publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the
		// parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}
		throw lastException;
	}
```

**附**：

**OAuth2AuthorizationCodeAuthenticationProvider**

```java
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		OAuth2AuthorizationCodeAuthenticationToken authorizationCodeAuthentication =
				(OAuth2AuthorizationCodeAuthenticationToken) authentication;

		OAuth2ClientAuthenticationToken clientPrincipal =
				getAuthenticatedClientElseThrowInvalidClient(authorizationCodeAuthentication);
		RegisteredClient registeredClient = clientPrincipal.getRegisteredClient();

		OAuth2Authorization authorization = this.authorizationService.findByToken(
				authorizationCodeAuthentication.getCode(), AUTHORIZATION_CODE_TOKEN_TYPE);
		if (authorization == null) {
			throw new OAuth2AuthenticationException(OAuth2ErrorCodes.INVALID_GRANT);
		}
		OAuth2Authorization.Token<OAuth2AuthorizationCode> authorizationCode =
				authorization.getToken(OAuth2AuthorizationCode.class);

		OAuth2AuthorizationRequest authorizationRequest = authorization.getAttribute(
				OAuth2AuthorizationRequest.class.getName());

		if (!registeredClient.getClientId().equals(authorizationRequest.getClientId())) {
			if (!authorizationCode.isInvalidated()) {
				// Invalidate the authorization code given that a different client is attempting to use it
				authorization = OAuth2AuthenticationProviderUtils.invalidate(authorization, authorizationCode.getToken());
				this.authorizationService.save(authorization);
			}
			throw new OAuth2AuthenticationException(OAuth2ErrorCodes.INVALID_GRANT);
		}

		if (StringUtils.hasText(authorizationRequest.getRedirectUri()) &&
				!authorizationRequest.getRedirectUri().equals(authorizationCodeAuthentication.getRedirectUri())) {
			throw new OAuth2AuthenticationException(OAuth2ErrorCodes.INVALID_GRANT);
		}

		if (!authorizationCode.isActive()) {
			throw new OAuth2AuthenticationException(OAuth2ErrorCodes.INVALID_GRANT);
		}

		// @formatter:off
		DefaultOAuth2TokenContext.Builder tokenContextBuilder = DefaultOAuth2TokenContext.builder()
				.registeredClient(registeredClient)
				.principal(authorization.getAttribute(Principal.class.getName()))
				.providerContext(ProviderContextHolder.getProviderContext())
				.authorization(authorization)
				.authorizedScopes(authorization.getAttribute(OAuth2Authorization.AUTHORIZED_SCOPE_ATTRIBUTE_NAME))
				.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
				.authorizationGrant(authorizationCodeAuthentication);
		// @formatter:on

		OAuth2Authorization.Builder authorizationBuilder = OAuth2Authorization.from(authorization);

		// ----- Access token -----
		OAuth2TokenContext tokenContext = tokenContextBuilder.tokenType(OAuth2TokenType.ACCESS_TOKEN).build();
		OAuth2Token generatedAccessToken = this.tokenGenerator.generate(tokenContext);
		if (generatedAccessToken == null) {
			OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
					"The token generator failed to generate the access token.", ERROR_URI);
			throw new OAuth2AuthenticationException(error);
		}
		OAuth2AccessToken accessToken = new OAuth2AccessToken(OAuth2AccessToken.TokenType.BEARER,
				generatedAccessToken.getTokenValue(), generatedAccessToken.getIssuedAt(),
				generatedAccessToken.getExpiresAt(), tokenContext.getAuthorizedScopes());
		if (generatedAccessToken instanceof ClaimAccessor) {
			authorizationBuilder.token(accessToken, (metadata) ->
					metadata.put(OAuth2Authorization.Token.CLAIMS_METADATA_NAME, ((ClaimAccessor) generatedAccessToken).getClaims()));
		} else {
			authorizationBuilder.accessToken(accessToken);
		}

		// ----- Refresh token -----
		OAuth2RefreshToken refreshToken = null;
		if (registeredClient.getAuthorizationGrantTypes().contains(AuthorizationGrantType.REFRESH_TOKEN) &&
				// Do not issue refresh token to public client
				!clientPrincipal.getClientAuthenticationMethod().equals(ClientAuthenticationMethod.NONE)) {

			tokenContext = tokenContextBuilder.tokenType(OAuth2TokenType.REFRESH_TOKEN).build();
			OAuth2Token generatedRefreshToken = this.tokenGenerator.generate(tokenContext);
			if (!(generatedRefreshToken instanceof OAuth2RefreshToken)) {
				OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
						"The token generator failed to generate the refresh token.", ERROR_URI);
				throw new OAuth2AuthenticationException(error);
			}
			refreshToken = (OAuth2RefreshToken) generatedRefreshToken;
			authorizationBuilder.refreshToken(refreshToken);
		}

		// ----- ID token -----
		OidcIdToken idToken;
		if (authorizationRequest.getScopes().contains(OidcScopes.OPENID)) {
			// @formatter:off
			tokenContext = tokenContextBuilder
					.tokenType(ID_TOKEN_TOKEN_TYPE)
					.authorization(authorizationBuilder.build())	// ID token customizer may need access to the access token and/or refresh token
					.build();
			// @formatter:on
			OAuth2Token generatedIdToken = this.tokenGenerator.generate(tokenContext);
			if (!(generatedIdToken instanceof Jwt)) {
				OAuth2Error error = new OAuth2Error(OAuth2ErrorCodes.SERVER_ERROR,
						"The token generator failed to generate the ID token.", ERROR_URI);
				throw new OAuth2AuthenticationException(error);
			}
			idToken = new OidcIdToken(generatedIdToken.getTokenValue(), generatedIdToken.getIssuedAt(),
					generatedIdToken.getExpiresAt(), ((Jwt) generatedIdToken).getClaims());
			authorizationBuilder.token(idToken, (metadata) ->
					metadata.put(OAuth2Authorization.Token.CLAIMS_METADATA_NAME, idToken.getClaims()));
		} else {
			idToken = null;
		}

		authorization = authorizationBuilder.build();

		// Invalidate the authorization code as it can only be used once
		authorization = OAuth2AuthenticationProviderUtils.invalidate(authorization, authorizationCode.getToken());
		// 持久化 token
		this.authorizationService.save(authorization);

		Map<String, Object> additionalParameters = Collections.emptyMap();
		if (idToken != null) {
			additionalParameters = new HashMap<>();
			additionalParameters.put(OidcParameterNames.ID_TOKEN, idToken.getTokenValue());
		}

		return new OAuth2AccessTokenAuthenticationToken(
				registeredClient, clientPrincipal, accessToken, refreshToken, additionalParameters);
	}
```

**ClientSecretAuthenticationProvider**

```java
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		OAuth2ClientAuthenticationToken clientAuthentication =
				(OAuth2ClientAuthenticationToken) authentication;

		if (!ClientAuthenticationMethod.CLIENT_SECRET_BASIC.equals(clientAuthentication.getClientAuthenticationMethod()) &&
				!ClientAuthenticationMethod.CLIENT_SECRET_POST.equals(clientAuthentication.getClientAuthenticationMethod())) {
			return null;
		}

		String clientId = clientAuthentication.getPrincipal().toString();
    // 核心位置
		RegisteredClient registeredClient = this.registeredClientRepository.findByClientId(clientId);
		if (registeredClient == null) {
			throwInvalidClient(OAuth2ParameterNames.CLIENT_ID);
		}

		if (!registeredClient.getClientAuthenticationMethods().contains(
				clientAuthentication.getClientAuthenticationMethod())) {
			throwInvalidClient("authentication_method");
		}

		if (clientAuthentication.getCredentials() == null) {
			throwInvalidClient("credentials");
		}

		String clientSecret = clientAuthentication.getCredentials().toString();
		if (!this.passwordEncoder.matches(clientSecret, registeredClient.getClientSecret())) {
			throwInvalidClient(OAuth2ParameterNames.CLIENT_SECRET);
		}

		// Validate the "code_verifier" parameter for the confidential client, if available
		this.codeVerifierAuthenticator.authenticateIfAvailable(clientAuthentication, registeredClient);

		return new OAuth2ClientAuthenticationToken(registeredClient,
				clientAuthentication.getClientAuthenticationMethod(), clientAuthentication.getCredentials());
	}
```

