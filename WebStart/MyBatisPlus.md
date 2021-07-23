## 引入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.18</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
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
<!--        代码生成器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.4.1</version>
        </dependency>
<!--        模板引擎（必须）-->
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.2</version>
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

## 编写自定义生成器

```java
@Test
    public void autoGenerate() {
        // Step1：代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // Step2：全局配置
        GlobalConfig gc = new GlobalConfig();
        // 填写代码生成的目录(需要修改)
        String projectPath = "D:\\WorkSpace\\MyBatisPlus";
        // 拼接出代码最终输出的目录
        gc.setOutputDir(projectPath + "/src/main/java");
        // 配置开发者信息（可选）（需要修改）
        gc.setAuthor("IvanTsui");
        // 配置是否打开目录，false 为不打开（可选）
        gc.setOpen(false);
        // 实体属性 Swagger2 注解，添加 Swagger 依赖，开启 Swagger2 模式（可选）
        //gc.setSwagger2(true);
        // 重新生成文件时是否覆盖，false 表示不覆盖（可选）
        gc.setFileOverride(false);
        // 配置主键生成策略，此处为 ASSIGN_ID（可选）
        gc.setIdType(IdType.ASSIGN_ID);
        // 配置日期类型，此处为 ONLY_DATE（可选）
        gc.setDateType(DateType.ONLY_DATE);
        // 默认生成的 service 会有 I 前缀
        gc.setServiceName("%sService");
        mpg.setGlobalConfig(gc);

        // Step3：数据源配置（需要修改）
        DataSourceConfig dsc = new DataSourceConfig();
        // 配置数据库 url 地址
        dsc.setUrl("jdbc:mysql://127.0.0.1:3306/warehouse?useUnicode=true&characterEncoding=utf8&useSSL=true&serverTimezone=UTC&useSSL=false");
        // dsc.setSchemaName("testMyBatisPlus"); // 可以直接在 url 中指定数据库名
        // 配置数据库驱动
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        // 配置数据库连接用户名
        dsc.setUsername("root");
        // 配置数据库连接密码
        dsc.setPassword("cui09876");
        mpg.setDataSource(dsc);

        // Step:4：包配置
        PackageConfig pc = new PackageConfig();
        // 配置父包名（需要修改）
        pc.setParent("com.ivan");
        // 配置模块名（需要修改）
        pc.setModuleName("user");
        // 配置 entity 包名
        pc.setEntity("entity");
        // 配置 mapper 包名
        pc.setMapper("mapper");
        // 配置 service 包名
        pc.setService("service");
        // 配置 controller 包名
        pc.setController("controller");
        mpg.setPackageInfo(pc);

        // Step5：策略配置（数据库表配置）
        StrategyConfig strategy = new StrategyConfig();
        // 指定表名（可以同时操作多个表，使用 , 隔开）（需要修改）
        strategy.setInclude("user");
        // 配置数据表与实体类名之间映射的策略
        strategy.setNaming(NamingStrategy.underline_to_camel);
        // 配置数据表的字段与实体类的属性名之间映射的策略
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        // 配置 lombok 模式
        strategy.setEntityLombokModel(true);
        // 配置 rest 风格的控制器（@RestController）
        strategy.setRestControllerStyle(true);
        // 配置驼峰转连字符
        strategy.setControllerMappingHyphenStyle(true);
        // 配置表前缀，生成实体时去除表前缀
        // 此处的表名为 test_mybatis_plus_user，模块名为 test_mybatis_plus，去除前缀后剩下为 user。
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);

        // Step6：执行代码生成操作
        mpg.execute();
    }
```

## 配置yaml

```yml
spring:
  devtools:
    livereload:
      enabled: true
    restart:
      enabled: true  #设置开启热部署
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/warehouse?useUnicode=true&characterEncoding=utf8&useSSL=true&serverTimezone=UTC&useSSL=false
    username: root
    password: cui09876
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #配置打印sql语句

```

## 自定义自动填充规则（可选）

1. 配置自定义内容

```java
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * 自动填充处理类，
 * insertFill() 表示插入时的填充规则，
 * updateFill() 表示更新时的填充规则。
 */
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    /**
     * 插入时的填充规则
     * @param metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject, "createTime", Date.class, new Date());
        this.strictInsertFill(metaObject, "updateTime", Date.class, new Date());
        this.strictInsertFill(metaObject, "deleteFlag", Integer.class, 0);
        this.strictInsertFill(metaObject, "version", Integer.class, 1);
    }

    /**
     * 更新时的填充规则
     * @param metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateTime", Date.class, new Date());
    }
}
```

