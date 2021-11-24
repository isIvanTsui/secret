<h1 align="center">Getter和Setter</h1>

## 前端传值给后端时设置默认值

1. 首先`Controller`层用`Vo`接收

   ```java
   @RestController
   public class AnimalController {
       @PostMapping("test")
       public void test(@RequestBody AnimalVo animalVo) {
           System.out.println(animalVo);
       }
   }
   ```

   `Vo`里这么写：

   ```java
   @Data
   public class AnimalVo {
       private Integer id;
   
       private String name;
   
       private Integer age;
   
       public void setName(String name) {
           if (StrUtil.isNotBlank(name)) {
               this.age = 18;
           }
           this.name = name;
       }
   }
   ```

   > 作用：当前端传来`name`属性时，`Controller`层接收时会调用`setName()`方法，在`setName()`方法里设置`name`属性时为`age`属性赋默认值

2. 测试

   `PostMan`发起请求:

   ![image-20211124160744113](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211124160744113.png)

   后端`Debug`查看是否赋上默认值：

   ![image-20211124160932210](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211124160932210.png)

   > 已经赋上默认值，该做法不仅限于赋默认值而已，可以在`setXXX()`方法里添加一些设置其他属性的逻辑

-----

## 后端返回值给前端时设置默认值

1. 修改刚才的`Controller`和`Vo`

   ```java
   @RestController
   public class AnimalController {
       @PostMapping("test")
       public AnimalVo test(@RequestBody AnimalVo animalVo) {
           animalVo.setId(1);
           return animalVo;
       }
   }
   ```

   ```java
   @Data
   public class AnimalVo {
       @JsonIgnore
       private Integer id;
   
       private String idCode;
       
       private String name;
   
       private Integer age;
   
       public void setName(String name) {
           if (StrUtil.isNotBlank(name)) {
               this.age = 18;
           }
           this.name = name;
       }
   
       public String getIdCode() {
           if (id != null) {
               //将加密后的id赋给idCode
               return id + "encode";
           }
           return idCode;
       }
   }
   ```

   > 在`id`属性上添加`@JsonIgnore`表示序列化时，忽略这个属性，也就是不传这个属性给前端。
   >
   > 在`Controller`层返回数据给前端时，会调用`getIdCode()`方法，我们在这个方法里，将`Id`加密后的值赋给`idCode`属性并返回这个属性给前端，做到只传加密后的`id`给前端，而不是传明文`id`给前端。

2. 测试

   ![image-20211124164606054](https://raw.githubusercontent.com/isIvanTsui/img/master/image-20211124164606054.png)
