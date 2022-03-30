<h1 align="center">SpringCache集成Ehcache</h1>

## 简介

`**SpringCache`默认是采用`CurrentHashMap`作为默认缓存框架；这里我们使用`Ehcache`去替换默认缓存。**

**Spring从3.1开始定义了org.springframework.cache.Cache 和 org.springframework.cache.CacheManager 接口来统一不同的缓存技术；**

- **Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；**
- **Cache接口下spring提供了各种xxxCache的实现，比如EhCacheCache、RedisCache等等**
- **每次调用需要缓存功能的方法时，Spring会检查指定参数的指定目标方法是否已经被调用过；如果有缓存就直接从缓存中获取结果，没有就调用方法并缓存结果后返回给用户。下次调用则直接从缓存中获取。**

## 引入依赖

```xml
<!--ehcache 缓存-->
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

## 添加`Ehcache`配置文件

**添加`ehcache.xml`配置文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <!-- 磁盘缓存位置 -->
    <diskStore path="\\CUIYINGFAN\share\ehcache"/>
    <!-- 默认缓存 -->
    <defaultCache
            maxEntriesLocalHeap="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxEntriesLocalDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
    <!-- cache 可以设置多个，例如localCache、UserCache和HelloWorldCache -->
    <cache name="localCache"
           eternal="true"
           maxElementsInMemory="100"
           maxElementsOnDisk="1000"
           overflowToDisk="true"
           diskPersistent="true"
           timeToIdleSeconds="0"
           timeToLiveSeconds="0"
           memoryStoreEvictionPolicy="LRU"/>

    <cache name="UserCache"
           maxElementsInMemory="1000"
           eternal="true"
           timeToIdleSeconds="10"
           timeToLiveSeconds="10"
           overflowToDisk="false"
           memoryStoreEvictionPolicy="LRU">
        <!--开启可搜索-->
        <searchable/>
    </cache>

    <!-- hello world缓存 -->
    <cache name="HelloWorldCache"
           maxElementsInMemory="1000"
           eternal="false"
           timeToIdleSeconds="5"
           timeToLiveSeconds="5"
           overflowToDisk="false"
           memoryStoreEvictionPolicy="LRU"/>
    <!-- memoryStoreEvictionPolicy Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）-->
    <!-- 缓存配置
     name:缓存名称。
     maxElementsInMemory：缓存最大个数。
     eternal:对象是否永久有效，一但设置了，timeout将不起作用。
     timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
     timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
     overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。 diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
     maxElementsOnDisk：硬盘最大缓存个数。
     diskPersistent：是否缓存虚拟机重启期数据 Whether the disk
     store persists between restarts of the Virtual Machine. The default value is false.
     diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
     memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
     clearOnFlush：内存数量最大时是否清除。
     -->
</ehcache>
```

> 我们这里只添加了3个缓存分组：`localCache`、`UserCache`、`HelloWorldCache`；可根据自己需要再添加

## 添加`application.yml`中的`ehcache`配置

**添加配置文件的路径进去：**

```yaml
spring:
  cache:
    ehcache:
      config: classpath:ehcache.xml
```

## 测试

```java
package com.ivan.cache.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.ArrayList;
import java.util.List;

@RestController
public class CacheController {
    @Autowired
    private CacheManager cacheManager;

    /**
     * 将list添加到缓存中（注解方式）
     *
     * @param id id
     * @return {@link List}
     */
    @GetMapping("cache")
    @Cacheable(cacheNames = "UserCache", key = "'user' + #id")
    public List getList(Integer id) {
        List<String> list = new ArrayList<>();
        list.add("Tom");
        list.add("Bob");
        list.add("Mary");
        list.add("Jim");
        list.add("Jerry");
        list.add("Angela");
        list.add("Kalun");
        return list;
    }

    /**
     * 获取缓存中的数据
     *
     * @return {@link List}
     */
    @GetMapping("test")
    public List getUser() {
        //获取第一个User
        List list = cacheManager.getCache("UserCache").get("user0", List.class);
        //获取SpringCache
        Cache cache = cacheManager.getCache("UserCache");
        //获取SpringCache集成的Ehcache
        Ehcache ehcache = (Ehcache) cache.getNativeCache();
        ehcache.put(new Element("user199", "哈哈哈哈"));
        Element value = ehcache.get("user199");
        System.out.println(value);
        //使用Ehcache进行模糊查询
        Attribute<String> key = ehcache.getSearchAttribute("key");
        Query query = ehcache.createQuery();
        query.includeKeys(); //查询结果中包含Key
        query.includeValues(); //查询结果中包含Value
        query.addCriteria(key.ilike("*users*"));
        Results results = query.execute();
        List<Result> resultList = results.all();
        return list;
    }

    /**
     * 清理缓存中的对应数据
     *
     * @param id id
     * @return {@link String}
     */
    @GetMapping("cache2")
    @CacheEvict(cacheNames = "UserCache", key = "'user' + #id")
    public String clean(Integer id) {
        return "清理缓存成功";
    }
}
```

