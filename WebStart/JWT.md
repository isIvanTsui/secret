<h1 align="center">SpringBoot集成JWT</h1>

`SpringBoot`集成`JWT`做用户登录认证

## 引入依赖

```xml
		<dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.4.0</version>
        </dependency>
```



-----

## 登录用户类

````java
@Data
public class User {
    private Integer id;
    private String username;
    private String password;
}
````

-----



## 自定义2个注解

> 作用：标记哪些接口需要`token`，哪些接口不需要`token`

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NeedToken {
    boolean required() default true;
}
```

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface PassToken {
    boolean required() default true;
}
```

## 自定义拦截器

```java
public class AuthenticationInterceptor implements HandlerInterceptor {
    @Autowired
    UserService userService;
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object object) throws Exception {
        String token = httpServletRequest.getHeader("token");// 从 http 请求头中取出 token
        // 如果不是映射到方法直接通过
        if(!(object instanceof HandlerMethod)){
            return true;
        }
        HandlerMethod handlerMethod=(HandlerMethod)object;
        Method method=handlerMethod.getMethod();
        //检查是否有passtoken注释，有则跳过认证
        if (method.isAnnotationPresent(PassToken.class)) {
            PassToken passToken = method.getAnnotation(PassToken.class);
            if (passToken.required()) {
                return true;
            }
        }
        //检查有没有需要用户权限的注解
        if (method.isAnnotationPresent(NeedToken.class)) {
            NeedToken needToken = method.getAnnotation(NeedToken.class);
            if (needToken.required()) {
                // 执行认证
                if (token == null) {
                    throw new RuntimeException("无token，请重新登录");
                }
                // 获取 token 中的 user id
                String userId;
                try {
                    userId = JWT.decode(token).getAudience().get(0);
                } catch (JWTDecodeException j) {
                    throw new RuntimeException("401");
                }
                User user = userService.findUserById(Integer.valueOf(userId));
                if (user == null) {
                    throw new RuntimeException("用户不存在，请重新登录");
                }
                // 验证 token
                JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256(user.getPassword())).build();
                try {
                    jwtVerifier.verify(token);
                } catch (JWTVerificationException e) {
                    throw new RuntimeException("401");
                }
                return true;
            }
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

-----

## 添加自定义拦截器到`WebMVC`

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(getAuthenticationInterceptor()).addPathPatterns("/**");//拦截所有请求
    }

    @Bean
    public AuthenticationInterceptor getAuthenticationInterceptor() {
        return new AuthenticationInterceptor();
    }
}
```

## 编写接口来测试

```java
@Service
public class UserService {
    public User findUserById(Integer uid) {
        User user = new User();
        user.setId(uid);
        user.setUsername("Tom");
        user.setPassword("tt123");
        return user;
    }

    public User getUserByUsername(String username) {
        User user = new User();
        user.setId(1);
        user.setUsername("Tom");
        user.setPassword("tt123");
        return user;
    }
}
```

```java
@RestController
public class JwtController {
    @Autowired
    UserService userService;

    @GetMapping("user1")
    @NeedToken
    public String neetToken() {
        return "需要Token";
    }

    @GetMapping("user2")
    @PassToken
    public String passToken() {
        return "不需要Token";
    }

    @PostMapping("login")
    public Object login(String username, String password) {
        Map<String, Object> result = new HashMap<>();
        User user = userService.getUserByUsername(username);
        if (user == null) {
            result.put("message", "登录失败,用户不存在");
            return result;
        } else {
            if (!user.getPassword().equals(password)) {
                result.put("message", "登录失败,密码错误");
                return result;
            } else {
                String token = getToken(user);
                result.put("token", token);
                result.put("user", user);
                return result;
            }
        }
    }

    public String getToken(User user) {
        String token = "";
        token = JWT.create().withAudience(user.getId() + "")
                .sign(Algorithm.HMAC256(user.getPassword()));
        return token;
    }
}
```

测试：

1. 访问：http://localhost:8080/user2，不需要`token`，可以正常访问

   ![image-20210914173655216](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210914173655216.png)

2. 访问：http://localhost:8080/user1，需要`token`，控制台已经打印出错误信息

   ![image-20210914173827893](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210914173827893.png)

3. 登录：http://localhost:8080/login，登录成功，并返回了`token`

   ![image-20210914174012011](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210914174012011.png)

4. 携带`token`信息，再次访问http://localhost:8080/user1，可以通过验证，正常访问

   ![image-20210914174227970](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210914174227970.png)

