<h1 align="center">JVM</h1>

## JVM总结构图

![jvmStructure](https://raw.githubusercontent.com/isIvanTsui/img/master/jvmStructure.png)

> 虚拟机栈、程序计数器、本地方法栈都是线程私有的，方法区、堆是线程共享的

## 类加载过程

**主要是五个阶段：**

<span style="color:red">装载 --> 链接(验证、准备、解析) --> 初始化 --> 使用 --> 卸载</span>

<img src="https://raw.githubusercontent.com/isIvanTsui/img/master/classProcess.png" alt="classProcess" style="zoom:150%;" />

1. **加载：**

   - **通过类的全限定名或者类的二进制字节流**，JVM并没有规定字节流一定要用某种方式，可以通过压缩包（jar、war包等）、从网络上获取、动态代理生成、其他文件（JSP）、数据库、加密文件（防反编译）等。
   - **将字节流所代表的静态存储结构转化为方法区的运行时数据结构**。
   - **在方法区中为这个类生成一个java.lang.Class对象，作为方法区这个类的访问入口。**

   > 总结这个过程就是加载二进制然后转换成JVM需要的结构，最后生成对应Class对象。

2. **链接:**

   2.1 **验证:**

   - **文件格式验证:**这一阶段主要验证字节流是否符合class文件格式规范，并且能被虚拟机处理，保证输入的字节流能被正确的解析并且存储在方法区

   - **元数据验证:**第二阶段主要是进行语法分析，以保证class文件符合Java语法规范，这个阶段的验证点包括：
         1、 这个类是否有父类(除java.lang.Object类之外都有父类)；
         2、 这个类的父类是否继承了不允许被继承的类(final修饰)；
         3、 如果这个类不是抽象类，是否实现了其父类或者接口中要求实现的所有方法；
         4、 类中的字段、方法是否跟父类中的字段、方法冲突；
         ......

   - **字节码验证:**这个阶段是语义分析，是验证过程中最复杂的一个阶段，主要目的是通过数据流和控制流，确定语义是合法并且符合逻辑的，这个阶段主要是针对方法体进行分析，以保证方法在运行过程中不会出现危害虚拟机的操作，验证点包括：
         1、 变量要在使用之前进行初始化；
         2、 方法调用与对象引用类型要匹配；
         3、 数据和方法的访问要符合权限设置规则；
         4、 对本地变量的访问都落在运行时本地方法栈内；
         5、 运行时堆栈没有溢出；
         ......

   - **符号引用验证**：校验行为发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在**解析阶段**中发生，验证是否有访问某些外部类、方法、字段的权限。

     > 整个过程可以理解为：整体结构、结构类型、结构内容、外部引用4个步骤。

   2.2 **准备:**

   **准备阶段是证实为类变量分配内存并且设置初始化值的阶段，这些变量所使用的内存都在方法区分配。**这个阶段进行初始化的数据只有静态字段，并且是赋值初始化值(final修饰的字段除外)，不是代码中定义的值。
       假设我们定义一个变量:

   ```
   public static int value = 123;
   ```
   在准备阶段，value在方法区分配内存，并且设置初始值0，如果value被final修饰，形如：

   ```
   public static final int value = 123;
   ```

   则该变量在准备阶段将会被赋值123，并且不会引起类的初始化过程，示例及说明见第二部分(类加载的时机)的示例。

   2.3 **解析:**

   **解析阶段是虚拟机将符号引用转化为直接引用的过程**，符号引用在之前已经介绍过了，在class文件中以形如"CONSTANT_Class_info"、"CONSTANT_Fieldref_info"、"CONSTANT_Methodref_info"格式存在

3. **初始化:**

   **执行\<clinit> 方法初始化类。**

   （1）类什么时候才被初始化 
   　　1）创建类的实例，也就是new一个对象 
   　　2）访问某个类或接口的静态变量，或者对该静态变量赋值 
   　　3）调用类的静态方法 
   　　4）反射（Class.forName(“com.lyj.load”)） 
   　　5）初始化一个类的子类（会首先初始化子类的父类） 
   　　6）JVM启动时标明的启动类，即文件名和类名相同的那个类 
   （2）类的初始化顺序 
   　　1）如果这个类还没有被加载和链接，那先进行加载和链接 
   　　2）假如这个类存在直接父类，并且这个类还没有被初始化（注意：在一个类加载器中，类只能初始化一次），那就初始化直接的父类（不适用于接口） 
   　　3）加入类中存在初始化语句（如static变量和static块），那就依次执行这些初始化语句。 

   　　4）总的来说，初始化顺序依次是：（静态变量、静态初始化块）–>（变量、初始化块）–> 构造器；

   如果有父类，则顺序是：父类static方法 –> 子类static方法 –> 父类构造方法- -> 子类构造方法 

## 类加载器

![classLoad](https://raw.githubusercontent.com/isIvanTsui/img/master/classLoad.png)

```java
public class Car {
    public static void main(String[] args) {
        //类是模板，对象是具体的。具体的对象可以拥有相同的类模板
        Car car1 = new Car();
        Car car2 = new Car();
        Car car3 = new Car();
        System.out.println(car1.hashCode());
        System.out.println(car2.hashCode());
        System.out.println(car3.hashCode());
        //获取类class模板
        Class<? extends Car> aClass1 = car1.getClass();
        Class<? extends Car> aClass2 = car2.getClass();
        Class<? extends Car> aClass3 = car3.getClass();
        System.out.println(aClass1.hashCode());
        System.out.println(aClass2.hashCode());
        System.out.println(aClass3.hashCode());
        /*结果: 三个实例对象的地址值不一样，但模板地址值一样
        621009875
        1265094477
        2125039532
        856419764
        856419764
        856419764*/
        ClassLoader classLoader = aClass1.getClassLoader();
        System.out.println(classLoader);// AppClassLoader
        System.out.println(classLoader.getParent()); // ExtClassLoader
        System.out.println(classLoader.getParent().getParent()); //null,Java程序抓不到。这是本地方法接口，由C或C++写的
    }
}
```

### 双亲委派机制（安全）

![parent_appointed](https://raw.githubusercontent.com/isIvanTsui/img/master/parent_appointed.jpg)

APP-->Ext-->BootStrap

![args](d:\Users\cuiyingfan\Desktop\secret\img\args.png)

## 程序计数器 PC Register

- 作用：记住下一条jvm指令的执行地址
- 特点：
  1. 是线程私有的
  2. 不会存在内存溢出

## 虚拟机栈 JVM Stacks

![stack](https://raw.githubusercontent.com/isIvanTsui/img/master/stack.jpg)

- 定义：线程运行时所需要的内存称为虚拟机栈
- 每个栈由多个栈帧组成，对应着每次方法调用时所占的内存
- 每个线程只能有一个活动栈帧，对应着正在执行的那个方法

栈:栈内存，主管程序的运行，生命周期和线程同步;线程结束，栈内存也就是释放，对于栈来说，不存在垃圾回收问题一旦线程结束，栈就Over!

<span style="color:red">**栈里存放的是:8大基本类型＋对象引用＋实例的方法**</span>

栈运行原理:栈帧

**栈溢出：**

1. 栈帧过多（递归）
2. 栈帧过大，一个栈帧比栈还大

**栈内存溢出实例：**

```java
public class Car {
    public static void main(String[] args) {
        new Car().a();
    }
    public void a() {
        b();
    }
    public void b() {
        a();
    }
}
```

### 对象的实例化

声明：

```java
class Student {
    String name = "Alice";//显示初始化
    int age = 18; //显示初始化
    public Student() {
        name = "Bob"; //构造方法初始化
        age = 24 ;  //构造方法初始化
    }
}
```

调用:

```java
Student s = new Student();
```

编译期间，会生成Student.class字节码文件。

初始化时：

1、类加载器ClassLoader，加载Student.class字节码到内存；

2、在栈里面为变量s申请一个空间，用来声明s；

3、new的时候，在堆内开辟空间。然后，开始进行默认初始化，String类型默认给null，int类型默认给0等。默认初始化后，开始进行显示初始化，比如成员变量里name默认值为Alice，所以这时会初始化name为Alice。

3、执行Student()构造方法，构造方法进栈，进行构造方法初始化

4、执行构造方法初始化，构造方法执行完毕后出栈，把堆内的对象物理地址，复制给栈内的s，也就是s存放的是对象的引用（物理地址）。

5、执行Student里面的方法时，方法进栈，方法里隐式的this指向堆内存空间s

## 本地方法栈 Native Method Stacks

* 作用：为本地方法运行提供内存空间

  ![native](https://raw.githubusercontent.com/isIvanTsui/img/master/native.png)

## 堆 Heap

![heap](https://raw.githubusercontent.com/isIvanTsui/img/master/heap.png)

* 通过new关键字，创建对象都会使用堆内存

* 特点
  1. 它是线程共享的，堆中的对象都要考虑线程安全问题
  2. 有垃圾回收机制

**不断的new对象就会造成堆内存溢出**

**对象的引用:**

- 句柄

  ![handle](https://raw.githubusercontent.com/isIvanTsui/img/master/handle.png)

- 直接引用

  ![direct](https://raw.githubusercontent.com/isIvanTsui/img/master/direct.png)

## 方法区 Method Area

方法区是被所有线程共享，所有字段和方法字节码，以及一些特殊方法，如构造函数，接口代码也在此定义,简单说，所有定义的方法的信息都保存在该区域，此区域属于共享区间;

<span style="color:red">静态变量(static)、常量(finale)、类信息(构造方法、接口定义)(Class模板)、运行时的常量池存在**方法区**中，但是实例变量存在堆内存中，和方法区无关</span>

* 方法区里存放着类的版本、字段、方法、接口和常量池（存储字面量和符号引用）

  符号引用包括：1、类的权限定名；2、字段名和属性；3、方法名和属性。

  ![methodAreaDetail](https://raw.githubusercontent.com/isIvanTsui/img/master/methodAreaDetail.png)

* 方法区在jdk1.6和1.7以后的区别

  ![methodArea](https://raw.githubusercontent.com/isIvanTsui/img/master/methodArea.png)

  >  Method Area都只是一种概念，Method Area在1.6以前是在jvm中，而1.7以后是在本地内存（操作系统内存）中。
>
  > 移除永久代的工作从JDK1.7就开始了。JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap。但永久代仍存在于JDK1.7中，并没完全移除，譬如符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。元空间本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此默认情况下元空间的大小仅受本地内存限制。



__StringTable特性:__

String s1 = "a";

String s2 = "b";

1. 常量池中的字符串仅是符号，第一次用到时才变为对象
2. 利用串池的机制，来避免重复创建字符串对象
3. 字符串变量拼接的原理是StringBuilder ( 1.8) s1 + s2
4. 字符串常量拼接的原理是编译期优化 "a" + "b"
5. 可以使用intern方法，主动将串池中还没有的字符串对象放入串池

```java
public class Main {
    public static void main(String[] args) {
        String s1 = new String("a") + new String("b"); //相当于new String("ab"), new出来的对象都放在堆中
        //此时串池中是["a", "b"]两个字符串
        //此时堆中是new String("a"),new String("b"),new String("ab")
        String s2 = s1.intern();//将这个字符串对象放入串池。如果串池有该字符串对象，则不放入且返回串池中的对象，如果没有，则放入并返回串池中的对象。并返回串池中的这个对象
        //此时串池中是["a", "b", "ab"]
        //刚才new出来的s1也被放入了串池
        System.out.println(s2 == "ab");//true
        System.out.println(s1 == "ab");//true
        System.out.println(s1 == s2);//true
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        String x = "ab";
        String s1 = new String("a") + new String("b");
        String s2 = s1.intern(); //不会将s1放入串池，因为串池中有“ab”字符串，直接返回串池中的“ab”就行
        System.out.println(x == s2); //true
        System.out.println(x == s1); //false
    }
}
```



StringBuilder是线程不安全的，但是它只是个**局部变量**，局部变量存储在**虚拟机栈**，**虚拟机栈**是线程隔离的，所以不会有线程安全问题



**重要例子:**

```java
		String test = "abc";
        String s1 = "a";
        String s2 = "b";
        String s3 = "c";
        System.out.println(test == "a" + "b" + "c"); //true
        System.out.println(test == s1 + s2 + s3); //false
//为什么出现上面的结果呢？这是因为，字符串字面量拼接操作是在Java编译器编译期间就执行了，也就是说编译器编译时，直接把"java"、"language"和"specification"这三个字面量进行"+"操作得到一个"javalanguagespecification" 常量，并且直接将这个常量放入字符串池中，这样做实际上是一种优化，将3个字面量合成一个，避免了创建多余的字符串对象。而字符串引用的"+"运算是在Java运行期间执行的，即str + str2 + str3在程序执行期间才会进行计算，它会在堆内存中重新创建一个拼接后的字符串对象。总结来说就是：字面量"+"拼接是在编译期间进行的，拼接后的字符串存放在字符串池中；而字符串引用的"+"拼接运算实在运行时进行的，新创建的字符串存放在堆中。
//		String str1 = new String("a") + new String("b");和 str1 = new String("ab")效果一样
```

## GC

### GC新生代对象晋升到老年代情况总结

(1)、Eden区满时，进行Minor GC，当Eden和一个Survivor区中依然存活的对象无法放入到Survivor中，则通过分配担保机制提前转移到老年代中。 

(2)、若对象体积太大, 新生代无法容纳这个对象，-XX:PretenureSizeThreshold即对象的大小大于此值, 就会绕过新生代, 直接在老年代分配, 此参数只对Serial及ParNew两款收集器有效。

(3)、长期存活的对象将进入老年代。

        虚拟机对每个对象定义了一个对象年龄（Age）计数器。当年龄增加到一定的临界值时，就会晋升到老年代中，该临界值由参数：-XX:MaxTenuringThreshold来设置。
    
        如果对象在Eden出生并在第一次发生MinorGC时仍然存活，并且能够被Survivor中所容纳的话，则该对象会被移动到Survivor中，并且设Age=1；以后每经历一次Minor GC，该对象还存活的话Age=Age+1。

(4)、动态对象年龄判定。

        虚拟机并不总是要求对象的年龄必须达到MaxTenuringThreshold才能晋升到老年代，如果在Survivor区中相同年龄（设年龄为age）的对象的所有大小之和超过Survivor空间的一半，年龄大于或等于该年龄（age）的对象就可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄。

### 查找算法

**引用计数算法:**每个对象添加到引用计数器，每被引用一次，计数器+1，失去引用，计数器-1，当计数器在一段时间内为0时，即认为该对象可以被回收了。(已废弃)

**根搜索算法:**从一个叫GC Roots的根节点出发，向下搜索，如果一个对象不能达到GC Roots的时候，说明该对象不再被引用，可以被回收。如上图中的Object5、Object6、Object7，虽然它们三个依然相互引用，但是它们其实已经没有作用了，这样就解决了引用计数算法的缺陷。

补充概念，在JDK1.2之后引入了四个概念：强引用、软引用、弱引用、虚引用。
       **强引用**：new出来的对象都是强引用，GC无论如何都不会回收，即使抛出OOM异常。
       **软引用**：只有当JVM内存不足时才会被回收。
       **弱引用**：只要GC,就会立马回收，不管内存是否充足。
       **虚引用**：可以忽略不计，JVM完全不会在乎虚引用，你可以理解为它是来凑数的，凑够”四大天王”。它唯一的作用就是做一些跟踪记录，辅助finalize函数的使用。

最后总结，什么样的类需要被回收：

> a.该类的所有实例都已经被回收；
>
>  b.加载该类的ClassLoad已经被回收；
>
>  c.该类对应的反射类java.lang.Class对象没有被任何地方引用。

### GC算法

**复制**：复制算法采用的方式为从根集合进行扫描，将存活的对象移动到一块空闲的区域，如图所示：

![copyAlgorithm](https://raw.githubusercontent.com/isIvanTsui/img/master/copyAlgorithm.png)

当存活的对象较少时，复制算法会比较高效（新生代的Eden区就是采用这种算法），其带来的成本是需要一块额外的空闲空间和对象的移动。

 **标记-清除**：该算法采用的方式是从跟集合开始扫描，对存活的对象进行标记，标记完毕后，再扫描整个空间中未被标记的对象，并进行清除。标记和清除的过程如下：

![markAlgorithm](https://raw.githubusercontent.com/isIvanTsui/img/master/markAlgorithm.png)

**标记-压缩**：该算法与标记-清除算法类似，都是先对存活的对象进行标记，但是在清除后会把活的对象向左端空闲空间移动，然后再更新其引用对象的指针，如下图所示：

![compressAlgorithm](https://raw.githubusercontent.com/isIvanTsui/img/master/compressAlgorithm.png)

