---
title: 'pringCloud-hystrix'
date: '2019-02-05 12:22:38'
---
# SpringCloud-hystrix

## 异常
No fallbackFactory instance of type class feign<br />分析：1、实现了接口_FallbackFactory的_实现类没有加@Component；<br />2、没有扫描到降级类，如：@ComponentScan(basePackages={"main",""降级类所在的包"})
<!--more-->
## 服务熔断
### 依赖配置
```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
```
### Controller
```java
		/**
     * 查询一条
     */
    @GetMapping(value = "dept/{id}")
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept getDept(@PathVariable("id")Long id) {
        Dept d = deptService.get(id);
        if(d==null){
            throw new RuntimeException("该ID：" + id + "没有没有对应的信息");
        }
        return d;
    }
    /**
     * 熔断-异常处理
     */
    public Dept processHystrix_Get(@PathVariable("id")Long id) {
        return new Dept().setId(id).setDname("该ID：" + id + "没有没有对应的信息,null--@HystrixCommand").setDbSource("no this database in MySQL");
    }
```

### 启动类
```java
@EnableEurekaClient
@SpringBootApplication
@EnableHystrix
public class SpringApplication14004 {
    public static void main(String[] args) {
        SpringApplication.run(SpringApplication14004.class, args);
    }
}
```
## 服务降级（provider已关闭）
在controller中指定熔断handler，增加了代码的耦合度，可以在service统一指定实现了FallbackFactory接口的熔断处理类。
### service指定

```java
package com.zhikuan.service;

import com.zhikuan.entities.Dept;
import com.zhikuan.service.hystrix.DeptServiceHandler;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

import java.util.List;

/**
 * @author Songzk
 * @date 2019/2/2
 */
@FeignClient(value = "CLOUD-PROVIDER1",fallbackFactory = DeptServiceHandler.class)
public interface DeptService {
    /**
     * 新增
     */
    @PostMapping(value = "dept")
    boolean addDept(@RequestBody Dept dept);
    /**
     * 查询一条
     */
    @GetMapping(value = "dept/{id}")
     Dept getDept(@PathVariable("id")Long id) ;
    /**
     * 查询全部
     */
    @GetMapping(value = "depts")
     List<Dept> getDepts();

}

```

### 实现类DeptServiceHandler
```java
package com.zhikuan.service.hystrix;

import com.zhikuan.entities.Dept;
import com.zhikuan.service.DeptService;
import feign.hystrix.FallbackFactory;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * @author Songzk
 * @date 2019/2/3
 */
@Component
public class DeptServiceHandler implements FallbackFactory<DeptService> {
    @Override
    public DeptService create(Throwable cause) {
        return new DeptService() {
            @Override
            public boolean addDept(Dept dept) {
                return false;
            }

            @Override
            public Dept getDept(Long id) {
                return new Dept().setId(id).setDname("改ID："+id+" 没有对应的记录,hystrix DeptServiceHandler");
            }

            @Override
            public List<Dept> getDepts() {
                return null;
            }
        };
    }
}

```

### 客户端yml
```groovy

spring:
  profiles:
    active: dev
  application:
    name: cloud_consumer_dept_feign
feign:
  hystrix:
    enabled: true
eureka:
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    service-url:
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://cloud1.com:16001/eureka/,http://cloud2.com:16002/eureka/,http://cloud3.com:16003/eureka/

```

## HystrixDashBoard
### provider要配置actuator
```yaml
management:
  endpoints:
    web:
      exposure:
        exclude: "*"
  server:
    port: 14104
```

### dashboard依赖
```groovy
 implementation 'org.springframework.boot:spring-boot-starter'
 implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix'
 implementation 'org.springframework.cloud:spring-cloud-starter-netflix-hystrix-dashboard'
```

### 启动类配置
@EnableHystrixDashboard
