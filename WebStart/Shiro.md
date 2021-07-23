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

```java
package com.ivan.user.realm;

import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.thymeleaf.util.StringUtils;

/**
 * @ClassName
 * @Description TODO
 * @Author IvanTsui
 * @Date 2021/2/9 13:24
 * @Version 1.0
 **/
public class UserRealm extends AuthorizingRealm {
    /**
     * 授权
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("授权了...");
        return null;
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
        //用户名、密码，数据库中取
        String username = "root";
        String password = "12345";
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        if (!StringUtils.equals(token.getUsername(), username)) {
            return null;
        }
        return new SimpleAuthenticationInfo("",password,"");
    }
}

```

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
        LinkedHashMap<String, String> fileterChain = new LinkedHashMap<>();
        fileterChain.put("/user/*", "authc");
        factoryBean.setFilterChainDefinitionMap(fileterChain);
//        设置未登录时跳转的页面
        factoryBean.setLoginUrl("/login");
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

