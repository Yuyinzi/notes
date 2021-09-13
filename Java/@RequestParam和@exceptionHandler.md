## @RequestParam
在使用`Spring Boot`的过程中，经常使用的是`@RequestParam`，可以获得请求参数值的解析。
它有四个属性：
- `value`：`url`中的参数名要与其值一致
```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(@RequestParam(value = "name", required = false) String firstName, String lastName) {
        return "hello " + firstName + ' ' + lastName;
    }
}
```
调用结果：
```
~ curl --location --request GET 'localhost:8080/hello?name=little&lastName=may'
hello little may
~ curl --location --request GET 'localhost:8080/hello?firstName=little&lastName=may'
hello null may
```
- `name`:与`value`一样，实际上互为别名
> `@RequestParam(name = "name", required = false) String firstName`与`@RequestParam(value = "name", required = false) String firstName`是一样的
- `required`：默认为`true`
> 如果不指明为`false`，则`value`或者`name`值与传入参数名不一致时，会报错。
- `defaultValue`:当参数没有传入时的默认值
```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(@RequestParam(name = "name", defaultValue = "big") String firstName, String lastName) {
        return "hello " + firstName + ' ' + lastName;
    }
}
```
调用结果：
```
 ~ curl --location --request GET 'localhost:8080/hello?firstName=little&lastName=may'
hello big may
```

比较特别的是，如果不显式使用`RequestParam`，相当于对于该参数默认使用了`@RequestParam(required=false, name=参数名)`
```
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(String firstName, String lastName) {
        return "hello " + firstName + ' ' + lastName;
    }
}
```
调用结果：
```
 ~ curl --location --request GET 'localhost:8080/hello?firstName=little&lastName=may'

hello little may
 ~ curl --location --request GET 'localhost:8080/hello?firstName1=little&lastName=may'
hello null may
```
## @exceptionHandler
实际开发中通常出现一些不可预料的错误，比如数据库查询语句错误，或者空指针，请求第三方超时之类的错误。需要些很多`try...catch`，才能尽可能覆盖所有可能出现的错误。一般处理的方式是将响应统一封装，然后指定错误码，不将错误信息暴露给前端，可以采用`exceptionHandler`进行全局的统一异常增强处理。
> 利用`@RestControllerAdvice`，可以不需要使用`@ResponseBody`：
```
// GlobalExceptionHandler.java
package com.littlemay.spring.demo.handler;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseStatus(code= HttpStatus.INTERNAL_SERVER_ERROR)
    public String exceptionHandler(Exception e){
// 通常在此进行log打印，重新封装响应进行返回
        System.out.println(e.getMessage());
        return "服务器开小差了~~";
    }
}
```
测试：
```
// Hello.java
package com.littlemay.spring.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        double t = 1 / 0;
        return "hello";
    }
}
```
调用结果：
```
~ curl --location --request GET 'localhost:8080/hello'
服务器开小差了~~
```