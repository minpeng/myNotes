## spring cloud+zuul+security+outh2微服务搭建

> 网关服务
1. zuul网关配置
```
#设置转发服务id
zuul.routes.XXX.service-id=auth
zuul.routes.XXX.path=/auth/**
zuul.routes.XXX.strip-prefix=false
##转发至其他服务可携带头部信息
zuul.routes.XXX.sensitive-headers=Cookie,Set-Cookie

```

2.配置serurity资源服务网关拦截

>继承ResourceServerConfigurerAdapter,重写config方法

```
@Configuration
//开启了一个spring security的filter，这个filter通过一个Oauth2的token进行认证请求
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    private Logger logger = LoggerFactory.getLogger(getClass());
    
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated().withObjectPostProcessor(

                new ObjectPostProcessor<FilterSecurityInterceptor>() {
                    @Override
                    public <O extends FilterSecurityInterceptor> O postProcess(
                            O fsi) {
                        //设置元数据
                        fsi.setSecurityMetadataSource(customMetadataSource());
                        //设置校验
                        fsi.setAccessDecisionManager(customAccessDecision());
                        return fsi;
                    }
                }
        );


    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        // 设置token远程验证
        resources
                .tokenServices(tokenServices())
                // invalid_token
                .authenticationEntryPoint(authenticationEntryPoint())
                // access_denied
                .accessDeniedHandler(accessDeniedHandler());
    }
    
    
    /token校验
    @Primary
    @Bean
    public RemoteTokenService tokenServices() {
        RemoteTokenService tokenServices = new RemoteTokenService();
        tokenServices.setClientId("xxx");
        tokenServices.setClientSecret("xxx");
        return tokenServices;
    }


    @Bean
    public AccessDeniedHandler accessDeniedHandler() {
        return new AccessDeniedHandler() {
            @Override
            public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
                response.setStatus(200);
                response.setContentType("application/json");
                response.setCharacterEncoding("utf-8");
                PrintWriter writer = response.getWriter();
                String resp = GwErrorCodeEnum.G_SERVICE_NOT_PERMISSION.toJson();
                WriteLogHelper.commonLog(logger,resp);
                writer.write(resp);
            }
        };
    }

    @Bean
    public AuthenticationEntryPoint authenticationEntryPoint() {
        return new AuthenticationEntryPoint() {
            @Override
            public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
                response.setStatus(200);
                response.setContentType("application/json");
                response.setCharacterEncoding("utf-8");
                PrintWriter writer = response.getWriter();
                String resp =GwErrorCodeEnum.G_SERVICE_TOKEN_ILLEGAL.toJson();
                WriteLogHelper.commonLog(logger,resp);
                writer.write(resp);
            }
        };
    }
    
    //全部资源信息
    @Bean
    public FilterInvocationSecurityMetadataSource customMetadataSource() {
        return new CustomMetadataSource();
    }
    
    //校验资源信息
    @Bean
    public AccessDecisionManager customAccessDecision() {
        return new CustomAccessDecisionManager();
    }
}

```


3.自定义资源信息(和角色绑定)
>实现FilterInvocationSecurityMetadataSource接口

```
public class CustomMetadataSource implements FilterInvocationSecurityMetadataSource {

    @Autowired
    private AuthorityService authorityService;
    
    private static final String[] EXCLUDE_URLS = {"/auth"};

    private static final String ROLE_ANONYMOUS = "ROLE_ANONYMOUS";
    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public Collection<ConfigAttribute> getAttributes(Object object) {
        FilterInvocation filterInvocation = (FilterInvocation) object;
        HttpServletRequest request = filterInvocation.getRequest();
        String requestURI = request.getRequestURI();
        String ip= IPUtil.getClientIp(request);
        //判断是否需要鉴权
        boolean isExcludeUrl = Stream.of(CustomAccessDecisionManager.EXCLUDE_URLS).anyMatch(requestURI::contains);
        if (isExcludeUrl) {
            return SecurityConfig.createList(ROLE_ANONYMOUS);
        }

        // 获取所有资源授权信息
        Map<String, List<ResourceAuthority>> authorities = authorityService.selAllAuthorities();

        对url进行匹配，如果匹配到，则返回对应的权限
        List<ResourceAuthority> authority = authorities.get(requestURI);
        if (authority != null) {
            Set<String> needRolesSet = new HashSet<>();
            for (ResourceAuthority resAuth : authority) {
                Set<String> roles = Stream.of(resAuth.getRoles().split(",")).map(String::trim).collect(Collectors.toSet());
                needRolesSet.addAll(roles);
            }

            String needRoles = String.join(",", needRolesSet);
            return SecurityConfig.createListFromCommaDelimitedString(needRoles);
        }

        //接口没有匹配到权限信息
       throw new AccessDeniedException("当前访问没有权限");
       return SecurityConfig.createList(ROLE_ANONYMOUS);
    }

    @Override
    public Collection<ConfigAttribute> getAllConfigAttributes() {
        return Collections.emptyList();
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return FilterInvocation.class.isAssignableFrom(aClass);
    }
}

```

