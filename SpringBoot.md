# SpringBoot

## pom中关键配置

### 1. <parent> springboot

### 2. <parent> springbootdependencies  进行真正的管理依赖

### 3. 未在dependencies中管理撤销，需要声明

### 4.导入 <Dependencies>spring-boot-start-web :场景启动器：

帮我导入web模块运行需要的组件，如mvc jackson等

> springboot 将功能场景抽取，并做成一个个的start(启动器)，我们只需入引入start就可以了，版本有springboot控制

## @SpringBootApplication:注解标注在某个类上，标明这个类是springboot的主配置类，SpringBoot 就运行这个类的main方法。

是一个组合注解：

1. springbootConfiguration: springboot配置：是 一个springBoot的配置类

   1. configuration: spring的注解：容器中的一个组件

2. EnableautoConfiguration:开启自动配置功能：将主配置类所在包及子包所有的组件扫描到spring容器中

   需要配置的东西，springBoot自动配置。

   1. @autoConfigurationPackage  自动配置包： @import(autoConfigurationPackages.Registrar.class);
   2. @import(Enable)

## spring中的容器和组件的概念？？

## 快速创建SpringBoot

spring initializer，选择我们需要的模块，会联网创建项目

## 配置文件注入

* @SpringBootTest

* @ConfigurationProperties（prefix = "persion"）  @Value

* @Autowired

* 如果只是在某个业务逻辑中需要获取某项值 ，是使用@Value,涉及复杂类型则使用@ConfigurationProperties

* 如果是专门编写了 javaBean 和配置文件映射，则使用ConfigurationPropertis

* persion.properties  @PropertySource(value = {"classpath:presion.properties"})  指定加载的配置文件

* @importResource 导入Spring的配置文件，让配置文件生效  设置到主配置类上

* 配置类：config.myAppConfig 需要加上 @Configuration 

  1. 添加组件： @Bean //将方法的返回值 ，添加到容器中，容器中组件默认的id是方法名

     ```java
     @Configuration
     public class MyConfig{
         @Bean
         public HelloService helloService(){
             return new HelloService();
         }
     }
     ```

## 自动配置的原理

1. 配置文件怎么写？写什么？官方文档
   1. 配置文件可以配置的属性参照官方文档
   2. 能配置的属性，都是来源于功能的Properties类

2. 自动配置原理：
   1. 启动时，加载的主配置类，开启了自动配置功能 @EnableAutoConfiguration
   2. @EnableAutoConfiguration:
      1. 利用EnableAutoConfigurationImportSelection 给容器中导入一些组件
      2. 主要是EnableAutoConfigurationImportSelection方法中的 selectImports()方法
      3. 扫描jar包，类目录下的 mata-INFO/spring.factories
      4. 将扫描到文件的内容 包装成Properties对象
      5. 从Properties中获取到 EnableAutonConfiguration的值 ，添加到容器中
   3. 自动配置原理：

   ## Bean 作用域：

   1. 单例：Singleton 在整个应用中，只创建一个bean的实例

      > 在大多数情况下，创建一个bean是可以的，因为里面只是方法和本地变量，没有属性变量，所以多线程访问时，也不会出现多线程共享的问题 

   2.  原型：Prototype:每次注入或者通过Spring应用上下文获取时的时候，都会创建一个新的bean实例

   3. 会话：Session:在web应用中，为每一个会话创建一个bean实例

   4. 请求：Request:在Web应用中，为每一个请求创建一个bean实例

   ```java
   @Component
   @Scope(ConfigurationBeanFactory.SCOPE_PROTOTYPE)
   public class Notepad{
       
   }
   ```

   ### bean中基于请求和会话的作用域的意义：同一个会话，共享购物车信息，而不是所有的链接共享购物车信息。

   #Spring表达式

   #Spring面向切面编程：

   ### 通知（Advice）：

   1. 前置通知（Before）,在目标方法执行前，调用通知的功能
   2. 后置通知（After）,在目标方法执行后，调用的通知，不会关心方法的输出是什么
   3. 返回通知（After-returning） 在目标方法成功执行之后调用的通知（执行成功之后）
   4. 异常通知（After-throwing）在目标方法抛出异常后调用通知
   5. 环绕通知（Around）通知包裹了被通知的方法，在被通知的方法调用之前、和之后执行自定义的行为

   ### 连接点(Join point)

   我们的应用有很多个时机应用通知，这些时机称为连接点。

   > 在应用执行过程中能够插入切面的一个点，可以是方法调用是地，抛出异常时，甚至修改一个字段时，切面代码可以利用这些点，插入到应用的正常流程中，并添加新的行为

   ### 切点(poincut)

   通知：定义了切面的“什么”和“何时”，切点定义了“何处”，

   > 切点的定义会匹配通知所要织入的一个或多个连接点，我们通常使用明确的类和方法的名称，或是利用正则表达式定义所匹配的类和方法名来来指定切点，

   ### 切面(Aspect)

   > 切面：通知和切点的结合，共同定义了切面的全部内容：是什么？在何时、何处完成其功能。

   ### 引入(Introduction)

   > 引入：允许我们向现有的类添加新方法、属性。

   ### 织入(Weaving)

   > 把切面应用到目标对象，并创建新的代理对象的过程。

   ## SpringMvc
# SpringBoot

   尽可能快的构建应用程序

   1. 不是服务器
   2. 不是JavaEE之类的规范
   3. 不是代码生成器
   4. 不是SpringFramework的升级版

   特性：

   1. 快速创建程序
   2. 内嵌tomcat,jetty,undertow
   3. 简化项目构建配置
   4. 为Spring及第三方库提供自动配置
   5. 提供生产级特性
   6. 无需生成代码或进行xml配置

   核心：

   1. 自动配置
   2. 起步依赖 start dependency
   3. 命令行界面
   4. Actuator

## springBoot 自动配置

classpath中出现的类，对应用程序作配置

spring-boot-autoconfiguration

@EnableAutoConfiguration

@SpringBootApplication 包含 EnableAutoConfiguration

# redo Log

   redo Log 是innoDb引擎的特有的，是顺序写磁盘，速度快，但记录的只能为innoDb使用，是物理的。 数据库插入数据时或者更新数据时，是先写redoLog文件，等空闲时或者redoLog没有写的空间时，才会真正的更新数据库中物理存储：因为物理存储有可能是不连续的，写起来速度更慢。



binLog 是MySql server层的，是逻辑的，任何引擎都可以拿去用来恢复数据库。



redoLog 和 binLog是分阶段提交，首先是 写入 redoLog文件，然后，redoLog的状态为prepare阶段，然后写入binLog 完事之后，redoLog设置为commit状态。

   

   







































































































