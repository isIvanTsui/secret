<h1 align="cneter">Jackson注解大全</h1>

## 序列化注解

该注解用在`getter`方法上，作用是序列化时修改属性名

### `@JsonGetter`

`Java`对象：

```java
public class MyBean {
    public int id;
    private String name;
 
    @JsonGetter("myName")
    public String getTheName() {
        return name;
    }
}
```

序列化后的结果：

```json
{
	"id":1,
	"myName":"Bob"
}
```

-----



### `@JsonAnyGetter`

该注解用于把可变的`Map`类型属性当做标准属性

```java
class ExtendableBean {
    public String name;
    public Map<String, String> properties;
    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
    public ExtendableBean(String name) {
        this.name = name;
        properties = new HashMap<>();
    }
    public void add(String key, String value){
        properties.put(key, value);
    }
}
```

序列化后的结果: 

```json
{
    "name": "My bean",
    "attr2": "val2",
    "attr1": "val1"
}
```

-----



### `@JsonPropertyOrder`

该注解可以指定实体属性序列化后的顺序

```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```

序列化后结果：

```json
{
    "name": "My bean",
    "id": 1
}
```

> 该注解有一个参数`alphabetic`, 如果为`true`, 表示按字母顺序序列化,此时输出结果:`{ "id":1, "name":"My bean"}`

-----



### `@JsonRawValue`

该注解可以让Jackson在序列化时把属性的值原样输出
下面的例子中, 我们给实体属性`attrs`赋值一个`json`字符串

```java
public class RawBean {
    public String name;
    @JsonRawValue
    public String attrs;
}
public void whenSerializingUsingJsonRawValue_thenCorrect()
  throws JsonProcessingException {  
    RawBean bean = new RawBean("My bean", "{\"attr\":false}");
    String result = new ObjectMapper().writeValueAsString(bean);
}
```

输出结果是: `{"name":"Mybean","attrs":{"attr":false}}`

-----



### `@JsonValue`

可以用在`get`方法或者属性字段上，<span style="color:red">**一个类只能用一个**</span>，当加上`@JsonValue`注解时，序列化时只返回这一个字段的值``

-----



### `@JsonRootName`

暂时领悟不到这个注解是干嘛的

-----



### `@JsonSerialize`

该注解用于指定一个自定义序列化器(`custom serializer`)来序列化实体例的某属性
下例中, 用`@JsonSerialize`的参数`using`指明实体类属性`eventDate`的序列化器是`CustomDateSerializer`类:

```java
public class Event {
    public String name;
 
    @JsonSerialize(using = CustomDateSerializer.class)
    public Date eventDate;
}
```

下面是类`CustomDateSerializer`的定义:

```java
public class CustomDateSerializer extends StdSerializer<Date> {
    private static SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    public CustomDateSerializer() { this(null); } 
    public CustomDateSerializer(Class<Date> t) { super(t); }
    @Override
    public void serialize(Date value, JsonGenerator gen, SerializerProvider arg2) 
    	throws IOException, JsonProcessingException {
        gen.writeString(formatter.format(value));
    }
}
```

序列化:

```java
public void whenSerializingUsingJsonSerialize_thenCorrect(){
	SimpleDateFormat df = new SimpleDateFormat("yyyy/MM/DD hh:mm:ss");
	String toParse = "2019/08/19 16:28:00";
	Date date = null;
	try {
	    date = df.parse(toParse);
	} catch (ParseException e) {
	    e.printStackTrace();
	}
	Event event = new Event();
	event.name = "party";
	event.eventDate = date;
	try {
	    String result = new ObjectMapper().writeValueAsString(event);
	    System.out.println(result);
	} catch (JsonProcessingException e) {
	    e.printStackTrace();
	}
}
```

>序列化结果: `{"name":"party","eventDate":"2019-08-19 04:28:00"}`
>而如果没有`@JsonSerialize`注解的序列化结果是: `{"name":"party","eventDate":1566203280000}`

## 反序列化注解

### `@JsonSetter`

该注解可用于属性或方法上，作用是：反序列化时隐射属性名；与`@JsonProperty(value = "myName")`具有相同的效果；

该注解是`@JsonProperty`的另一个作用, 和`@JsonGetter`相对, 标记一个方法是`setter`方法

`Json`对象：

```json
{
	"id": 3,
	"myName": "Tom"
}
```

反序列化结果：

