### 全局配置

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {

    private CorsConfiguration buildConfig() {

        CorsConfiguration corsConfiguration = new CorsConfiguration();
        // 1. 设置访问源地址
        corsConfiguration.addAllowedOrigin("*");
        // 2. 设置访问源请求头
        corsConfiguration.addAllowedHeader("*");
        // 3. 设置访问源请求方法
        corsConfiguration.addAllowedMethod("*");

        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter() {

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        // 4. 对接口配置跨域设置
        source.registerCorsConfiguration("/**", buildConfig());

        return new CorsFilter(source);
    }
}
```

### 另外一种全局配置

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
        .allowedOrigins("http://localhost:8081")
        .allowedHeaders("*")
        .allowedMethods("*");
    }
}
```

### 单个配置用@CrossOrigin注解

```java
@RestController
public class HelloController {
	@CrossOrigin(value = "http://localhost:8081")
	@GetMapping("/hello")    
	public String hello() {        
		return "hello";
	}
    
	@CrossOrigin(value = "http://localhost:8081")    
	@PostMapping("/hello")    
	public String hello2() {        
		return "post hello";    
	}
}
```

