---
title: 'pringCloud-Feign'
date: '2019-02-05 12:22:38'
---
# SpringCloud-Feign

## 公共模块
### 依赖

```groovy
implementation "org.springframework.cloud:spring-cloud-starter-openfeign"
```
<!--more-->
### 接口——DeptService
```java
package com.zhikuan.service;

import com.zhikuan.entities.Dept;
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
@FeignClient(value = "CLOUD-PROVIDER1")
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

## 客服端
### 依赖
```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-ribbon'
implementation "org.springframework.cloud:spring-cloud-starter-openfeign"
```
### controller
```java
package com.zhikuan.cloud.controller;

import com.zhikuan.service.DeptService;
import com.zhikuan.entities.Dept;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;

/**
 * @author Songzk
 * @date 2019/1/24
 */
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;
    @PostMapping("/consumer/dept/add")
    public boolean add(Dept dept){
        return deptService.addDept(dept);
    }

    @GetMapping("/consumer/dept/{id}")
    public Dept getDept(@PathVariable("id")Long id) {
        return deptService.getDept(id);
    }

    @GetMapping("/consumer/depts")
    public List<Dept> getDepts() {
        return deptService.getDepts();
    }
}

```
### 启动类

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableEurekaClient
@EnableFeignClients(basePackages= {"com.zhikuan.service"})//公共服务接口包
@ComponentScan("com.zhikuan.cloud")
public class SpringApplication15002 {
    public static void main(String[] args) {
            SpringApplication.run(SpringApplication15002.class, args);
    }
}
```