```
public class MyBean {
    public int id;
    @JsonSetter("myName")
    private String name;
 
    @JsonSetter("myName")
    public void setTheName(String name) {
        this.name = name;
    }
}
```

### `@JsonCreator`

该注解可以调整反序列化时`构造器/构造工厂`的行为
当我们需要反序列化的Json字符串和目标实体类不完全匹配时, 这个注解非常有用
假设我们要反序列化下面的Json字符串:

```json
{
    "id":1,
    "theName":"My bean"
}
```

但是, 我们的目标实体类并没有一个名为`theName`的属性. 现在, 我们不想改变实体类本身, 我们只需在数据导出时做一些控制, 方法就是在构造器中使用`@JsonCreator`和`@JsonProperty`注解:

```java
public class BeanWithCreator {
    public int id;
    public String name;
 
    @JsonCreator
    public BeanWithCreator(
      @JsonProperty("id") int id, 
      @JsonProperty("theName") String name) {
        this.id = id;
        this.name = name;
    }
}
```
### `@JacksonInject`

### `@JsonAnySetter`

该注解允许我们把一个可变的`map`属性作为标准属性, 在反序列过程中, 从Json字符串得到的属性值会加入到`map`属性中
实体例和注解:

`Json`对象：

```json
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

反序列化后的`Java`对象：

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;
 
    @JsonAnySetter
    public void add(String key, String value) {
        properties.put(key, value);
    }
}
```

### `@JsonDeserialize`

该注解标明使用自定义反序列化器(`custom deserializer`)

`Java`对象：

```java
public class Event {
    public String name;
 
    @JsonDeserialize(using = CustomDateDeserializer.class)
    public Date eventDate;
}
```

自定义反序列化器:

```java
public class CustomDateDeserializer  extends StdDeserializer<Date> { 
    private static SimpleDateFormat formatter = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
     public CustomDateDeserializer() { 
        this(null); 
    }  
    public CustomDateDeserializer(Class<?> vc) { 
        super(vc); 
    }
    @Override
    public Date deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException {         
        String date = jsonparser.getText();
        try {
            return formatter.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### `@JsonAlias`

该注解在反序列化过程中为属性定义一个或多个别名

```java
public class AliasBean {
    @JsonAlias({ "fName", "f_name" })
    private String firstName;   
    private String lastName;
}
```

`Json`字符串中`fName`, `f_name`或`firstName`的值都可以被反序列到属性`firstName`

-----



## 属性包含注解

### `@JsonIgnoreProperties`

该注解是一个**类级别**的注解, 标记一个或多个属性被Jackson**忽略**

```java
@JsonIgnoreProperties({ "id" })
public class BeanWithIgnore {
    public int id;
    public String name;
}
```

> 参数`ignoreUnknown`为`true`时, `Json`字符串如果有未知的属性名, 则不会抛出异常

### `@JsonIgnore`

该注解用于**属性级别**, 用于标明一个属性可以被Jackson**忽略**

```java
public class BeanWithIgnore {
    @JsonIgnore
    public int id;
 
    public String name;
}
```

### `@JsonIgnoreType`

该注解标记类型是注解作用的类型的属性都会被忽略
**必须作用于类, 标明以该类为类型的属性都会被Jackson忽略**

```java
public class User {
    public int id;
    public Name name;
 
