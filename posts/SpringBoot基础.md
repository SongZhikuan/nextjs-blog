---
title: 'SpringBoot基础'
date: '2019-02-05 12:22:38'
---
# SpringBoot基础

# SpringBoot简介
## 版本要求    

本次学习SpringBoot2.0版本，jdk1.8以上
<!--more-->
## SpringBoot和SpringMVC、SpringCloud区别
SpringBoot
是一个快速开发的框架,能够快速的整合第三方框架，简化XML配置，全部采用注解形式，内置Tomcat容器,帮助开发者能够实现快速开发，SpringBoot的Web组件 默认集成的是SpringMVC框架。<br />SpringMVC是控制层<br />SpringCloud依赖与SpringBoot组件，使用SpringMVC编写Http协议接口，同时SpringCloud是一套完整的微服务解决框架

<br />
# 注解
## @SpringBootApplication
@SpringBootApplication 被 @Configuration、@EnableAutoConfiguration、@ComponentScan 注解所修饰，
## @Configuration
@Configuration用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。

```java
public class AppConfig {

    @Bean
    public MyBean myBean() {
        // instantiate, configure and return bean ...
    }
}
```
## @ComponentScan
ComponentScan 用于扫包
## @EnableAutoConfiguration
EnableAutoConfiguration自动装配
## @RestController
@RestController返回JSON对象
## @responseBody
@responseBody注解的作用是将controller的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到response对象的body区，通常用来返回JSON数据或者是XML<br />　　数据，需要注意的呢，在使用此注解之后不会再走试图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。
# 静态资源访问
Spring Boot默认提供静态资源目录位置需置于classpath下，目录名需符合如下规则：<br />/static<br />/public<br />/resources          <br />/META-INF/resources<br />举例：我们可以在src/main/resources/目录下创建static，在该位置放置一个图片文件。启动程序后，尝试访问http://localhost:8080/D.jpg。如能显示图片，配置成功
# 模板引擎
提供了默认配置的模板引擎主要有以下几种<br />·        
Thymeleaf<br />·        
FreeMarker<br />·        
Velocity<br />·        
Groovy<br />·        
Mustache<br />它们默认的模板配置路径为：src/main/resources/templates<br />freemarker的使用案例：
1. 引入相关包
```xml
<!-- 引入freeMarker的依赖包. gradle配置-->

implementation 'org.springframework.boot:spring-boot-starter-freemarker'

<!-- 引入freeMarker的依赖包. pom配置-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>

```

1. 在src/main/resources/创建一个templates文件夹,后缀为*.ftl
1. 后台代码
```java
@RequestMapping("/index")
	public String index(Map<String, Object> map) {
	    map.put("name","美丽的天使...");
	   return "index";
	}

```
1. applycation配置
```groovy
spring:
  freemarker:
    allow-request-override: false
    cache: true
    check: true
    charset: UTF-8
    content-type:text/html
    expose-request-attributes:false
    expose-session-attributes:false
    expose-spring-macro-helpers:false
    suffix:
    template-loader-path:.ftl
    template-loader-path:classpath:/templates/

```

# 全局捕获异常:

@ExceptionHandler
表示拦截异常<br />
@ControllerAdvice 是 controller 的一个辅助类，最常用的就是作为全局异常处理的切面类<br />@ControllerAdvice 可以指定扫描范围<br />
@ControllerAdvice 约定了几种可行的返回值，如果是直接返回 model 类的话，需要使用 @ResponseBody 进行 json 转换<br /> 返回 String，表示跳到某个 view<br />返回 modelAndView<br /> 返回 model +
@ResponseBody

```java
@ControllerAdvice
public class GlobalExceptionHandler {
	@ExceptionHandler(RuntimeException.class)
	@ResponseBody
	public Map<String, Object> exceptionHandler() {
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("errorCode", "101");
		map.put("errorMsg", "系統错误!");
		return map;
	}
}

```

