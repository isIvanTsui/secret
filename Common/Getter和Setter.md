<h1 align="center">Getter和Setter</h1>

## 后端返回值给前端时设置默认值

1. `Controller`层正常查询

   ```java
   @RestController
   public class IvanUserController {
       @Autowired
       private IvanUserServiceImpl userService;
   
       @GetMapping("getUsers")
       public List getUsers() {
           List<IvanUser> list = userService.list();
           List<IvanUserVo> resultList = BeanUtil.copyToList(list, IvanUserVo.class);
           return resultList;
       }
   }
   ```

   `Vo`里这么写

   ```
   @Data
   public class IvanUserVo {
       private String uidCode;
       @JsonIgnore
       private Integer uid;
       private String name;
       private Integer age;
       private Integer jid;
   
       public String getUidCode() {
           if (this.uid != null) {
               //加密
               return uid + "encrypt";
           }
           return uidCode;
       }
   }
   ```

   

   > 在`uid`属性上添加`@JsonIgnore`表示序列化时，忽略这个属性，也就是不传这个属性给前端。
   >
   > 在`Controller`层返回数据给前端时，会调用`getUidCode()`方法，我们在这个方法里，将`uid`加密后的值赋给`uidCode`属性并返回这个属性给前端，做到只传加密后的`id`给前端，而不是传明文`id`给前端。

2. 测试

   ![image-20211125164906895](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211125164906895.png)

-----



## 前端传值给后端时设置默认值

1. `Controller`层用`Vo`接收

   ```java
   @RestController
   public class IvanUserController {
       @Autowired
       private IvanUserServiceImpl userService;
   
       @GetMapping("getUsersByUidCode")
       public List getUsersByUidCode(@RequestBody IvanUserVo userVo) {
           List resultList = null;
           if (StrUtil.isNotBlank(userVo.getUidCode())) {
               List<IvanUser> list = userService.list(new LambdaQueryWrapper<IvanUser>().eq(IvanUser::getUid, userVo.getUid()));
               resultList = BeanUtil.copyToList(list, IvanUserVo.class);
           }
           return resultList;
       }
   }
   ```

   `Vo`里这么写：

   ```java
   @Data
   public class IvanUserVo {
       private String uidCode;
       @JsonIgnore
       private Integer uid;
       private String name;
       private Integer age;
       private Integer jid;
   
       public String getUidCode() {
           if (this.uid != null) {
               //加密
               return uid + "encrypt";
           }
           return uidCode;
       }
   
       public void setUidCode(String uidCode) {
           if (StrUtil.isNotBlank(uidCode)) {
               //解密
               this.uid = NumberUtil.parseInt(uidCode.split("encrypt")[0]);
           }
           this.uidCode = uidCode;
       }
   }
   ```

   > 作用：当前端传来`uidCode`属性时，`Controller`层接收时会调用`setUidCode()`方法，在`setUidCode()`方法里将`uidCode`解密，解出后的明文赋给`uid`

2. 测试

   `PostMan`发起请求:

   ![image-20211125165131806](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211125165131806.png)

   后端`Debug`查看是否赋上默认值：

   ![image-20211125165305898](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211125165305898.png)

   > 已经赋上默认值，该做法不仅限于赋默认值而已，可以在`setXXX()`方法里添加一些设置其他属性的逻辑

