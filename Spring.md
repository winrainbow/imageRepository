## Spring 容器：

从概念上讲：Spring容器是spring框架的核心，用来管理对象。容器将创建对象，把它们连接在一起，配置他们，并管理他们的生命周期

Spring容器就是某个实现了ApplicationContext接口的类的实例，也就是说，从代码层面，Spring容器其实就是一个ApplicationContext的对象

一个javaweb中只有一个 上下文实例



#### Thymeleaf: 模板引擎

1. @Controller @RestController
2. @ModelAttribute

#### 静态资源/缓存

1. 核心配置：WebMvcConfigurer.addResourceHandlers()
2. 常用配置：spring.mvc.static-patch-pattern=/**
3. 缓存：ResourceProperties.Cache
4. Controller方法，设置缓存，返回时，返回ResponseEntity.ok().cacheController()

用户：CDN  ->静态资源

用户：GateWay->应用：缓存->应用

#### MVC的异常处理

HandlerExceptionResolver

SimpleMappingExceptionResolver

DefaultHandlerExceptionResolver

ResponseStatusExceptionResolver

....

处理方法：@ExcetionHandler

添加位置：@Controller / RestController 或者 ControllerAdvice(注解)优先级低于Controller中

#### Spring Mvc 切入点

1. HandlerIntecepter

   1. boolean preHandler()

   2. void postHandler()

   3. void afterCompletion()
2. ResponseBodyAdvice
3. AsyncHandlerInterceptor
4. 配置：在webMvcConfigurer.addInterceptors()
5. spring-boot配置，创建 webMvcConfigurer类
6. stopwatch,记录时间工具类


#### Spring中的applicationContext

#### spring mvc 基本使用

#### sping  mvc视频 json\xml等 

#### spring mvc 机制

#### 课程答疑：

1. mysql代替 h2

   1. application.properties设置数据源相关的属性，spring.datasource打头

   2. show databases

   3. ddl-auto

   4. lambda

      1. (params)->表达式

      2. （params）->{语句;}

      3. strem:

         1. forEach

         2. filter

         3. map

         4. flatMap()元素映射成流  ：把流中的元素拍平

            

2. MyBatisGenerator

   1. 生成的放在 auto目录下
   2. 手写的内容放在 manual目录下

3. MyBatis-Plus

#### 通过RestTemplate访问Web资源

1. new RestTemplate 或者RestTemplateBuilder.build()

2. getForObject()/getForEntity()

3. postForObject/postForEntity()

4. put()

5. delete()

6. 构造URI:uriComponentBuilder

7. ResTemplate.exchange() 传递httpheader

8. @JsonComponent 序列化，反 序列化

9. ClientHttpRequestFactory

10. SimpleClientHttpRequestFactory

11. OKHttp3ClientRequestFactory

#### 如何实现Restfull　web service

1. REST提供了一组架构约束，当作为一个整体应用时，强调组件交互的可伸缩性，接口的通用性、组件的独立部署、交
2. Richardson成熟度模型

   1. level0:一个Url 处理所有请求
   2. level:1 只处理get 和post
   3. level2: 使用 get post delete put
   4. level3:超媒体
 3. 识别资源
 4. 选择合适的资源粒度
 5. 设计URI
 6. 选择合适的http方法和返回码
 7. 设计资源的表述


#### HATEOAS

Hybermedia as the engin of application State

REST统一接口的必要组成部分

1. 表述中的超链接会提供服务所需的各种REST接口信息
2. 无需事先约定如何访问服务
3. 超链接类型：self,edit, collection,search,related,first,last,previous,next

#### 使用SpringDataRest实现简单的服务

1. HAL hypertext application language
2. 链接、内嵌资源，状态
3. spring-boot-state-data-rest
4. @RespositoryRestResource\Resource<T> PagedResource<T>

```java

```

#### SPring session 

支持：Redias,mongoDb,等

定制httpSession,

#### WebFlux

spring 5.0 之后，提供的组件

基于 reactivie streams API

为什么需要WebFlux: 1,对于非阻塞Web应用的需要，函数式编程

性能：耗时不会有很大的改善，使用相对少的线程和少的内存来实现扩展

webFlux和Nio 有关么？



####  Spring xml配置

* init-method: bean 被初始化时，执行的方法

* 注解的方式实现
   * 指定组建的init方法和destroy的几种方法

   * 1：在配置类中 @Bean(initMethod = "init",destroyMethod = "destory")注解指定

   * 2：实现InitializingBean重写其afterPropertiesSet方法，重写DisposableBean重写destroy方法

   *      3：利用java的JSR250规范中的@PostConstruct标注在init方法上，@PreDestroy标注在destroy注解上
   

需要注意的是：

 单实例bean：容器启动时创建对象
 多实例bean：每次获取时创建对象
 初始化：
 对象创建完成，赋值完成，调用初始化方法
 销毁：
 单实例：容器关闭时调用
 多实例：容器不会销毁，只能手动调用销毁方法











