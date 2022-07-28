---
date: 2022/3/8 20:13
title: spring security的使用
---





## 配置

`WebSecurityConfigurerAdapter`这个类，是spring security中最重要的一个类，我们需要自定义的话，就需要继承该类



```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {

    }
}
```

如果我们只做了上面部分，那么我们的页面并没有任何的验证，所有的东西都需要我们重新配置

所以我们需要指定登录页面，登录的操作逻辑是哪个



```java
//自定义登录页面
http.formLogin()
  //自定义登录逻辑
  .loginProcessingUrl("/login")
  .loginPage("/login.html");

//授权
http.authorizeRequests()
  //放行登录页面
  .antMatchers("/login.html").permitAll()
  //所有请求都需要认证
  .anyRequest().authenticated();

//禁止csrf()
http.csrf().disable();
```

如果我们不指定`.loginProcessingUrl("/login")`的话，那么我们点击提交，是不能够登录成功的，后面的这个`/login`是spring security，如果我们在某个controller中也有一个`/login`，他不会走这个controller中的

> ```java
> @PostMapping("/login")
> public String login(String username,String password) {
>   log.info("执行了");
>   return "aurora1";
> }
> ```
>
> 不会走这个controller，而且就算我们controller中有一个`/login2`，我们设置处理逻辑未`.loginProcessingUrl("/login2")`，也是不会走这个controller的，并且点击提交的时候，会立马回到登录页



如果我们要放行某个请求，那么需要通过`http.authorizeRequests()`配置

`.antMatchers("xxxx").permitAll()`就是放行某个请求，或者页面

` .anyRequest().authenticated()`是其他的任何请求，都需要授权验证，但是这些其他的请求，并不包含我们通过`antMatchers("xxxx").permitAll()`进行放行的



如果我们不加上`http.csrf().disable()`的话，那么是访问不到页面的，直接网络出问题



## 自定义登录成功处理

设置登录成功是通过`successForwardUrl()`进行设置的，但是这个，存在一个问题，这里我们只能指定当前应用中的一个post请求的地址，假如我们登录成功就要跳转到百度，那么是做不到的



### 源码

```java
public FormLoginConfigurer<H> successForwardUrl(String forwardUrl) {
  successHandler(new ForwardAuthenticationSuccessHandler(forwardUrl));
  return this;
}
```

通过点进`http.formLogin().successForwardUrl()`方法中，我们发现他执行的就是上面这个代码，从中我们可以发现，登录成功后，是这个`new ForwardAuthenticationSuccessHandler(forwardUrl)`对象进行处理的，这个类实现了`AuthenticationSuccessHandler`接口



因为我们可以指定登录成功的处理类，通过`.successHandler()`进行设置，所以我们如果需要自定义登录成功逻辑的话，那么我们可以自己写一个类，实现`AuthenticationSuccessHandler`接口，然后在此`.successHandler()`方法中，使用这个处理类



代码如下

```java
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        response.sendRedirect("https://aurora.xcye.xyz");
    }
}
```



