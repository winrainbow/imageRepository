# Spring MVC

## 前端控制器：

web.xml中配置servlet 使用 DispatcherServlet

init-param 加载 spring-mvc.xml

spring-mvc.xml中开启扫描，

spring-mvc.xml中配置视图解析器

spring-mvc.xml中开启 注解

dispatcherServlet->controller-dispatcherServlet->viewResolver->.jsp->dispatcherServlet->浏览器

handlerMapping（处理映射器）:让controller知道用哪个方法执行

HandlerExecutionChain{

​	HandlerInterceptor1

}

处理适配器（HandlerAdapter）：

<mvc:annotation-driven>:相当于配置了适配器和映射器

## RequestMapping

建立请求与方法建立映射关联

作用在方法上，类上

属性：

* value  path和value的作用是一样的 value属性可以不写
* path
* method:当前方法可以接收什么样的请求方式：GET、PUT、POST、DELETE
* params:限定请求参数条件，必须带有指定的参数 ?hello=ss

## 请求参数绑定

username=hhe&password=123

mvc框架：sayHello(String username,String password) 通过反射的方法拿到参数名

<form>

<input type="text" name="username"/>

<input type="text" name="user.age"/>

<input type="text" name="list[0].uname"/> 集合list

<input type="text" name="map['one'].uname"/> map

</form>

### 特殊情况：自定义类型转换器

提交给后台的全部是 字符串，需要转换

```xml
<!-- 配置自定义类型转换器 -->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.hgj.utils.StringToDateConverter"/>
        </set>
    </property>
</bean>
<!-- 开启spring mvc 注解支持 -->
<mvc:annotation-driven  conversion-service="conversionService"/>
```

## servlet原生API

```java
@RequestMapping("/account")
public String getServletApi(HttpServletResponse response, HttpServletRequest request){
    
    return "success";
}
```

## RequestParam 注解

``` java
@RequestMapping("/requestParam")
public String requestParam(@RequestParam(name="name") String username){

    return "success";
}
```

## RequestBody

```java
// 拿到整个body
@RequestMapping("/requestParam")
    public String requestBody(@RequestBody String body){

        return "success";
    }
}
```

## PathVariable注解

> RESTfull风格：结构清晰，利于缓存
>
> 请求路径一样，但用不同的请求方式决定由哪个方法来执行
>
> path="/user/{id}"
>
> findById(id)

```java
@RequestMapping("/requestParam/{id}")
public String findById(@PathVariable(name="id") String id){

    return "success";
}
```

## HiddenHttpMethodFilter过滤器

浏览器不容易发送Put、DELETE请求等，新增过滤器用来模拟 PUT DELETE等请求

## RequestHeader注解

用于获取请求消息头

```java
@RequestMapping("/testrequestheader")
public String getHeader(@RequestHeader(name = "content-type") String header){

    return "success";
}
```

## CookieValue注解

```java
@RequestMapping("/testrequestheader")
public String getCookie(@CookieValue(value = "JSESSIONID") String cookieValue){

    return "success";
}
```

## ModelAttribute注解

作用上方法上：方法在控制器方法执行前先执行  类似于aop中的 before

提前处理数据

作用上参数上：当modelAttribute的方法没有返回值 时，需要用这个取出

## SessionAttribute 注解

```java
@SessionAttribute(value="msg")
public class TestController{
@RequestMapping("/testsessionAttribute")
// @SessionAttribute只能在类上
public String getCookie(Model model){
model.addAttribute("msg","美美")
return "success";
}
    }
```
## controller方法返回String

```java
public String getUser(Model model){
    User user = new User();
    model.addAttribute("user",user);
    return "success"
}
```

## controller 方法返回Void

默认返回请求路径.jsp 

```java
// 转发是一次请求
public void testVoid(HttpServletRequest request){
    request.getRequestDiaptcher("/WEB-INF/pages/success.jsp").forward(request,response);
   // 后面代码还会执行，不想后面代码执行，使用return
    
        //
}
// 重定向是两次请求
public void testVoid(HttpServletResponse response){
    response.sendRedirect(request.getContextPath()+"/index.jsp")
   // 后面代码还会执行，不想后面代码执行，使用return
    
        //
}
public void testVoid(HttpServletResponse response){
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=utf-8")
    response.getWriter().printer("hello");
   // 后面代码还会执行，不想后面代码执行，使用return
    
        //
}
```

## Controller方法返回 ModelAndView对象

```java
public ModelAndView test(){
   ModelAndView mv = new ModelAndView();
   User user = new User("sss");
    mv.addObject("user",user);
    mv.setViewName("success");
    return mv; //使用视图解析器
}
```

## Controller forward 和Redirect

```java
public String testKey(){
    return "forward:/WEB-INF/pages/success.jsp";
}
public String testKey(){
    return "redirect:/index.jsp";
}
```

## ResponseBody 响应JSON数据

```java
// 后端将jsonStr 转成 User对象，需引入jackson相关包
public @ResponseBody User testAjax(@Reqeustbody User user){
    System.out.println(user.toString());
    user.setName("heheheh");
    // 做响应
    return user;// 返回的是user对象
    
}
```

## 文件上传

1. enctype=multipart/form-data
2. post方法
3. <input type="file"/>

借助第三方组件实现上传

Commons-fileupload 帮助解析

## 跨域服务器文件上传

请求->应用服务器->图片服务器->浏览器

1. 搭建图片服务器（tomcat）

2. 应用服务器，controller 上传方法编写

   ```java
   1.定义上传服务器的路径：path = http://localhost:9090/uploads/
   2.导入jar包
     <groupId>com.sun.jersey</groupId>
     <artifactId>jersey-core</artifactId>
     <version>1.18.1</version>
     <groupId>com.sun.jersey</groupId>
     <artifactId>jersey-client</artifactId>
     <version>1.18.1</version>
   3.创建客户端对象
   	Client client = Client.create();
   4.和图片服务器建立链接
   	WebResources resources = client.resource(path+fileName);
   5.上传文件
   	resources.put(upload.getBytes());
   	
   ```

   ​

## 异常处理，跳友好页面

springMvc 框架

异常处理器组件

```xml
1.编写自定义异常类
2.编写异常处理器 implements HandlerExceptionResolver
2.配置异常处理器 spring-mvc.xml中配置
<bean id = "" class="异常处理器">
</bean>

引入包：

```

## 拦截器

类似于filter,进行预处理和后处理

拦截器，在controller执行之前，和执行之后调用

拦截器可以是多个，拦截器链

拦截器与filter区别：

 	1. filter是servlet一部分
 	2. 拦截器是 springmvc 特有
 	3. 拦截器只会拦截controller方法，filter可拦范围广

<mvc:interceptors>

</mvc:interceptors>

## ssm环境整合

分层：web service dao

web层：springMvc

service: spring

dao: Mybatis

spring 为基础，整合另外两个框架

环境搭建

spring 整合 springmvc 启动tomcat时加载spring配置文件

servletContext 对象：只会被 创建 一次，在服务器启动时创建，服务器关闭时销毁

监听器：监听servletContext 创建和销毁 （）

监听器去加载spring的配置文件，创建web版本工厂，存储到servletContext对象中

## 整合myBatis

1. dao 生成的代理对象，存到ioc容器