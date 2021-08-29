### **1、什么是 MyBatis？**

答：MyBatis 是一个可以自定义 SQL、存储过程和高级映射的持久层框架。

### **2、讲下 MyBatis 的缓存**

答：MyBatis 的缓存分为一级缓存和二级缓存,一级缓存放在 session 里面,默认就有,二级缓
存放在它的命名空间里,默认是不打开的,使用二级缓存属性类需要实现 Serializable 序列化
接口(可用来保存对象的状态),可在它的映射文件中配置<cache/>

### **3、Mybatis 是如何进行分页的？分页插件的原理是什么？**

答：
1）Mybatis 使用 RowBounds 对象进行分页，也可以直接编写 sql 实现分页，也可以使用
Mybatis 的分页插件。
2）分页插件的原理：实现 Mybatis 提供的接口，实现自定义插件，在插件的拦截方法内拦
截待执行的 sql，然后重写 sql。
举例：select from student，拦截 sql 后重写为：select t. from （select from student）t
limit 0，10

### **4、简述 Mybatis 的插件运行原理，以及如何编写一个插件？**

答：
1）Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、
Executor 这 4 种接口的插件，Mybatis 通过动态代理，为需要拦截的接口生成代理对象以实
现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是
InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。
2）实现 Mybatis 的 Interceptor 接口并复写 intercept()方法，然后在给插件编写注解，指定
要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

### **5、Mybatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？**

答：
1）Mybatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑
判断和动态拼接 sql 的功能。
2）Mybatis 提供了 9 种动态 sql 标签：
trim|where|set|foreach|if|choose|when|otherwise|bind。
3）其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼
接 sql，以此来完成动态 sql 的功能。

### **6、#{}和${}的区别是什么？**

答：
1）#{}是预编译处理，${}是字符串替换。
2）Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的 set 方法
来赋值；
3）Mybatis 在处理${}时，就是把${}替换成变量的值。
4）使用#{}可以有效的防止 SQL 注入，提高系统安全性。

### **7、为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？**

答：Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象
时，可以根据对象关系模型直接获取，所以它是全自动的。而 Mybatis 在查询关联对象或
关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

### **8、Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？**

答：
1）Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association
指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否
启用延迟加载 lazyLoadingEnabled=true|false。
2）它的原理是，使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方
法，比如调用 a.getB().getName()，拦截器 invoke()方法发现 a.getB()是 null 值，那么就会单
独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的
对象 b 属性就有值了，接着完成 a.getB().getName()方法的调用。这就是延迟加载的基本原
理。

### **9、MyBatis 与 Hibernate 有哪些不同？**

答：
1）Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己
编写 Sql 语句，不过 mybatis 可以通过 XML 或注解方式灵活配置要运行的 sql 语句，并将
java 对象和 sql 语句映射生成最终执行的 sql，最后将 sql 执行的结果再映射生成 java 对
象。
2）Mybatis 学习门槛低，简单易学，程序员直接编写原生态 sql，可严格控制 sql 执行性
能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运
营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的
前提是 mybatis 无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定
义多套 sql 映射文件，工作量大。
3）Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如
需求固定的定制化软件）如果用 hibernate 开发可以节省很多代码，提高效率。但是
Hibernate 的缺点是学习门槛高，要精通门槛更高，而且怎么设计 O/R 映射，在性能和对象
模型之间如何权衡，以及怎样用好 Hibernate 需要具有很强的经验和能力才行。
总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都
是好架构，所以框架只有适合才是最好。

### **10、MyBatis 的好处是什么？**

答：
1）MyBatis 把 sql 语句从 Java 源程序中独立出来，放在单独的 XML 文件中编写，给程序的
维护带来了很大便利。
2）MyBatis 封装了底层 JDBC API 的调用细节，并能自动将结果集转换成 Java Bean 对象，
大大简化了 Java 数据库编程的重复工作。
3）因为 MyBatis 需要程序员自己去编写 sql 语句，程序员可以结合数据库自身的特点灵活
控制 sql 语句，因此能够实现比 Hibernate 等全自动 orm 框架更高的查询效率，能够完成复
杂查询。

### **11、简述 Mybatis 的 Xml 映射文件和 Mybatis 内部数据结构之间的映射关系？**

答：Mybatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在
Xml 映射文件中，<parameterMap>标签会被解析为 ParameterMap 对象，其每个子元素会
被解析为 ParameterMapping 对象。<resultMap>标签会被解析为 ResultMap 对象，其每个子
元素会被解析为 ResultMapping 对象。每一个<select>、<insert>、<update>、<delete>标签
均会被解析为 MappedStatement 对象，标签内的 sql 会被解析为 BoundSql 对象。

### **12、什么是 MyBatis 的接口绑定,有什么好处？**

答：接口映射就是在 MyBatis 中任意定义接口,然后把接口里面的方法和 SQL 语句绑定,我们
直接调用接口方法就可以,这样比起原来了 SqlSession 提供的方法我们可以有更加灵活的选
择和设置.

### **13、接口绑定有几种实现方式,分别是怎么实现的?**

答：接口绑定有两种实现方式,一种是通过注解绑定,就是在接口的方法上面加上
@Select@Update 等注解里面包含 Sql 语句来绑定,另外一种就是通过 xml 里面写 SQL 来绑
定,在这种情况下,要指定 xml 映射文件里面的 namespace 必须为接口的全路径名.

### **14、什么情况下用注解绑定,什么情况下用 xml 绑定？**

答：当 Sql 语句比较简单时候,用注解绑定；当 SQL 语句比较复杂时候,用 xml 绑定,一般用
xml 绑定的比较多

### **15、MyBatis 实现一对一有几种方式?具体怎么操作的？**

答：有联合查询和嵌套查询,联合查询是几个表联合查询,只查询一次,通过在 resultMap 里面
配置 association 节点配置一对一的类就可以完成;嵌套查询是先查一个表,根据这个表里面
的结果的外键 id,去再另外一个表里面查询数据,也是通过 association 配置,但另外一个表的
查询通过 select 属性配置。*

### 16、**DBC编程有哪些不足之处，MyBatis是如何解决这些问题的？**

① 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。

解决：在SqlMapConfig.xml中配置数据链接池，使用连接池管理数据库链接。

② Sql语句写在代码中造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。

解决：将Sql语句配置在XXXXmapper.xml文件中与java代码分离。

③ 向sql语句传参数麻烦，因为sql语句的where条件不一定，可能多也可能少，占位符需要和参数一一对应。

解决： Mybatis自动将java对象映射至sql语句。

④ 对结果集解析麻烦，sql变化导致解析代码变化，且解析前需要遍历，如果能将数据库记录封装成pojo对象解析比较方便。

解决：Mybatis自动将sql执行结果映射至java对象。

### 17、**MyBatis编程步骤是什么样的？**

① 创建SqlSessionFactory

② 通过SqlSessionFactory创建SqlSession

③ 通过sqlsession执行数据库操作

④ 调用session.commit()提交事务

⑤ 调用session.close()关闭会话

### 18、**SqlMapConfig.xml中配置有哪些内容？**

SqlMapConfig.xml中配置的内容和顺序如下：

properties（属性）

settings（配置）

typeAliases（类型别名）

typeHandlers（类型处理器）

objectFactory（对象工厂）

plugins（插件）

environments（环境集合属性对象）

environment（环境子属性对象）

transactionManager（事务管理）

dataSource（数据源）

mappers（映射器）