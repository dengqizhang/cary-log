# SpringMVC 与 SpringBoot

Date：2024-12-3

作者：Cary

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [SpringMVC 与 SpringBoot](#springmvc-与-springboot)
  - [SpringMvc](#springmvc)
    - [使用 SpringMVC 框架处理 web 请求的顺序](#使用-springmvc-框架处理-web-请求的顺序)
  - [Springboot](#springboot)
    - [自动装配](#自动装配)
      - [SpringBoot 是怎么自动配置的？](#springboot-是怎么自动配置的)
        - [启动引导](#启动引导)
        - [自动配置类的加载：AutoConfigurationImportSelector](#自动配置类的加载autoconfigurationimportselector)
        - [哪些需要自动配置?](#哪些需要自动配置)
      - [自定义自动装配组件](#自定义自动装配组件)
        - [了解自动配置的 Bean](#了解自动配置的-bean)
        - [定位自动配置候选者](#定位自动配置候选者)
    - [IOC（控制反转）](#ioc控制反转)
      - [IOC 的好处](#ioc-的好处)
      - [IOC 是怎么实现的？](#ioc-是怎么实现的)
    - [AOP](#aop)
      - [AOP 核心概念](#aop-核心概念)
      - [AOP 通知分类](#aop-通知分类)
      - [AOP 使用方式](#aop-使用方式)
        - [基于注解](#基于注解)
        - [自定义注解 @interface](#自定义注解-interface)
          - [元注解](#元注解)
        - [自定义注解结合 AOP 的使用](#自定义注解结合-aop-的使用)
      - [AOP 的实现原理](#aop-的实现原理)
    - [Spring Bean 和依赖注入](#spring-bean-和依赖注入)
      - [@Bean 注解的使用](#bean-注解的使用)
      - [Bean 的生命周期](#bean-的生命周期)
    - [SpringBoot 的启动流程](#springboot-的启动流程)
    - [SpringBoot 事务](#springboot-事务)
      - [声明式事务管理 @Transactional](#声明式事务管理-transactional)
      - [事务使用规则](#事务使用规则)
    - [Springboot 上下文](#springboot-上下文)
      - [SpringBoot 上下文的类型](#springboot-上下文的类型)
      - [使用和获取 SpringBoot 上下文](#使用和获取-springboot-上下文)
      - [SpringBoot 上下文与 IOC 的联系](#springboot-上下文与-ioc-的联系)

<!-- /code_chunk_output -->

## SpringMvc

SpringMvc 是 Spring 框架的一部分，替代 Servlet 来接受请求，处理请求，返回处理结果给客户端。

SpringMvc 相比较传统的 Servlet 相比，有以下优点：

- 基于 MVC 架构，功能分工明确，解耦。

- 作为 Spring 框架的一部分，可以使用 Spring 的 IOC 和 AOP，方便整合 Spring 生态。

- SpringMVC 强化注解的使用，在控制器，Service，DAO 都可以使用注解。

### 使用 SpringMVC 框架处理 web 请求的顺序

![SpringMVC](https://upload-images.jianshu.io/upload_images/27448672-f2a9ad088a2bd7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Springboot

使用 Springboot 的目的呢，就是快速生成一个基于 Spring 的应用程序。

### 自动装配

自动装配机制官网的解释是，会试图根据你所添加的依赖来配置你的 Spring 应用程序。

例如在你添加了 `spring-boot-starter-web` 依赖后，Springboot 会自动配置一个嵌入式的 Tomcat 服务器，并配置好 DispatcherServlet，视图解析器等 SpringMVC 组件。

#### SpringBoot 是怎么自动配置的？

##### 启动引导

以一个带有 `@SpringBootApplication` 的注解的主类作为入口，这个注解是一个组合注解，它包含了

- @Configuration：表明该类为配置类，类似于 Spring 中的 xml 配置文件。

- ComponentScan： 用于扫描当前包及其子包下的组件如 `@controller` 等，将它们注册为 Spring 容器中的 Bean。

- @EnableAutoConfiguration：用于触发自动配置，开启自动配置功能，通过 `AutoConfigurationImportSelector` 类来加载自动配置类。并通过 `AutoConfigurationPackage` 将入口类的包及子包都纳入扫描范围，可以让自动配置能找到其中的组件。

##### 自动配置类的加载：AutoConfigurationImportSelector

- 当 `@EnableAutoConfiguration` 被解析时，自动配置类的加载：AutoConfigurationImportSelector 开始工作，会从类路径下的 `META - INF/spring.factories` 文件中查找并加载自动配置类。

> 在 Springboot 2.7.0 版本后，自动装配路径改为 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

##### 哪些需要自动配置?

Springboot 基于@Conditional 注解家族，可以配置自动装配的规则。

例如：

- @ConditionalOnClass：检查特定的类是否在类路径中存在。例如，在 DataSourceAutoConfiguration 中，可能会有@ConditionalOnClass({DataSource.class, EmbeddedDatabaseType.class})。这意味着只有当 DataSource 类和 EmbeddedDatabaseType 类都在类路径中时，这个自动配置类才有可能被启用，因为这表示项目可能需要配置数据源。

- @ConditionalOnMissingClass：与@ConditionalOnClass 相反，用于检查特定的类是否不存在于类路径中。如果存在，相应的自动配置可能不会进行。

#### 自定义自动装配组件

开发自己的自动装配，使用者只需要添加依赖到项目中，SpringBoot 会自动根据装配规则完成配置。

##### 了解自动配置的 Bean

实现自动配置的类需要用 `@AutoConfiguration` 注解。

##### 定位自动配置候选者

Springboot 会检查 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 的文件。该文件应列出您的配置类，每行一个类名，如下例所示：

```
com.mycorp.libx.autoconfigure.LibXAutoConfiguration
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

### IOC（控制反转）

IOC 的概念是一个容器，容器管理着很多的对象也被称为 Bean。

原本对象的创建和对象之间的依赖关系是由我们手动决定的，比如 new 一个 service 实例，然后再调用 service 实例的方法。但是 IOC 的概念是这些事情不再是我们手动做，而是交给容器。

从

`private UserRepository userRepository = new UserRepository();`

变成了

```
@Autowired
private UserRepository userRepository;
```

#### IOC 的好处

- 降低耦合度：

在传统模式下，例如我在 `UserService` 调用 `UserMapper时`，UserMapper 是在 UserService 里通过 New 关键字创建的，这种情况下，如果 UserMapper 发生变化时，UserService 就需要适应的改变代码来符合变化。

但在 IOC 下，UserMapper 是由 IOC 创建的不再受 UserService 的管理。

#### IOC 是怎么实现的？

有两种方式，Java 注解和 Java 代码配置

Java 注解：

- @Component：当前类是组件，没有明确的意思

- @Service： 当前类在业务逻辑层使用

- @Repository：当前类在数据访问层

- @Controller： 当前类在展示层（MVC）使用

使用注解在实体类上，会告诉 Spring，在项目运行时，帮我创建实体类的 Bean 添加到 IOC。

然后可以在例如 Service 中，使用这个 Bean

语法：

- @Autowired：根据 Bean 的 Class 类型来自动装配

- @Resource：翻译为“资源”，根据 Bean 得属性名称（id 或 name）自动装配

### AOP

AOP 主要用于将一些像日志记录，事务管理等功能，从业务逻辑代码中抽离出来。

例如定义一个日志切面，只需要在一个方法上加上注解，就可以完成记录日志的功能，同理适用于鉴权等业务场景。

#### AOP 核心概念

- Joinpoint（连接点）：指可以被动态代理拦截目标类的方法

- Pointcut（切入点）：指要对哪些 Joinpoint 进行拦截

- Advice（通知）：指拦截到 Joinpoint 之后要做的事

- Target（目标）：指代理的目标对象

- Weaving（织入）：指把增强代码应用到目标上，生成代理对象的过程。

- Proxy（代理）：指生成的代理对象。

- Aspect（切面）：切入点和通知的结合。

#### AOP 通知分类

- before（前置通知）：通知方法在目标方法调用之前执行

- after（后置通知）：通知方法在目标方法返回或异常后调用

- after-returning（返回后通知）：通知方法会在目标方法返回后调用

- after-throwing（抛出异常通知）：通知方法会在目标方法抛出异常后调用

- around（环绕通知）：通知方法会将目标方法封装起来

#### AOP 使用方式

##### 基于注解

- 引入 `spring-boot-starter-aop` 依赖

- 定义一个服务类

```
@Service
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello() {
        System.out.println("Hello World!");
    }
}
```

- 定义切入点：

```
@Aspect
@Component
public class MyAspect {
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceMethods() {}
    @Before("serviceMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("方法 " + joinPoint.getSignature().getName() + " 即将被调用");
    }
}
```

##### 自定义注解 @interface

###### 元注解

- @Target：用于描述注解的使用范围，该注解可以使用在什么地方

- @Retention：表明该注解的生命周期

- @Inherited：是一个标记注解，@Inherited 阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited 修饰的 annotation 类型被用于一个 class，则这个 annotation 将被用于该 class 的子类。

- @Documented：表明该注解标记的元素可以被 Javadoc 或类似的工具文档化

##### 自定义注解结合 AOP 的使用

- 创建自定义注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomAnnotation {
}

```

- 创建切面类

```
@Aspect
@Component
public class MyAspect {
    // 定义切入点为带有自定义注解的方法
    @Pointcut("@annotation(com.example.demo.MyCustomAnnotation)")
    public void customAnnotatedMethods() {}

    @Around("customAnnotatedMethods()")
    public Object aroundCustomAnnotatedMethod(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Before executing method with custom annotation.");
        Object result = pjp.proceed();
        System.out.println("After executing method with custom annotation.");
        return result;
    }
}

```

- 在服务类中使用自定义注解

```
public interface HelloService {
    void sayHello();

    @MyCustomAnnotation
    void customAnnotatedMethod();
}


@Service
public class HelloServiceImpl implements HelloService {
    @Override
    public void sayHello() {
        System.out.println("Hello World!");
    }

    @Override
    public void customAnnotatedMethod() {
        System.out.println("This is a custom annotated method.");
    }
}

```

#### AOP 的实现原理

### Spring Bean 和依赖注入

SpringBean 是用于声明一个被 IOC 管理的对象，这些对象可以是任何 java 类的实例。

#### @Bean 注解的使用

- @Bean 注解作用在方法上，产生一个 Bean 对象，然后这个 Bean 对象交给 Spring 管理，剩下的你就不用管了。产生这个 Bean 对象的方法 Spring 只会调用一次，随后这个 Spring 将会将这个 Bean 对象放在自己的 IOC 容器中。

- @Bean 方法名与返回类名一致，首字母小写。

- @Component、@Repository、@Controller、@Service 这些注解只局限于自己编写的类，而 @Bean 注解能把第三方库中的类实例加入 IOC 容器中并交给 Spring 管理。
- @Bean 一般和 @Component 或者 @Configuration 一起使用

#### Bean 的生命周期

Bean 的生命周期概括就是四个阶段：

- 实例化

- 属性赋值

- 初始化

- 销毁

### SpringBoot 的启动流程

1，创建 SpringApplication 对象

2，进入 run()方法

```
 public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
```

3，run 方法会 new 一个 SpringApplication 对象并完成一些赋值操作, 例如设置一些横幅信息，自动类型转换的开启，上下文工厂，初始化器，监听器等等。

4，调用 SpringApplication 对象的 run 方法，xxxx，最后返回 ConfigurableApplicationContext 对象。

SpringApplication 对象的 run 方法主要做了几件事：

- 启动准备：

  - Startup startup = SpringApplication.Startup.create();：创建一个用于跟踪启动过程的 Startup 对象。

  - 如果设置了注册关闭钩子（this.registerShutdownHook 为 true），则通过 shutdownHook.enableShutdownHookAddition();启用关闭钩子的添加。关闭钩子在应用程序退出时执行一些清理操作。

  - DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();：创建一个默认的引导上下文，用于在应用启动的早期阶段初始化关键组件。

- 获取监听器并触发启动事件：

  - SpringApplicationRunListeners listeners = this.getRunListeners(args);：获取启动监听器列表。

  - listeners.starting(bootstrapContext, this.mainApplicationClass);：触发启动事件，通知监听器应用即将开始启动。

- 准备应用环境；

  - ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);：创建应用程序参数对象，用于处理命令行参数。

  - ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);：准备应用程序的环境，包括加载系统属性、环境变量和配置文件中的属性等。监听器可以参与环境的准备过程。

- 打印启动横幅

  - Banner printedBanner = this.printBanner(environment);：如果配置了启动横幅显示，则打印启动横幅。

- 创建应用上下文

  - context = this.createApplicationContext();：根据应用类型创建应用上下文，可能是 AnnotationConfigServletWebServerApplicationContext（Web 应用）或 AnnotationConfigApplicationContext（非 Web 应用）。

  - context.setApplicationStartup(this.applicationStartup);：设置应用程序的启动方式。

- 准备和刷新上下文：

  - this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);：准备应用上下文，包括设置父上下文、加载 Bean 定义、应用环境等。监听器可以参与上下文的准备过程。

  - this.refreshContext(context);：刷新应用上下文，这是 Spring 框架启动应用程序的关键步骤，在此过程中创建和初始化 Bean、进行属性赋值和依赖注入等操作。

  - this.afterRefresh(context, applicationArguments);：在上下文刷新后执行一些额外的操作。

- 启动完成和记录启动信息： 程序启动完成，并通知监听器应用程序已经成功启动，并传递启动时间。

- 调用运行器和处理就绪事件：调用应用程序中的运行器（如果有），用于在应用启动后执行特定任务。如果应用上下文正在运行，则触发就绪事件

- 异常处理：在整个过程中，如果发生异常，则通过 throw this.handleRunFailure(context, ex, listeners);或 throw this.handleRunFailure(context, ex, (SpringApplicationRunListeners)null);处理启动失败的情况。

### SpringBoot 事务

在 springboot 中，实现事务管理有两种方式，声明式和编程式，我主要用的是声明式。

#### 声明式事务管理 @Transactional

声明式事务管理主要基于 AOP，对方法进行拦截，在方法开始或创建之前加入一个事务，在执行完目标方法后根据情况回滚或者提交。

```

@Service
public class UserService {
    @Transactional
    public void updateUserAndOrders(User user) {
        // 更新用户信息
        userRepository.update(user);
        // 更新用户相关订单信息
        orderRepository.updateOrdersForUser(user.getId());
    }
}
```

@Transactional 注解的属性介绍：

- propagation： 事务的默认传播属性，默认值为 Propagation.REQUIRED。

- isolation： 事务隔离级别，Isolation.DEFAULT 默认值。

- timeout： 事务超时时间

- readOnly： 只读事务属性

- rollbackFor 和 noRollbackFor（事务回滚规则）属性

#### 事务使用规则

- @Transactional 注解的必须是 public 的方法

- @Transactional 注解建议在方法和类上使用

- 只有来自外部的方法调用才会被 aop 捕获

- 受检异常不会导致回滚，只有 RuntimeException 才会自动回滚，也可以自定义异常回滚规则

### Springboot 上下文

SpringBoot 上下文是 Spring 框架中的核心概念，用于管理和维护应用程序中的各种 Bean，存储所有被 Spring 管理的对象，并负责对象的创建，配置，生命周期管理以及依赖注入等操作。

#### SpringBoot 上下文的类型

- AnnotationConfigApplicationContext： 用于非 Web 应用程序或者单元测试环境管理 Bean

- AnnotationConfigServletWebServerApplicationContext： 在 Springboot 启动时创建，用于管理 Bean，集成 web 服务器，用于处理 HTTP 请求。

- AnnotationConfigReactiveWebServerApplicationContext： 用于构建响应式 Web 应用程序，WebFlux。

#### 使用和获取 SpringBoot 上下文

- 在组件内部获取上下文： 可以通过实现 `ApplicationContextAware` 接口来获取 `ApplicationContext`

```
@Component
public class MyComponent implements ApplicationContextAware {
    private ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    public void doSomethingWithContext() {
        // 从上下文获取Bean
        MyService myService = applicationContext.getBean(MyService.class);
        myService.doSomething();
    }
}
```

或者可以通过在一个 Springboot 组件，如@Service ，@Controller 里通过 `@Autowired` 获取 ApplicationContext

```
@Service
public class AnotherService {
    @Autowired
    private ApplicationContext applicationContext;
    public void useContext() {
        // 获取并使用Bean
        MyBean myBean = applicationContext.getBean(MyBean.class);
        myBean.doSomething();
    }
}
```

#### SpringBoot 上下文与 IOC 的联系

IOC 是一种设计模式，将对象的创建和管理转移到容器(在 Spring 种就是 ApplicationContext)，ApplicationContext 是 IOC 容器的具体实现，负责管理应用程序中所有的 Bean，包括 Bean 的创建，索引，配置等。
