# Shiro简介

![](C:\Users\Administrator\Desktop\secret\img\shiro_sumarry.png)

**Authentication**：身份认证/登录，验证用户是不是拥有相应的身份,例如账号密码登陆；

**Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

**Session Manager**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

**Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

**Web Support**：Web支持，可以非常容易的集成到Web环境；

**Caching**：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

**Concurrency**：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

**Testing**：提供测试支持；

**Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

**Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

# Shiro架构

![](C:\Users\Administrator\Desktop\secret\img\shiro_structure.png)

**Subject**：主体，应用代码直接交互的对象是 Subject，也就是说 Shiro 的对外 API 核心就是 Subject , 代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器；

**Realm**：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。

![](C:\Users\Administrator\Desktop\secret\img\shiro_structure_in.png)

**Subject**：主体，可以看到主体可以是任何可以与应用交互的“用户”；

**SecurityManager**：相当于SpringMVC中的DispatcherServlet或者Struts2中的FilterDispatcher；是Shiro的心脏；所有具体的交互都通过SecurityManager进行控制；它管理着所有Subject、且负责进行认证和授权、及会话、缓存的管理。

**Authenticator**：认证器，负责主体认证的，这是一个扩展点，如果用户觉得Shiro默认的不好，可以自定义实现；其需要认证策略（Authentication Strategy），即什么情况下算用户认证通过了；

**Authrizer**：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

**Realm**：可以有1个或多个Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是JDBC实现，也可以是LDAP实现，或者内存实现等等；由用户提供；注意：Shiro不知道你的用户/权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的Realm；

**SessionManager**：如果写过Servlet就应该知道Session的概念，Session呢需要有人去管理它的生命周期，这个组件就是SessionManager；而Shiro并不仅仅可以用在Web环境，也可以用在如普通的JavaSE环境、EJB等环境；所有呢，Shiro就抽象了一个自己的Session来管理主体与应用之间交互的数据；这样的话，比如我们在Web环境用，刚开始是一台Web服务器；接着又上了台EJB服务器；这时想把两台服务器的会话数据放到一个地方，这个时候就可以实现自己的分布式会话（如把数据放到Memcached服务器）；

**SessionDAO**：DAO大家都用过，数据访问对象，用于会话的CRUD，比如我们想把Session保存到数据库，那么可以实现自己的SessionDAO，通过如JDBC写到数据库；比如想把Session放到Memcached中，可以实现自己的Memcached SessionDAO；另外SessionDAO中可以使用Cache进行缓存，以提高性能；

**CacheManager**：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能

**Cryptography**：密码模块，Shiro提高了一些常见的加密组件用于如密码加密/解密的。

# 过滤器

| anon              | org.apache.shiro.web.filter.authc.AnonymousFilter            |
| ----------------- | ------------------------------------------------------------ |
| authc             | org.apache.shiro.web.filter.authc.FormAuthenticationFilter   |
| authcBasic        | org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter |
| perms             | org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter |
| port              | org.apache.shiro.web.filter.authz.PortFilter                 |
| rest              | org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter |
| roles             | org.apache.shiro.web.filter.authz.RolesAuthorizationFilter   |
| ssl               | org.apache.shiro.web.filter.authz.SslFilter                  |
| user              | org.apache.shiro.web.filter.authc.UserFilter                 |
| logout            | org.apache.shiro.web.filter.authc.LogoutFilter               |
| noSessionCreation | org.apache.shiro.web.filter.session.NoSessionCreationFilter  |

解释：

1. `/admins/**=anon # 表示该 url 可以匿名访问`
2. `/admins/**=authc # 表示该 url 需要认证才能访问`
3. `/admins/**=authcBasic # 表示该 url 需要 httpBasic 认证`
4. `/admins/**=perms[user:add:*] # 表示该 url 需要认证用户拥有 user:add:* 权限才能访问`
5. `/admins/**=port[8081] # 表示该 url 需要使用 8081 端口`
6. `/admins/**=rest[user] # 相当于 /admins/**=perms[user:method]，其中，method 表示 get、post、delete 等`
7. `/admins/**=roles[admin] # 表示该 url 需要认证用户拥有 admin 角色才能访问`
8. `/admins/**=ssl # 表示该 url 需要使用 https 协议`
9. `/admins/**=user # 表示该 url 需要认证或通过记住我认证才能访问`
10. `/logout=logout # 表示注销,可以当作固定配置`

