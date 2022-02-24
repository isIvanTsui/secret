<h1 align="center">SpringBean声明周期</h1>

## 声明周期流程图

简单来说`SpringBean`的生命周期就`init-method`和`destroy-method`；但`SpringBean`的产生首先需要有`Spring`容器；所以`Spring Bean`的完整生命周期从创建`Spring`容器开始，直到最终`Spring`容器销毁`Bean`，这其中包含了一系列关键点。

![springbean](https://raw.githubusercontent.com/isIvanTsui/img/master/springbean.png)

> 主要接口：
>
> 1. `BeanNameAware`
> 2. `BeanFactoryAware`
> 3. ` ApplicationContextAware` 
> 4. `BeanPostProcessor`
> 5.  `InitializingBean`
> 6.  `DisposableBean`

-----



## Demo

`Person`实体类实现`BeanNameAware, BeanFactoryAware, ApplicationContextAware,BeanPostProcessor, InitializingBean, DisposableBean`接口

```java
package com.medicom.springbean;

import lombok.Data;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

import javax.annotation.PostConstruct;


/**
 * SpringBean生命周期测试类
 *
 * @author Ivan Tsui
 * @date 2022/02/24
 */
@Data
public class Person implements BeanNameAware, BeanFactoryAware, ApplicationContextAware,
        BeanPostProcessor, InitializingBean, DisposableBean {

    private String name = "Ivan";
    private String address = "成都市";
    private int phone = 1552545654;

    public Person() {
        System.out.println("【构造器】调用Person的构造器实例化");
    }

    public void setName(String name) {
        System.out.println("【注入属性】注入属性name");
        this.name = name;
    }

    public void setAddress(String address) {
        System.out.println("【注入属性】注入属性address");
        this.address = address;
    }

    public void setPhone(int phone) {
        System.out.println("【注入属性】注入属性phone");
        this.phone = phone;
    }

    /**
     * 1.这是BeanNameAware接口方法
     *
     * @param arg0 arg0
     */
    @Override
    public void setBeanName(String arg0) {
        System.out.println("1.【BeanNameAware接口】调用BeanNameAware.setBeanName()");
    }

    /**
     * 2.这是BeanFactoryAware接口方法
     *
     * @param arg0 arg0
     * @throws BeansException 豆子例外
     */
    @Override
    public void setBeanFactory(BeanFactory arg0) throws BeansException {
        System.out.println("2.【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("3.【ApplicationContextAware】调用setApplicationContext方法");
    }

    /**
     * 3.这是@PostConstruct注解修饰的方法
     */
    @PostConstruct
    public void postConstruct() {
        System.out.println("4.这是@PostConstruct修饰的方法");
    }

    /**
     * 3.这是InitializingBean接口方法
     *
     * @throws Exception 异常
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("5.【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
    }

    /**
     * 4.通过<bean>的init-method属性指定的初始化方法
     */
    public void myInit() {
        System.out.println("6.【init-method】调用<bean>的init-method属性指定的初始化方法");
    }

    /**
     * 5.目标方法
     *
     * @return {@link String}
     */
    @Override
    public String toString() {
        return "Person [name=" + name + ", address=" + address + ", phone=" + phone + "]";
    }


    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！");
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }

    /**
     * 9.这是DiposibleBean接口方法
     *
     * @throws Exception 异常
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("9.【DiposibleBean接口】调用DiposibleBean.destory()");
    }

    /**
     * 10.通过<bean>的destroy-method属性指定的初始化方法
     */
    public void myDestory() {
        System.out.println("10.【destroy-method】调用<bean>的destroy-method属性指定的销毁方法");
    }

}
```

将`Person`类配置成`SpringBean`

```java
package com.medicom.springbean;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class PersonConfig {
    @Bean(initMethod = "myInit", destroyMethod = "myDestory")
    public Person getPerson() {
        return new Person();
    }
}
```

建立单元测试：

```java
package com.medicom;

import com.medicom.springbean.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class SpringBeanTest {
    @Autowired
    private Person person;
    
    @Test
    public void test() {
        System.out.println(person);
    }
}
```

控制台上打印出的结果：

```
【构造器】调用Person的构造器实例化
1.【BeanNameAware接口】调用BeanNameAware.setBeanName()
2.【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
3.【ApplicationContextAware】调用setApplicationContext方法
4.这是@PostConstruct修饰的方法
5.【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
6.【init-method】调用<bean>的init-method属性指定的初始化方法
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
............
...........
...........
...........
...........
2022-02-24 16:46:40.871  INFO 12224 --- [           main] com.medicom.SpringBeanTest               : Started SpringBeanTest in 2.877 seconds (JVM running for 3.634)
7.BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
8.BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
Person [name=Ivan, address=成都市, phone=1552545654]
9.【DiposibleBean接口】调用DiposibleBean.destory()
10.【destroy-method】调用<bean>的destroy-method属性指定的销毁方法
```

> 第7步和第8步循环了很多次

### 换`xml`的方式来实例化`Bean`

添加`bean.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="person" class="com.medicom.springbean.Person" init-method="myInit"
          destroy-method="myDestory" scope="singleton" >
        <property name="name" value="张三"/>
        <property name="address" value="广州"/>
        <property name="phone" value="1539039133"/>
    </bean>
</beans>
```

编写测试类：

```java
package com.medicom.springbean;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class BeanLifeCycle {

    public static void main(String[] args) {
        System.out.println("现在开始初始化容器");
        ApplicationContext factory = new ClassPathXmlApplicationContext("classpath:/bean.xml");
        System.out.println("容器初始化成功");
        Person person = factory.getBean("person", Person.class);
        System.out.println(person);
        System.out.println("现在开始关闭容器！");
        ((ClassPathXmlApplicationContext) factory).registerShutdownHook();
    }
}
```

控制台打印出结果：

```
现在开始初始化容器
16:51:46.991 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@16f65612
16:51:47.106 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 1 bean definitions from class path resource [bean.xml]
16:51:47.128 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'person'
【构造器】调用Person的构造器实例化
【注入属性】注入属性name
【注入属性】注入属性address
【注入属性】注入属性phone
1.【BeanNameAware接口】调用BeanNameAware.setBeanName()
2.【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
3.【ApplicationContextAware】调用setApplicationContext方法
5.【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
6.【init-method】调用<bean>的init-method属性指定的初始化方法
容器初始化成功
Person [name=张三, address=广州, phone=1539039133]
现在开始关闭容器！
16:51:47.197 [SpringContextShutdownHook] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Closing org.springframework.context.support.ClassPathXmlApplicationContext@16f65612, started on Thu Feb 24 16:51:46 CST 2022
9.【DiposibleBean接口】调用DiposibleBean.destory()
10.【destroy-method】调用<bean>的destroy-method属性指定的销毁方法
```

