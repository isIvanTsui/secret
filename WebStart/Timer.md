# 定时器的四种实现方式

### 一：使用Timer创建简单的定时任务

> 多线程并行处理定时任务时，Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异常，其他任务便会自动终止运行，使用ScheduledExecutorService则没有这个问题

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Timer;
import java.util.TimerTask;

/**
 * @author cuiyingfan
 * @ClassName Main
 * @Description TODO
 * @date 2021/7/22 17:16
 * @Version 1.0
 */
public class Main {
    public static void main(String[] args) {
        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println("TimerTask1 run" + LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss")));
            }
        }, 1000, 5000);//延时1s，之后每隔5s运行一次
    }
}

```

### 二：使用ScheduledThreadPoolExecutor创建定时任务

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @author cuiyingfan
 * @ClassName Main
 * @Description TODO
 * @date 2021/7/22 17:16
 * @Version 1.0
 */
public class Main {
    public static void main(String[] args) {
        ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(5);
        executorService.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                String now = LocalDateTime.now().format(DateTimeFormatter.ofPattern("HH:mm:ss"));
                System.out.println("ScheduledThreadPoolExecutor1 run:"+now);
            }
        },1,2, TimeUnit.SECONDS);
    }
}
```

### 三：在springboot环境下使用@Scheduled创建定时任务

1. 启动类添加@EnableScheduling

   ```java
   import org.mybatis.spring.annotation.MapperScan;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.scheduling.annotation.EnableScheduling;
   
   @SpringBootApplication
   @MapperScan("com.ivan.*.mapper")
   @EnableScheduling
   public class IvanApplication {
       public static void main(String[] args) {
           SpringApplication.run(IvanApplication.class, args);
       }
   }
   ```

2. 定时任务方法上添加@Scheduled

   > <a href="https://cron.qqe2.com">在线cron表达式</a>

   ```java
   import org.springframework.scheduling.annotation.Scheduled;
   import org.springframework.stereotype.Component;
   import java.time.LocalDateTime;
   import java.time.format.DateTimeFormatter;
   
   /**
    * @ClassName TimerDemo
    * @Description TODO
    * @author cuiyingfan
    * @date 2021/7/30 9:59
    * @Version 1.0
    */
   @Component
   public class TimerDemo {
       @Scheduled(cron = "1/5 * * * * ?")
       public void show() {
           System.out.println("springScheduled run:" + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
       }
   }
   ```

   

### 四：使用Quartz框架

> 详见于Quartz文档

