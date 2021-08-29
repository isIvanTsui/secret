<h1 align="center">Java8新特性</h1>

## 函数式接口

### 概念

函数式接口(Functional Interface)就是一个**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口。

> 注：只有1个抽象方法，但可有有多个default修饰的默认方法和多个静态方法

例：

```java
@FunctionalInterface
public interface TestInterface {
    // 抽象方法(只有1个)
    public void sub();
 
    // java.lang.Object中的方法不是抽象方法
    public boolean equals(Object var1);
 
    // default不是抽象方法
    public default void defaultMethod(){
 
    }
 
    // static不是抽象方法
    public static void staticMethod(){
 
    }
}
```

---

### 使用

1. 声明一个函数式接口

   ```java
   @FunctionalInterface
   public interface UserService {
       Integer getSum(Integer a, Integer b);
   }
   ```

2. 使用

   ```java
   public class Main {
       public static void main(String[] args) {
           UserService userService = (a, b) -> a + b;//这是其实是getSum（）方法的实现
           System.out.println(userService.getSum(1, 2));
       }
   }
   ```

   结果：

   3

---

### Java自带的函数式接口

- java.lang.Runnable
- java.util.concurrent.Callable
- java.security.PrivilegedAction
- java.util.Comparator
- java.io.FileFilter
- java.nio.file.PathMatcher
- java.lang.reflect.InvocationHandler
- java.beans.PropertyChangeListener
- java.awt.event.ActionListener
- javax.swing.event.ChangeListener
- java.util.function

平时用得比较多的函数式接口：

![image-20210819112918309](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20210819112918309.png)

| 序号 | 接口&描述                                          |
| :--: | :------------------------------------------------- |
|  1   | Consumer<T> 代表了接受一个输入参数并且无返回的操作 |
|  2   | Function<T,R> 接受一个输入参数，返回一个结果       |
|  3   | Predicate<T> 接受一个输入参数，返回一个布尔值结果  |
|  4   | Comparator<T> 接收2个输入参数，返回Integer类型     |
|  5   | Runnable 无参无返回                                |
|  6   | Callable<V> 无传入参数，返回V泛型                  |

---

## 方法引用

方法引用通过方法的名字来指向一个方法。

方法引用使用一对冒号 **::** 。

直接上实例：

```java
public class Animail {

    private String name;
    private Integer age;

    public Animail(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    public void show() {
        System.out.println(this.name + ":" + this.age);
    }
    public static void staMethod() {
        System.out.println("staMethod");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Animail dog = new Animail("Dog", 12);
        //特定对象的方法引用：它的语法是instance::method
        Runnable getAge = dog::show;
        new Thread(getAge).start();
        //构造器引用：它的语法是Class::new，或者更一般的Class< T >::new实例
        BiFunction<String, Integer, Animail> aNew = Animail::new;
        //静态方法引用：它的语法是Class::static_method
        Runnable staMethod = Animail::staMethod;
        List<Integer> list = Arrays.asList(1, 2, 3, 4);
    }
}
```

## Stream

### forEach

Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

```java
Random random = new Random(); random.ints().limit(10).forEach(System.out::println);
```

------

### map

map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); // 获取对应的平方数 
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```

------

### filter

filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); // 获取空字符串的数量 

long count = strings.stream().filter(string -> string.isEmpty()).count();
```

------

### limit

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

```java
Random random = new Random(); random.ints().limit(10).forEach(System.out::println);
```

------

### sorted

sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

```java
Random random = new Random(); random.ints().limit(10).sorted().forEach(System.out::println);
```

------

### 并行（parallel）程序

parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
// 获取空字符串的数量 
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```

我们可以很容易的在顺序运行和并行直接切换。

------

### Collectors

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl"); 
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList()); 
System.out.println("筛选列表: " + filtered); 
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", ")); System.out.println("合并字符串: " + mergedString);
```

------

### 统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5); 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics(); 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin()); 
System.out.println("所有数之和 : " + stats.getSum()); 
System.out.println("平均数 : " + stats.getAverage());
```

