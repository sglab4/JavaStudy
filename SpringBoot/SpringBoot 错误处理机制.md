# SpringBoot 默认处理机制

在 `ErrorMvcAutoConfiguration` 类中存放了所有关于错误信息的自动配置。

访问步骤：

- 首先客户端访问了错误界面。例：404 或者 500
- `SpringBoot`注册错误请求`/error`。通过`ErrorPageCustomizer`组件实现，`ErrorPageCustomizer`为 `ErrorMvcAutoConfiguration` 的一个静态内部类
- 通过`BasicErrorController` 类处理`/error`，对错误信息进行了自适应处理，在 BasicErrorController 中，分为 errorHtml 和 error 两个主要处理方法，分别是处理浏览器发送的请求和其它浏览器发送的请求的。浏览器会响应一个界面，其他端会响应一个`json`数据
- 如果响应一个界面，通过`DefaultErrorViewResolver`类来进行具体的解析。可以通过模板引擎解析也可以解析静态资源文件，如果两者都不存在则直接返回默认的错误`JSON` 或者错误`View`
- 通过`DefaultErrorAttributes`来添加具体的错误信息

源代码如下：

```java
//错误信息的自动配置
public class ErrorMvcAutoConfiguration {
    //包括响应具体的错误信息(errorAttributes)、处理错误请求(basicErrorController)、注册错误界面(errorPageCustomizer)
    ....
}
```

```java
//注册错误界面,错误界面的路径为/error
private static class ErrorPageCustomizer implements ErrorPageRegistrar, Ordered {
	....
}
```

```java
//处理/error请求,从配置文件中取出请求的路径
@RequestMapping({"${server.error.path:${error.path:/error}}"})
public class BasicErrorController extends AbstractErrorController {
    //浏览器行为，通过请求头来判断，浏览器返回一个视图，ModelAndView
    @RequestMapping(produces = {"text/html"}) 
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response){
        ....
    }
    
    //其他客户端行为处理，返回一个JSON数据
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ....
    }
}
```

```java
public class DefaultErrorViewResolver implements ErrorViewResolver, Ordered {
    //使用 resolveErrorView 方法根据传入的状态码进行匹配
	....
}
```

```java
//添加错误信息
public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
   
	....
}
```

# 自定义处理机制

## @ControllerAdvice + @ExceptionHandler

自定义一个异常处理类

```java
/**
 * @ControllerAdvice 有 @Component 注解，因此会将类添加到容器中
 */
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * @ExceptionHandler 用来处理异常，参数为可处理的异常类型
     */
    @ExceptionHandler({ArithmeticException.class, NullPointerException.class})
    //页面跳转
    public String handleException() {
        return "exception";
    }
}
```

对于访问时，在 Controller 中出现的计算异常和空指针异常，都将由 handleException 方法来进行处理。

SpringBoot 使用 ExceptionHandlerExceptionResolver 来支持这种自定义异常处理机制，在 DispatcherServlet 初始化时，会将 ExceptionHandlerExceptionResolver 注册到 handlerExceptionResolvers 中，调用的关键方法是 ExceptionHandlerMethodResolver 类中的 `protected ServletInvocableHandlerMethod getExceptionHandlerMethod` 方法 。出现异常时，首先会从 ExceptionHandlerExceptionResolver  中的 exceptionHandlerCache 中去找参数 handlerMethod 所属 bean 的 class 对应的 ExceptionHandlerMethodResolver， 如果找不到则 new 一个 ExceptionHandlerMethodResolver 并缓存起来。 然后从ExceptionHandlerMethodResolver 去找该 exception 对应的异常处理方法。

```java
@Nullable
protected ServletInvocableHandlerMethod getExceptionHandlerMethod(@Nullable HandlerMethod handlerMethod, Exception exception) {
	....
}
```

![img](https://pict-picgo.oss-cn-hangzhou.aliyuncs.com/picture3/202209191600196.png)

ExceptionHandlerMethodResolver 中存的是各个 Exception 到各个异常处理方法映射。

## @ResponseStatus + 自定义异常

在使用控制器 Controller 时，可手动 throw 抛出异常

```java
//可返回错误状态码，HttpStatus.FORBIDDEN 是 403 错误
@ResponseStatus(value = HttpStatus.FORBIDDEN, reason = "用户数量太多")
public class UserException extends RuntimeException {
    
}
```

```java
@Controller
public class CommonController {

    @RequestMapping("common")
    public String common() {
        throw new UserException();
        //return "common";
    }
}
```

底层仍然调用 ExceptionHandlerExceptionResolver 类的 `protected ServletInvocableHandlerMethod getExceptionHandlerMethod`方法构造传入错误码和错误信息的 Model

