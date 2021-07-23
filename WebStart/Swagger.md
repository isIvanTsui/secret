# swagger配置及注解详解

## 1.加入依赖的jar包

```xml
 <!--引入swagger的依赖-->
     <dependency>
         <groupId>io.springfox</groupId>
         <artifactId>springfox-swagger2</artifactId>
         <version>2.9.2</version>
     </dependency>

     <dependency>
         <groupId>io.springfox</groupId>
         <artifactId>springfox-swagger-ui</artifactId>
     	 <version>2.9.2</version>
     </dependency>
```

## 2.配置swagger配置文件

```java
package com.ivan.user.config;

import io.swagger.annotations.ApiOperation;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @ClassName Swagger
 * @Description TODO
 * @Author IvanTsui
 * @Date 2021/2/19 13:16
 * @Version 1.0
 **/
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket swaggerSpringMvcPlugin() {
        return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
                // (第一种方式)扫描所有有注解的api，用这种方式更灵活
                .apis( RequestHandlerSelectors.withMethodAnnotation( ApiOperation.class ) )
                // (第二种方式)扫描指定包中的swagger注解
                //.apis(RequestHandlerSelectors.basePackage("com.ivan.user.controller"))
                // (第三种方式)扫描所有
                //.apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //标题
                .title("Swagger API Doc")
                //描述
                .description("SwaggerUI")
                //名片
                .contact(new Contact("IvanTsui", "", "is.ivan.tsui@gmail.com"))
                //版本
                .version("1.0")
                //所有者
                .license("Supermarket Manager")
                //构造
                .build();
    }
}

```

## 3.生成swagger文档

```java
@Api(description = "商户平台应用接口")
@RestController
public class MerchantController {

    @org.apache.dubbo.config.annotation.Reference
    private MerchantService merchantService;


    /**
     * 根据id获取商户信息
     * @param id
     * @return
     */
    @ApiOperation( "根据商户id获取商户信息" )
    @GetMapping("/merchants/{id}")
    public MerchantDTO queryMerchantById(@PathVariable("id") Long id){
        return merchantService.queryMerchantById( id );
    }

    @ApiOperation( value = "测试swagger其他常用注解",httpMethod = "POST")
    @PostMapping("/hello")
    @ApiImplicitParam(name = "id", value = "商户id", dataType = "Long", paramType ="query", required = true)
    public String hello(Long id){
        return "hello "+ merchantService.queryMerchantById( id ).getMerchantName();
    }

}

```

## 4.打开链接进行测试

http://localhost:8080/swagger-ui.html

## 5.swagger常用注解

##### @Api：修饰整个类,描述的是controller的作用，写在类的名称上面；

```java
@Api(value = "/merchant", description = "商户管理")
```

| 属性名称       | 备注                                             |
| -------------- | ------------------------------------------------ |
| value          | url的路径值                                      |
| tags           | 如果设置这个值、value的值会被覆盖                |
| description    | 对api资源的描述                                  |
| basePath       | 基本路径可以不配置                               |
| position       | 如果配置多个Api 想改变显示的顺序位置             |
| produces       | For example, “application/json, application/xml” |
| consumes       | For example, “application/json, application/xml” |
| protocols      | Possible values: http, https, ws, wss.           |
| authorizations | 高级特性认证时配置                               |

##### @ApiOperation：修饰类的一个方法，用来描述该方法的作用，写在方法上；

```java
 @ApiOperation(value = "商户新增", httpMethod = "POST")
```

| 属性名称          | 备注                                                         |
| ----------------- | ------------------------------------------------------------ |
| value             | url的路径值                                                  |
| tags              | 如果设置这个值、value的值会被覆盖                            |
| description       | 对api资源的描述                                              |
| basePath          | 基本路径可以不配置                                           |
| position          | 如果配置多个Api 想改变显示的顺序位置                         |
| produces          | For example, “application/json, application/xml”             |
| consumes          | For example, “application/json, application/xml”             |
| protocols         | Possible values: http, https, ws, wss.                       |
| authorizations    | 高级特性认证时配置                                           |
| hidden            | 配置为true 将在文档中隐藏                                    |
| response          | 返回的对象                                                   |
| responseContainer | 这些对象是有效的 “List”, “Set” or “Map”.，其他无效           |
| httpMethod        | “GET”, “HEAD”, “POST”, “PUT”, “DELETE”, “OPTIONS” and “PATCH” |
| code              | http的状态码 默认 200                                        |
| extensions        | 扩展属性                                                     |

##### @ApiParam:用对象来接收参数

```java
public ResponseEntity<Merchant> merchantAdd(@RequestBody @ApiParam(value = "Merchant", required = true)  Merchant merchant)
```

| 属性名称        | 备注         |
| --------------- | ------------ |
| name            | 属性名称     |
| value           | 属性值       |
| defaultValue    | 默认属性值   |
| allowableValues | 可以不配置   |
| required        | 是否属性必填 |
| access          | 不过多描述   |
| allowMultiple   | 默认为false  |
| hidden          | 隐藏该属性   |
| example         | 举例子       |

##### @ApiModelProperty：用对象接受参数时，用来描述对象中一个字段的作用

```java
@ApiModelProperty("名称")
private String name;
```

##### @ ApiResponses :HTTP响应整体的描述，响应集配置

```java
@ApiResponses({ @ApiResponse(code = 400, message = "无效得商户") })
```

##### @ ApiResponse :HTTP响应其中一个的描述， 响应集配置

```java
@ApiResponse(code = 400, message = "提供无效得用户")
```

| 属性名称          | 备注                             |
| ----------------- | -------------------------------- |
| code              | http的状态码                     |
| message           | 描述                             |
| response          | 默认响应类 Void                  |
| reference         | 参考ApiOperation中配置           |
| responseHeaders   | 参考 ResponseHeader 属性配置说明 |
| responseContainer | 参考ApiOperation中配置           |

##### @ApiImplicitParams：用在方法上包含一组参数说明；

```java
   @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "商户id", dataType = "Long", required = true, paramType = "query"),
            @ApiImplicitParam(name = "name", value = "商户名称", dataType = "string", paramType = "query", required = true)
    })
```

##### @ApiImplicitParam：用在方法上包含一个参数说明；

```java
 @ApiImplicitParam(name = "id", value = "商户id", dataType = "Long", required = true, paramType = "query")
```

| 属性名称     | 备注                     |
| ------------ | ------------------------ |
| paramType    | 参数放在哪个地方         |
| name         | 参数代表的含义           |
| value        | 参数名称                 |
| dataType     | 参数类型，有String/Long… |
| required     | 是否必填                 |
| defaultValue | 参数的默认值             |