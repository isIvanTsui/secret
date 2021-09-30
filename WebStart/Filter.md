<h1 align="center">自定义Filter</h1>

## 自定义`2`个`Filter`

> 自定义`Filter1`实现`Filter`接口并实现`init`、`doFilter`、`destory`方法，在类上添加`@WebFilter`注解

```java
import org.springframework.core.annotation.Order;
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(filterName = "filter1", urlPatterns = "*")
@Order(1) //执行顺序，数字越小越先执行
public class MyFilter1 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("MyFilter1 初始化");
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("MyFilter1 请求处理之前");
        //将请求传递给下一个过滤器
        filterChain.doFilter(request, response);
        System.out.println("MyFilter1 请求处理之后");
    }
    @Override
    public void destroy() {
        System.out.println("MyFilter1 销毁");
    }
}

```

```java
import org.springframework.core.annotation.Order;
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(filterName = "filter2", urlPatterns = "*")
@Order(2) //执行顺序，数字越小越先执行
public class MyFilter2 implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("MyFilter2 初始化");
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("MyFilter2 请求处理之前");
        //将请求传递给下一个过滤器
        filterChain.doFilter(request, response);
        System.out.println("MyFilter2 请求处理之后");
    }
    @Override
    public void destroy() {
        System.out.println("MyFilter2 销毁");
    }
}

```

-----

## 在启动类上加上`@ServletComponentScan`注解

```java
@SpringBootApplication
@ServletComponentScan
public class IvanApplication {
    public static void main(String[] args) {
        SpringApplication.run(IvanApplication.class, args);
    }
}
```

## 编写测试接口并测试

```java
@RestController
public class UserController {
    @GetMapping("test")
    public void test() {
        System.out.println("测试测试方法");
    }
}
```

启动项目时：

```
MyFilter1 初始化
MyFilter2 初始化
```

访问`localhost:8080/test`，控制台打印如下：

```
MyFilter1 请求处理之前
MyFilter2 请求处理之前
测试测试方法
MyFilter2 请求处理之后
MyFilter1 请求处理之后
```

关闭项目时，控制台打印如下：

```
MyFilter1 销毁
MyFilter2 销毁
```

## 总结

`Filter`执行顺序：

![filter](https://raw.githubusercontent.com/isIvanTsui/img/master/filter.png)