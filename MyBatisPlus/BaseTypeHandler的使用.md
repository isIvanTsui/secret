<h1 align="center">BaseTypeHandler</h1>

# 概述

![image-20220114172158507](https://s2.loli.net/2022/01/14/vPJOidAVY9unWLC.png)

**用户`user`表里的`jid`字段存的是该用户对应的职位`id`，但该用户有多个职位，所以存了多个职位`id`且以`'`号分隔；现在想要查询出来的结果集就是下面这样的数据结构：**

```java
public class IvanUser {
    /**
     * 用户id
     */
    private Integer uid;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 职位id
     */
    private String jid;

	/**
     * 职位id集合
     */
    private List<String> jids;
}
```

> 即想得到`List`类型的职位`id`集合`jids`；
>
> 以往我们需要在往数据库中存或者在数据库取出来时转换类型或者对值做某些处理，这样十分不方便，在这里我们就可以继承`BaseTypeHandler<T>`，自己实现各种转换

-----



## 一、自定义`handler`继承`BaseTypeHandler<T>`



```java
@SuppressWarnings("unchecked")
public class ListTypeHandler extends BaseTypeHandler<List<String>> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, List<String> param, JdbcType jdbcType) throws SQLException {
        String str = String.join("", param);
        ps.setString(i, str);
    }


    @Override
    public List<String> getNullableResult(ResultSet rs, String columnName)
            throws SQLException {
        String result = rs.getString(columnName);
        return Arrays.asList(result.split(","));
    }

    @Override
    public List<String> getNullableResult(ResultSet rs, int columnIndex)
            throws SQLException {
        String result = rs.getString(columnIndex);
        return Arrays.asList(result.split(","));
    }


    @Override
    public List<String> getNullableResult(CallableStatement cs, int columnIndex)
            throws SQLException {
        String result = cs.getString(columnIndex);
        return Arrays.asList(result.split(","));
    }
}
```

## 二、绑定`handler`

1. 注解的形式绑定

   实体类上加注解`@TableName(value ="ivan_user", autoResultMap = true)`

   对应对象的属性也需要加注解`@TableField(value = "jid", typeHandler = ListTypeHandler.class)`

   ```java
   @TableName(value ="ivan_user", autoResultMap = true)
   @Data
   public class IvanUser implements Serializable {
       /**
        * 用户id
        */
       @TableId(type = IdType.AUTO)
       private Integer uid;
   
       /**
        * 用户名
        */
       private String name;
   
       /**
        * 年龄
        */
       private Integer age;
   
       /**
        * 职位id
        */
       private String jid;
   
       /**
        * 职位id集合
        */
       @TableField(value = "jid", typeHandler = ListTypeHandler.class)
       private List<String> jids;
   }
   ```

2. 写`XML`形式的绑定

   ```xml
   <resultMap id="resultList" type="com.ivan.user.entity.IvanUser">
       <id property="uid" column="uid"/>
       <result property="name" column="name"/>
       <result property="age" column="age"/>
       <result property="jid" column="jid"/>
       <result property="jids" column="jid" typeHandler="com.ivan.user.handler.ListTypeHandler"/>
   </resultMap>
   <select id="getALL" resultMap="resultList">
       SELECT
           uid,
           name,
           age,
           jid
       FROM
           ivan_user
   </select>
   ```

   ## 三、测试

   ```java
   @SpringBootTest
   class IvanUserMapperTest {
       @Resource
       private IvanUserMapper userMapper;
   
       @Test
       void getALL() {
           //这个是注解的形式
           List<IvanUser> ivanUsers = userMapper.selectList(null);
           //这个是自己写SQL，XML的形式
           List<IvanUser> list = userMapper.getALL();
           System.out.println(list);
       }
   }
   ```

   `debug`后发现已经绑定进来了：

   ![image-20220114173444778](https://s2.loli.net/2022/01/14/dDFtn7vpiXZP54j.png)