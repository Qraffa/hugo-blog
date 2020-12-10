---
title: Spring
author: John Doe
date: 2020-03-17 01:14:14
tags:
- spring
categories: 
- spring
---

## Spring常见注入方式

1. 设值注入
   通过调用set方法注入
2. 构造注入
   构造方法注入

## Bean作用域

1. singleton
   单例模式,一个context中只有一个对象实例
2. prototype
   多例模式,每次调用getbean方法时会再次创建一个实例

<!--more-->

## Bean生命周期

1. init-method和destroy-method,可以设置bean的初始化方法和销毁方法
2. default-init-method和default-destroy-method,在全局配置默认的初始化方法和销毁方法

- InitializingBean接口为bean提供了初始化方法的方式
- DisposableBean接口为bean提供了销毁方法
- 当同时设置了**全局配置default-init-method**和**bean初始化方法init-method**以及**实现了InitializingBean接口**时,全局配置的方法不再执行,并且**InitializingBean接口**的初始化方法执行顺序先与**bean初始化方法init-method**

## Bean装配方式

- autowire实现自动装配
  - autowire中的byName方式是通过bean中set方法名称,去掉set再将首字母小写,再根据这个名称从bean中查找然后注入

## Bean注解方式

- `<context:component-scan>`
- 默认情况下,类被自动发现并被加载bean的条件:使用`@Component`,`@Repository`,`@Service`,`@Controller`进行注解
- 注解方式的命名策略:
    1. 默认使用把首字母小写的类名
    2. 通过指定注解的name属性
    3. 可以自定义默认命名策略
- `@Scope`注解用于制定bean的作用域
- `@Autowired`注解可能会无法找到合适的bean而抛出异常,使用`@Autowired(required=false)`可避免
- 如果一个属性是必要的,建议使用`@Required`注解
- `@Order`注解可用于装配时的排序(Spirng4以上)
- `@Qualifier`注解,对于多个bean,可以使用`@Qualifier`指定需要注解的bean,可用于函数参数中
- `@Bean`注解,作用于方法,动态产生一个bean对象,交由Spring管理,需要在`@Configuration`注解下使用.默认使用方法名作为bean名称,可使用`name`属性指定名称
- `@Resource`注解
- `@PostConstruct`注解,注解在方法上,在对象完成加载注入后执行该方法
- `@PreDestroy`注解,在bean销毁前执行该方法

## Spring AOP

- 带参数的around通知要求参数名称一致
- 使用注解:
  - `@Pointcut`指定切入点
  - `@AfterReturning`可以用`returning`属性来获取返回值
  - `@AfterThrowing`可以用`throwing`属性来获取异常,但只能匹配相同类型的异常
