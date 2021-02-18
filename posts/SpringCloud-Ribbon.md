---
title: 'pringCloud-Ribbon'
date: '2019-02-05 12:22:38'
---
# SpringCloud-Ribbon

SpringCloud Ribbon 是基于Netflix Ribbon实现的一套客户端均衡负载的工具。<br />本次实验基于Ribbon+RestTemplate
<!--more-->
## 报错异常
1. java.lang.IllegalStateException: Request URI does not contain a valid hostname: http://CLOUD_PROVIDER1:14001/depts（服务名不能有下划线）

## 客户端依赖

```groovy
    implementation 'org.springframework.cloud:spring-cloud-starter-eureka'
    implementation 'org.springframework.cloud:spring-cloud-starter-ribbon'
    implementation 'org.springframework.cloud:spring-cloud-starter-config'
```
## yml配置

```yaml
eureka:
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    service-url:
      defaultZone: http://cloud1.com:16001/eureka/,http://cloud2.com:16002/eureka/,http://cloud3.com:16003/eureka/

```

## RestTemplate

```java
@Configuration
public class ConfigBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```
## 启动类

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableEurekaClient
public class SpringApplication15001 {
    public static void main(String[] args) {
            SpringApplication.run(SpringApplication15001.class, args);
    }
}
```

## IRule组件
七大算法
1. RoundRobinRule
1. RandomRule
1. AvailabilityFilteringRule
1. WeightedResponseTimeRule
1. RetryRule
1. BestAvailableRule
1. ZoneAvoidanceRule
### 切换默认算法

```java
  @Bean
    public IRule definedRule(){
//        return new RandomRule();
        return new RetryRule();
    }
```

### 自定义算法
参考 [https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/RandomRule.java](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/RandomRule.java)<br />轮询，要求每个服务调用5次后在轮询
#### 启动类配置
自定义规则类不能与主启动类同包
```java
@RibbonClient(name = "CLOUD-PROVIDER1",configuration = Rule.class)//服务名，自定义规则类
```
#### 自定义规则类——Rule

```java
@Configuration
public class Rule {
    @Bean
    public IRule getRule(){
        return new RoundRobinRule_ZK();
    }
}
```

#### 自定义算法类——RoundRobinRule_ZK
```java
package com.zhikuan.ribbon;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

/**
 * @author Songzk
 * @date 2019/2/2
 */
public class RoundRobinRule_ZK extends AbstractLoadBalancerRule {
    private static final int NUM = 5;
    private int totals = 0;
    private int currentIndex = 0;
    /**
     * Randomly choose from all living servers
     */
//    @edu.umd.cs.findbugs.annotations.SuppressWarnings(value = "RCN_REDUNDANT_NULLCHECK_OF_NULL_VALUE")
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            int index = chooseRandomInt(serverCount);
            server = upList.get(index);

            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }

            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }

        return server;

    }

    protected int chooseRandomInt(int serverCount) {
//        return ThreadLocalRandom.current().nextInt(serverCount);//原始
        return getCurrentIndex(serverCount);//自定义
    }
    private int getCurrentIndex(int serverCount){
        if(totals<RandomRule_ZK.NUM){
            totals ++;
        }else{
            totals = 0;
            if(currentIndex>=serverCount-1){
                currentIndex = 0;
            }else {
                currentIndex ++;
            }
        }
        return currentIndex;
    }
    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // TODO Auto-generated method stub

    }
}

```

