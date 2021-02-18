---
title: 'pringCloud-Eureka'
date: '2019-02-05 12:22:38'
---
# SpringCloud-Eureka

## 版本信息
Boot版本：2.0.7.RELEASE，Cloud版本：Finchley.SR2
<!--more-->
## 报错处理
1. [ClassNotFoundException for 'com.google.common.collect.Lists'](https://stackoverflow.com/questions/49903191/classnotfoundexception-for-com-google-common-collect-lists)
1.  java.lang.NoClassDefFoundError: com/google/common/base/Function

诸如此类报错的，需要加入一下依赖
```groovy
implementation group: 'com.google.guava', name: 'guava', version: '27.0-jre'
```

## 服务端
### 一、依赖配置

```groovy
dependencies {
    implementation "org.springframework.cloud:spring-cloud-starter-eureka-server"//2.0以下
 	  implementation "org.springframework.cloud:spring-cloud-starter-netflix-eureka-server"//2.0以上
}
```


### 二、yml配置

```yaml
eureka: 
  instance:
    hostname: eureka16001.com #eureka服务端的实例名称
  client: 
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url: 
      #单机 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

## 客服端
### 一、依赖配置

```groovy
implementation 'org.springframework.cloud:spring-cloud-starter-config'
implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
implementation 'org.springframework.boot:spring-boot-starter-actuator' //可选，监控中心
```
### 二、yml配置

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:16001/eureka  //服务中心地址
  instance:
  	instance-id: cloud_prover1_14001 #服务别名
  	prefer-ip-address: true #访问路劲显示ip
info:	#服务info接口信息 以下字段可以任意的定义
  app.name: cloud_provider1
  company.name: www.songzhikuan.com
  group: $ group $
  version: $version$
```
### 三、服务发现
关于@EnableEurekaClient 与 @EnableDiscoveryClien的[详细解释](https://blog.csdn.net/u012734441/article/details/78256256?locationNum=1&fps=1)

```java
@Autowired
private DiscoveryClient client;
@GetMapping(value = "/dept/discovery")
    public Object discovery()
    {
        List<String> list = client.getServices();
        System.out.println("**********" + list);

        List<ServiceInstance> srvList = client.getInstances("cloud_provider1");
        for (ServiceInstance element : srvList) {
            System.out.println(element.getServiceId() + "\t" + element.getHost() + "\t" + element.getPort() + "\t"
                    + element.getUri());
        }
        return this.client;
    }
```


## 自我保护机制
某一时刻某个微服务不可用了，Eureka不会立即清理，依旧会对该服务的信息进行保存。默认情况下，如果Eureka在一定的时间内没有接收到某个微服务实例的心跳，EurekaServer会注销该实例（默认为90秒）。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身是健康的，此时不该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题。当EurekaServer节点在短时间内丢失过多的客户端时（可能发生网络分区故障），这个节点就会进入自我保护模式，EurekaServer会保护服务注册表中的信息，不在删除服务注册表中的数据（也就是不会注销任何的微服务），当网络故障恢复后，该EurekaServer节点会自动退出自我保护模式。<br />在自我保护模式中，EurekaServer会保护服务注册表中的信息，不会注销任何的服务实例，当它收到的心跳数重新恢复到阈值以上时，该EurekaServer节点自动退出自我保护模式，他的设计哲学就是宁可保留错误的服务注册信息，也不盲目的注销可能健康的服务实例。
## 
## Eureka集群
1. 单机集群测试（不想开虚拟机），修改hosts文件，增加域名映射
```
127.0.0.1 localhost
127.0.0.1 cloud1.com
127.0.0.1 cloud2.com
127.0.0.1 cloud3.com

```
1. 服务端yml
```yaml
eureka:
  instance:
    hostname: cloud3.com
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
#      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/       #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址（单机）。
      defaultZone: http://cloud1.com:16001/eureka/,http://cloud2.com:16002/eureka/
```

1. 客服端yml
```yaml
eureka:
  client:
    service-url:
#      defaultZone: http://localhost:16001/eureka  #服务中心注册地址 单机版
      defaultZone: http://cloud1:16001/eureka, http://cloud2:16002/eureka, http://cloud3:16003/eureka,  #服务中心注册地址
  instance:
    instance-id: ${spring.application.name}_14301 #服务别名
    prefer-ip-address: true #显示ip
info: #服务info接口信息
  app.name: ${spring.application.name}
  company.name: www.songzhikuan.com
  group: ${ group }
  version: ${version}
```



