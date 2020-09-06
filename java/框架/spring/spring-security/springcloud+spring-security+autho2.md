# SpringCould整合spring-security+oauth2(亲测)

 **作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**



## 1.OAuth2 概念

-  OAuth2 其实是一个关于授权的网络标准，它制定了设计思路和运行流程，利用这个标准我们其实是可以自己实现 OAuth2 的认证过程的。

  ![oauth2.png](https://ae03.alicdn.com/kf/U4451e76dbf5345e0b5d74f5d4c240dc69.jpg) 

OAuth 2 有四种授权模式:

- 授权码模式（authorization code)

- 简化模式（implicit)

- 密码模式（resource owner password credentials）

- 客户端模式（client credentials）

  具体 OAuth2 是什么，可以参考这篇文章。(http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 

## 2.什么情况下需要用 OAuth2

例子：

```
    首先大家最熟悉的就是几乎每个人都用过的，比如用微信登录、用 QQ 登录、用微博登录、用 Google 账号登录、用 github 授权登录等等，这些都是典型的 OAuth2 使用场景。假设我们做了一个自己的服务平台，如果不使用 OAuth2 登录方式，那么我们需要用户先完成注册，然后用注册号的账号密码或者用手机验证码登录。而使用了 OAuth2 之后，相信很多人使用过、甚至开发过公众号网页服务、小程序，当我们进入网页、小程序界面，第一次使用就无需注册，直接使用微信授权登录即可，大大提高了使用效率。因为每个人都有微信号，有了微信就可以马上使用第三方服务，这体验不要太好了。而对于我们的服务来说，我们也不需要存储用户的密码，只要存储认证平台返回的唯一ID 和用户信息即可。 
```

​       以上是使用了 OAuth2 的授权码模式，利用第三方的权威平台实现用户身份的认证。当然了，如果你的公司内部有很多个服务，可以专门提取出一个认证中心，这个认证中心就充当上面所说的权威认证平台的角色，所有的服务都要到这个认证中心做认证

这样一说，发现没，这其实就是个单点登录的功能。这就是另外一种使用场景，对于多服务的平台，可以使用 OAuth2 实现服务的单点登录，只做一次登录，就可以在多个服务中自由穿行，当然仅限于授权范围内的服务和接口。

## 3.具体使用

​     OAuth2 其实是一个关于授权的网络标准，它制定了设计思路和运行流程，利用这个标准我们其实是可以自己实现 OAuth2 的认证过程的。今天要介绍的 spring-cloud-starter-oauth2 ，其实是 Spring Cloud 按照 OAuth2 的标准并结合 spring-security 封装好的一个具体实现。 

### 3.1 系统架构说明

![OAuth2架构时序图.png](https://ae03.alicdn.com/kf/U4a9fe5c89c134b6797305b03415baa5fr.jpg)

-  认证服务：OAuth2 主要实现端，Token 的生成、刷新、验证都在认证中心完成。 
- 后台服务： 接收到请求后会到认证中心验证
- 前端：认证服务、后台服务之间的联调

上图描述了使用了 前端与OAuth2 认证服务、微服务间的请求过程。大致的过程就是前端用用户名和密码到后台服务登录，成功后后台服务到认证服务端换取 token，返回给前端，前端拿着 token 去各个微服务请求数据接口，一般这个 token 是放到 header 中的。当微服务接到请求后，先要拿着 token 去认证服务端检查 token 的合法性，如果合法，再根据用户所属的角色及具有的权限动态的返回数据 



### 3.2 创建并配置认证服务端

配置最多的就是认证服务端，验证账号、密码，存储 token，检查 token ,刷新 token 等都是认证服务端的工作。 

#### 3.2.1 引入需要的maven包

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

 之所以引入 redis 包，是因为下面会介绍一种用 redis 存储 token 的方式。 

#### 3.2.2 配置好 application.yml

```
spring:
  application:
    name: auth-server
  redis:
    database: 2
    host: localhost
    port: 6379

server:
  port: 6001

management:
  endpoint:
    health:
      enabled: true
```

#### 3.2.3 spring security 基础配置

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * 允许匿名访问所有接口 主要是 oauth 接口
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/**").permitAll();
    }
}
```

使用`@EnableWebSecurity`注解修饰，并继承自`WebSecurityConfigurerAdapter`类。

这个类的重点就是声明 `PasswordEncoder` 和 `AuthenticationManager`两个 Bean。稍后会用到。其中 `BCryptPasswordEncoder`是一个密码加密工具类，它可以实现不可逆的加密，`AuthenticationManager`是为了实现 OAuth2 的 password 模式必须要指定的授权管理 Bean。

#### 3.2.4  实现 UserDetailsService

如果你之前用过 Security 的话，那肯定对这个类很熟悉，它是实现用户身份验证的一种方式，也是最简单方便的一种。另外还有结合 `AuthenticationProvider`的方式，有机会讲 Security 的时候再展开来讲吧。

`UserDetailsService`的核心就是 `loadUserByUsername`方法，它要接收一个字符串参数，也就是传过来的用户名，返回一个 `UserDetails`对象。

```java
@Component(value = "kiteUserDetailsService")
public class KiteUserDetailsService implements UserDetailsService {


    @Autowired
    private UserRepository userRepository;

    /**
     * Security的登录，User赋予权限
     *
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if (StringUtils.isBlank(username)) {
            throw new UsernameNotFoundException("the username is not null");
        }
        
        //校验用户是否存在
        User user = userRepository.getById(username);
        if (null == user){
            throw new UsernameNotFoundException("the user is not exist");
        }

        //给用户添加角色权限
        String role = user.getRole();
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority(role));

        //返回用户token
        return new org.springframework.security.core.userdetails.User(username, user.getOauthpassword(), authorities);
    }

```

#### 3.2.5  OAuth2 配置文件

 创建一个配置文件继承自 `AuthorizationServerConfigurerAdapter` 

```java
@Configuration
@EnableAuthorizationServer
public class OAuth2Config extends AuthorizationServerConfigurerAdapter {

    /**
     * 指定密码的加密方式
     */
    @Autowired
    public PasswordEncoder passwordEncoder;

    /**
     * 该对象为刷新token提供支持
     */
    @Autowired
    public UserDetailsService kiteUserDetailsService;

    /**
     * 该对象用来支持password模式
     */
    @Autowired
    private AuthenticationManager authenticationManager;

    /**
     * 该对象用来讲令牌信息存储到内存中
     */
    @Autowired
    private TokenStore redisTokenStore;

    /**
     * 密码模式下配置认证管理器 AuthenticationManager,并且设置 AccessToken的存储介质tokenStore,如        果不设置，则会默认使用内存当做存储介质。
     * 而该AuthenticationManager将会注入 2个Bean对象用以检查(认证)
     * 1、ClientDetailsService的实现类 JdbcClientDetailsService (检查 ClientDetails 对象)
     * 2、UserDetailsService的实现类 KiteUserDetailsService (检查 UserDetails 对象)
     */
    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        /** redis token 方式*/
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(kiteUserDetailsService)
                .tokenStore(redisTokenStore);

    }

    /**
     * 配置 oauth_client_details【client_id和client_secret等】信息的认证【检查ClientDetails的合        法性】服务
     * 设置 认证信息的来源：数据库 (可选项：数据库和内存,使用内存一般用来作测试)
     * 自动注入：ClientDetailsService的实现类 JdbcClientDetailsService (检查 ClientDetails 对        象)
     * 1.inMemory 方式存储的，将配置保存到内存中，相当于硬编码了。正式环境下的做法是持久化到数据库中，比如        mysql 中。
     * 2. secret加密是client_id:secret 然后通过base64编码后的字符串
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //添加客户端信息
        //使用内存存储OAuth客服端信息
        clients.inMemory()
                // client_id 客户单ID
                .withClient("order-client")
                // client_secret 客户单秘钥
                .secret(passwordEncoder.encode("order-secret-8888"))
                // 该客户端允许的授权类型，不同的类型，则获取token的方式不一样
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                // token 有效期
                .accessTokenValiditySeconds(3600)
                // 允许的授权范围
                .scopes("all")
                .and()
                .withClient("user-client")
                .secret(passwordEncoder.encode("user-secret-8888"))
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                .accessTokenValiditySeconds(3600)
                .scopes("all");
    }

    /**
     * 配置：安全检查流程
     * 默认过滤器：BasicAuthenticationFilter
     * 1、oauth_client_details表中clientSecret字段加密【ClientDetails属性secret】
     * 2、CheckEndpoint类的接口 oauth/check_token 无需经过过滤器过滤，默认值：denyAll()
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        ///允许客户表单认证
        security.allowFormAuthenticationForClients();
        //对于CheckEndpoint控制器[框架自带的校验]的/oauth/check端点允许所有客户端发送器请求而不会被  Spring-security拦截
        security.checkTokenAccess("isAuthenticated()");
        security.tokenKeyAccess("isAuthenticated()");
    }
}
```

有三个 configure 方法的重写。

`AuthorizationServerEndpointsConfigurer`参数的重写

```java
endpoints.authenticationManager(authenticationManager)
                .userDetailsService(kiteUserDetailsService)
                .tokenStore(redisTokenStore);
```

`authenticationManage()` 调用此方法才能支持 password 模式。

`userDetailsService()` 设置用户验证服务。

`tokenStore()` 指定 token 的存储方式。

redisTokenStore Bean 的定义如下：

```
@Configuration
public class RedisTokenStoreConfig {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public TokenStore redisTokenStore (){
        return new RedisTokenStore(redisConnectionFactory);
    }
}
```

ClientId、Client-Secret：这两个参数对应请求端定义的 cleint-id 和 client-secret

authorizedGrantTypes 可以包括如下几种设置中的一种或多种：

- authorization_code：授权码类型。
- implicit：隐式授权类型。
- password：资源所有者（即用户）密码类型。
- client_credentials：客户端凭据（客户端ID以及Key）类型。
- refresh_token：通过以上授权获得的刷新令牌来获取新的令牌。

accessTokenValiditySeconds：token 的有效期

scopes：用来限制客户端访问的权限，在换取的 token 的时候会带上 scope 参数，只有在 scopes 定义内的，才可以正常换取 token。

上面代码中是使用 inMemory 方式存储的，将配置保存到内存中，相当于硬编码了。正式环境下的做法是持久化到数据库中，比如 mysql 中。(**优化认证服务有实例**)

#### 3.3.6 创建数据库SpringCloud、user表、实体User、UserRepository

**实体bean**

```java
@Entity
@Table(
        name = "user"
)
@Setter
@Getter
public class User implements Serializable {
    @Id
    @GeneratedValue(generator = "uuidGenerator")
    @GenericGenerator(name = "uuidGenerator", strategy = "uuid")
    @Column(name = "id", nullable = false)
    private String id;
    @Column(name = "username")
    private String username;
    @Column(name = "oauth_password")
    private String oauthpassword;
    @Column(name = "role")
    private String role;
}
```

**jpa接口**

```java
public interface UserRepository extends JpaRepository<User, String> {

    @Query("select r from User r where r.id = ?1 ")
    User getById(String username);

}
```

**user表**

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `oauth_password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `role` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'admin', '$2a$10$D3PEtxvJ.N9Ko6osFaO4SO/jYcC8v7RHP34gZNk5THMvX7H5g8/NS', 'ROLE_ADMIN');
INSERT INTO `user` VALUES ('2', 'Custon', '$2a$10$D3PEtxvJ.N9Ko6osFaO4SO/jYcC8v7RHP34gZNk5THMvX7H5g8/NS', 'ROLE_ADMIN');

SET FOREIGN_KEY_CHECKS = 1;

```

#### 3.2.6 启动认证服务

 完成之后，启动项目，如果你用的是 IDEA 会在下方的 Mapping 窗口中看到 oauth2 相关的 RESTful 接口。 

![copy.png](https://ae02.alicdn.com/kf/U1770cdc935ac433c87a68cf651ed2e4eH.jpg)

主要有如下几个：

```
POST /oauth/authorize  授权码模式认证授权接口
GET/POST /oauth/token  获取 token 的接口
POST  /oauth/check_token  检查 token 合法性接口
```

### 3.3 创建用户客户端项目

 上面创建完成了认证服务端，下面开始创建一个客户端，对应到我们系统中的业务相关的微服务。我们假设这个微服务项目是管理用户相关数据的，所以叫做用户客户端。 

#### 3.3.1  引用相关的 maven 包

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 3.3.2  application.yml 配置文件

```java
spring:
  application:
    name: client-user
  redis:
    database: 2
    host: localhost
    port: 6379

server:
  port: 6101
  servlet:
    context-path: /client-user

security:
  oauth2:
    client:
      client-id: user-client
      client-secret: user-secret-8888
      user-authorization-uri: http://localhost:6001/oauth/authorize
      access-token-uri: http://localhost:6001/oauth/token
    resource:
      id: user-client
      user-info-uri: user-info
    authorization:
      check-token-access: http://localhost:6001/oauth/check_token
```

上面是常规配置信息以及 redis 配置，重点是下面的 security 的配置，这里的配置稍有不注意就会出现 401 或者其他问题。

client-id、client-secret 要和认证服务中的配置一致，如果是使用 inMemory 还是 jdbc 方式。

user-authorization-uri 是授权码认证方式需要的，下一篇文章再说。

access-token-uri 是密码模式需要用到的获取 token 的接口。

authorization.check-token-access 也是关键信息，当此服务端接收到来自客户端端的请求后，需要拿着请求中的 token 到认证服务端做 token 验证，就是请求的这个接口.

#### 3.3.3 资源配置文件

在 OAuth2 的概念里，所有的接口都被称为资源，接口的权限也就是资源的权限，所以 Spring Security OAuth2 中提供了关于资源的注解 `@EnableResourceServer`，和 `@EnableWebSecurity`的作用类似。

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String secret;

    @Value("${security.oauth2.authorization.check-token-access}")
    private String checkTokenEndpointUrl;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public TokenStore redisTokenStore (){
        return new RedisTokenStore(redisConnectionFactory);
    }

    @Bean
    public RemoteTokenServices tokenService() {
        RemoteTokenServices tokenService = new RemoteTokenServices();
        tokenService.setClientId(clientId);
        tokenService.setClientSecret(secret);
        tokenService.setCheckTokenEndpointUrl(checkTokenEndpointUrl);
        return tokenService;
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenServices(tokenService());
    }
}
```

 因为使用的是 redis 作为 token 的存储，所以需要特殊配置一下叫做 tokenService 的 Bean，通过这个 Bean 才能实现 token 的验证。 

#### 3.3.4  最后，添加一个 RESTful 接口

```
@Slf4j
@RestController
public class UserController {

    @GetMapping(value = "get")
    @PreAuthorize("hasAnyRole('ROLE_ADMIN')")
    public Object get(){
         Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
         
        return authentication.getName();
    }
}
```

一个 RESTful 方法，只有当访问用户具有 ROLE_ADMIN 权限时才能访问，否则返回 401 未授权。

通过 Authentication 参数或者 `SecurityContextHolder.getContext().getAuthentication()` 可以拿到授权信息进行查看。

### 3.4 测试

#### 3.4.1 获取token

```
http://localhost:6001/oauth/token?username=2&password=123456&grant_type=password&scope=all&client_id=user-client&client_secret=user-secret-8888
```

![copy.png](https://ae04.alicdn.com/kf/Uea945b8707be4c6280cb744ee491cea6J.jpg)



#### 3.4.3 校验token

![checktoken.png](https://ae02.alicdn.com/kf/U0101ecaf690d4376b4ab678d518d8d45q.jpg)

接口地址 http://localhost:6001/oauth/check_token?token=5f861834-9c6f-4424-af1d-df35fefddee3 

正常返回结果：

```
{
    "active": true,
    "exp": 1597915851,
    "user_name": "2",
    "authorities": [
        "ROLE_ADMIN"
    ],
    "client_id": "user-client",
    "scope": [
        "all"
    ]
}
```

校验失败结果：

```
{
    "error": "invalid_token",
    "error_description": "Token was not recognised"
}
```

#### 3.4.3 获取refresh_token

![copy.png](https://ae03.alicdn.com/kf/U94c5421026e6428c9147b40fbbbc267dM.jpg)

访问地址： http://localhost:6001/oauth/token?username=2&password=123456&grant_type=refresh_token&scope=all&client_id=user-client&client_secret=user-secret-8888&refresh_token=323a3662-c997-4af0-b5d9-ea1a7f76fc84 

grant_type: refresh_token

refresh_token: 从获取token里面取出

#### 3.4.2 客户端携带token访问接口

![test.png](https://ae02.alicdn.com/kf/U0c1fd4714c5f466dac05dde3560020b7E.jpg)

 http://localhost:6101/client-user/get 

返回结果： “2” （登录username）

token到了过期时间，再次访问，返回结果

```
{
    "error": "invalid_token",
    "error_description": "f7520be0-fb2c-4386-9ffc-e64977314b2f"
}
```

#### 

### 3.5 优化方案

#### 3.5.1 认证服务OAuth2Config的configure(ClientDetailsServiceConfigurer clients) 换成数据库存储

```
 @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //添加客户端信息
        //使用内存存储OAuth客服端信息
        clients.inMemory()
                // client_id 客户单ID
                .withClient("order-client")
                // client_secret 客户单秘钥
                .secret(passwordEncoder.encode("order-secret-8888"))
                // 该客户端允许的授权类型，不同的类型，则获取token的方式不一样
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                // token 有效期
                .accessTokenValiditySeconds(3600)
                // 允许的授权范围
                .scopes("all")
                .and()
                .withClient("user-client")
                .secret(passwordEncoder.encode("user-secret-8888"))
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                .accessTokenValiditySeconds(3600)
                .scopes("all");
    }
```

把OAuth2Config.java文件的configure(ClientDetailsServiceConfigurer clients)替换成下面的

```

    @Autowired
    private DataSource dataSource;


    /**
     * jdbc配置
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        JdbcClientDetailsServiceBuilder jcsb = clients.jdbc(dataSource);
        jcsb.passwordEncoder(passwordEncoder);
    }

```

在application.yml添加数据库连接

```
    #数据库连接
  datasource:
    url: jdbc:mysql://localhost:3306/springcloud?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: 123456
```

1. 在数据库中增加表，并插入数据

```mysql
create table oauth_client_details (
    client_id VARCHAR(256) PRIMARY KEY,
    resource_ids VARCHAR(256),
    client_secret VARCHAR(256),
    scope VARCHAR(256),
    authorized_grant_types VARCHAR(256),
    web_server_redirect_uri VARCHAR(256),
    authorities VARCHAR(256),
    access_token_validity INTEGER,
    refresh_token_validity INTEGER,
    additional_information VARCHAR(4096),
    autoapprove VARCHAR(256)
);
INSERT INTO oauth_client_details
    (client_id, client_secret, scope, authorized_grant_types,
    web_server_redirect_uri, authorities, access_token_validity,
    refresh_token_validity, additional_information, autoapprove)
VALUES
    ('user-client', '$2a$10$o2l5kA7z.Caekp72h5kU7uqdTDrlamLq.57M1F6ulJln9tRtOJufq', 'all',
    'authorization_code,refresh_token,password', null, null, 3600, 36000, null, true);

INSERT INTO oauth_client_details
    (client_id, client_secret, scope, authorized_grant_types,
    web_server_redirect_uri, authorities, access_token_validity,
    refresh_token_validity, additional_information, autoapprove)
VALUES
    ('order-client', '$2a$10$GoIOhjqFKVyrabUNcie8d.ADX.qZSxpYbO6YK4L2gsNzlCIxEUDlW', 'all',
    'authorization_code,refresh_token,password', null, null, 3600, 36000, null, true);
```

*注意：* client_secret 字段不能直接是 secret 的原始值，需要经过加密。因为是用的 `BCryptPasswordEncoder`，所以最终插入的值应该是经过 `BCryptPasswordEncoder.encode()`之后的值。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SecurityServerSystemApplication.class)
public class OAuth2PasswordTest {
    @Autowired
    public PasswordEncoder passwordEncoder;

    @Test
    public  void passwordEncode() {
        //secret
        System.out.println(passwordEncoder.encode("user-secret-8888"));
    }
}
```

### 3.6 JWT替换 redisToke

上面 token 的存储用的是 redis 的方案，Spring Security OAuth2 还提供了 jdbc 和 jwt 的支持，jdbc 的暂不考虑，现在来介绍用 JWT 的方式来实现 token 的存储。

用 JWT 的方式就不用把 token 再存储到服务端了，JWT 有自己特殊的加密方式，可以有效的防止数据被篡改，只要不把用户密码等关键信息放到 JWT 里就可以保证安全性。

#### 3.6.1  认证服务端改造

##### 3.6.1.1  添加 JwtConfig 配置类

```java
@Configuration
public class JwtTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();
        accessTokenConverter.setSigningKey("dev");
        return accessTokenConverter;
    }
}
```

 `JwtAccessTokenConverter`是为了做 JWT 数据转换，这样做是因为 JWT 有自身独特的数据格式。如果没有了解过 JWT ，可以搜索一下先了解一下。 

3.6.1.2  更改 OAuthConfig 配置类

```
@Autowired
private TokenStore jwtTokenStore;

@Autowired
private JwtAccessTokenConverter jwtAccessTokenConverter;

@Override
public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        /**
         * 普通 jwt 模式
         */
         endpoints.tokenStore(jwtTokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .userDetailsService(kiteUserDetailsService)
                /**
                 * 支持 password 模式
                 */
                .authenticationManager(authenticationManager);
}
```

 注入 JWT 相关的 Bean，然后修改 `configure(final AuthorizationServerEndpointsConfigurer endpoints)` 方法为 JWT 存储模式。 

#### 3.6.2  改造用户客户端

##### 3.6.2.1  修改 application.yml 配置文件

```
security:
  oauth2:
    client:
      client-id: user-client
      client-secret: user-secret-8888
      user-authorization-uri: http://localhost:6001/oauth/authorize
      access-token-uri: http://localhost:6001/oauth/token
    resource:
      jwt:
        key-uri: http://localhost:6001/oauth/token_key
        key-value: dev
```

 注意认证服务端 `JwtAccessTokenConverter`设置的 SigningKey 要和配置文件中的 key-value 相同，不然会导致无法正常解码 JWT ，导致验证不通过。 

##### 3.6.2.2  ResourceServerConfig 类的配置

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();

        accessTokenConverter.setSigningKey("dev");
        accessTokenConverter.setVerifierKey("dev");
        return accessTokenConverter;
    }

    @Autowired
    private TokenStore jwtTokenStore;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenStore(jwtTokenStore);
    }
}
```

#### 3.6.3 测试

跟上面一样（这里就不重复了）

#### 3.6.4  增强 JWT

 如果我想在 JWT 中加入额外的字段(比方说用户的其他信息)怎么办呢，当然可以。spring security oauth2 提供了 `TokenEnhancer` 增强器。其实不光 JWT ，RedisToken 的方式同样可以。 

##### 3.6.4.1 OAuthConfig 配置类修改

 **声明一个增强器**

```java
public class JWTokenEnhancer implements TokenEnhancer {

    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {
        Map<String, Object> info = new HashMap<>();
        info.put("jwt-ext", "JWT 扩展信息");
        ((DefaultOAuth2AccessToken) oAuth2AccessToken).setAdditionalInformation(info);
        return oAuth2AccessToken;
    }
}
```

 通过 oAuth2Authentication 可以拿到用户名等信息，通过这些我们可以在这里查询数据库或者缓存获取更多的信息，而这些信息都可以作为 JWT 扩展信息加入其中。 

 **在JwtTokenConfig.java 注入增强器** TokenEnhancer

```
@Configuration
public class JwtTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();
        accessTokenConverter.setSigningKey("dev");
        return accessTokenConverter;
    }

    @Bean
    public TokenEnhancer jwtTokenEnhancer() {
        return new JWTokenEnhancer();
    }

}
```

OAuthConfig.java 修改 `configure(final AuthorizationServerEndpointsConfigurer endpoints)`方法 

```
@Override
public void configure( final AuthorizationServerEndpointsConfigurer endpoints ) throws Exception{
	/**
	 * jwt 增强模式
	 */
	TokenEnhancerChain	enhancerChain	= new TokenEnhancerChain();
	List<TokenEnhancer>	enhancerList	= new ArrayList<>();
	enhancerList.add( jwtTokenEnhancer );
	enhancerList.add( jwtAccessTokenConverter );
	enhancerChain.setTokenEnhancers( enhancerList );
	endpoints.tokenStore( jwtTokenStore )
	.userDetailsService( kiteUserDetailsService )
	/**
	 * 支持 password 模式
	 */
	.authenticationManager( authenticationManager )
	.tokenEnhancer( enhancerChain )
	.accessTokenConverter( jwtAccessTokenConverter );
}
```

##### 3.6.4.2 测试

 **再次请求 token ，返回内容中多了个刚刚加入的 jwt-ext 字段** 

```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiJKV1Qg5omp5bGV5L-h5oGvIiwic2NvcGUiOlsiYWxsIl0sImV4cCI6MTU3MTc0NTE3OCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJhNDU1MWQ5ZS1iN2VkLTQ3NTktYjJmMS1mMGI5YjIxY2E0MmMiLCJjbGllbnRfaWQiOiJ1c2VyLWNsaWVudCJ9.5j4hNsVpktG2iKxNqR-q1rfcnhlyV3M6HUBx5cd6PiQ",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiJKV1Qg5omp5bGV5L-h5oGvIiwic2NvcGUiOlsiYWxsIl0sImF0aSI6ImE0NTUxZDllLWI3ZWQtNDc1OS1iMmYxLWYwYjliMjFjYTQyYyIsImV4cCI6MTU3MTc3NzU3OCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJmNTI3ODJlOS0wOGRjLTQ2NGUtYmJhYy03OTMwNzYwYmZiZjciLCJjbGllbnRfaWQiOiJ1c2VyLWNsaWVudCJ9.UQMf140CG8U0eWh08nGlctpIye9iJ7p2i6NYHkGAwhY",
  "expires_in": 3599,
  "scope": "all",
  "jwt-ext": "JWT 扩展信息",
  "jti": "a4551d9e-b7ed-4759-b2f1-f0b9b21ca42c"
}
```

#### 3.6.4  用户客户端解析 JWT 数据

 我们如果在 JWT 中加入了额外信息，这些信息我们可能会用到，而在接收到 JWT 格式的 token 之后，用户客户端要把 JWT 解析出来。 

###### 引入 JWT 包

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

###### 加一个 RESTful 接口，在其中解析 JWT

```java
@GetMapping(value = "jwt")
@PreAuthorize("hasAnyRole('ROLE_ADMIN')")
public Object jwtParser(Authentication authentication){
    authentication.getCredentials();
    OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails)authentication.getDetails();
    String jwtToken = details.getTokenValue();
    Claims claims = Jwts.parser()
                .setSigningKey("dev".getBytes(StandardCharsets.UTF_8))
                .parseClaimsJws(jwtToken)
                .getBody();
    return claims;
}
```

同样注意其中签名的设置要与认证服务端相同

**测试**

 **用上一步的 token 请求上面的接口** 

返回内容如下：

```json
{
  "user_name": "admin",
  "jwt-ext": "JWT 扩展信息",
  "scope": [
    "all"
  ],
  "exp": 1571745178,
  "authorities": [
    "ROLE_ADMIN"
  ],
  "jti": "a4551d9e-b7ed-4759-b2f1-f0b9b21ca42c",
  "client_id": "user-client"
}
```

# SpringCould整合spring-security+oauth2(亲测)

 **作者：星晴（当地小有名气，小到只有自己知道的杰伦粉）**



## 1.OAuth2 概念

- OAuth2 其实是一个关于授权的网络标准，它制定了设计思路和运行流程，利用这个标准我们其实是可以自己实现 OAuth2 的认证过程的。

  ![oauth2.png](https://ae03.alicdn.com/kf/U4451e76dbf5345e0b5d74f5d4c240dc69.jpg) 

OAuth 2 有四种授权模式:

- 授权码模式（authorization code)

- 简化模式（implicit)