## 启动类上添加`@EnableCaching`注解

```java
@SpringBootApplication
@EnableCaching
public class McdexNetApplication {
    public static void main(String[] args) {
        SpringApplication.run(McdexNetApplication.class, args);
    }
}
```

## `Spring`提供的注解

| Cache              | 缓存接口，定义缓存操作。实现有：EhCacheCache、RedisCache等等 |
| :----------------- | :----------------------------------------------------------- |
| **CacheManager**   | **缓存管理器，管理各种缓存组件**                             |
| **@Cacheable**     | **主要针对方法配置，能够根据方法的请求参数对其结果进行缓存** |
| **@CacheEvict**    | **清空缓存**                                                 |
| **@CachePut**      | **保证方法被调用，又希望结果被缓存**                         |
| **@EnableCaching** | **开启基于注解的缓存**                                       |

## SpringCache集成了多个第三方缓存

1. 自定义`CacheManager`接口实现的第三方缓存`Bean`

   ```java
   /**
    * ehcache配置
    *
    * @author cuiyingfan
    * @date 2022/03/15
    */
   @Configuration
   public class EhcacheConfig {
   
       @Bean("ehCacheCacheManager")
       public EhCacheCacheManager getEhCacheCacheManager() throws IOException {
           EhCacheCacheManager ehCacheCacheManager = new EhCacheCacheManager(CacheManager.create(new ClassPathResource("ehcache.xml").getInputStream()));
           return ehCacheCacheManager;
       }
   }
   ```

2. `@Cacheable`注解里配置`cacheManager`属性来切换不同的`CacheManage`即可切换不同的第三方缓存

   ```java
   @Cacheable(cacheNames = "CPR", cacheManager = "ehCacheCacheManager", key = "drugclas")
   public List<DrugClas> getDrugClas() {
       return drugClasService.list();
   }
   ```
   

## API的方式添加缓存

```java
@SpringBootTest
public class Main {
    @Resource
    private EhCacheCacheManager ehCacheCacheManager;

    @Test
    public void byApi() {
        CacheManager ehCachemanager = ehCacheCacheManager.getCacheManager();
        //添加缓存
        ehCachemanager.addCache("hhh");
        //获取缓存
        Cache cache = ehCachemanager.getCache("hhh");
        //修改缓存设置
        CacheConfiguration config = cache.getCacheConfiguration();
        config.setTimeToIdleSeconds(60);
        config.setTimeToLiveSeconds(120);
        System.out.println("");
    }
}
```



## 坑

<span style="color:red">**@Cacheable注解中：** </span>

<span style="color:red">**一个方法A调同一个类里的另一个有缓存注解的方法B，这样是不走缓存的。**</span>

<span style="color:red">**例如在同一个service里面两个方法的调用，缓存是不生效的；**</span>

> 这里涉及到的是SpringAOP，Spring的所有注解都是基于SpringAOP动态代理实现的，

解决方案：

1.不使用注解的方式，直接取 Ehcache 的 CacheManger 对象，把需要缓存的数据放到里面，类似于使用 Map，缓存的逻辑自己控制；或者可以使用redis的缓存方式去添加缓存；

2.把方法A和方法B放到两个不同的类里面，例如：如果两个方法都在同一个service接口里，把方法B放到另一个service里面，这样在A方法里调B方法，就可以使用B方法的缓存。

