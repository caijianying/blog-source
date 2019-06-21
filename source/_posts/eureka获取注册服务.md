---
title: eureka获取注册服务
date: 2018-07-22 18:47:55
tags: 开发笔记
categories:
         - springcloud
         - java
         - 分布式框架
---
# eureka获取注册服务

## eureka client获取注册服务

首先在启动类中增加```@EnableDiscoveryClient```

其次在spring 项目中注入
```	
@Autowired
	private DiscoveryClient discoveryClient;
```

然后通过以下方式进行获取
```
List<ServiceInstance> list = discoveryClient.getInstances("analyze");
```
analyze为注册服务名



## eureka server获取注册服务
1. 方法1

首先在配置文件中增加
```
eureka.client.fetch-registry=true
```

其次在spring 项目中注入
```	
@Autowired
	private DiscoveryClient discoveryClient;
```

然后通过以下方式进行获取
```
List<ServiceInstance> list = discoveryClient.getInstances("analyze");
```
analyze为注册服务名

这种方法有一定的延迟，原理和Client一样，如果需要及时更新那么需要配置一下其他参数，及时更新注册信息.

server
```
#清理时间间隔
eureka.server.eviction-interval-timer-in-ms=10000
#关闭自我保护模式。自我保护模式是指，出现网络分区、eureka在短时间内丢失过多客户端时，会进入自我保护模式。
#自我保护：一个服务长时间没有发送心跳包，eureka也不会将其删除，默认为true。
eureka.server.enable-self-preservation=false
```
client
```
#发送时间间隔
eureka.instance.lease-renewal-interval-in-seconds=10
#多长时间过期
eureka.instance.lease-expiration-duration-in-seconds=30
```
2. 方法2

如果需要在server中获取，强烈建议使用这种方式，优点就是和dashboard状态保持一致。
```
PeerAwareInstanceRegistry registry = EurekaServerContextHolder.getInstance().getServerContext().getRegistry();
    Applications applications = registry.getApplications();

    applications.getRegisteredApplications().forEach((registeredApplication) -> {
        registeredApplication.getInstances().forEach((instance) -> {
            System.out.println(instance.getAppName() + " (" + instance.getInstanceId() + ") : " + response);
        });
    });
```

## 参考
1. [Spring Cloud中文文档](https://springcloud.cc/spring-cloud-dalston.html#netflix-eureka-server-starter)
2. [Spring Cloud doc](https://cloud.spring.io/spring-cloud-static/Finchley.RELEASE/single/spring-cloud.html)
3. [Eureka Server - list all registered instances](https://stackoverflow.com/questions/42427492/eureka-server-list-all-registered-instances)