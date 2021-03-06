<h1 align="center">Spring Config</h1>

## 准备Git仓库

Github上面新建一个用于存储Spring配置文件的仓库

![image-20210823160047513](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823160047513.png)

再在SpringCloudConfig仓库里创建文件夹user，文件夹user里创建**user-dev.yml**

```yaml
server:
  port: 10087
test: 
  hh: hahahahaha
```

具体如下：

![image-20210824115236011](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210824115236011.png)

> user-dev.yml为待会儿将要创建的config-client模块的配置文件

## 创建config-server

**项目整体结构：**

![image-20210823171956971](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823171956971.png)

1. 创建

   ![image-20210823163644394](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823163644394.png)

   ![image-20210823163715061](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823163715061.png)

2. 配置application.yml

   ```yaml
   spring:
     application:
       name: config-server
     cloud:
       config:
         server:
           git:
             uri: https://github.com/isIvanTsui/SpringCloudConfig.git  #配置文件所在仓库
             default-label: main  #主分支
             search-paths: /**  #搜寻（即配置文件将在SpringCloudConfig仓库的所有文件夹下去搜索）
             username:  #公开仓库不需要配置用户名密码
             password:
             basedir: D:\basedir
   server:
     port: 10086
   ```

   

3. 启动类上开启@EnableConfigServer注解

   ```java
   @SpringBootApplication
   @EnableConfigServer
   public class ConfigServerApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ConfigServerApplication.class, args);
       }
   
   }
   ```

   

4. 启动项目访问：http://localhost:10086/user-dev.yml

   ![image-20210823164330985](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823164330985.png)

   > confi-server已经从Git上拉取到了配置文件

## 创建config-client

1. 创建

   ![image-20210823164718469](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823164718469.png)

   ![image-20210823164752993](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823164752993.png)

   <span style="color:red">**还需要在pom.xml里添加一个bootstrap的依赖**</span>

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
   </dependency>
   ```

   > 原因：SpringCloud2020.x.x以上的版本默认禁止了bootstrap.yml，需要加入此依赖开启

2. 配置bootstrap.yml依赖

   > 不再是添加application.yml，而是添加bootstrap.yml，因为bootstrap.yml会在application.yml执行前执行

   ```yaml
   spring:
     application:
       name: config-client
     cloud:
       config:
         uri: http://localhost:10086/
         profile: dev # 环境（读取后缀）
         label: main # 分支名称
         name: user # 配置文件名称 列如user-dev.yml 名称就是user
   ```

   注意此时config-client的bootstrap配置文件中没有配置自身端口以及其他信息，而是配置了config-server所在ip，是因为我们刚才在config-server模块中已经能在GitHub远程仓库上获取到user-dev.yml配置文件信息了，此时我们只需要让config-client去config-server里拉取它获取到的user-dev.yml来作为config-client自己的其他配置

3. 写一个controller方便待会儿观察

   ```java
   @SpringBootApplication
   @RestController
   public class ConfigClientApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ConfigClientApplication.class, args);
       }
   
       @Value("${test.hh}")
       private String hh;
   
       @GetMapping("hh")
       public String hh() {
           return hh;
       }
   }
   ```

4. 启动项目

   启动项目，查看控制台

   ![image-20210824115851568](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210824115851568.png)

   > config-client去config-server里拉取到了配置文件信息，添加了port  10087配置，而config-server是从远端git仓库拉取配置信息，如图：

   ![a2021_8_24](https://raw.githubusercontent.com/isIvanTsui/img/master/a2021_8_24.png)

   测试一下刚才写的controller

   http://localhost:10087/hh
   
   ![image-20210823171125068](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210823171125068.png)
   
   

