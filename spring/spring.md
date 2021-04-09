[toc]

# spring

> springbean的生命周期讲一下

SpringBean的生命周期指一个Bean对象从创建、到销毁的过程。SpringBean不等于普通对象，实例化一个java对象只是Bean生命周期过程的一步，只有走完了流程才称之为SpringBean。核心过程如下：
1. 实例化Bean：主要通过反射技术，实例化Java对象；
2. 设置对象属性（依赖注入）：向实例化后的Java对象中注入属性；
3. 处理Aware接口：接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给Bean：
a. 如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的id值；
b. 如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()方法，传递的就是Spring工厂自身；
c. 如果这个Bean已经实现了ApplicationCOntextAware接口，会调用setApplicationContext(ApplicationContext context)方法，传入Spring上下文
4. BeanPostProcessor：如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。
5. InitializingBean与init-method：实现接口InitializingBean完成一些初始化逻辑，如果Bean在Spring配置文件中配置了init-method属性，则会自动调用其配置的初始化方法，完成一些初始化逻辑。
6. 如果这个Bean实现了BeanProcessor接口，那将会调用postProcessAfterInitialzation(Object obj, String s)方法，在这个过程中比如可以做代理增强；
7. DisposableBean：当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；
8. destroy-method：最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

> spring mvc的工作流程讲一下

![](https://gitee.com/super-jimwang/img/raw/master/img/20210303142310.png)
1.用户发送请求至前端控制器 DispatcherServlet 

2.DispatcherServlet 收到请求调用 HandlerMapping 处理器映射器。HandlerMapping根据url寻找Handler

3.处理器映射器根据请求 url 找到具体的处理器, 生成处理器对象及处理器拦截器(如果有则生成)一并返回给 DispatcherServlet。

4.DispatcherServlet 通过 HandlerAdapter 处理器适配器调用处理器

5.执行处理器(Controller,也叫后端控制器)。

6.Controller 执行完成返回 ModelAndView 

7.HandlerAdapter 将 controller 执行结果 ModelAndView 返回给 DispatcherServlet 

8.DispatcherServlet 将 ModelAndView 传给 ViewReslover 视图解析器

9.ViewReslover 解析后返回具体 View 

10.DispatcherServlet 对 View 进行渲染视图(即将模型数据填充至视图中)。

11.DispatcherServlet 响应用

> Spring中用了哪些设计模式

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

> spring能否支持事务与什么有关

与数据库有关，数据库支持事务就可以

> spring管理事务的方式有几种

1. 编程式事务，在代码中硬编码。(不推荐使用)
2. 声明式事务，在配置文件中配置（推荐使用）

**声明式事务又分为两种：**

1. 基于XML的声明式事务
2. 基于注解的声明式事务

> @Resource和@Autowired的区别

- 都是用来自动装配，都可以在属性字段上
- @Autowired通过byType的方式实现，并且必须要求这个对象存在！
- @Resource默认通过byname的方式实现，如果名字找不到，则通过byType！如果两个都找不到，就报错
- 执行顺序：
  - @Autowired 先类型 再名字
  - @Resource 先名字 再类型

> 讲一下AOP中各个名词的含义

（1）切面（Aspect）：被抽取的公共模块，可能会横切多个对象。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @AspectJ 注解来实现。

（2）连接点（Join point）：指方法，在Spring AOP中，一个连接点 总是 代表一个方法的执行。

（3）通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。通知有各种类型，其中包括“around”、“before”和“after”等通知。许多AOP框架，包括Spring，都是以拦截器做通知模型， 并维护一个以连接点为中心的拦截器链。

（4）切入点（Pointcut）：切入点是指 我们要对哪些Join point进行拦截的定义。通过切入点表达式，指定拦截的方法，比如指定拦截add、search。

（5）引入（Introduction）：（也被称为内部类型声明（inter-type declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现 IsModified 接口，以便简化缓存机制。

（6）目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

（7）织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。


![](https://gitee.com/super-jimwang/img/raw/master/img/20210312202201.png)

> 讲一下@SpringBootApplication

这个注解可以看作是
- @EnableAutoCinfuguration:根据情况自动加载配置类，将注解类的包下所有类注入spring（不包括子包）
  - @autoConfigurationPackages注解类包下所有类注入spring
  - 又通过@Import自动注入需要的配置累
- @ComponentScan:将包及其子包与指定的包和子包内的所有类注入spring
- @SpringBootConfiguration
  - @Configuration: 允许在Spring上下文中注册额外的bean



> 前后端传值需要用哪些注解

- 对于get方法有@PathVariable和@RequestParam
  - @PathVariable是从/{}/中读数据
  - @RequestParam是从?xxx=读数据
- 对于post请求
  - @RequestBody

> 配置文件中的信息怎么读取到bean中

- `@Value("${xxx}")`放在变量上面
- `@ConfigurationProperties(xxx)` 放在类上面，会直接初始化bean对象

> 如何自定义处理Controller层的异常

- 在类的上面注解`@ControllerAdvice`

> 注解是如何生效的

**注解只是metadata，不包含任何业务逻辑**。 注解只是在其内部定义了一些方法（也就是属性）。

那么要让注解生效就需要消费者。消费者可以通过发射机制获得注解的属性，然后再进行处理。

```java
Class businessLogicClass = BusinessLogic.class;
for(Method method : businessLogicClass.getMethods()) {
    Todo todoAnnotation = (Todo)method.getAnnotation(Todo.class); // 获取注解
    if(todoAnnotation != null) { // 获取注解的属性
        System.out.println(" Method Name : " + method.getName());
        System.out.println(" Author : " + todoAnnotation.author());
        System.out.println(" Priority : " + todoAnnotation.priority());
        System.out.println(" Status : " + todoAnnotation.status());
    }
}
```

> spring的三级缓存指哪三个，分别什么用

三级：singletonFactories ： 单例对象工厂的cache。存储对象工程ObjectFactory
二级：earlySingletonObjects ：提前暴光的单例对象的Cache。存半成品的bean
一级：singletonObjects：单例对象的cache。就是spring容器，存的就是完全初始化的实例

![](https://gitee.com/super-jimwang/img/raw/master/img/20210308213743.png)

> 如何解决循环依赖

我们假设现在有这样的场景AService依赖BService，BService依赖AService

1. AService首先实例化，实例化将ObjectFactory存在三级缓存中

2. 填充属性BService，发现BService还未进行过加载，就会先去加载BService

3. 再加载BService的过程中，实例化，也通过ObjectFactory半成品暴露在三级缓存

4. 填充属性AService的时候，这时候能够从三级缓存中拿到半成品的ObjectFactory

拿到ObjectFactory对象后，调用ObjectFactory.getObject()方法最终会调用getEarlyBeanReference()方法，getEarlyBeanReference这个方法主要逻辑大概描述下如果bean被AOP切面代理则返回的是beanProxy对象，如果未被代理则返回的是原bean实例，这时我们会发现能够拿到bean实例(属性未填充)，然后从三级缓存移除beanA，并放到二级缓存earlySingletonObjects中，而此时B注入的是一个半成品的实例A对象，不过随着B初始化完成后，A会继续进行后续的初始化操作，最终B会注入的是一个完整的A实例，因为在内存中它们是同一个对象。

![](https://gitee.com/super-jimwang/img/raw/master/img/20210308214155.png)

> 为什么需要二级缓存

因为如果A被AOP切面了，那每次去三级缓singletonFactory.getObject()方法都是一个新的代理对象。不是同一个。因此把三级缓存生成的动态代理对象存到二级缓存去，下次直接去二级缓存拿。

> BeanFactory和ApplicationContext的区别

BeanFactory：

是Spring里面最低层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能；

 

ApplicationContext：

应用上下文，继承BeanFactory接口，它是Spring的一各更高级的容器，提供了更多的有用的功能；

1) 国际化（MessageSource）

2) 访问资源，如URL和文件（ResourceLoader）

3) 载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层  

4) 消息发送、响应机制（ApplicationEventPublisher）

