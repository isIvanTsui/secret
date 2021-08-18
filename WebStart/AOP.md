### 1.添加依赖

```xml
<!-- aop -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

## 2. 编写拦截规则的注解

```java
@Target({ ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Action {

    String value() default "";
}
```

### 3.在控制器的方法上使用注解@Action

```java
@RestController
public class HelloController {

    @RequestMapping("/")
    @Action("hello") //连接点
    public String hello() {
        return "Hello Spring Boot";
    }
}
```

### 4.编写切面

```java
@Aspect
@Component
public class LogAspect {

    // pointCut
    @Pointcut("@annotation(com.ivan.aop.annotation.Action)")
    public void log() {

    }

    /**
     * 前置通知
     */
    @Before("log()")
    public void doBeforeController(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        Action action = method.getAnnotation(Action.class);
        System.out.println("action名称 " + action.value()); // ⑤
    }
    
    /**
     * 环绕通知
     */
    @Around("log()")
    public void aroundController(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("前置环绕通知");
            joinPoint.proceed(); //执行targetMethod
            System.out.println("后置环绕通知");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }

    /**
     * 后置通知
     */
    @AfterReturning(pointcut = "log()", returning = "retValue")
    public void doAfterController(JoinPoint joinPoint, Object retValue) {
        System.out.println("retValue is:" + retValue);
    }
}
```

### 5.测试

Web页面:

![aopWeb](https://raw.githubusercontent.com/isIvanTsui/img/master/aopWeb.png)

控制台:

![aopConsole](https://raw.githubusercontent.com/isIvanTsui/img/master/aopConsole.png)



### 另外一种配置：

```java
import org.springframework.stereotype.Component;

@Component("knight")
public class BraveKnight {
    public void saying(){
        System.out.println("我是骑士..（切点方法）");
    }
}
```



```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

/**
 * 注解方式声明aop
 * 1.用@Aspect注解将类声明为切面(如果用@Component("")注解注释为一个bean对象，那么就要在spring配置文件中开启注解扫描，<context:component-scan base-package="com.cjh.aop2"/>
 *      否则要在spring配置文件中声明一个bean对象)
 * 2.在切面需要实现相应方法的前面加上相应的注释，也就是通知类型。
 * 3.此处有环绕通知，环绕通知方法一定要有ProceedingJoinPoint类型的参数传入，然后执行对应的proceed()方法，环绕才能实现。
 */
@Component("annotationTest")
@Aspect
public class AnnotationTest {
    //定义切点
    @Pointcut("execution(* *.saying(..))")
    public void sayings(){}
    /**
     * 前置通知(注解中的sayings()方法，其实就是上面定义pointcut切点注解所修饰的方法名，那只是个代理对象,不需要写具体方法，
     * 相当于xml声明切面的id名，如下，相当于id="embark",用于供其他通知类型引用)
     * <aop:config>
        <aop:aspect ref="mistrel">
            <!-- 定义切点 -->
            <aop:pointcut expression="execution(* *.saying(..))" id="embark"/>
            <!-- 声明前置通知 (在切点方法被执行前调用) -->
            <aop:before method="beforSay" pointcut-ref="embark"/>
            <!-- 声明后置通知 (在切点方法被执行后调用) -->
            <aop:after method="afterSay" pointcut-ref="embark"/>
        </aop:aspect>
       </aop:config>
     */
    @Before("sayings()")
    public void sayHello(){
        System.out.println("注解类型前置通知");
    }
    //后置通知
    @After("sayings()")
    public void sayGoodbey(){
        System.out.println("注解类型后置通知");
    }
    //环绕通知。注意要有ProceedingJoinPoint参数传入。
    @Around("sayings()")
    public void sayAround(ProceedingJoinPoint pjp) throws Throwable{
        System.out.println("注解类型环绕通知..环绕前");
        pjp.proceed();//执行方法
        System.out.println("注解类型环绕通知..环绕后");
    }
}
```

