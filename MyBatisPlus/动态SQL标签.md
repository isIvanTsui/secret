<h1 align="center">动态SQL标签</h1>

表：

```sql
CREATE TABLE `ivan_user` (
	`user_id` INT NOT NULL AUTO_INCREMENT,
	`user_name` VARCHAR ( 64 ) DEFAULT NULL,
	`user_sex` CHAR ( 8 ) DEFAULT NULL,
	`user_age` INT DEFAULT NULL,
	PRIMARY KEY ( `user_id` ) 
)
```

实体类：

```java
@TableName(value ="ivan_user")
@Data
public class User implements Serializable {
    
    @TableId(type = IdType.AUTO)
    private Integer userId;

    private String userName;

    private String userSex;

    private Integer userAge;

    @TableField(exist = false)
    private static final long serialVersionUID = 1L;
}
```



## sql

用于提前定义多出都需要用到的SQL语句

```xml
	<sql id="Base_Column_List">
        user_id
        ,user_name,user_sex,
        user_age
    </sql>
    <select id="getUser" resultType="com.ivan.user.entity.User">
        select
        <include refid="Base_Column_List"/>   -- 在要用到的地方用include标签引入进来
        from `ivan_user`
    </select>
```

## if

用于条件判断

```xml
<select id="getUser" resultType="com.ivan.user.entity.User">
        select
        * 
        from `ivan_user` u
        where 1 = 1
        <if test="user.user_name != null and user.user_name != ''">
            and u.user_name = #{user.uesrName}
        </if>
    </select>
```



## where

```xml
<select id="getUser" resultType="com.ivan.user.entity.User">
        select
        *
        from `ivan_user` u
        <where>
            <if test="user.userSex != null and user.userSex != ''">
                and u.user_sex = #{user.userSex}
            </if>
            <if test="user.userAge != null and user.userAge != ''">
                and u.user_age = #{user.userAge}
            </if>
        </where>
    </select>
```

> `where`标签一般与`if`标签同用。至少有一个`if`条件符合的时候，才会插入`where`语句并且会将第1个符合条件的`if`标签里的 and 去掉，上面的动态SQL执行后是这样的：

```
==>  Preparing: select * from `ivan_user` u WHERE u.user_sex = ? and u.user_age = ? 
==> Parameters: 男(String), 18(Integer)
<==    Columns: user_id, user_name, user_sex, user_age
<==        Row: 1, Tom, 男, 18
<==        Row: 4, Jim, 男, 18
<==      Total: 2
```

## trim

```xml
<select id="getUser" resultType="com.ivan.user.entity.User">
        select
        *
        from `ivan_user` u
        <trim prefix="where" suffixOverrides="and">
            <if test="user.userSex != null and user.userSex != ''">
                u.user_sex = #{user.userSex} and
            </if>
            <if test="user.userAge != null and user.userAge != ''">
                u.user_age = #{user.userAge} and
            </if>
        </trim>
    </select>
```

> 在第一个`if`标签前加`where`语句，在最后一个`if`标签后覆盖调`and`语句；执行后SQL如下：

````
==>  Preparing: select * from `ivan_user` u where u.user_sex = ? and u.user_age = ? 
==> Parameters: 男(String), 18(Integer)
<==    Columns: user_id, user_name, user_sex, user_age
<==        Row: 1, Tom, 男, 18
<==        Row: 4, Jim, 男, 18
<==      Total: 2
````

## set

动态前置`SET`关键词

```xml
<update id="updateUser" >
        update `ivan_user` u
        <set>
            <if test="user.userName != null and user.userName != ''">
                u.user_name = #{user.userName},
            </if>
            <if test="user.userAge != null and user.userAge != ''">
                u.user_age = #{user.userAge}
            </if>
        </set>
        <where>
            u.user_id = #{user.userId}
        </where>
    </update>
```

> 若`user.userName`有值，而`user.userAge`没有值，只会拼接第一个`if`标签里的SQL语句，且后面的`,`号会自动帮你去掉；完整SQL如下：

```
==>  Preparing: update `ivan_user` u SET u.user_name = ? WHERE u.user_id = ? 
==> Parameters: Tomm(String), 1(Integer)
<==    Updates: 1
```

## choose

有时我们不想应用到所有的条件语句，而只想从中择其一项。

```xml
<select id="getUser" resultType="com.ivan.user.entity.User">
        select
        *
        from `ivan_user` u
        <trim prefix="where" suffixOverrides="and">
            <choose>
                <when test="user.userSex != null and user.userSex != ''">
                    u.user_sex = #{user.userSex} and
                </when>
                <when test="user.userAge != null and user.userAge != ''">
                    u.user_age = #{user.userAge} and
                </when>
                <otherwise>
                    1 = 1 and
                </otherwise>
            </choose>
        </trim>
    </select>
```

> 执行后SQL语句：

```
==>  Preparing: select * from `ivan_user` u where u.user_sex = ? 
==> Parameters: 男(String)
<==    Columns: user_id, user_name, user_sex, user_age
<==        Row: 1, Tomm, 男, 18
<==        Row: 2, Bob, 男, 22
<==        Row: 4, Jim, 男, 18
<==        Row: 5, Dave, 男, 22
<==      Total: 4
```

## foreach

```xml
<select id="selectUsersByIds" resultType="com.ivan.user.entity.User">
        select * from `ivan_user` u
        <where>
            u.user_id in
            <foreach collection="ids" item="id" separator="," open="(" close=")">
                #{id}
            </foreach>
        </where>
    </select>
```

> 执行后完整SQL如下：

```
==>  Preparing: select * from `ivan_user` u WHERE u.user_id in ( ? , ? , ? , ? ) 
==> Parameters: 1(Integer), 2(Integer), 3(Integer), 4(Integer)
<==    Columns: user_id, user_name, user_sex, user_age
<==        Row: 1, Tomm, 男, 18
<==        Row: 2, Bob, 男, 22
<==        Row: 3, Mary, 女, 17
<==        Row: 4, Jim, 男, 18
<==      Total: 4
```



