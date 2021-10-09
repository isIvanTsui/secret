<h1 align="center">用Redis实现Session共享</h1>

## 创建工程

首先 创建一个 Spring Boot 工程，引入 Web、Spring Session 以及 Redis:

![sessionRedis](https://raw.githubusercontent.com/isIvanTsui/img/master/sessionRedis.png)

**创建后依赖如下:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

## 配置Redis：**application.yml配置如下**

```yaml
redis:
  # redis数据库索引(默认为0)，我们使用索引为3的数据库，避免和其他数据库冲突
  database: 3
  # redis服务器地址（默认为loaclhost）
  host: localhost
  # redis端口（默认为6379）
  port: 6379
  # redis访问密码（默认为空）
  password: 
  # redis连接超时时间（单位毫秒）
  timeout: 0
  # redis连接池配置
  pool:
    # 最大可用连接数（默认为8，负数表示无限）
    max-active: 8
    # 最大空闲连接数（默认为8，负数表示无限）
    max-idle: 8
    # 最小空闲连接数（默认为0，该值只有为正数才有用）
    min-idle: 0
    # 从连接池中获取连接最大等待时间（默认为-1，单位为毫秒，负数表示无限）
    max-wait: -1
```

## 在启动类上添加`@EnableRedisHttpSession`注解

```java
@SpringBootApplication
@EnableRedisHttpSession
public class SpringSessionApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSessionApplication.class, args);
    }

}
```



## 编写`Controller`测试

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpSession;

/**
 * @ClassName
 * @Description TODO
 * @Author IvanTsui
 * @Date 2021/3/10 11:31
 * @Version 1.0
 **/
@RestController
public class HelloController {
    @GetMapping("set")
    public String set(HttpSession session) {
        session.setAttribute("user", "ivan");
        return "ok";
    }

    @GetMapping("get")
    public String get(HttpSession session) {
        return (String) session.getAttribute("user");
    }
}
```

测试：

访问http://localhost:8080/set

![image-20211009152357991](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211009152357991.png)

打开`RDM`查看`Redis`中的数据：

![image-20211009152637970](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211009152637970.png)

> 发现`Session`已经存入`Redis`了，默认存在`db0`

访问http://localhost:8080/get，发现已经能从`Redis`中获取`Session`了

![image-20211009152905832](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211009152905832.png)