5) AOP（拦截器）

两者装载bean的区别

 

BeanFactory：

BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化；

 

ApplicationContext：

ApplicationContext在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化；

> 拦截器和过滤器的区别

![](https://gitee.com/super-jimwang/img/raw/master/img/20210318205016.png)

①拦截器是基于java的反射机制的，而过滤器是基于函数回调。

②拦截器不依赖与servlet容器，过滤器依赖与servlet容器。

③拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。

④拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。

⑤在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。

⑥拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

> Controller的作用域是什么，在Controller中有一个成员变量@Resource的HttpRequest怎么解决？

Controller是单例的。

spring框架在初始IOC的时候，创建了一个Request对象的代理类，从而完成了初始注入，代理类负责从ThreadLocal中获取真正的Request对象并调用相应的方法。每次对成员变量request（实际上是个代码类）的任何方法的调用，最终转化成了该次请求真正的request对象的方法，因此不存在线程安全问题。

> IoC的时候什么时候创建对象，什么时候是代理类？

　1）service方法添加@Transactional注解或者加入其它的aop拦截配置，没有实现任何接口。   对应CGLIB代理类

　　2）service方法添加@Transactional注解或者加入其它的aop拦截配置，实现了接口。              对应JAVAJDK代理类

　　3）serice方法没有添加@Transactional注解或者其它的aop拦截配置。                                      对应对象


> spring的bean为什么默认是单例

（1）减少了新生成实例的消耗 新生成实例消耗包括两方面，首先，Spring会通过反射或者cglib来生成bean实例这都是耗性能的操作，其次给对象分配内存也会涉及复杂算法

（2）减少jvm垃圾回收 由于不会给每个请求都新生成bean实例，所以自然回收的对象少了

（3）可以快速获取到bean 因为单例的获取bean操作除了第一次生成之外其余的都是从缓存里获取的所以很快


> FactoryBean中的beanName讲一下。

https://segmentfault.com/a/1190000039691809

当name前面加了`&`的时候，表示获取FactoryBean本身，而如果没加`&`表示获取FactoryBean所维护的Bean。

```java
// 注意这里没有注解，一旦有了注解，就会交给spring来管理
public class BeanA {

}

@Component
public class MyFactoryBean implements FactoryBean {
  @Override
  public Object getObject() throws Exception {
    return new BeanA();
  }

  @Override
  public Class<?> getObjectType() {return BeanA.class};
}
```

![](https://gitee.com/super-jimwang/img/raw/master/img/20210402142329.png)

==这里需要注意一个点: 不管是获取FactoryBean还是FactoryBean维护的bean。首先从spring单例池获取的bean都为FactoryBean，因为spring中没有一个bean会包含&符号，最终都是根据去除name中的&符号的beanName去获取bean。然后再根据当前bean的类型和name中是否包含&符号来确定返回的是FactoryBean还是FactoryBean维护的bean。所以我们要想获取FactoryBean，要在beanName前添加&符号==

这里的beanName都是myFactoryBean。但是name不同。一个是&myFactoryBean（对应本身），另一个是myFactoryBean（对应维护的bean）

**总结**
- 要获取FactoryBean则添加&符号。eg: context.getBean("&myFactoryBean")
- 要获取FactoryBean维护的bean则不添加&符号。eg: context.getBean("myFactoryBean")