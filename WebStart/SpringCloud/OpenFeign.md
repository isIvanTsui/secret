<h1 align="center">OpenFeign的GET/POST请求</h1>

## 服务提供者

```java
package com.itlaoqi.springcloud.bookservice.controller;

import com.itlaoqi.springcloud.bookservice.entity.Book;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import java.util.ArrayList;
import java.util.List;

@RestController
public class BookController {
    @PostMapping("/create")
    public String createBook(@RequestBody Book book){
        return book.getName() + "创建成功";
    }

    @GetMapping("/search")
    public List<Book> search(Book book){
        List list = new ArrayList();
        if(book.getSn().equals("1111") && book.getName().equals("x")){
            list.add(new Book("1111", "XXXX", ""));
        }
        return list;
    }
}
```

## FeignApi

```java
package com.itlaoqi.springcloud.memberserviceopenfeign.client;
import com.itlaoqi.springcloud.memberserviceopenfeign.entity.Book;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;
import java.util.Map;

@FeignClient(name="book-service")
public interface BookApi {

    @PostMapping("/create")
    public String createBook(@RequestBody Book book);

     // 注意为map 进行get 传递
    @GetMapping("/search")
    public List<Book> search(@RequestParam Map book);
}
```

> 注意：
>
> `GET` 需要`@RequestParam`注解，并且需要传递`Map`
> `POST` 需要加 `@RequestBody` 注解

## 服务消费者

```java
@Controller
public class MemberController {
    @Resource
    BookService bookService;

    @GetMapping("/createBook")
    @ResponseBody
    public String compensate(){
        Book book = new Book();
        book.setName("赔偿图书");
        book.setSn("5555");
        String result = bookService.createBook(book);
        return result;
    }

    @GetMapping("/search")
    @ResponseBody
    public List<Book> search(){
        //通过map 进行信息传递
        Map param = new HashMap();
        param.put("sn", "1111");
        param.put("name", "x");
        List<Book> list = bookService.search(param);
        return list;
    }
}
```