4.校验资源
>实现AccessDecisionManager接口

```
public class CustomAccessDecisionManager implements AccessDecisionManager {
    private Logger logger = LoggerFactory.getLogger(getClass());
    // 官方默认匿名用户
    private static final String DEFAULT_ANONYMITY = "anonymousUser";
    // 设置过滤接口
    private static final String[] EXCLUDE_URLS = {"/auth"};

    @Override
    public void decide(Authentication authentication, Object o, Collection<ConfigAttribute> configAttributes)  {

        String requestURI = ((FilterInvocation) o).getRequest().getRequestURI();
        //判断是否校验
        boolean isExcludeUrl = Stream.of(EXCLUDE_URLS).anyMatch(requestURI::startsWith);
        if (isExcludeUrl) {
            return;
        }

        // 对token失效或匿名(anonymousUser)访问禁止
        String user = (String) authentication.getPrincipal();

        if (user == null || DEFAULT_ANONYMITY.equals(user)) {
            throw new AccessDeniedException("Access Deny:" + user);
        }
        //获取用户角色
        Set<String> userRoles = authentication.getAuthorities().stream().map(GrantedAuthority::getAuthority).collect(Collectors.toSet());
        //获取改url需要的角色
        Set<String> needRoles = configAttributes.stream().map(ConfigAttribute::getAttribute).collect(Collectors.toSet());
        boolean roleMatched = userRoles.stream().anyMatch(needRoles::contains);
        if (!roleMatched) {
            WriteLogHelper.ErrorLog(logger, "Access Deny:Could not match roles", user, String.valueOf(userRoles), String.valueOf(needRoles),requestURI);
            throw new AccessDeniedException("Access Deny!");
        }

    }

    @Override
    public boolean supports(ConfigAttribute configAttribute) {
        return true;
    }

    @Override
    public boolean supports(Class<?> aClass) {
        return true;
    }
}

```


5.token校验
> 实现ResourceServerTokenServices 


```
public class RemoteTokenService implements ResourceServerTokenServices {
    private Logger logger = LoggerFactory.getLogger(getClass());

    private String clientId;
    private String clientSecret;

    private AccessTokenConverter tokenConverter = new DefaultAccessTokenConverter();

    @Autowired
    private AuthService authService;


    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public void setClientSecret(String clientSecret) {
        this.clientSecret = clientSecret;
    }

    public void setAccessTokenConverter(AccessTokenConverter accessTokenConverter) {
        this.tokenConverter = accessTokenConverter;
    }
    

    @Override
    public OAuth2Authentication loadAuthentication(String accessToken) {
        String authorizationHeader = getAuthorizationHeader(clientId, clientSecret);
        //通过auth服务嘉苑token是否有效
        Map<String, Object> response = authService.checkToken(authorizationHeader, accessToken);
        return tokenConverter.extractAuthentication(response);
    }

    @Override
    public OAuth2AccessToken readAccessToken(String accessToken) {
        throw new UnsupportedOperationException("Not supported: read access token");
    }
    
    //获取头部校验信息(jwt)
    private String getAuthorizationHeader(String clientId, String clientSecret) {
        if (clientId == null || clientSecret == null) {
            WriteLogHelper.ErrorLog(logger, "getAuthorizationHeader error", "Null Client ID or Client Secret detected. Endpoint that requires authentication will reject request with 401 error.");

        }

        String creds = String.format("%s:%s", clientId, clientSecret);
        try {
            return "Basic " + new String(Base64.getEncoder().encode(creds.getBytes("UTF-8")));
        } catch (UnsupportedEncodingException e) {
            throw new IllegalStateException("Could not convert String");
        }
    }

}
```

