---
title: Spring Boot 整合 Security 实现 JWT
updated:
tags:
  - Spring Boot
  - Spring Security
categories:
  - Spring Security
  - Spring Boot
---

[项目代码 已经加入到Github Spring Boot Study学习项目中](https://github.com/zlatanlong/springboot-study/tree/master/sb-09-jwt)

## 一、TokenProvider

对于JWT本身的学习可以参照百度到的内容，在项目中使用时，我们主要需要两个方法，一个是创建token，一个是解析token。我们保存的token相当于Spring Security中当前认证信息的一种序列化。

1. 继承**InitializingBean**类，主要为了初始化一些配置项的参数。

   ```java
   @Override
   public void afterPropertiesSet() throws Exception {
       byte[] keyBytes = Decoders.BASE64.decode(jwtProperties.getBase64Secret());
       this.key = Keys.hmacShaKeyFor(keyBytes);
   }
   ```

   **jwtProperties.getBase64Secret()**实际是从配置项中获取一段密码，生成的token会用这个密码加密。

2. 创建token

   ```java
   public String createToken(Authentication authentication) {
       String authorities = authentication.getAuthorities().stream()
               .map(GrantedAuthority::getAuthority)
               .collect(Collectors.joining(","));
   
   	Date validity = new Date(new Date().getTime() + jwtProperties.getTokenValidityInSeconds());
   
   	return Jwts.builder()
               .signWith(key, SignatureAlgorithm.HS512)
               .claim(AUTHORITIES_KEY, authorities)
               .setSubject(authentication.getName())
               .setExpiration(validity)
               .compact();
   }
   ```

3. 解析token

   ```java
   Authentication getAuthentication(String token) {
       Claims claims = Jwts.parser()
               .setSigningKey(key)
               .parseClaimsJws(token)
               .getBody();
       Collection<? extends GrantedAuthority> authorities =
               Arrays.stream(claims.get(AUTHORITIES_KEY).toString().split(","))
                       .map(SimpleGrantedAuthority::new)
                       .collect(Collectors.toList());
       User principal = new User(claims.getSubject(), "******", authorities);
       return new UsernamePasswordAuthenticationToken(principal, token, authorities);
   }
   ```

<!--more-->

## 二、配置过滤器

通过添加过滤器，截取每次请求中的header对应的jwt的键值对信息，判断登录情况。

1. 过滤器类**JWTFilter**， 继承**GenericFilter**类，重写`doFilter()`方法

```java
/**
 * 从请求中获取到token
 * @param request httpRequest
 * @param response httpResponse
 * @param chain filterChain
 */
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    // 获得我们token对应开头的
    String token = resolveToken((HttpServletRequest) request);
    if (StringUtils.hasText(token) && tokenProvider.validateToken(token)) {
        // 如果成功获取了就保存在SecurityContextHolder中
        Authentication authentication = tokenProvider.getAuthentication(token);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        log.info("set Authentication to security context for '{}'", authentication.getName());
    } else {
        log.info("no valid JWT token found");
    }
    chain.doFilter(request, response);
}

/**
 * 以“Bearer ”开头
 */
private String resolveToken(HttpServletRequest request) {
    String bearerToken = request.getHeader(jwtProperties.getHeader());
    if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(jwtProperties.getTokenStartWith())) {
        return bearerToken.substring(7);
    }
    return null;
}
```

2. 将过滤器添加

```java
http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
```

## 三、UserDetailsService实现类

创建UserDetailsService的实现类UserModelDetailsService，重写loadUserByUsername()方法：

```java
@Component("userDetailsService")
public class UserModelDetailsService implements UserDetailsService {

   private final Logger log = LoggerFactory.getLogger(UserModelDetailsService.class);

   private final UserRepository userRepository;

   public UserModelDetailsService(UserRepository userRepository) {
      this.userRepository = userRepository;
   }

   @Override
   @Transactional
   public UserDetails loadUserByUsername(final String username) {
      log.debug("Authenticating user '{}'", username);

      // 邮箱查询
      if (Validator.isEmail(username)) {
         return userRepository.findOneWithAuthoritiesByEmailIgnoreCase(username)
            .map(user -> createSpringSecurityUser(username, user))
            .orElseThrow(() -> new UsernameNotFoundException("User with email " + username + " was not found in the database"));
      }

      // 用户名查询
      String lowercaseLogin = username.toLowerCase(Locale.ENGLISH);
      return userRepository.findOneWithAuthoritiesByUsername(lowercaseLogin)
         .map(user -> createSpringSecurityUser(lowercaseLogin, user))
         .orElseThrow(() -> new UsernameNotFoundException("User " + lowercaseLogin + " was not found in the database"));

   }

   /**
    * 将数据库中的user转化为SpringSecurity中的user
    * @param lowercaseUsername
    * @param user
    * @return
    */
   private org.springframework.security.core.userdetails.User createSpringSecurityUser(String lowercaseUsername, User user) {
      if (!user.isActivated()) {
         throw new UserNotActivatedException("User " + lowercaseUsername + " was not activated");
      }
      List<GrantedAuthority> grantedAuthorities = user.getAuthorities().stream()
         .map(authority -> new SimpleGrantedAuthority(authority.getName()))
         .collect(Collectors.toList());
      return new org.springframework.security.core.userdetails.User(user.getUsername(),
         user.getPassword(),
         grantedAuthorities);
   }
}
```

loadUserByUsername实际上就是通过用户名/邮箱，从数据库中获取真实的用户，并且获取到权限。然后将用户以及权限转化为Spring Security中的类，交给Spring Security进行后续处理。

## 四、其他配置

1. 0Configure BCrypt password encoder 

    ```java
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    ```
    
1. 放行静态资源

   ```java
   @Override
   public void configure(WebSecurity web) {
   	web.ignoring()
   			.antMatchers(HttpMethod.OPTIONS, "/**")
               // allow anonymous resource requests
               .antMatchers(
                       "/",
                       "/*.html",
                       "/favicon.ico",
                       "/**/*.html",
                       "/**/*.css",
                       "/**/*.js",
                       "/h2-console/**"
               );
   }
   ```
   
1. no session

   ```java
   httpSecurity
       .sessionManagement()
       .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
   ```
   
1. 异常处理

   ```java
   httpSecurity.
       .exceptionHandling()
       .authenticationEntryPoint(authenticationErrorHandler)
       .accessDeniedHandler(jwtAccessDeniedHandler)
   ```
   JwtAuthenticationEntryPoint类，处理认证失败（未登录）
   
   ```java
   @Component
   public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
   
      @Override
      public void commence(HttpServletRequest request,
                           HttpServletResponse response,
                           AuthenticationException authException) throws IOException {
         // This is invoked when user tries to access a secured REST resource without supplying any credentials
         // We should just send a 401 Unauthorized response because there is no 'login page' to redirect to
         // Here you can place any message you want
         response.sendError(HttpServletResponse.SC_UNAUTHORIZED, authException.getMessage());
      }
   }
   ```
   JwtAccessDeniedHandler类，处理没有权限的操作
   ```java
   @Component
   public class JwtAccessDeniedHandler implements AccessDeniedHandler {
   
      @Override
      public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
         // This is invoked when user tries to access a secured REST resource without the necessary authorization
         // We should just send a 403 Forbidden response because there is no 'error' page to redirect to
         // Here you can place any message you want
         response.sendError(HttpServletResponse.SC_FORBIDDEN, accessDeniedException.getMessage());
      }
   }
   ```
   

## 五、登陆写法

登陆可以直接放在controller中

```java
@PostMapping("/authenticate")
public ResponseEntity<JWTToken> authorize(@Valid @RequestBody LoginDto loginDto) {

      UsernamePasswordAuthenticationToken authenticationToken =
      new UsernamePasswordAuthenticationToken(loginDto.getUsername(), loginDto.getPassword());

      Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
   SecurityContextHolder.getContext().setAuthentication(authentication);

      boolean rememberMe = (loginDto.isRememberMe() == null) ? false : loginDto.isRememberMe();
   String jwt = tokenProvider.createToken(authentication);

      HttpHeaders httpHeaders = new HttpHeaders();
   httpHeaders.add(jwtProperties.getHeader(), "Bearer " + jwt);

      return new ResponseEntity<>(new JWTToken(jwt), httpHeaders, HttpStatus.OK);
}
```

## 六、认证究竟在哪里执行

可以从controller中开始找

1.  `Authentication authentication = `

   ​					`authenticationManagerBuilder.getObject().authenticate(authenticationToken);`

   **authenticate()**方法传入前端传递的用户名和密码构成的**UsernamePasswordAuthenticationToken**类对象，Spring Security接管认证。

2. **WebSecurityConfigurerAdapter**类的**init()**方法中的`final HttpSecurity http = getHttp();`获取**HttpSecurity**类对象，

   在**getHttp()**方法中`AuthenticationManager authenticationManager = authenticationManager();`

   在**authenticationManager()**方法中`authenticationManager = authenticationConfiguration.getAuthenticationManager();`
   
   在**getAuthenticationManager()**方法中通过**AuthenticationManagerBuilder**类的对象的**build()**方法创建了一个**AuthenticationManager**的实现。
   
   **AuthenticationManager**接口如下：
   
   ```java
   public interface AuthenticationManager {
       Authentication authenticate(Authentication var1) throws AuthenticationException;
   }
   ```
   
   **AuthenticationManager**接口的实现类**ProviderManager**中对**authenticate()**方法的实现中 关注到：
   
   ```java
   
   ,.....
   for (AuthenticationProvider provider : getProviders()) {
       ......
       try {
           result = provider.authenticate(authentication);
           if (result != null) {
               copyDetails(authentication, result);
               break;
           }
       }
       catch ...
   }
   ......
   ```
   
   对每个**AuthenticationProvider**对象进行认证。
   
   **AuthenticationProvider**  ->  **AbstractUserDetailsAuthenticationProvider**  ->  **DaoAuthenticationProvider**
   
   **AuthenticationProvider** 类的后代**AbstractUserDetailsAuthenticationProvider **中对**authenticate()**方法进行了重写：
   
   ```java
   public Authentication authenticate(Authentication authentication)
         throws AuthenticationException {
      ......
      UserDetails user = this.userCache.getUserFromCache(username);
      if (user == null) {
         cacheWasUsed = false;
         try {
            user = retrieveUser(username,
                  (UsernamePasswordAuthenticationToken) authentication);
         }
         catch ......
      }
      try {
         // 第一重验证
         preAuthenticationChecks.check(user);
         // 第二重验证
         additionalAuthenticationChecks(user,
               (UsernamePasswordAuthenticationToken) authentication);
      }
      catch ......
      }
   ```
   
   其中**retrieveUser()**方法在**AbstractUserDetailsAuthenticationProvider **是一个抽象方法，在**DaoAuthenticationProvider**中实现如下：
   
   ```java
   protected final UserDetails retrieveUser(String username,
   		UsernamePasswordAuthenticationToken authentication)
   		throws AuthenticationException {
   	prepareTimingAttackProtection();
   	try {
   		UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
   		if (loadedUser == null) {
   			throw new InternalAuthenticationServiceException(
   					"UserDetailsService returned null, which is an interface contract viola
   		}
   		return loadedUser;
   	}
   	catch ......
   }
   ```
   
   可以看到调用了我们重写的**loadUserByUsername()**方法，获取到一个正确的用户。
   
   然后再进行第一重验证：locked/disabled/expired 
   
   与第二重验证：密码的验证:
   
   ```java
   protected void additionalAuthenticationChecks(UserDetails userDetails,
   		UsernamePasswordAuthenticationToken authentication)
   		throws AuthenticationException {
   	if (authentication.getCredentials() == null) {
   		logger.debug("Authentication failed: no credentials provided");
   		throw new BadCredentialsException(messages.getMessage(
   				"AbstractUserDetailsAuthenticationProvider.badCredentials",
   				"Bad credentials"));
   	}
   	String presentedPassword = authentication.getCredentials().toString();
   	if (!passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
   		logger.debug("Authentication failed: password does not match stored value");
   		throw new BadCredentialsException(messages.getMessage(
   				"AbstractUserDetailsAuthenticationProvider.badCredentials",
   				"Bad credentials"));
   	}
   }
   ```
   
   