> **注意：**
>
> anon，authcBasic，auchc，user 是认证过滤器。
>
> perms，roles，ssl，rest，port 是授权过滤器。

# Shiro认证过程中可以用的异常

```java

Subject subject = SecurityUtils.getSubject();
if(!subject.isAuthenticated()){
  UsernamePasswordToken usernamePasswordToken = new UsernamePasswordToken(userName,password);
  usernamePasswordToken.setRememberMe(true);
  try{
     subject.login(usernamePasswordToken);
  }catch (UnknownAccountException e){
     System.out.print("账户不存在:"+e.getMessage());
  }catch (LockedAccountException e){
     System.out.print("账户被锁定:"+e.getMessage());
  }catch (IncorrectCredentialsException e){
     System.out.print("密码不匹配:"+e.getMessage());
  }

```

# 示例：

1. 引入依赖:

   ```xml
   <dependency>
               <groupId>org.apache.shiro</groupId>
               <artifactId>shiro-spring-boot-web-starter</artifactId>
               <version>1.7.1</version>
   </dependency>
   ```

2. 自定义Realm

   ```java
   package com.ivan.user.realm;
   
   import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
   import com.ivan.user.entity.User;
   import com.ivan.user.service.UserService;
   import org.apache.commons.collections.CollectionUtils;
   import org.apache.shiro.SecurityUtils;
   import org.apache.shiro.authc.*;
   import org.apache.shiro.authz.AuthorizationInfo;
   import org.apache.shiro.authz.SimpleAuthorizationInfo;
   import org.apache.shiro.realm.AuthorizingRealm;
   import org.apache.shiro.subject.PrincipalCollection;
   import org.apache.shiro.subject.Subject;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.thymeleaf.util.ObjectUtils;
   import org.thymeleaf.util.StringUtils;
   
   import java.util.Arrays;
   
   /**
    * @ClassName
    * @Description TODO
    * @Author IvanTsui
    * @Date 2021/2/9 13:24
    * @Version 1.0
    **/
   public class UserRealm extends AuthorizingRealm {
       @Autowired
       private UserService userService;
       /**
        * 授权
        * @param principalCollection
        * @return
        */
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
           System.out.println("授权了...");
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           Subject subject = SecurityUtils.getSubject();
           //获取当前用户
           User currentUser = (User) subject.getPrincipal();
           //设置当前用户的权限
           info.addStringPermission(currentUser.getPerms());
           return info;
       }
   
       /**
        * 认证
        * @param authenticationToken
        * @return
        * @throws AuthenticationException
        */
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
           System.out.println("认证了...");
           UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
           QueryWrapper<User> wrapper = new QueryWrapper<>();
           wrapper.eq("username", token.getUsername());
           //用户名、密码，数据库中取
           User user = userService.getOne(wrapper);
           if (user == null) { //没有这个人
               return null;
           }
           //可以加密
           //密码认证，Shiro做
           return new SimpleAuthenticationInfo(user,user.getPassword(),"");
       }
   }
   ```