2. 使用

   ```java
   import com.baomidou.mybatisplus.annotation.*;
   import java.util.Date;
   import java.io.Serializable;
   import io.swagger.annotations.ApiModel;
   import io.swagger.annotations.ApiModelProperty;
   import lombok.Data;
   import lombok.EqualsAndHashCode;
   import lombok.experimental.Accessors;
   
   /**
    * <p>
    * 用户表
    * </p>
    *
    * @author lyh
    * @since 2020-06-08
    */
   @Data
   @EqualsAndHashCode(callSuper = false)
   @Accessors(chain = true)
   @TableName("back_user")
   @ApiModel(value="User对象", description="用户表")
   public class User implements Serializable {
   
       private static final long serialVersionUID=1L;
   
       @ApiModelProperty(value = "用户 ID")
       @TableId(value = "id", type = IdType.ASSIGN_ID)
       private Long id;
   
       @ApiModelProperty(value = "用户名")
       private String name;
   
       @ApiModelProperty(value = "用户手机号")
       private String mobile;
   
       @ApiModelProperty(value = "用户密码")
       private String password;
   
       @TableField(fill = FieldFill.INSERT)
       @ApiModelProperty(value = "创建时间")
       private Date createTime;
   
       @TableField(fill = FieldFill.INSERT_UPDATE)
       @ApiModelProperty(value = "修改时间")
       private Date updateTime;
   
       @TableField(fill = FieldFill.INSERT)
       @ApiModelProperty(value = "逻辑删除标志，0 表示未删除， 1 表示删除")
       private Integer deleteFlag;
   
       @TableField(fill = FieldFill.INSERT)
       @ApiModelProperty(value = "版本号")
       private Integer version;
   
   }
   ```

3. 测试

   ```java
   import com.lyh.admin_template.back.common.utils.Result;
   import com.lyh.admin_template.back.entity.User;
   import com.lyh.admin_template.back.service.UserService;
   import io.swagger.annotations.Api;
   import io.swagger.annotations.ApiOperation;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   
   /**
    * 用于测试环境搭建各个功能是否成功
    */
   @RestController
   @RequestMapping("/test")
   @Api(tags = "测试页面")
   public class TestController {
   
       @Autowired
       private UserService userService;
   
       @ApiOperation(value = "获取所有用户信息")
       @GetMapping("/getListOfUser")
       public Result testGetListOfUser() {
           return Result.ok().data("user", userService.list());
       }
   
       @ApiOperation(value = "测试 MyBatis Plus 自动填充数据功能")
       @PostMapping("/testMyBatisPlus/AutoFill")
       public Result testMyBatisPlusAutoFill(@RequestBody User user) {
           if(userService.save(user)) {
               return Result.ok().message("数据添加成功");
           }
           return Result.error().message("数据添加失败");
       }
      
   }
   ```

   # 分页

   1. 配置分页config

      ```java
      import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
      import org.springframework.context.annotation.Bean;
      import org.springframework.context.annotation.Configuration;
      import org.springframework.transaction.annotation.EnableTransactionManagement;
      
      @EnableTransactionManagement
          @Configuration
          public class MybatisPlusConfig {
          
              /**
               * 分页插件
               */
              @Bean
              public PaginationInterceptor paginationInterceptor() {
                  return new PaginationInterceptor();
              }
          }
      ```

   2. 使用

      ```java
      @RestController
          @RequestMapping("/student")
          public class StudentController {
          
              @Autowired
              IStudentService studentService;
          
              @RequestMapping(value = "/findAll",method = RequestMethod.POST)
              public Object findAll(HttpServletRequest request){
                  //获取前台发送过来的数据
                  Integer pageNo = Integer.valueOf(request.getParameter("pageNo"));
                  Integer pageSize = Integer.valueOf(request.getParameter("pageSize"));
                  IPage<Student> page = new Page<>(pageNo, pageSize);
                  QueryWrapper<Student> wrapper = new QueryWrapper<>();
                  Student student = new Student();
                  student.setId(1);
                  wrapper.setEntity(student);
                  return studentService.page(page,wrapper);
              }
          
          }
      ```

      效果：

      ![](C:\Users\Administrator\Desktop\secret\img\splitPage.png)