- 密码模式（resource owner password credentials）

- 客户端模式（client credentials）

  具体 OAuth2 是什么，可以参考这篇文章。(http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 

## 2.什么情况下需要用 OAuth2

例子：

```
    首先大家最熟悉的就是几乎每个人都用过的，比如用微信登录、用 QQ 登录、用微博登录、用 Google 账号登录、用 github 授权登录等等，这些都是典型的 OAuth2 使用场景。假设我们做了一个自己的服务平台，如果不使用 OAuth2 登录方式，那么我们需要用户先完成注册，然后用注册号的账号密码或者用手机验证码登录。而使用了 OAuth2 之后，相信很多人使用过、甚至开发过公众号网页服务、小程序，当我们进入网页、小程序界面，第一次使用就无需注册，直接使用微信授权登录即可，大大提高了使用效率。因为每个人都有微信号，有了微信就可以马上使用第三方服务，这体验不要太好了。而对于我们的服务来说，我们也不需要存储用户的密码，只要存储认证平台返回的唯一ID 和用户信息即可。 
```

​       以上是使用了 OAuth2 的授权码模式，利用第三方的权威平台实现用户身份的认证。当然了，如果你的公司内部有很多个服务，可以专门提取出一个认证中心，这个认证中心就充当上面所说的权威认证平台的角色，所有的服务都要到这个认证中心做认证

这样一说，发现没，这其实就是个单点登录的功能。这就是另外一种使用场景，对于多服务的平台，可以使用 OAuth2 实现服务的单点登录，只做一次登录，就可以在多个服务中自由穿行，当然仅限于授权范围内的服务和接口。

## 3.具体使用

​     OAuth2 其实是一个关于授权的网络标准，它制定了设计思路和运行流程，利用这个标准我们其实是可以自己实现 OAuth2 的认证过程的。今天要介绍的 spring-cloud-starter-oauth2 ，其实是 Spring Cloud 按照 OAuth2 的标准并结合 spring-security 封装好的一个具体实现。 

### 3.1 系统架构说明

![OAuth2架构时序图.png](https://ae03.alicdn.com/kf/U4a9fe5c89c134b6797305b03415baa5fr.jpg)

-  认证服务：OAuth2 主要实现端，Token 的生成、刷新、验证都在认证中心完成。 
-  后台服务： 接收到请求后会到认证中心验证
-  前端：认证服务、后台服务之间的联调

上图描述了使用了 前端与OAuth2 认证服务、微服务间的请求过程。大致的过程就是前端用用户名和密码到后台服务登录，成功后后台服务到认证服务端换取 token，返回给前端，前端拿着 token 去各个微服务请求数据接口，一般这个 token 是放到 header 中的。当微服务接到请求后，先要拿着 token 去认证服务端检查 token 的合法性，如果合法，再根据用户所属的角色及具有的权限动态的返回数据 



### 3.2 创建并配置认证服务端

配置最多的就是认证服务端，验证账号、密码，存储 token，检查 token ,刷新 token 等都是认证服务端的工作。 

#### 3.2.1 引入需要的maven包

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

 之所以引入 redis 包，是因为下面会介绍一种用 redis 存储 token 的方式。 

#### 3.2.2 配置好 application.yml

```
spring:
  application:
    name: auth-server
  redis:
    database: 2
    host: localhost
    port: 6379

server:
  port: 6001

management:
  endpoint:
    health:
      enabled: true
```

#### 3.2.3 spring security 基础配置

```
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * 允许匿名访问所有接口 主要是 oauth 接口
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/**").permitAll();
    }
}
```

使用`@EnableWebSecurity`注解修饰，并继承自`WebSecurityConfigurerAdapter`类。

这个类的重点就是声明 `PasswordEncoder` 和 `AuthenticationManager`两个 Bean。稍后会用到。其中 `BCryptPasswordEncoder`是一个密码加密工具类，它可以实现不可逆的加密，`AuthenticationManager`是为了实现 OAuth2 的 password 模式必须要指定的授权管理 Bean。

#### 3.2.4  实现 UserDetailsService

如果你之前用过 Security 的话，那肯定对这个类很熟悉，它是实现用户身份验证的一种方式，也是最简单方便的一种。另外还有结合 `AuthenticationProvider`的方式，有机会讲 Security 的时候再展开来讲吧。

`UserDetailsService`的核心就是 `loadUserByUsername`方法，它要接收一个字符串参数，也就是传过来的用户名，返回一个 `UserDetails`对象。

```java
@Component(value = "kiteUserDetailsService")
public class KiteUserDetailsService implements UserDetailsService {


    @Autowired
    private UserRepository userRepository;

    /**
     * Security的登录，User赋予权限
     *
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if (StringUtils.isBlank(username)) {
            throw new UsernameNotFoundException("the username is not null");
        }
        
        //校验用户是否存在
        User user = userRepository.getById(username);
        if (null == user){
            throw new UsernameNotFoundException("the user is not exist");
        }

        //给用户添加角色权限
        String role = user.getRole();
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority(role));

        //返回用户token
        return new org.springframework.security.core.userdetails.User(username, user.getOauthpassword(), authorities);
    }

```

#### 3.2.5  OAuth2 配置文件

 创建一个配置文件继承自 `AuthorizationServerConfigurerAdapter` 

```java
@Configuration
@EnableAuthorizationServer
public class OAuth2Config extends AuthorizationServerConfigurerAdapter {

    /**
     * 指定密码的加密方式
     */
    @Autowired
    public PasswordEncoder passwordEncoder;

    /**
     * 该对象为刷新token提供支持
     */
    @Autowired
    public UserDetailsService kiteUserDetailsService;

    /**
     * 该对象用来支持password模式
     */
    @Autowired
    private AuthenticationManager authenticationManager;

    /**
     * 该对象用来讲令牌信息存储到内存中
     */
    @Autowired
    private TokenStore redisTokenStore;

    /**
     * 密码模式下配置认证管理器 AuthenticationManager,并且设置 AccessToken的存储介质tokenStore,如        果不设置，则会默认使用内存当做存储介质。
     * 而该AuthenticationManager将会注入 2个Bean对象用以检查(认证)
     * 1、ClientDetailsService的实现类 JdbcClientDetailsService (检查 ClientDetails 对象)
     * 2、UserDetailsService的实现类 KiteUserDetailsService (检查 UserDetails 对象)
     */
    @Override
    public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        /** redis token 方式*/
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(kiteUserDetailsService)
                .tokenStore(redisTokenStore);

    }

    /**
     * 配置 oauth_client_details【client_id和client_secret等】信息的认证【检查ClientDetails的合        法性】服务
     * 设置 认证信息的来源：数据库 (可选项：数据库和内存,使用内存一般用来作测试)
     * 自动注入：ClientDetailsService的实现类 JdbcClientDetailsService (检查 ClientDetails 对        象)
     * 1.inMemory 方式存储的，将配置保存到内存中，相当于硬编码了。正式环境下的做法是持久化到数据库中，比如        mysql 中。
     * 2. secret加密是client_id:secret 然后通过base64编码后的字符串
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //添加客户端信息
        //使用内存存储OAuth客服端信息
        clients.inMemory()
                // client_id 客户单ID
                .withClient("order-client")
                // client_secret 客户单秘钥
                .secret(passwordEncoder.encode("order-secret-8888"))
                // 该客户端允许的授权类型，不同的类型，则获取token的方式不一样
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                // token 有效期
                .accessTokenValiditySeconds(3600)
                // 允许的授权范围
                .scopes("all")
                .and()
                .withClient("user-client")
                .secret(passwordEncoder.encode("user-secret-8888"))
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                .accessTokenValiditySeconds(3600)
                .scopes("all");
    }

    /**
     * 配置：安全检查流程
     * 默认过滤器：BasicAuthenticationFilter
     * 1、oauth_client_details表中clientSecret字段加密【ClientDetails属性secret】
     * 2、CheckEndpoint类的接口 oauth/check_token 无需经过过滤器过滤，默认值：denyAll()
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        ///允许客户表单认证
        security.allowFormAuthenticationForClients();
        //对于CheckEndpoint控制器[框架自带的校验]的/oauth/check端点允许所有客户端发送器请求而不会被  Spring-security拦截
        security.checkTokenAccess("isAuthenticated()");
        security.tokenKeyAccess("isAuthenticated()");
    }
}
```

有三个 configure 方法的重写。

`AuthorizationServerEndpointsConfigurer`参数的重写

```java
endpoints.authenticationManager(authenticationManager)
                .userDetailsService(kiteUserDetailsService)
                .tokenStore(redisTokenStore);
```

`authenticationManage()` 调用此方法才能支持 password 模式。

`userDetailsService()` 设置用户验证服务。

`tokenStore()` 指定 token 的存储方式。

redisTokenStore Bean 的定义如下：

```
@Configuration
public class RedisTokenStoreConfig {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public TokenStore redisTokenStore (){
        return new RedisTokenStore(redisConnectionFactory);
    }
}
```

ClientId、Client-Secret：这两个参数对应请求端定义的 cleint-id 和 client-secret

authorizedGrantTypes 可以包括如下几种设置中的一种或多种：

- authorization_code：授权码类型。
- implicit：隐式授权类型。
- password：资源所有者（即用户）密码类型。
- client_credentials：客户端凭据（客户端ID以及Key）类型。
- refresh_token：通过以上授权获得的刷新令牌来获取新的令牌。

accessTokenValiditySeconds：token 的有效期

scopes：用来限制客户端访问的权限，在换取的 token 的时候会带上 scope 参数，只有在 scopes 定义内的，才可以正常换取 token。

上面代码中是使用 inMemory 方式存储的，将配置保存到内存中，相当于硬编码了。正式环境下的做法是持久化到数据库中，比如 mysql 中。(**优化认证服务有实例**)

#### 3.3.6 创建数据库SpringCloud、user表、实体User、UserRepository

**实体bean**

```java
@Entity
@Table(
        name = "user"
)
@Setter
@Getter
public class User implements Serializable {
    @Id
    @GeneratedValue(generator = "uuidGenerator")
    @GenericGenerator(name = "uuidGenerator", strategy = "uuid")
    @Column(name = "id", nullable = false)
    private String id;
    @Column(name = "username")
    private String username;
    @Column(name = "oauth_password")
    private String oauthpassword;
    @Column(name = "role")
    private String role;
}
```

**jpa接口**

```java
public interface UserRepository extends JpaRepository<User, String> {

    @Query("select r from User r where r.id = ?1 ")
    User getById(String username);

}
```

**user表**

```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `username` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `oauth_password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `role` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'admin', '$2a$10$D3PEtxvJ.N9Ko6osFaO4SO/jYcC8v7RHP34gZNk5THMvX7H5g8/NS', 'ROLE_ADMIN');
INSERT INTO `user` VALUES ('2', 'Custon', '$2a$10$D3PEtxvJ.N9Ko6osFaO4SO/jYcC8v7RHP34gZNk5THMvX7H5g8/NS', 'ROLE_ADMIN');

SET FOREIGN_KEY_CHECKS = 1;

```

#### 3.2.6 启动认证服务

 完成之后，启动项目，如果你用的是 IDEA 会在下方的 Mapping 窗口中看到 oauth2 相关的 RESTful 接口。 

![copy.png](https://ae02.alicdn.com/kf/U1770cdc935ac433c87a68cf651ed2e4eH.jpg)

主要有如下几个：

```
POST /oauth/authorize  授权码模式认证授权接口
GET/POST /oauth/token  获取 token 的接口
POST  /oauth/check_token  检查 token 合法性接口
```

### 3.3 创建用户客户端项目

 上面创建完成了认证服务端，下面开始创建一个客户端，对应到我们系统中的业务相关的微服务。我们假设这个微服务项目是管理用户相关数据的，所以叫做用户客户端。 

#### 3.3.1  引用相关的 maven 包

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 3.3.2  application.yml 配置文件

```java
spring:
  application:
    name: client-user
  redis:
    database: 2
    host: localhost
    port: 6379

server:
  port: 6101
  servlet:
    context-path: /client-user

security:
  oauth2:
    client:
      client-id: user-client
      client-secret: user-secret-8888
      user-authorization-uri: http://localhost:6001/oauth/authorize
      access-token-uri: http://localhost:6001/oauth/token
    resource:
      id: user-client
      user-info-uri: user-info
    authorization:
      check-token-access: http://localhost:6001/oauth/check_token
```

上面是常规配置信息以及 redis 配置，重点是下面的 security 的配置，这里的配置稍有不注意就会出现 401 或者其他问题。

client-id、client-secret 要和认证服务中的配置一致，如果是使用 inMemory 还是 jdbc 方式。

user-authorization-uri 是授权码认证方式需要的，下一篇文章再说。

access-token-uri 是密码模式需要用到的获取 token 的接口。

authorization.check-token-access 也是关键信息，当此服务端接收到来自客户端端的请求后，需要拿着请求中的 token 到认证服务端做 token 验证，就是请求的这个接口.

#### 3.3.3 资源配置文件

在 OAuth2 的概念里，所有的接口都被称为资源，接口的权限也就是资源的权限，所以 Spring Security OAuth2 中提供了关于资源的注解 `@EnableResourceServer`，和 `@EnableWebSecurity`的作用类似。

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String secret;

    @Value("${security.oauth2.authorization.check-token-access}")
    private String checkTokenEndpointUrl;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public TokenStore redisTokenStore (){
        return new RedisTokenStore(redisConnectionFactory);
    }

    @Bean
    public RemoteTokenServices tokenService() {
        RemoteTokenServices tokenService = new RemoteTokenServices();
        tokenService.setClientId(clientId);
        tokenService.setClientSecret(secret);
        tokenService.setCheckTokenEndpointUrl(checkTokenEndpointUrl);
        return tokenService;
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenServices(tokenService());
    }
}
```

 因为使用的是 redis 作为 token 的存储，所以需要特殊配置一下叫做 tokenService 的 Bean，通过这个 Bean 才能实现 token 的验证。 

#### 3.3.4  最后，添加一个 RESTful 接口

```
@Slf4j
@RestController
public class UserController {

    @GetMapping(value = "get")
    @PreAuthorize("hasAnyRole('ROLE_ADMIN')")
    public Object get(){
         Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
         
        return authentication.getName();
    }
}
```

一个 RESTful 方法，只有当访问用户具有 ROLE_ADMIN 权限时才能访问，否则返回 401 未授权。

通过 Authentication 参数或者 `SecurityContextHolder.getContext().getAuthentication()` 可以拿到授权信息进行查看。

### 3.4 测试

#### 3.4.1 获取token

```
http://localhost:6001/oauth/token?username=2&password=123456&grant_type=password&scope=all&client_id=user-client&client_secret=user-secret-8888
```

![copy.png](https://ae04.alicdn.com/kf/Uea945b8707be4c6280cb744ee491cea6J.jpg)



#### 3.4.3 校验token

![checktoken.png](https://ae02.alicdn.com/kf/U0101ecaf690d4376b4ab678d518d8d45q.jpg)

接口地址 http://localhost:6001/oauth/check_token?token=5f861834-9c6f-4424-af1d-df35fefddee3 

正常返回结果：

```
{
    "active": true,
    "exp": 1597915851,
    "user_name": "2",
    "authorities": [
        "ROLE_ADMIN"
    ],
    "client_id": "user-client",
    "scope": [
        "all"
    ]
}
```

校验失败结果：

```
{
    "error": "invalid_token",
    "error_description": "Token was not recognised"
}
```

#### 3.4.3 获取refresh_token

![copy.png](https://ae03.alicdn.com/kf/U94c5421026e6428c9147b40fbbbc267dM.jpg)

访问地址： http://localhost:6001/oauth/token?username=2&password=123456&grant_type=refresh_token&scope=all&client_id=user-client&client_secret=user-secret-8888&refresh_token=323a3662-c997-4af0-b5d9-ea1a7f76fc84 

grant_type: refresh_token

refresh_token: 从获取token里面取出

#### 3.4.2 客户端携带token访问接口

![test.png](https://ae02.alicdn.com/kf/U0c1fd4714c5f466dac05dde3560020b7E.jpg)

 http://localhost:6101/client-user/get 

返回结果： “2” （登录username）

token到了过期时间，再次访问，返回结果

```
{
    "error": "invalid_token",
    "error_description": "f7520be0-fb2c-4386-9ffc-e64977314b2f"
}
```

#### 

### 3.5 优化方案

#### 3.5.1 认证服务OAuth2Config的configure(ClientDetailsServiceConfigurer clients) 换成数据库存储

```
 @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //添加客户端信息
        //使用内存存储OAuth客服端信息
        clients.inMemory()
                // client_id 客户单ID
                .withClient("order-client")
                // client_secret 客户单秘钥
                .secret(passwordEncoder.encode("order-secret-8888"))
                // 该客户端允许的授权类型，不同的类型，则获取token的方式不一样
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                // token 有效期
                .accessTokenValiditySeconds(3600)
                // 允许的授权范围
                .scopes("all")
                .and()
                .withClient("user-client")
                .secret(passwordEncoder.encode("user-secret-8888"))
                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
                .accessTokenValiditySeconds(3600)
                .scopes("all");
    }
```

把OAuth2Config.java文件的configure(ClientDetailsServiceConfigurer clients)替换成下面的

```
    @Autowired
    private DataSource dataSource;


    /**
     * jdbc配置
     *
     * @param clients
     * @throws Exception
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        JdbcClientDetailsServiceBuilder jcsb = clients.jdbc(dataSource);
        jcsb.passwordEncoder(passwordEncoder);
    }

```

在application.yml添加数据库连接

```
    #数据库连接
  datasource:
    url: jdbc:mysql://localhost:3306/springcloud?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
    username: root
    password: 123456
```

1. 在数据库中增加表，并插入数据

```mysql
create table oauth_client_details (
    client_id VARCHAR(256) PRIMARY KEY,
    resource_ids VARCHAR(256),
    client_secret VARCHAR(256),
    scope VARCHAR(256),
    authorized_grant_types VARCHAR(256),
    web_server_redirect_uri VARCHAR(256),
    authorities VARCHAR(256),
    access_token_validity INTEGER,
    refresh_token_validity INTEGER,
    additional_information VARCHAR(4096),
    autoapprove VARCHAR(256)
);
INSERT INTO oauth_client_details
    (client_id, client_secret, scope, authorized_grant_types,
    web_server_redirect_uri, authorities, access_token_validity,
    refresh_token_validity, additional_information, autoapprove)
VALUES
    ('user-client', '$2a$10$o2l5kA7z.Caekp72h5kU7uqdTDrlamLq.57M1F6ulJln9tRtOJufq', 'all',
    'authorization_code,refresh_token,password', null, null, 3600, 36000, null, true);

INSERT INTO oauth_client_details
    (client_id, client_secret, scope, authorized_grant_types,
    web_server_redirect_uri, authorities, access_token_validity,
    refresh_token_validity, additional_information, autoapprove)
VALUES
    ('order-client', '$2a$10$GoIOhjqFKVyrabUNcie8d.ADX.qZSxpYbO6YK4L2gsNzlCIxEUDlW', 'all',
    'authorization_code,refresh_token,password', null, null, 3600, 36000, null, true);
```

*注意：* client_secret 字段不能直接是 secret 的原始值，需要经过加密。因为是用的 `BCryptPasswordEncoder`，所以最终插入的值应该是经过 `BCryptPasswordEncoder.encode()`之后的值。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SecurityServerSystemApplication.class)
public class OAuth2PasswordTest {
    @Autowired
    public PasswordEncoder passwordEncoder;

    @Test
    public  void passwordEncode() {
        //secret
        System.out.println(passwordEncoder.encode("user-secret-8888"));
    }
}
```

### 3.6 JWT替换 redisToke

上面 token 的存储用的是 redis 的方案，Spring Security OAuth2 还提供了 jdbc 和 jwt 的支持，jdbc 的暂不考虑，现在来介绍用 JWT 的方式来实现 token 的存储。

用 JWT 的方式就不用把 token 再存储到服务端了，JWT 有自己特殊的加密方式，可以有效的防止数据被篡改，只要不把用户密码等关键信息放到 JWT 里就可以保证安全性。

#### 3.6.1  认证服务端改造

##### 3.6.1.1  添加 JwtConfig 配置类

```java
@Configuration
public class JwtTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();
        accessTokenConverter.setSigningKey("dev");
        return accessTokenConverter;
    }
}
```

 `JwtAccessTokenConverter`是为了做 JWT 数据转换，这样做是因为 JWT 有自身独特的数据格式。如果没有了解过 JWT ，可以搜索一下先了解一下。 

3.6.1.2  更改 OAuthConfig 配置类

```
@Autowired
private TokenStore jwtTokenStore;

@Autowired
private JwtAccessTokenConverter jwtAccessTokenConverter;

@Override
public void configure(final AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        /**
         * 普通 jwt 模式
         */
         endpoints.tokenStore(jwtTokenStore)
                .accessTokenConverter(jwtAccessTokenConverter)
                .userDetailsService(kiteUserDetailsService)
                /**
                 * 支持 password 模式
                 */
                .authenticationManager(authenticationManager);
}
```

 注入 JWT 相关的 Bean，然后修改 `configure(final AuthorizationServerEndpointsConfigurer endpoints)` 方法为 JWT 存储模式。 

#### 3.6.2  改造用户客户端

##### 3.6.2.1  修改 application.yml 配置文件

```
security:
  oauth2:
    client:
      client-id: user-client
      client-secret: user-secret-8888
      user-authorization-uri: http://localhost:6001/oauth/authorize
      access-token-uri: http://localhost:6001/oauth/token
    resource:
      jwt:
        key-uri: http://localhost:6001/oauth/token_key
        key-value: dev
```

 注意认证服务端 `JwtAccessTokenConverter`设置的 SigningKey 要和配置文件中的 key-value 相同，不然会导致无法正常解码 JWT ，导致验证不通过。 

##### 3.6.2.2  ResourceServerConfig 类的配置

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();

        accessTokenConverter.setSigningKey("dev");
        accessTokenConverter.setVerifierKey("dev");
        return accessTokenConverter;
    }

    @Autowired
    private TokenStore jwtTokenStore;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.tokenStore(jwtTokenStore);
    }
}
```

#### 3.6.3 测试

跟上面一样（这里就不重复了）

#### 3.6.4  增强 JWT

 如果我想在 JWT 中加入额外的字段(比方说用户的其他信息)怎么办呢，当然可以。spring security oauth2 提供了 `TokenEnhancer` 增强器。其实不光 JWT ，RedisToken 的方式同样可以。 

##### 3.6.4.1 OAuthConfig 配置类修改

 **声明一个增强器**

```java
public class JWTokenEnhancer implements TokenEnhancer {

    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken oAuth2AccessToken, OAuth2Authentication oAuth2Authentication) {
        Map<String, Object> info = new HashMap<>();
        info.put("jwt-ext", "JWT 扩展信息");
        ((DefaultOAuth2AccessToken) oAuth2AccessToken).setAdditionalInformation(info);
        return oAuth2AccessToken;
    }
}
```

 通过 oAuth2Authentication 可以拿到用户名等信息，通过这些我们可以在这里查询数据库或者缓存获取更多的信息，而这些信息都可以作为 JWT 扩展信息加入其中。 

 **在JwtTokenConfig.java 注入增强器** TokenEnhancer

```
@Configuration
public class JwtTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter accessTokenConverter = new JwtAccessTokenConverter();
        accessTokenConverter.setSigningKey("dev");
        return accessTokenConverter;
    }

    @Bean
    public TokenEnhancer jwtTokenEnhancer() {
        return new JWTokenEnhancer();
    }

}
```

OAuthConfig.java 修改 `configure(final AuthorizationServerEndpointsConfigurer endpoints)`方法 

```
@Override
public void configure( final AuthorizationServerEndpointsConfigurer endpoints ) throws Exception{
	/**
	 * jwt 增强模式
	 */
	TokenEnhancerChain	enhancerChain	= new TokenEnhancerChain();
	List<TokenEnhancer>	enhancerList	= new ArrayList<>();
	enhancerList.add( jwtTokenEnhancer );
	enhancerList.add( jwtAccessTokenConverter );
	enhancerChain.setTokenEnhancers( enhancerList );
	endpoints.tokenStore( jwtTokenStore )
	.userDetailsService( kiteUserDetailsService )
	/**
	 * 支持 password 模式
	 */
	.authenticationManager( authenticationManager )
	.tokenEnhancer( enhancerChain )
	.accessTokenConverter( jwtAccessTokenConverter );
}
```

##### 3.6.4.2 测试

 **再次请求 token ，返回内容中多了个刚刚加入的 jwt-ext 字段** 

```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiJKV1Qg5omp5bGV5L-h5oGvIiwic2NvcGUiOlsiYWxsIl0sImV4cCI6MTU3MTc0NTE3OCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJhNDU1MWQ5ZS1iN2VkLTQ3NTktYjJmMS1mMGI5YjIxY2E0MmMiLCJjbGllbnRfaWQiOiJ1c2VyLWNsaWVudCJ9.5j4hNsVpktG2iKxNqR-q1rfcnhlyV3M6HUBx5cd6PiQ",
  "token_type": "bearer",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiJKV1Qg5omp5bGV5L-h5oGvIiwic2NvcGUiOlsiYWxsIl0sImF0aSI6ImE0NTUxZDllLWI3ZWQtNDc1OS1iMmYxLWYwYjliMjFjYTQyYyIsImV4cCI6MTU3MTc3NzU3OCwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiJmNTI3ODJlOS0wOGRjLTQ2NGUtYmJhYy03OTMwNzYwYmZiZjciLCJjbGllbnRfaWQiOiJ1c2VyLWNsaWVudCJ9.UQMf140CG8U0eWh08nGlctpIye9iJ7p2i6NYHkGAwhY",
  "expires_in": 3599,
  "scope": "all",
  "jwt-ext": "JWT 扩展信息",
  "jti": "a4551d9e-b7ed-4759-b2f1-f0b9b21ca42c"
}
```

#### 3.6.4  用户客户端解析 JWT 数据

 我们如果在 JWT 中加入了额外信息，这些信息我们可能会用到，而在接收到 JWT 格式的 token 之后，用户客户端要把 JWT 解析出来。 

###### 引入 JWT 包

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

###### 加一个 RESTful 接口，在其中解析 JWT

```java
@GetMapping(value = "jwt")
@PreAuthorize("hasAnyRole('ROLE_ADMIN')")
public Object jwtParser(Authentication authentication){
    authentication.getCredentials();
    OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails)authentication.getDetails();
    String jwtToken = details.getTokenValue();
    Claims claims = Jwts.parser()
                .setSigningKey("dev".getBytes(StandardCharsets.UTF_8))
                .parseClaimsJws(jwtToken)
                .getBody();
    return claims;
}
```

同样注意其中签名的设置要与认证服务端相同

**测试**

 **用上一步的 token 请求上面的接口** 

返回内容如下：

```json
{
  "user_name": "admin",
  "jwt-ext": "JWT 扩展信息",
  "scope": [
    "all"
  ],
  "exp": 1571745178,
  "authorities": [
    "ROLE_ADMIN"
  ],
  "jti": "a4551d9e-b7ed-4759-b2f1-f0b9b21ca42c",
  "client_id": "user-client"
}
```

关注公众号，有更多好玩的等着你！！！

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YicpKkSXicfO23aLicEHTNZibc8zxtW31NSibuCibDgOk3UhJBq90Z1ibXdotRAzibukOAiaicYmWNZFm6R3YzolcOdbdE9Q/640?wx_fmt=jpeg)  