3. 配置ShiroConfig

   ```java
   package com.ivan.user.config;
   
   import com.ivan.user.realm.UserRealm;
   import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
   import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   import java.util.LinkedHashMap;
   
   /**
    * @ClassName
    * @Description TODO
    * @Author IvanTsui
    * @Date 2021/2/9 13:22
    * @Version 1.0
    **/
   @Configuration
   public class ShiroConfig {
       /**
        * 3.ShiroDefualtFactoryBean
        * @param securityManager
        * @return
        */
       @Bean("shiroFilterFactoryBean")
       public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("defaultWebSecurityManager") DefaultWebSecurityManager securityManager) {
           ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
           factoryBean.setSecurityManager(securityManager);
           //拦截
           LinkedHashMap<String, String> fileterChain = new LinkedHashMap<>();
           //授权
           //需要权限的请求
           fileterChain.put("/user/add", "perms[user:add]");
           fileterChain.put("/user/update", "perms[user:update]");
           fileterChain.put("/user/*", "authc");
           //不需要权限的请求
               filterMap.put("/webjars/**", "anon");
           filterChain.put("/druid/**", "anon");
           filterChain.put("/sys/login", "anon");
           filterChain.put("/swagger/**", "anon");
           filterChain.put("/v2/api-docs", "anon");
           filterChain.put("/swagger-ui.html", "anon");
           filterChain.put("/swagger-resources/**", "anon");
          	//拦截所有请求
           filterChain.put("/**", "auth"); 
           factoryBean.setFilterChainDefinitionMap(fileterChain);
   //        设置未登录时跳转的页面
           factoryBean.setLoginUrl("/login");
           //未授权跳转页面
           factoryBean.setUnauthorizedUrl("/noauth");
           return factoryBean;
       }
   
       /**
        * 2.SecurityManager
        * @param userRealm
        * @return
        */
       @Bean("defaultWebSecurityManager")
       public DefaultWebSecurityManager getSecurityManager(@Qualifier("userRealm") UserRealm userRealm) {
           return new DefaultWebSecurityManager(userRealm);
       }
   
    /**
        * 1.UserRealm
        *
        * @return
        */
       @Bean("userRealm")
       public UserRealm getRealm() {
           return new UserRealm();
       }
   }
   ```
   
4. controller

   ```java
   @RestController("/test")
   public class TestController {
    
    
       @RequiresPermissions({"save"}) //没有的话 AuthorizationException
       @PostMapping("/save")
       public Map<String, Object> selectRole(String token) {
    
           System.out.println("save");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有save的权力");
           return map;
       }
    
       @RequiresPermissions({"delete"}) //没有的话 AuthorizationException
       @PostMapping("/delete")
       public Map<String, Object> managerRole(String token) {
           System.out.println("delete");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有delete的权力");
           return map;
       }
    
       @RequiresPermissions({"update"}) //没有的话 AuthorizationException
       @PostMapping("update")
       public Map<String, Object> selectUser(String token) {
           System.out.println("update");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有update的权力");
           return map;
       }
    
       @RequiresPermissions({"select"}) //没有的话 AuthorizationException
       @PostMapping("select")
       public Map<String, Object> managerUser(String token) {
           System.out.println("select");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有select的权力");
           return map;
       }
    
       @RequiresRoles({"vip"}) //没有的话 AuthorizationException
       @PostMapping("/vip")
       public Map<String, Object> vip(String token) {
           System.out.println("vip");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有VIP角色");
           return map;
       }
       @RequiresRoles({"svip"}) //没有的话 AuthorizationException
       @PostMapping("/svip")
       public Map<String, Object> SVIP(String token) {
           System.out.println("svip");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有SVIP角色");
           return map;
       }
       @RequiresRoles({"p"}) //没有的话 AuthorizationException
       @PostMapping("/p")
       public Map<String, Object> P(String token) {
           System.out.println("p");
           Map<String, Object> map = new HashMap<String, Object>();
           map.put("success", true);
           map.put("msg", "当前用户有P角色");
           return map;
       }
   }
   ```

   ### 完整pom
   
   ```xml
    <properties>
           <java.version>1.8</java.version>
           <shiro.version>1.7.1</shiro.version>
       </properties>
       <dependencies>
   <!--        thymeleaf-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-thymeleaf</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   <!--        shiro-->
           <dependency>
               <groupId>org.apache.shiro</groupId>
               <artifactId>shiro-spring-boot-web-starter</artifactId>
               <version>${shiro.version}</version>
           </dependency>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>8.0.18</version>
           </dependency>
           <!--        devtools-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <optional>true</optional>
               <scope>true</scope>
           </dependency>
   <!--        lombok-->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
           <dependency>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-boot-starter</artifactId>
               <version>3.4.2</version>
           </dependency>
           <!--        代码生成器-->
           <dependency>
               <groupId>com.baomidou</groupId>
               <artifactId>mybatis-plus-generator</artifactId>
               <version>3.4.1</version>
           </dependency>
           <!--        模板引擎-->
           <dependency>
               <groupId>org.apache.velocity</groupId>
               <artifactId>velocity-engine-core</artifactId>
               <version>2.2</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <version>RELEASE</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <excludes>
                           <exclude>
                               <groupId>org.projectlombok</groupId>
                               <artifactId>lombok</artifactId>
                           </exclude>
                       </excludes>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   ```
   
   