> auth鉴权服务

1.auth授权服务配置
>继承AuthorizationServerConfigurerAdapter

```

@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private UserService userService;

    /**
     * Auth服务器安全配置
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security
                // 请求token所需权限
                .tokenKeyAccess("permitAll()")
                // 验证token所需权限(需要验证通过)
                .checkTokenAccess("isAuthenticated()");
    }

    /**
     * 客户端相关配置
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 客户端信息使用in-memory存储
        clients.inMemory()
                //需要和网关保持一致
                // 客户端名
                .withClient("XXX")
                // 设置客户端secret，默认bcypt编码
                .secret(PasswordEncoderFactories.createDelegatingPasswordEncoder().encode("XXX"))
                // 授权类型(设置为密码)
                .authorizedGrantTypes("password")
                .scopes("XXX");
    }

    /**
     * Auth服务器非安全配置，如token等
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // token增强链，添加token自定义增强器
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Arrays.asList(customTokenEnhancer(), jwtAccessTokenConverter()));

        //端点映射
        endpoints.pathMapping("/oauth/token", "/auth/token")
                .pathMapping("/oauth/check_token", "/auth/check_token")
                // password授权类型必须显式添加authenticationManager
                .authenticationManager(authenticationManager)
                // token存储
                .tokenStore(jwtTokenStore())
                // token增强
                .tokenEnhancer(tokenEnhancerChain)
                // token转换器，定义token如何转换为请求token时的响应
                .accessTokenConverter(jwtAccessTokenConverter())
                // 自定义认证失败响应
                .exceptionTranslator(customWebResponseExceptionTranslator());

    }


    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new CustomJwtAccessTokenConverter();
        //设置签名密钥(密钥最好定期更换)
        jwtAccessTokenConverter.setSigningKey("XXXX");
        return jwtAccessTokenConverter;
    }

    //使用JWT作为token
    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    /**
     * 自定义认证失败响应
     */
    @Bean
    public WebResponseExceptionTranslator customWebResponseExceptionTranslator() {
        return new CustomAuthExceptionTranslator();
    }

    /**
     * token 加强
     */
    @Bean
    @SuppressWarnings("unchecked")
    public TokenEnhancer customTokenEnhancer() {
        return (accessToken, authentication) -> {
            String username = ((UserDetails) authentication.getPrincipal()).getUsername();
            //根据账号查询用户信息
            TokenUserInfo userInfo = userService.findUserInfo(username);
            if (userInfo == null) {
                WriteLogHelper.ErrorLog(logger, "not found user", username);
                throw new CustomAuthException(JSONObject.toJSONString(AuthResultState.NAME_PASSWORD_ERROR.toCommonResult()));
            }
            if (userInfo.getCompanyStatus()!=null && 1!=userInfo.getCompanyStatus()) {
                WriteLogHelper.ErrorLog(logger, "user company is delete", username, String.valueOf(userInfo));
                throw new CustomAuthException(JSONObject.toJSONString(AuthResultState.COMPANY_DELETE.toCommonResult()));
            }
            Map<String, Object> map = new HashMap<>();
            map.put(AuthRoleConstants.TOKEN_USER, userInfo);
            ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(map);
            return accessToken;
        };
    }

    /**
     * token服务 注销时需要提供
     */
    @Bean
    public DefaultTokenServices tokenServices() {
        DefaultTokenServices tokenServices = new DefaultTokenServices();
        tokenServices.setAuthenticationManager(authenticationManager);
        tokenServices.setTokenStore(jwtTokenStore());
        tokenServices.setTokenEnhancer(jwtAccessTokenConverter());
        return tokenServices;
    }
}
```

2.web安全配置
>继承WebSecurityConfigurerAdapter

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    CustomAuthProvider customAuthProvider;

    @Autowired
    VerificationCodeFilter verificationCodeFilter;

    //设置用户认证方式
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthProvider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //增加验证码
        http.csrf().ignoringAntMatchers("/auth/**").and()
                .addFilterBefore(verificationCodeFilter, UsernamePasswordAuthenticationFilter.class);
    }

    // 采用密码授权模式需要显式配置AuthenticationManager
    @Bean(name = "authenticationManage")
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return authentication -> customAuthProvider.authenticate(authentication);

    }

}
```