```java
//使用
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //自定义登录页面
        http.formLogin()
                //自定义登录逻辑
                xxxx
//                .successForwardUrl("/loginSuccess")
                .successHandler(new CustomAuthenticationSuccessHandler())
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

> `.successHandler()`不能和`.successForwardUrl()`一起使用


上面的地址，我们可以自己在`CustomAuthenticationSuccessHandler`类中，添加一个构造方法，传入进去



当登陆成功之后，我们可以通过认证的信息，获取到很多的用户信息，比如用户的ip地址等

```java
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        //response.sendRedirect("https://aurora.xcye.xyz");
        Object principal = authentication.getPrincipal();

        WebAuthenticationDetails webAuthenticationDetails = (WebAuthenticationDetails) authentication.getDetails();
        String remoteAddress = webAuthenticationDetails.getRemoteAddress();
        String sessionId = webAuthenticationDetails.getSessionId();
        System.out.println("ip: " + remoteAddress);

    }
}
```

> 但是需要注意的是，如果我们的访问路径未`localhost:8080`这种，那么ip地址为`0:0:0:0:0:0:0:1`，如果路径为`127.0.0.1:8080`，那么ip地址为`127.0.0.1`，我们需要特别注意localhost的ip地址

### 自定义登录失败

既然登录成功，我们可以自定义，那么登录失败也是同样的道理，只是他们实现的类不同而已

自定义登录失败的类，需要实现`AuthenticationFailureHandler`接口



## antMatchers() ant匹配规则

这是一个ant的匹配规则，请自行搜索引擎查看

## 权限控制

security默认的权限控制有以下这几种

![](https://picture.xcye.xyz/image-20220312202920765.png)

1. 第一种就是允许所有
2. denyAll就是禁止所有
3. 第三种就是匿名的
4. 第四种就是认证的
5. 第五种就是每次访问都需要经过完整验证，也就是输入用户名和密码进行验证的
6. 第六种就是记住我，多少天内，可以免登陆





- hasAuthority()

  ```java
  .antMatchers("/main.html").hasAuthority("admin")
  ```

  也就是说，访问`/main.html`页面，登录用户必须拥有admin的权限，否则便会报一个403错误

- hasAnyAuthority()

  ```java
  .antMatchers("/main.html").hasAnyAuthority("admin","noram")
  ```

  登录用户必须要拥有admin和noram权限中的其中一个，只要用户有其中一个权限就可以了

- hasAnyRole()

  ```java
  .antMatchers("/main.html").hasAnyRole("man","women")
  ```

  登录用户必须拥有`ROLE_man`或者`ROLE_women`两个角色中的其中一个

- hasRole()

  ```java
  .antMatchers("/main.html").hasRole("man")
  ```

  登录用户必须拥有`ROLE_man`角色

- hasIpAddress()

  ```java
  .antMatchers("/main.html").hasIpAddress("127.0.0.1")
  ```

  登录用户的ip地址，必须为`127.0.0.1`



### 角色和权限的区别

角色和权限是两个不同的，角色在security中，需要保证角色前缀是`ROLE_`，比如`ROLE_admin,ROLE_man`，但是权限可以是任何内容，不推荐加上`ROLE_`前缀，因为设置用户权限是一起添加的



```java
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

  String encodePassword = passwordEncoder.encode("123456");
  log.info(encodePassword);
  return new User("aurora",encodePassword,
                  AuthorityUtils.commaSeparatedStringToAuthorityList("admin,chuchen,ROLE_man,ROLE_men"));
}
```

上面的代码，就是我们为aurora这个用户增加admin,chuchen权限，增加ROLE_man,ROLE_men两个角色



还有一点需要注意，为用户增加权限，角色的时候，如果需要设置多个，我们是直接使用`,`进行隔开的，但是在`.antMatchers("/main.html").hasAnyAuthority("admin","sdlfkj")`中，是多个字符串，里面是一个可变长度的参数



## 自定义403权限错误

自定义403权限错误，我们首先需要创建一个类，实现`AccessDeniedHandler`接口

```java
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        response.setContentType("application/json;charset=utf-8");
        response.setStatus(403);
        PrintWriter writer = response.getWriter();
        writer.write("{\"code\": \"403\",\"msg\": \"权限不足\"}");
        writer.flush();
        writer.close();

    }
}
```

然后我们在通过下面这个进行设置

```java
http.exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler());
```

![](https://picture.xcye.xyz/image-20220312214330819.png)

当出现权限错误时，就是我们上面写的这个进行处理

而且我们也可以在这个里面写json数据



同理，还存在下面的这些异常，我们也可以通过这种方法进行自定义处理

![](https://picture.xcye.xyz/image-20220312214555569.png)



## access()方法

此方法的定义为

```java
public ExpressionInterceptUrlRegistry access(String attribute) {
  if (this.not) {
    attribute = "!" + attribute;
  }
  interceptUrl(this.requestMatchers, SecurityConfig.createList(attribute));
  return ExpressionUrlAuthorizationConfigurer.this.REGISTRY;
}
```

参数说明

> Allows specifying that URLs are secured by an arbitrary expression
> Params:
> attribute – the expression to secure the URLs (i.e. "hasRole('ROLE_USER') and hasRole('ROLE_SUPER')")





### 源码分析

比如我们随便看一个内置的权限控制

```java
public ExpressionInterceptUrlRegistry hasRole(String role) {
  return access(ExpressionUrlAuthorizationConfigurer
                .hasRole(ExpressionUrlAuthorizationConfigurer.this.rolePrefix, role));
}
```





![](https://picture.xcye.xyz/image-20220312220552534.png)

最终`access()`方法接收的参数为`hasRole(\"men\")`这种，所以我们可以直接在配置中，这样写

```java
.antMatchers("/main.html").access("hasRole(\"men\")")
```



### 自定义判断逻辑

并且我们也可以自定义判断逻辑



1. 创建一个类，类中定义一个方法，返回boolean值

   ```java
   public class CustomAccess {
       public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
           String requestURI = request.getRequestURI();
   
           Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
           for (GrantedAuthority grantedAuthority : authorities) {
               String authority = grantedAuthority.getAuthority();
               if (authority.equals(requestURI)) {
                   return true;
               }
           }
           return false;
       }
   }
   ```

2. 在`WebSecurityConfigurerAdapter`的子类上，实例化一个`CustomAccess customAccess = new CustomAccess();`对象

3. 通过`http.authorizeRequests().antMatchers("/main.html").access("@customAccess.hasPermission(request,authentication)")`使用

> 因为我们在`hasPermission()`方法中，需要用到`HttpServletRequest和Authentication`对象，但是`WebSecurityConfigurerAdapter`的子类里没有，我们只管传就行，框架会自动赋值的
>
> 一定要加上`@`否则会报错@customAccess.hasPermission(request,authentication)
>
> 一定要保证，我们自定义的这个类中的`hasPermission()`方法返回的是boolean值，不管你类名和方法名叫什么



> 一定要保证access("")里面@xxx对象名字和类名一样，知识首字母小写，比如access("@customAccess(request,authentication)")可以，但是access("@access(request,authentication)")就不行，尽管我们创建的CustomAccess的对象名为access，是不行的



## 基于注解的权限控制

自行必应，使用差不多



## 记住我

记住我这个功能，比如在登录页面或者是接口的方式发送请求，提供一个复选框，或者是直接`remember-me=true`，那么security就会将最后一次登录的时间保存在内容中，或者是保存在mysql数据库中，默认为两周免登录，那么下次我们就可以不同再自己登录了

```java
public static final int TWO_WEEKS_S = 1209600;
```



并且我们也可以自己定义记住我的逻辑实现类，这样就可以做到自定义的方式

![](https://picture.xcye.xyz/image-20220313091405706.png)



### 设置最后登录时间存储方式

```java
@Bean
public PersistentTokenRepository persistentTokenRepository() {
  JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();

  //设置数据源
  jdbcTokenRepository.setDataSource(dataSource);

  //启动时是否创建表 第一次要
  //jdbcTokenRepository.setCreateTableOnStartup(true);
  return jdbcTokenRepository;
}
```

这里我们使用jdbc作为数据的存储方式，当`jdbcTokenRepository.setCreateTableOnStartup(true)`时候，程序启动的时候，便会自动创建一张名为`persistent_logins`的表，但是我第二次启动的时候，就报错了，报错原因是因为重复创建表

![](https://picture.xcye.xyz/image-20220313085017707.png)



然后我们通过`http.xxx`进行设置

```java
http.rememberMe()
  //设置多长时间过期
  .tokenValiditySeconds(1000)
  //自定义登录逻辑
  .userDetailsService(new CustomUserDetailsService())
  //自定义存储方式
  .tokenRepository(persistentTokenRepository());
```

> `.userDetailsService(new CustomUserDetailsService())`这个好像是必须设置的

### 自定义记住我逻辑

我们可以通过下面这种方式设置记住我的逻辑实现

```java
http.rememberMe()
    //自定义记住我逻辑实现
    .rememberMeServices(new CustomRememberMeServices())
```



```java
public class CustomRememberMeServices implements RememberMeServices {

    @Override
    public Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {
        System.out.println("自动登录");
        return null;
    }

    @Override
    public void loginFail(HttpServletRequest request, HttpServletResponse response) {
        System.out.println("登录失败");
    }

    @Override
    public void loginSuccess(HttpServletRequest request, HttpServletResponse response, Authentication successfulAuthentication) {
        System.out.println("登录成功");

    }
}
```



然后里面的逻辑就是我们自己实现了，想怎么存储，就怎么存储



但是如果我们像上面这样，什么都不做的话，当我们退出登录，重新登录时，数据库中也不会有任何的东西，并且我们关闭浏览器，重新打开，也是没有这个记住我的功能



因为最开始的基于内存和数据库的存储，都是通过默认的记住我逻辑类去实现的，现在我们使用自己的记住我逻辑类，也就没有上面的功能，需要我们自己去实现



## Oauth2

http://localhost:8080/oauth/authorize?response_type=code&client_id=admin&redirect_url=http://aurora.xcye.xyz&scope=all

![](https://picture.xcye.xyz/image-20220313124252661.png)

关于`Oauth2`的介绍，可以看一下这篇文章，非常的详细，而且易懂[OAuth 2.0 的一个简单解释 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/04/oauth_design.html)，这部分主要介绍的就是如何使用，如何配置spring security一起来使用



### 授权服务器

oauth2的验证，我们可以分成两部分，一个是授权服务器，另一个是资源服务器，授权服务器和资源服务器他们可以同时在一个系统中

![](https://picture.xcye.xyz/image-20220313124146903.png)

- 授权服务器的作用就是一个客户端，点击下面这个链接`http://localhost:8080/oauth/authorize?response_type=code&client_id=admin&redirect_url=http://aurora.xcye.xyz&scope=all`，然后就会出来一个页面，需要用户点击授权还是拒绝，就和使用微信登录是一样的

    ![](https://picture.xcye.xyz/image-20220313124952413.png)

- 当点击授权之后，那么就会将网页重定向到我们设置的那个地址，并且会加上一个参数

    ![](https://picture.xcye.xyz/image-20220313125619797.png)

    其中`60DDkz`就是我们授权服务器返回回来的授权码，我们需要使用该授权码去请求token，然后使用token去请求资源服务器上的资源





#### 配置

首先就是我们需要用到的依赖了

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>

    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-oauth2 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
        <version>2.2.4.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



security配置

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        //禁止csrf()
        http.csrf().disable();

        http.authorizeRequests()
                .antMatchers("/ouath/**","/login/**","/logout/**").permitAll()
                .anyRequest().authenticated();

        http.formLogin()
                .loginProcessingUrl("/login")
                .successForwardUrl("/login/success")
                .loginPage("/login.html")
                .permitAll();

        //自定义异常处理
        http.exceptionHandling()
                .accessDeniedHandler(new CustomAccessDeniedHandler());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

因为获取授权码和通过授权码获取token等的请求路径是不变的，这里可以将他们放行



授权服务器配置

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServer extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()

            //设置客户端id 正常是存放在数据库中的
            .withClient("admin")
            //设置客户端密码
            .secret(passwordEncoder.encode("123456"))
            //重定向地址，可以随便
            .redirectUris("https://aurora.xcye.xyz")
            //授权范围
            .scopes("all")
            //授权类型
            .authorizedGrantTypes("authorization_code");
    }
}
```



> 一定要加上`EnableAuthorizationServer`注解，开启授权服务器
>
> 数据可以存储在本地，也可以存储在数据库中

这里的重定向地址不一定非要是localhost这种，你当前程序运行的地址，可以是任何的地址，反正是重定向操作

如果把授权类型改为`password`，那么这个授权服务器就是密码授权，同理改为其他的







### 资源客户端

```java
@Configuration
@EnableResourceServer
public class ResourceConfig extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .requestMatchers().antMatchers("/resource/userInfo");
    }
}
```

上面这个就是配置资源服务器，其实这一块和security的配置是差不多的



使用`EnableResourceServer`注解标注的类的`configure()`方法中，被设置为需要认证的路径，都是我们需要使用oauth获得的token才能进行访问的资源(`尽管你已经通过security的登录认证`)，如果我们直接发送请求，那么就会出现下面信息

![](https://picture.xcye.xyz/image-20220313131345508.png)

![](https://picture.xcye.xyz/image-20220313122513228.png)



### 使用步骤

1. 点击http://localhost:8080/oauth/authorize?response_type=code&client_id=admin&redirect_url=http://aurora.xcye.xyz&scope=all网址进行授权

    ![](https://picture.xcye.xyz/image-20220313131637211.png)

    ![](https://picture.xcye.xyz/image-20220313131941758.png)

    记住这个授权码

1. 打开postmen或者其他的工具

    按照下面这个步骤，向`/oauth/token`发送一个post请求去获取token

    ![](https://picture.xcye.xyz/image-20220313132948833.png)

    > 无论是什么模式，客户端凭证都是不能少的，也就是上面的admin,123456(可自定义)

    ![image-20220313133130234](/Users/aurora/Library/Application%20Support/typora-user-images/image-20220313133130234.png)
    
    然后点击发送，我们便可以得到一个token
    
    ```json
    {
        "access_token": "de122e45-f7ac-4d68-81bd-85f7922cdb7e",
        "token_type": "bearer",
        "expires_in": 43199,
        "scope": "all"
    }
    ```
    
3. 请求资源

    ![](https://picture.xcye.xyz/image-20220313133331479.png)

    这个`/resource/userInfo`就是我们需要通过上一步获取到的token才能去访问的资源，可以是任何的请求，只要携带上需要的参数就行

    ![](https://picture.xcye.xyz/image-20220313133628240.png)

    ![](https://picture.xcye.xyz/image-20220313133645975.png)

    这样就可以了，下图这个，就是我们直接在浏览器中，访问的结果页面

    ![](https://picture.xcye.xyz/image-20220313133742346.png)



### 密码模式

我们也可以配置密码模式

![image-20220313135156523](/Users/aurora/Library/Application%20Support/typora-user-images/image-20220313135156523.png)

![](https://picture.xcye.xyz/image-20220313135236116.png)

获取token



![](https://picture.xcye.xyz/image-20220313135303154.png)

![](https://picture.xcye.xyz/image-20220313135348650.png)

然后和前面一样，你已经获取到了token



## 使用redis存储token

```java
@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    endpoints
        //自定义登录逻辑
        .userDetailsService(customUserDetailsService)
        .authenticationManager(authenticationManager)

        //设置令牌的存储位置
        .tokenStore(tokenStore);
}
```

通过`tokenStore`进行设置

















## jwt

### 头部

头部用于描述关于该jwt的最基本的信息，例如其类型（jwt）以及签名所用的算法（例如HMAC，SHA256或RSA）等，这也可以被表示成一个json对象

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- Type: 是类型
- alg: 签名的算法



我们对头部的json字符串进行base64编码，编码后的字符串如下

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

> 这一部分是不会变的，还有头部，如果数据都一样的话，编码后的字符串都不会变



### 负载

负载就是存放有效信息的地方，除了标准中注册的声明外(建议但不强制)，我们也可以自定义一些声明



因为负载也是通过`base64`进行编码的对称加密，通过通过解密拿到里面的数据，所以敏感信息，比如密码，不能存放在里面



#### 标准中注册的声明

```json
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的
lat: jwt的签发时间
jtl: jwt的唯一身份标识，主要用来作为一次性token，从而回避攻击
```



- 公共的声明

    公共的声明可以添加任何的信息，一般添加用户的相关信息或者其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可解密

- 私有的声明

    私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息

这个指的就是自定义的claim，比如下面这个举例中的name都属于自定的claim，这些claim跟jwt标准规定的claim区别在于，jwt规定的claim，jwt的接收方在拿到jwt之后，都知道怎么对这些标准的claim进行验证（还不知道是否验证通过），而私有的claim不会验证，除非明确告诉接收方要对这些claim进行验证以及规则才行



```json
{
    "sub": "123456789",
    "naem": "aurora",
    "lat": 34876459234
}
```

其中sub是标准的声明，name是自定义的声明（公共的或者私有的）



### 签名，签证

jwt的第三部分是一个签证信息，这个签证信息是由三部分组成

1. header(base64后的)
2. payload(base64)后的
3. secret(盐，一定要保密)

这个部分需要base64加密后的header和payload使用`.`链接组成的字符串，然后通过header中声明的加密算法进行加盐`secret`组合加密，然后就构成了jwt的第三部分，如

`oHuQ_qOfk-bUfRTQWcdTyrEfFi878vms4URzlBW2tXI`

将这三部分使用`.`连接符，组合在一起，就是一个完整的jwt字符串了，比如下面这个

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjEwMDA5ODc2fQ.oHuQ_qOfk-bUfRTQWcdTyrEfFi878vms4URzlBW2tXI`

> `secret`是保存在服务器端的，jwt的签发也是在服务器端的，`secret`就是用来进行jwt的签发和jwt的验证，所以，他就是你服务端的私钥，在任何场景都不应该流露出来，一旦客户端将得知这个`secret`，那就意味着客户端可以自我签发jwt了





### 使用

 创建jwt

```java
//System.out.println(System.currentTimeMillis());
String token = Jwts.builder()
    .setId("aurora_docs")
    .setSubject("qsyyke")
    .setIssuer("by aurora")
    .setIssuedAt(new Date(1647181460864l))
    .signWith(SignatureAlgorithm.HS512, "secret").compact();

System.out.println(token);
String[] split = token.split("\\.");
//头部
System.out.println(Base64Codec.BASE64.decodeToString(split[0]));
//payload
System.out.println(Base64Codec.BASE64.decodeToString(split[1]));
//
System.out.println(Base64Codec.BASE64.decodeToString(split[2]));
```



> 这里的签发时间我直接写成了字符串，什么都没改变，所以jwt也就不会改变



解析jwt

```java
//解析
Claims claims = Jwts.parser()
    .setSigningKey("secret")
    .parseClaimsJws(token)
    .getBody();

System.out.println("jti: " + claims.getId());
System.out.println("sub: " + claims.getSubject());
System.out.println("lat: " + claims.getIssuedAt());
```





### jwt过期检验

jwt也是有生命的，如果使用上面这种方式，因为没有设置过期时间，所以生成的token是永久的

> 从服务器发出的token，服务器自己并不会做记录，就存在一个弊端就是，服务端无法主动控制某个token的立即失效



```java
Date date = new Date(1647173241736l);
//设置一分钟失效
long l = date.getTime() + (60 * 1000);

String token = Jwts.builder()
    .setId("aurora_docs")
    .setSubject("qsyyke")
    .setIssuer("by aurora")
    .setIssuedAt(new Date())
    .setExpiration(new Date(l))
    .signWith(SignatureAlgorithm.HS512, "secret").compact();

System.out.println(token);
String[] split = token.split("\\.");

//解析
Claims claims = Jwts.parser()
    .setSigningKey("secret")
    .parseClaimsJws(token)
    .getBody();


SimpleDateFormat simpleFormatter = new SimpleDateFormat("yyyy-MM-dd HH:m:ss");
System.out.println("当前时间: " + simpleFormatter.format(new Date()));
System.out.println("过期时间: " + simpleFormatter.format(claims.getExpiration()));
```

```java
当前时间: 2022-03-13 20:4:39
过期时间: 2022-03-13 20:5:38
```



如果此jwt已经过期，那么就会抛出一个异常

![](https://picture.xcye.xyz/image-20220313201117954.png)



### 自定义声明

```java
String token = Jwts.builder()
    .setId("aurora_docs")
    .setSubject("qsyyke")
    .setIssuer("by aurora")
    .setIssuedAt(new Date())
    .setExpiration(new Date(l))
    .claim("logo","aurora.jpg")
    .claim("site","https://aurora.xcye.xyz")
    .signWith(SignatureAlgorithm.HS512, "secret").compact();

Claims claims = Jwts.parser()
    .setSigningKey("secret")
    .parseClaimsJws(token)
    .getBody();


System.out.println((String)claims.get("logo"));
System.out.println((String)claims.get("site"));
```



> 自定义声明是通过`.claim("key","value")`进行设置的
>
> 获取值和map一样，使用`claims.get("key")`就可以获取到了



