    @JsonIgnoreType
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

## 其他注解

### `@JsonProperty`

该注解可以指定属性在Json字符串中的名字
下例中在非标准的`setter`和`getter`方法上使用该注解, 可以成功序列化和反序列化

```java
public class MyBean {
    public int id;
    private String name;
 
    @JsonProperty("name")
    public void setTheName(String name) {
        this.name = name;
    }
 
    @JsonProperty("name")
    public String getTheName() {
        return name;
    }
}
```

### `@JsonFormat`

```java
public class Event {
    public String name;
 
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
}
```

### `@JsonUnwrapped`

该注解指定值在序列化和反序列化时, 去除对应属性的外包装(根节点)

```java
public class UnwrappedUser {
    public int id;
 
    @JsonUnwrapped
    public Name name;
 
    public static class Name {
        public String firstName;
        public String lastName;
    }
}
```

序列化结果：

```json
{
    "id":1,
    "firstName":"John",
    "lastName":"Doe"
}
```

### `@JsonManagedReference, @JsonBackReference`

这两个注解配合使用, 可以解决两个不同类的属性的父子关系`(parent/child relationships)`和循环引用`(work around loops)`
使用`@JsonBackReference`可以在序列化时阻断循环引用, 原理是忽略被注解的属性, 否则会导致异常
本例中, 我们用这组注解来序列化`ItemWithRef`实体类:

```java
public class ItemWithRef {
    public int id;
    public String itemName;
 
    @JsonManagedReference
    public UserWithRef owner;
}
public class UserWithRef {
    public int id;
    public String name;
 
    @JsonBackReference
    public List<ItemWithRef> userItems;
}
```

序列化结果: `{"id":2,"itemName":"book","owner":{"id":1,"name":"John"}}`
如果把注解对调并序列化user结果是: `{"id":1,"name":"John","userItems":[{"id":2,"itemName":"book"}]}`

### `@JsonFilter`

该注解可以在序列化时指定一个过滤器
下面为一个实体类指定一个过滤器:

```java
@JsonFilter("myFilter")
public class BeanWithFilter {
    public int id;
    public String name;
}
```

```java
public void whenSerializingUsingJsonFilter_thenCorrect()  throws JsonProcessingException {
    BeanWithFilter bean = new BeanWithFilter(1, "My bean"); 
    FilterProvider filters = new SimpleFilterProvider().addFilter("myFilter",    
     						SimpleBeanPropertyFilter.filterOutAllExcept("name")); 
    String result = new ObjectMapper().writer(filters).writeValueAsString(bean);
}
```

序列化结果:`{"name":"My bean"}`
**这里添加了一个`SimpleBeanPropertyFilter.filterOutAllExcept`过滤器, 该过滤器的含义是除`name`属性外, 其他属性都被过滤掉(不序列化)**

### `@JsonAppend`

有时候我们需要在不改变类的属性字段的情况下，添加字段，`Jackson`提供了`@JsonAppend`注解来实现这个功能，使用方式如下：

```java
@Getter
    @Setter
    @AllArgsConstructor(staticName = "of")
    @NoArgsConstructor
    @JsonAppend(attrs = {@JsonAppend.Attr(value = "age"),@JsonAppend.Attr(value = "height")},
                props = {@JsonAppend.Prop(value =TestWriter.class ,type = String.class,name = "version")})
    class JsonPropertyPojo{


        private String sex;

        private String name;

        private String unknown;

    }
@NoArgsConstructor
    class TestWriter extends VirtualBeanPropertyWriter{


        private TestWriter(BeanPropertyDefinition propDef, Annotations contextAnnotations, JavaType declaredType) {
            super(propDef, contextAnnotations, declaredType);
        }


        @Override
        protected Object value(Object bean, JsonGenerator gen, SerializerProvider prov) throws Exception {

            Field nameField = ReflectionUtils.findField(bean.getClass(),"name");
            ReflectionUtils.makeAccessible(nameField);

           if( nameField.get(bean).toString().length()>2){

               return "1.2";


           }

           return "1.0";

        }

        @Override
        public VirtualBeanPropertyWriter withConfig(MapperConfig<?> config, AnnotatedClass declaringClass, BeanPropertyDefinition propDef, JavaType type) {
            return new TestWriter(propDef, declaringClass.getAnnotations(), type);
        }
    }
测试代码如下
  @Test
    public void JsonAppendTest() throws Exception{

        CombineJacksonAnnotation.JsonPropertyPojo pojo = CombineJacksonAnnotation.JsonPropertyPojo.of("男","小小刘","some");
        System.out.println(om.writerFor(CombineJacksonAnnotation.JsonPropertyPojo.class).withAttribute("age","10").withAttribute("height","12").writeValueAsString(pojo));

    }
序列化结果如下
{
  "sex" : "男",
  "name" : "小小刘",
  "unknown" : "some",
  "age" : "10",
  "height" : "12",
  "version" : "1.2"
}
```

> 通过上面的例子可以看到，@JsonAppend提供了两种方式来动态的添加虚拟字段
>  1 attrs
>  此种方式需要在序列化时候手动的添加Attribute，如下
>  om.writerFor(CombineJacksonAnnotation.JsonPropertyPojo.class).withAttribute("age","10").withAttribute("height","12")
>  2 props
>  此种方式比较灵活，但是要实现一个VirtualBeanPropertyWriter类即可，如果真的有这种需求，推荐使用第二种方式来实现