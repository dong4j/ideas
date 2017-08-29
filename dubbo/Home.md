Dubbo
---

`Dubbo`是一个分布式服务框架，致力于提供高性能和透明化的`RPC`远程服务调用方案，是阿里巴巴`SOA`服务化治理方案的核心框架，每天为**_2,000+_**个服务提供**_3,000,000,000+_**次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点。

### 概述

`Dubbo`是阿里巴巴SOA服务化治理方案的核心框架，每天为`2,000+`个服务提供`3,000,000,000+`次访问量支持，并被广泛应用于阿里巴巴集团的各成员站点：

[[images/alibaba.png]] [[images/1688.png]] [[images/aliexpress.png]] [[images/aliyun.png]] [[images/aliloan.png]] [[images/alibado.png]] [[images/lp.png]] [[images/laiwang.png]]

自开源后，已有不少非阿里系公司在使用`Dubbo`，参见：已知用户

#### 那么，`Dubbo`是什么？

`Dubbo` |ˈdʌbəʊ| 是一个分布式服务框架，致力于提供高性能和透明化的`RPC`远程服务调用方案，以及`SOA`服务治理方案。

其核心部分包含:

* 远程通讯: 提供对多种基于长连接的`NIO`框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
* 集群容错: 提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
* 自动发现: 基于注册中心目录服务，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。

#### `Dubbo`能做什么？

* 透明化的远程方法调用，就像调用本地方法一样调用远程方法，只需简单配置，没有任何`API`侵入。
* 软负载均衡及容错机制，可在内网替代`F5`等硬件负载均衡器，降低成本，减少单点。
* 服务自动注册与发现，不再需要写死服务提供方地址，注册中心基于接口名查询服务提供者的`IP`地址，并且能够平滑添加或删除服务提供者。

### 快速启动

> `Dubbo`采用全`Spring`配置方式，透明化接入应用，对应用没有任何`API`侵入，只需用`Spring`加载`Dubbo`的配置即可，`Dubbo`基于`Spring`的`Schema`扩展进行加载。
> 
> 如果不想使用`Spring`配置，而希望通过`API`的方式进行调用（不推荐），请参见：[API配置](user-guide-api-ref#配置api)

#### 服务提供者

> 完整安装步骤，请参见：[示例提供者安装](admin-guide-install-manual#示例提供者安装)

定义服务接口: (该接口需单独打包，在服务提供方和消费方共享)

> DemoService.java

```java
package com.alibaba.dubbo.demo;
 
public interface DemoService {
 
    String sayHello(String name);
 
}
```

在服务提供方实现接口：(对服务消费方隐藏实现)

> DemoServiceImpl.java

```java
package com.alibaba.dubbo.demo.provider;
 
import com.alibaba.dubbo.demo.DemoService;
 
public class DemoServiceImpl implements DemoService {
 
    public String sayHello(String name) {
        return "Hello " + name;
    }
 
}
```

用Spring配置声明暴露服务：

> provider.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="hello-world-app"  />
 
    <!-- 使用multicast广播注册中心暴露服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20880" />
 
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" />
 
    <!-- 和本地bean一样实现服务 -->
    <bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl" />
 
</beans>
```

加载`Spring`配置：

> Provider.java

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Provider {
 
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://dubbo.io/provider.xml"});
        context.start();
 
        System.in.read(); // 按任意键退出
    }
 
}
```

#### 服务消费者

> 完整安装步骤，请参见：[示例消费者安装](admin-guide-install-manual#示例消费者安装)

通过`Spring`配置引用远程服务：

> consumer.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" />
 
</beans>
```

加载`Spring`配置，并调用远程服务：(也可以使用`IoC`注入)

> Consumer.java

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;
import com.alibaba.dubbo.demo.DemoService;
 
public class Consumer {
 
    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://dubbo.io/consumer.xml"});
        context.start();
 
        DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
        String hello = demoService.sayHello("world"); // 执行远程方法
 
        System.out.println( hello ); // 显示调用结果
    }
 
}
```

想了解更多？阅读一下《[用户指南](user-guide-intro)》吧~~　