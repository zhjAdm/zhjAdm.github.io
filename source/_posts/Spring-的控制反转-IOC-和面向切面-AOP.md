---
title: Spring 的控制反转(IoC)和面向切面(AOP)
tags:
  - Java
  - Spring
categories: Java
description: Spring 框架核心，控制反转和面向切面编程的理解
abbrlink: 21589
date: 2024-03-16 14:38:28
---

# 控制反转（IoC）

### 是什么

> 控制反转本身是一种设计思想，非技术。是将对象交给容器控制，非传统通过在对象内部直接控制。传统的开发方式，我们通过new的方式创建一个对象，程序主动去创建并建立依赖。控制反转是将对象的创建交给容器，并让容器管理对象的整个生命周期，容器实现依赖的注入。相对与传统开发的主动创建对象建立依赖关系，通过容器创建管理对象，注入依赖所以叫作控制反转。

### 有什么作用

> 因为有原来在类内部自己创建对象，导致类与类之间存在很强的耦合关系。现在将控制权交给容器后，所以类与类之间变成了松耦合，有一利于代码功能的复用。

### Ioc 和 DI

> IoC是设计思想，DI（依赖注入）是实现方式

### IoC 配置方式

- xml 配置方式

  将bean对象的信息配置在xml文件中，Spring 加载配置文件根据配置的信息创建bean。但是配置方式繁琐，不利于维护。在 Spring boot 中该方式已经被抛弃。

  ~~~xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!--suppress ALL -->
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
  		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
  	">
      <!-- 使用无参构造器来创建对象。id属性：要求唯一 。class属性：要写类的完整的名称。 -->
      <bean id="a1" class="first.Apple"/>
      <bean id="date1" class="java.util.Date"/>
      <!-- 使用静态工厂方法来创建对象。 factory-method属性：用来指定静态方法名。注：Spring容器会调用该类的静态方法来创建一个对象。 -->
      <bean id="cal1" class="java.util.Calendar" factory-method="getInstance"/>
      <!-- 使用实例工厂方法来创建对象： factory-bean属性：指定要调用的bean的id, factory-method属性：指定要调用的实例方法。 注：Spring容器会调用该bean的实例方法来创建对象。 在Spring框架里面，所谓的bean指的是由Spring容器管理的对象。-->
      <bean id="date2" factory-bean="cal1" factory-method="getTime"/>
  </beans>
  ~~~

  

- Java 配置方式

  通过Java的配置类实现bean的创建，本质的xml的方式相同不过是将配置信息转移到了Java配置类中。同样存在配置繁琐，存在大量配置的话不利于威化，可读变差。但是第三方资源还是需要通过这种或者xml的方式。
  
  1. 创建一个配置类 ，添加 **@Configuration** 注解声明为配置类。
  2. 方法上添加 **@Bean** 注解。方法内创建一个对象并返回。返回的该对象就会被IoC容器所管理。
  
  ~~~java
  @Configuration
  public class BeansConfig {
  
      /**
       * @return user dao
       */
      @Bean("userDao")
      public UserDaoImpl userDao() {
          return new UserDaoImpl();
      }
  
      /**
       * @return user service
       */
      @Bean("userService")
      public UserServiceImpl userService() {
          UserServiceImpl userService = new UserServiceImpl();
          userService.setUserDao(userDao());
          return userService;
      }
  }
  ~~~

- 注解方式

  Spring 会扫描 @Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器。使用起来方便快捷、便于维护，但第三方资源无法添加。

### 依赖注入方式

> 常用的注入方式主要有三种：构造方法注入（Construct注入），setter注入，基于注解的注入（接口注入）

# 面向切面编程 (AOP)

### 是什么

> 和 IoC 同样是一种设计思想。理解为将不同业务模块相同的代进行抽离封装，从而降低模块之间的耦合度。Spring 中通过代理实现 AOP

### 有什么作用

> 可以通过AOP 实现模块之间的解耦，例如：系统中常见的日志记录功能，可以将各个模块中相同的代码抽取成独立的模块，利用   AOP 实现解耦。
