<h1 align="center">IDEA热部署</h1>

![image-20210825163215072](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210825163215072.png)

![image-20210825163254857](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210825163254857.png)

![image-20210825163325692](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210825163325692.png)

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
```

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: cui09876
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf8
  devtools:
    restart:
      enabled: true  #设置开启热部署
      additional-paths: src/main/java #重启目录
      exclude: WEB-INF/**
    freemarker:
      cache: false    #页面不加载缓存，修改即时生效
```

