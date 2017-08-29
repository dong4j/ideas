[**HOME**](Home) > **用户指南**

### 配置

#### XML配置

* 配置项说明: 详细配置项，请参见：[配置参考手册](user-guide-configuration-ref)
* API使用说明: 如果不想使用Spring配置，而希望通过API的方式进行调用，请参见：[API配置](user-guide-configuration#api配置)
* 配置使用说明: 想知道如何使用配置，请参见：[快速启动](user-guide-quick-start)

示例：

**provider.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="hello-world-app"  />
 
    <dubbo:registry address="multicast://224.5.6.7:1234" />
 
    <dubbo:protocol name="dubbo" port="20880" />
 
    <dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoServiceLocal" />
 
    <dubbo:reference id="demoServiceRemote" interface="com.alibaba.dubbo.demo.DemoService" />
 
</beans>
```

所有标签都支持自定义参数，用于不同扩展点实现的特殊配置。

如：

```xml
<dubbo:protocol name="jms">
    <dubbo:parameter key="queue" value="http://10.20.160.198/wiki/display/dubbo/10.20.31.22" />
</dubbo:protocol>
```

或：

> 2.1.0开始支持
> 注意声明：xmlns:p="http://www.springframework.org/schema/p"

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:protocol name="jms" p:queue="http://10.20.160.198/wiki/display/dubbo/10.20.31.22" />
</beans>
```

##### 配置关系

[[/user-guide/images/dubbo-config.jpg]]

* `<dubbo:service/>` 服务配置，用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心。
* `<dubbo:reference/>` 引用配置，用于创建一个远程服务代理，一个引用可以指向多个注册中心。
* `<dubbo:protocol/>` 协议配置，用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受。
* `<dubbo:application/>` 应用配置，用于配置当前应用信息，不管该应用是提供者还是消费者。
* `<dubbo:module/>` 模块配置，用于配置当前模块信息，可选。
* `<dubbo:registry/>` 注册中心配置，用于配置连接注册中心相关信息。
* `<dubbo:monitor/>` 监控中心配置，用于配置连接监控中心相关信息，可选。
* `<dubbo:provider/>` 提供方的缺省值，当ProtocolConfig和ServiceConfig某属性没有配置时，采用此缺省值，可选。
* `<dubbo:consumer/>` 消费方缺省配置，当ReferenceConfig某属性没有配置时，采用此缺省值，可选。
* `<dubbo:method/>` 方法配置，用于ServiceConfig和ReferenceConfig指定方法级的配置信息。
* `<dubbo:argument/>` 用于指定方法参数配置。

##### 配置覆盖

[[/user-guide/images/dubbo-config-override.jpg]]

* 上图中以timeout为例，显示了配置的查找顺序，其它retries, loadbalance, actives等类似。
    * 方法级优先，接口级次之，全局配置再次之。
    * 如果级别一样，则消费方优先，提供方次之。
* 其中，服务提供方配置，通过URL经由注册中心传递给消费方。
* 建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置。
* 理论上ReferenceConfig的非服务标识配置，在ConsumerConfig，ServiceConfig, ProviderConfig均可以缺省配置。

#### 属性配置

* 如果公共配置很简单，没有多注册中心，多协议等情况，或者想多个Spring容器想共享配置，可以使用dubbo.properties作为缺省配置。
* Dubbo将自动加载classpath根目录下的dubbo.properties，可以通过JVM启动参数：-Ddubbo.properties.file=xxx.properties 改变缺省配置位置。
* 如果classpath根目录下存在多个dubbo.properties，比如多个jar包中有dubbo.properties，Dubbo会任意加载，并打印Error日志，后续可能改为抛异常。

##### 映射规则
    
* 将XML配置的标签名，加属性名，用点分隔，多个属性拆成多行：
    * 比如：dubbo.application.name=foo等价于`<dubbo:application name="foo" />`
    * 比如：dubbo.registry.address=10.20.153.10:9090等价于`<dubbo:registry address="10.20.153.10:9090" />`
* 如果XML有多行同名标签配置，可用id号区分，如果没有id号将对所有同名标签生效：
    * 比如：dubbo.protocol.rmi.port=1234等价于`<dubbo:protocol id="rmi" name="rmi" port="1099" />` (协议的id没配时，缺省使用协议名作为id)
    * 比如：dubbo.registry.china.address=10.20.153.10:9090等价于`<dubbo:registry id="china" address="10.20.153.10:9090" />`
    
典型配置如：

**dubbo.properties**

```sh
dubbo.application.name=foo
dubbo.application.owner=bar
dubbo.registry.address=10.20.153.10:9090
```

##### 覆盖策略

[[/user-guide/images/dubbo-properties-override.jpg]]

* JVM启动-D参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。
* XML次之，如果在XML中有配置，则dubbo.properties中的相应配置项无效。
* Properties最后，相当于缺省值，只有XML没有配置时，dubbo.properties的相应配置项才会生效，通常用于共享公共配置，比如应用名。

#### 注解配置

> 2.2.1 以上版本支持

##### 服务提供方注解

```java
import com.alibaba.dubbo.config.annotation.Service;
 
@Service(version="1.0.0")
public class FooServiceImpl implements FooService {
 
    // ......
 
}
```

##### 服务提供方配置

```xml
<!-- 公共信息，也可以用dubbo.properties配置 -->
<dubbo:application name="annotation-provider" />
<dubbo:registry address="127.0.0.1:4548" />
 
<!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
<dubbo:annotation package="com.foo.bar.service" />
```

##### 服务消费方注解

```java
import com.alibaba.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Component;
 
@Component
public class BarAction {
 
    @Reference(version="1.0.0")
    private FooService fooService;
 
}
```

##### 服务消费方配置

```xml
<!-- 公共信息，也可以用dubbo.properties配置 -->
<dubbo:application name="annotation-consumer" />
<dubbo:registry address="127.0.0.1:4548" />
 
<!-- 扫描注解包路径，多个包用逗号分隔，不填pacakge表示扫描当前ApplicationContext中所有的类 -->
<dubbo:annotation package="com.foo.bar.action" />
```

也可以使用：(等价于前面的：`<dubbo:annotation package="com.foo.bar.service" />`)

```xml
<dubbo:annotation />
<context:component-scan base-package="com.foo.bar.service">
    <context:include-filter type="annotation" expression="com.alibaba.dubbo.config.annotation.Service" />
</context:component-scan>
```

> Spring2.5及以后版本支持component-scan，如果用的是Spring2.0及以前版本，需配置：

> ```xml
<!-- Spring2.0支持@Service注解配置，但不支持package属性自动加载bean的实例，需人工定义bean的实例。-->
<dubbo:annotation />
<bean id="barService" class="com.foo.BarServiceImpl" />
```

#### API配置

**API使用范围**

API仅用于OpenAPI, ESB, Test, Mock等系统集成，普通服务提供方或消费方，请采用配置方式使用Dubbo，请参见：[XML配置](user-guide-configuration#xml配置)

**API属性含义参考**

API属性与配置项一对一，各属性含义，请参见：[配置参考手册](user-guide-configuration-ref)，
比如：`ApplicationConfig.setName("xxx")` 对应 `<dubbo:application name="xxx" />`

##### (1) 服务提供者：

```java
import com.alibaba.dubbo.rpc.config.ApplicationConfig;
import com.alibaba.dubbo.rpc.config.RegistryConfig;
import com.alibaba.dubbo.rpc.config.ProviderConfig;
import com.alibaba.dubbo.rpc.config.ServiceConfig;
import com.xxx.XxxService;
import com.xxx.XxxServiceImpl;
 
// 服务实现
XxxService xxxService = new XxxServiceImpl();
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("xxx");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 服务提供者协议配置
ProtocolConfig protocol = new ProtocolConfig();
protocol.setName("dubbo");
protocol.setPort(12345);
protocol.setThreads(200);
 
// 注意：ServiceConfig为重对象，内部封装了与注册中心的连接，以及开启服务端口
 
// 服务提供者暴露服务配置
ServiceConfig<XxxService> service = new ServiceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接，请自行缓存，否则可能造成内存和连接泄漏
service.setApplication(application);
service.setRegistry(registry); // 多个注册中心可以用setRegistries()
service.setProtocol(protocol); // 多个协议可以用setProtocols()
service.setInterface(XxxService.class);
service.setRef(xxxService);
service.setVersion("1.0.0");
 
// 暴露及注册服务
service.export();
```

##### (2) 服务消费者：

```java
import com.alibaba.dubbo.rpc.config.ApplicationConfig;
import com.alibaba.dubbo.rpc.config.RegistryConfig;
import com.alibaba.dubbo.rpc.config.ConsumerConfig;
import com.alibaba.dubbo.rpc.config.ReferenceConfig;
import com.xxx.XxxService;
 
// 当前应用配置
ApplicationConfig application = new ApplicationConfig();
application.setName("yyy");
 
// 连接注册中心配置
RegistryConfig registry = new RegistryConfig();
registry.setAddress("10.20.130.230:9090");
registry.setUsername("aaa");
registry.setPassword("bbb");
 
// 注意：ReferenceConfig为重对象，内部封装了与注册中心的连接，以及与服务提供方的连接
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
reference.setApplication(application);
reference.setRegistry(registry); // 多个注册中心可以用setRegistries()
reference.setInterface(XxxService.class);
reference.setVersion("1.0.0");
 
// 和本地bean一样使用xxxService
XxxService xxxService = reference.get(); // 注意：此代理对象内部封装了所有通讯细节，对象较重，请缓存复用
```

##### (3) 特殊场景

注：下面只列出不同的地方，其它参见上面的写法

###### (3.1) 方法级设置：

```java
...
// 方法级配置
List<MethodConfig> methods = new ArrayList<MethodConfig>();
MethodConfig method = new MethodConfig();
method.setName("createXxx");
method.setTimeout(10000);
method.setRetries(0);
methods.add(method);
 
// 引用远程服务
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
...
reference.setMethods(methods); // 设置方法级配置
...
```

###### (3.2) 点对点直连：

```java
...
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
// 如果点对点直连，可以用reference.setUrl()指定目标地址，设置url后将绕过注册中心，
// 其中，协议对应provider.setProtocol()的值，端口对应provider.setPort()的值，
// 路径对应service.setPath()的值，如果未设置path，缺省path为接口名
reference.setUrl("dubbo://10.20.130.230:20880/com.xxx.XxxService"); 
...
```