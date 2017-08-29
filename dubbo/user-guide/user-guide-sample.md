[**HOME**](Home) > **用户指南**

### 示例

想完整的运行起来，请参见：[快速启动](user-guide-quick-start)，这里只列出各种场景的配置方式。以下示例全部使用基于Spring的[Xml配置](user-guide-configuration#xml配置)作为参考，如果不想使用Spring，而希望通过API的方式进行调用，请参见：[API配置](user-guide-configuration#api配置)

* [启动时检查](#启动时检查)
* [集群容错](#集群容错)
* [负载均衡](#负载均衡)
* [线程模型](#线程模型)
* [直连提供者](#直连提供者)
* [只订阅](#只订阅)
* [只注册](#只注册)
* [静态服务](#静态服务)
* [多协议](#多协议)
* [多注册中心](#多注册中心)
* [服务分组](#服务分组)
* [多版本](#多版本)
* [分组聚合](#分组聚合)
* [参数验证](#参数验证)
* [结果缓存](#结果缓存)
* [泛化引用](#泛化引用)
* [泛化实现](#泛化实现)
* [回声测试](#回声测试)
* [上下文信息](#上下文信息)
* [隐式传参](#隐式传参)
* [异步调用](#异步调用)
* [本地调用](#本地调用)
* [参数回调](#参数回调)
* [事件通知](#事件通知)
* [本地存根](#本地存根)
* [本地伪装](#本地伪装)
* [延迟暴露](#延迟暴露)
* [并发控制](#并发控制)
* [Load Balance 均衡](#Load Balance 均衡)
* [连接控制](#连接控制)
* [延迟连接](#延迟连接)
* [粘滞连接](#粘滞连接)
* [令牌验证](#令牌验证)
* [路由规则](#路由规则)
* [条件路由规则](#条件路由规则)
* [脚本路由规则](#脚本路由规则)
* [配置规则](#配置规则)
* [服务降级](#服务降级)
* [优雅停机](#优雅停机)
* [主机绑定](#主机绑定)
* [日志适配](#日志适配)
* [访问日志](#访问日志)
* [服务容器](#服务容器)
* [Reference Config缓存](#Reference Config缓存)
* [分布式事务](#分布式事务)

#### 启动时检查

Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止Spring初始化完成，以便上线时，能及早发现问题，默认 `check="true"`。如果你的Spring容器是懒加载的，或者通过API编程延迟引用服务，请关闭check，否则服务临时不可用时，会抛出异常，拿到null引用，如果 `check="false"`，总是会返回引用，当服务恢复时，能自动连上。

可以通过 `check="false"` 关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

##### 关闭某个服务的启动时检查：
> 没有提供者时报错

```xml
<dubbo:reference interface="com.foo.BarService" check="false" />
```

##### 关闭所有服务的启动时检查：
> 没有提供者时报错

```xml
<dubbo:consumer check="false" />
```

##### 关闭注册中心启动时检查：
> 注册订阅失败时报错

```xml
<dubbo:registry check="false" />
```

##### 也可以用dubbo.properties配置

```sh
dubbo.reference.com.foo.BarService.check=false
dubbo.reference.check=false
dubbo.consumer.check=false
dubbo.registry.check=false
```

##### 也可以用-D参数

```sh
java -Ddubbo.reference.com.foo.BarService.check=false
java -Ddubbo.reference.check=false
java -Ddubbo.consumer.check=false 
java -Ddubbo.registry.check=false
```

**注意区别**

* dubbo.reference.check=false，强制改变所有reference的check值，就算配置中有声明，也会被覆盖。
* dubbo.consumer.check=false，是设置check的缺省值，如果配置中有显式的声明，如：`<dubbo:reference check="true"/>`，不会受影响。
* dubbo.registry.check=false，前面两个都是指订阅成功，但提供者列表是否为空是否报错，如果注册订阅失败时，也允许启动，需使用此选项，将在后台定时重试。

引用缺省是延迟初始化的，只有引用被注入到其它Bean，或被getBean()获取，才会初始化。
如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：

```xml
<dubbo:reference interface="com.foo.BarService" init="true" />
```

#### 集群容错

在集群调用失败时，Dubbo提供了多种容错方案，缺省为failover重试。

[[/user-guide/images/cluster.jpg]]

各节点关系：

* 这里的Invoker是Provider的一个可调用Service的抽象，Invoker封装了Provider地址及Service接口信息。
* Directory代表多个Invoker，可以把它看成List<Invoker>，但与List不同的是，它的值可能是动态变化的，比如注册中心推送变更。
* Cluster将Directory中的多个Invoker伪装成一个Invoker，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个。
* Router负责从多个Invoker中按路由规则选出子集，比如读写分离，应用隔离等。
* LoadBalance负责从多个Invoker中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选

##### 集群容错模式

可以自行扩展集群容错策略，参见：[集群扩展](dev-guide-spi-reference-manual#集群扩展)

###### Failover Cluster

* 失败自动切换，当出现失败，重试其它服务器。(缺省)
* 通常用于读操作，但重试会带来更长延迟。
* 可通过 `retries="2"` 来设置重试次数(不含第一次)。

###### Failfast Cluster

* 快速失败，只发起一次调用，失败立即报错。
* 通常用于非幂等性的写操作，比如新增记录。

###### Failsafe Cluster

* 失败安全，出现异常时，直接忽略。
* 通常用于写入审计日志等操作。

###### Failback Cluster

* 失败自动恢复，后台记录失败请求，定时重发。
* 通常用于消息通知操作。

###### Forking Cluster

* 并行调用多个服务器，只要一个成功即返回。
* 通常用于实时性要求较高的读操作，但需要浪费更多服务资源。
* 可通过 `forks="2"` 来设置最大并行数。

###### Broadcast Cluster

* 广播调用所有提供者，逐个调用，任意一台报错则报错。(2.1.0开始支持)
* 通常用于通知所有提供者更新缓存或日志等本地资源信息。

重试次数配置如: (failover集群模式生效)

```xml
<dubbo:service retries="2" />
```

或

```xml
<dubbo:reference retries="2" />
```

或

```xml
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```

集群模式配置如:

```xml
<dubbo:service cluster="failsafe" />
```

或

```xml
<dubbo:reference cluster="failsafe" />
```

#### 负载均衡

在集群负载均衡时，Dubbo提供了多种均衡策略，缺省为random随机调用。可以自行扩展负载均衡策略，参见：[负载均衡扩展](dev-guide-spi-reference-manual#负载均衡扩展)

##### Random LoadBalance

* 随机，按权重设置随机概率。
* 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

##### RoundRobin LoadBalance

* 轮循，按公约后的权重设置轮循比率。
* 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

##### LeastActive LoadBalance

* 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。
* 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

##### ConsistentHash LoadBalance

* 一致性Hash，相同参数的请求总是发到同一提供者。
* 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
* 算法参见：http://en.wikipedia.org/wiki/Consistent_hashing。
* 缺省只对第一个参数Hash，如果要修改，请配置`<dubbo:parameter key="hash.arguments" value="0,1" />`
* 缺省用160份虚拟节点，如果要修改，请配置 `<dubbo:parameter key="hash.nodes" value="320" />`

配置如：

```xml
<dubbo:service interface="..." loadbalance="roundrobin" />
```

或

```xml
<dubbo:reference interface="..." loadbalance="roundrobin" />
```

或

```xml
<dubbo:service interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:service>
```

或

```xml
<dubbo:reference interface="...">
    <dubbo:method name="..." loadbalance="roundrobin"/>
</dubbo:reference>
```

#### 线程模型

[[/user-guide/images/dubbo-protocol.jpg]]

事件处理线程说明

如果事件处理的逻辑能迅速完成，并且不会发起新的IO请求，比如只是在内存中记个标识，则直接在IO线程上处理更快，因为减少了线程池调度。

但如果事件处理逻辑较慢，或者需要发起新的IO请求，比如需要查询数据库，则必须派发到线程池，否则IO线程阻塞，将导致不能接收其它请求。

如果用IO线程处理事件，又在事件处理过程中发起新的IO请求，比如在连接事件中发起登录请求，会报“可能引发死锁”异常，但不会真死锁。

##### Dispatcher

* `all` 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。
* `direct` 所有消息都不派发到线程池，全部在IO线程上直接执行。
* `message` 只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在IO线程上执行。
* `execution` 只请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在IO线程上执行。
* `connection` 在IO线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池。

##### ThreadPool

* `fixed` 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)
* `cached` 缓存线程池，空闲一分钟自动删除，需要时重建。
* `limited` 可伸缩线程池，但池中的线程数只会增长不会收缩。(为避免收缩时突然来了大流量引起的性能问题)。

配置如下:

```xml
<dubbo:protocol name="dubbo" dispatcher="all" threadpool="fixed" threads="100" />
```

#### 直连提供者

在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，点对点直联方式，将以服务接口为单位，忽略注册中心的提供者列表，A接口配置点对点，不影响B接口从注册中心获取列表。

[[/user-guide/images/dubbo-directly.jpg]]

* 如果是线上需求需要点对点，可在 `<dubbo:reference>` 中配置url指向提供者，将绕过注册中心，多个地址用分号隔开，配置如下：
    > 1.0.6及以上版本支持

    ```xml
    <dubbo:reference id="xxxService" interface="com.alibaba.xxx.XxxService" url="dubbo://localhost:20890" />
    ```
    
* 在JVM启动参数中加入-D参数映射服务地址，如：

    > key为服务名，value为服务提供者url，此配置优先级最高，1.0.15及以上版本支持

    ```sh
    java -Dcom.alibaba.xxx.XxxService=dubbo://localhost:20890
    ```

* 如果服务比较多，也可以用文件映射，如：

    > 用-Ddubbo.resolve.file指定映射文件路径，此配置优先级高于`<dubbo:reference>`中的配置，1.0.15及以上版本支持
    > 2.0以上版本自动加载${user.home}/dubbo-resolve.properties文件，不需要配置

    ```sh
    java -Ddubbo.resolve.file=xxx.properties
    ```
    
    然后在映射文件xxx.properties中加入：(key为服务名，value为服务提供者url)
    
    ```sh
    com.alibaba.xxx.XxxService=dubbo://localhost:20890
    ```
    
**注意**

为了避免复杂化线上环境，不要在线上使用这个功能，只应在测试阶段使用。

#### 只订阅

**问题**

为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。

**解决方案**

可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，而不注册正在开发的服务，通过直连测试正在开发的服务。

[[/user-guide/images/subscribe-only.jpg]]

禁用注册配置

```xml
<dubbo:registry address="10.20.153.10:9090" register="false" />
```

或者

```xml
<dubbo:registry address="10.20.153.10:9090?register=false" />
```

#### 只注册

**问题**

如果有两个镜像环境，两个注册中心，有一个服务只在其中一个注册中心有部署，另一个注册中心还没来得及部署，而两个注册中心的其它应用都需要依赖此服务，所以需要将服务同时注册到两个注册中心，但却不能让此服务同时依赖两个注册中心的其它服务。

**解决方案**

可以让服务提供者方，只注册服务到另一注册中心，而不从另一注册中心订阅服务。

禁用订阅配置

```xml
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090" subscribe="false" />
```

或者

```xml
<dubbo:registry id="hzRegistry" address="10.20.153.10:9090" />
<dubbo:registry id="qdRegistry" address="10.20.141.150:9090?subscribe=false" />
```

#### 静态服务

有时候希望人工管理服务提供者的上线和下线，此时需将注册中心标识为非动态管理模式

```xml
<dubbo:registry address="10.20.141.150:9090" dynamic="false" />
```

或者

```xml
<dubbo:registry address="10.20.141.150:9090?dynamic=false" />
```

服务提供者初次注册时为禁用状态，需人工启用，断线时，将不会被自动删除，需人工禁用。

如果是一个第三方独立提供者，比如memcached等，可以直接向注册中心写入提供者地址信息，消费者正常使用：

> 通常由脚本监控中心页面等调用

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("memcached://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo"));
```

#### 多协议

可以自行扩展协议，参见：[协议扩展](dev-guide-spi-reference-manual#协议扩展)

##### (1) 不同服务不同协议

比如：不同服务在性能上适用不同协议进行传输，比如大数据用短连接协议，小数据大并发用长连接协议

**consumer.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
 
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="rmi" port="1099" />
 
    <!-- 使用dubbo协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" protocol="dubbo" />
    <!-- 使用rmi协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" protocol="rmi" /> 
</beans>
```

##### (2) 多协议暴露服务

比如：需要与http客户端互操作

**consumer.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
 
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="hessian" port="8080" />
 
    <!-- 使用多个协议暴露服务 -->
    <dubbo:service id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" protocol="dubbo,hessian" />
</beans>
```

#### 多注册中心

可以自行扩展注册中心，参见：[注册中心扩展](dev-guide-spi-reference-manual#注册中心扩展)

##### (1) 多注册中心注册

比如：中文站有些服务来不及在青岛部署，只在杭州部署，而青岛的其它应用需要引用此服务，就可以将服务同时注册到两个注册中心。

**consumer.xml**

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="hangzhouRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="qingdaoRegistry" address="10.20.141.151:9010" default="false" />
 
    <!-- 向多个注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" registry="hangzhouRegistry,qingdaoRegistry" />
</beans>
```

##### (2) 不同服务使用不同注册中心

比如：CRM有些服务是专门为国际站设计的，有些服务是专门为中文站设计的。

**consumer.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="chinaRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="intlRegistry" address="10.20.154.177:9010" default="false" />
 
    <!-- 向中文站注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" registry="chinaRegistry" />
 
    <!-- 向国际站注册中心注册 -->
    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" registry="intlRegistry" />
</beans>
```

##### (3) 多注册中心引用

比如：CRM需同时调用中文站和国际站的PC2服务，PC2在中文站和国际站均有部署，接口及版本号都一样，但连的数据库不一样。

**consumer.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置 -->
    <dubbo:registry id="chinaRegistry" address="10.20.141.150:9090" />
    <dubbo:registry id="intlRegistry" address="10.20.154.177:9010" default="false" />
 
    <!-- 引用中文站服务 -->
    <dubbo:reference id="chinaHelloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" registry="chinaRegistry" />
 
    <!-- 引用国际站站服务 -->
    <dubbo:reference id="intlHelloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" registry="intlRegistry" />
</beans>
```

如果只是测试环境临时需要连接两个不同注册中心，使用竖号分隔多个不同注册中心地址：

**consumer.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://code.alibabatech.com/schema/dubbohttp://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <dubbo:application name="world"  />
 
    <!-- 多注册中心配置，竖号分隔表示同时连接多个不同注册中心，同一注册中心的多个集群地址用逗号分隔 -->
    <dubbo:registry address="10.20.141.150:9090|10.20.154.177:9010" />
 
    <!-- 引用服务 -->
    <dubbo:reference id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" />
</beans>
```

#### 服务分组

当一个接口有多种实现时，可以用group区分。

**服务**

```xml
<dubbo:service group="feedback" interface="com.xxx.IndexService" />
<dubbo:service group="member" interface="com.xxx.IndexService" />
```

**引用**

```xml
<dubbo:reference id="feedbackIndexService" group="feedback" interface="com.xxx.IndexService" />
<dubbo:reference id="memberIndexService" group="member" interface="com.xxx.IndexService" />
```

任意组：

> 2.2.0以上版本支持，总是只调一个可用组的实现

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" group="*" />
```

#### 多版本

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

在低压力时间段，先升级一半提供者为新版本，再将所有消费者升级为新版本，然后将剩下的一半提供者升级为新版本

**老版本服务**

```xml
<dubbo:service interface="com.foo.BarService" version="1.0.0" />
```

**新版本服务**

```xml
<dubbo:service interface="com.foo.BarService" version="2.0.0" />
```

**引用老版本**

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
```

**引用新版本**

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />
```

不区分版本：(2.2.0以上版本支持)

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" version="*" />
```

#### 分组聚合

按组合并返回结果，比如菜单服务，接口一样，但有多种实现，用group区分，现在消费方需从每种group中调用一次返回结果，合并结果返回，这样就可以实现聚合菜单项。

从2.1.0版本开始支持, 代码参见：https://github.com/alibaba/dubbo/tree/master/dubbo-test/dubbo-test-examples/src/main/java/com/alibaba/dubbo/examples/merge

配置如：

**搜索所有分组**

```xml
<dubbo:reference interface="com.xxx.MenuService" group="*" merger="true" />
```

**合并指定分组**

```xml
<dubbo:reference interface="com.xxx.MenuService" group="aaa,bbb" merger="true" />
```

**指定方法合并结果，其它未指定的方法，将只调用一个Group**

```xml
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger="true" />
</dubbo:service>
```

**某个方法不合并结果，其它都合并结果**

```xml
<dubbo:reference interface="com.xxx.MenuService" group="*" merger="true">
    <dubbo:method name="getMenuItems" merger="false" />
</dubbo:service>
```

**指定合并策略，缺省根据返回值类型自动匹配，如果同一类型有两个合并器时，需指定合并器的名称**
参见：[合并结果扩展](dev-guide-spi-reference-manual#合并结果扩展)

```xml
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger="mymerge" />
</dubbo:service>
```

**指定合并方法，将调用返回结果的指定方法进行合并，合并方法的参数类型必须是返回结果类型本身**

```xml
<dubbo:reference interface="com.xxx.MenuService" group="*">
    <dubbo:method name="getMenuItems" merger=".addAll" />
</dubbo:service>
```

#### 参数验证

参数验证功能是基于 [JSR303](https://jcp.org/en/jsr/detail?id=303) 实现的，用户只需标识JSR303标准的验证Annotation，并通过声明filter来实现验证。

2.1.0以上版本支持, 完整示例代码参见：https://github.com/alibaba/dubbo/tree/master/dubbo-test/dubbo-test-examples/src/main/java/com/alibaba/dubbo/examples/validation

验证方式可扩展，参见：[验证扩展](dev-guide-spi-reference-manual#验证扩展)

##### 参数标注示例

```java

import java.io.Serializable;
import java.util.Date;
 
import javax.validation.constraints.Future;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Past;
import javax.validation.constraints.Pattern;
import javax.validation.constraints.Size;
 
public class ValidationParameter implements Serializable {
     
    private static final long serialVersionUID = 7158911668568000392L;
 
    @NotNull // 不允许为空
    @Size(min = 1, max = 20) // 长度或大小范围
    private String name;
 
    @NotNull(groups = ValidationService.Save.class) // 保存时不允许为空，更新时允许为空 ，表示不更新该字段
    @Pattern(regexp = "^\\s*\\w+(?:\\.{0,1}[\\w-]+)*@[a-zA-Z0-9]+(?:[-.][a-zA-Z0-9]+)*\\.[a-zA-Z]+\\s*$")
    private String email;
 
    @Min(18) // 最小值
    @Max(100) // 最大值
    private int age;
 
    @Past // 必须为一个过去的时间
    private Date loginDate;
 
    @Future // 必须为一个未来的时间
    private Date expiryDate;
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public String getEmail() {
        return email;
    }
 
    public void setEmail(String email) {
        this.email = email;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
    public Date getLoginDate() {
        return loginDate;
    }
 
    public void setLoginDate(Date loginDate) {
        this.loginDate = loginDate;
    }
 
    public Date getExpiryDate() {
        return expiryDate;
    }
 
    public void setExpiryDate(Date expiryDate) {
        this.expiryDate = expiryDate;
    }
 
}
```

##### 分组验证示例

```java
public interface ValidationService { // 缺省可按服务接口区分验证场景，如：@NotNull(groups = ValidationService.class)
     
    @interface Save{} // 与方法同名接口，首字母大写，用于区分验证场景，如：@NotNull(groups = ValidationService.Save.class)，可选
    void save(ValidationParameter parameter);
    void update(ValidationParameter parameter);
}
```

##### 关联验证示例

```java
import javax.validation.GroupSequence;
 
public interface ValidationService {
     
    @GroupSequence(Update.class) // 同时验证Update组规则
    @interface Save{}
    void save(ValidationParameter parameter);
 
    @interface Update{} 
    void update(ValidationParameter parameter);
}
```

##### 参数验证示例

```java
import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
 
public interface ValidationService {
 
    void save(@NotNull ValidationParameter parameter); // 验证参数不为空
 
    void delete(@Min(1) int id); // 直接对基本类型参数验证
}
```

##### 在客户端验证参数

```xml
<dubbo:reference id="validationService" interface="com.alibaba.dubbo.examples.validation.api.ValidationService" validation="true" />
```

##### 在服务器端验证参数

```xml
<dubbo:service interface="com.alibaba.dubbo.examples.validation.api.ValidationService" ref="validationService" validation="true" />
```

##### 验证异常信息

```java
import javax.validation.ConstraintViolationException;
import javax.validation.ConstraintViolationException;
 
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
import com.alibaba.dubbo.examples.validation.api.ValidationParameter;
import com.alibaba.dubbo.examples.validation.api.ValidationService;
import com.alibaba.dubbo.rpc.RpcException;
 
public class ValidationConsumer {
     
    public static void main(String[] args) throws Exception {
        String config = ValidationConsumer.class.getPackage().getName().replace('.', '/') + "/validation-consumer.xml";
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(config);
        context.start();
        ValidationService validationService = (ValidationService)context.getBean("validationService");
        // Error
        try {
            parameter = new ValidationParameter();
            validationService.save(parameter);
            System.out.println("Validation ERROR");
        } catch (RpcException e) { // 抛出的是RpcException
            ConstraintViolationException ve = (ConstraintViolationException) e.getCause(); // 里面嵌了一个ConstraintViolationException
            Set<ConstraintViolation<?>> violations = ve.getConstraintViolations(); // 可以拿到一个验证错误详细信息的集合
            System.out.println(violations);
        }
    } 
}
```

##### 需要加入依赖

```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.0.0.GA</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>4.2.0.Final</version>
</dependency>
```

#### 结果缓存

结果缓存，用于加速热门数据的访问速度，Dubbo提供声明式缓存，以减少用户加缓存的工作量。

2.1.0以上版本支持, 示例代码：https://github.com/alibaba/dubbo/tree/master/dubbo-test/dubbo-test-examples/src/main/java/com/alibaba/dubbo/examples/cache

* `lru` 基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。
* `threadlocal` 当前线程缓存，比如一个页面渲染，用到很多portal，每个portal都要去查用户信息，通过线程缓存，可以减少这种多余访问。
* `jcache` 与 [JSR107](http://jcp.org/en/jsr/detail?id=107%27) 集成，可以桥接各种缓存实现。

缓存类型可扩展，参见：[CacheFactory扩展点](dev-guide-spi-reference-manual#缓存扩展)

配置如：

```xml
<dubbo:reference interface="com.foo.BarService" cache="lru" />
```

或：

```xml
<dubbo:reference interface="com.foo.BarService">
    <dubbo:method name="findBar" cache="lru" />
</dubbo:reference>
```

#### 泛化引用

泛接口调用方式主要用于客户端没有API接口及模型类元的情况，参数及返回值中的所有POJO均用Map表示，通常用于框架集成，比如：实现一个通用的服务测试框架，可通过GenericService调用所有服务实现。

```xml
<dubbo:reference id="barService" interface="com.foo.BarService" generic="true" />
```

```java
GenericService barService = (GenericService) applicationContext.getBean("barService");
Object result = barService.$invoke("sayHello", new String[] { "java.lang.String" }, new Object[] { "World" });
```

```java
import com.alibaba.dubbo.rpc.service.GenericService; 
... 
 
// 引用远程服务 
ReferenceConfig<GenericService> reference = new ReferenceConfig<GenericService>(); // 该实例很重量，里面封装了所有与注册中心及服务提供方连接，请缓存
reference.setInterface("com.xxx.XxxService"); // 弱类型接口名 
reference.setVersion("1.0.0"); 
reference.setGeneric(true); // 声明为泛化接口 
 
GenericService genericService = reference.get(); // 用com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口引用 
 
// 基本类型以及Date,List,Map等不需要转换，直接调用 
Object result = genericService.$invoke("sayHello", new String[] {"java.lang.String"}, new Object[] {"world"}); 
 
// 用Map表示POJO参数，如果返回值为POJO也将自动转成Map 
Map<String, Object> person = new HashMap<String, Object>(); 
person.put("name", "xxx"); 
person.put("password", "yyy"); 
Object result = genericService.$invoke("findPerson", new String[]{"com.xxx.Person"}, new Object[]{person}); // 如果返回POJO将自动转成Map 
 
...
```

假设存在POJO如：

```java
package com.xxx;

public class PersonImpl implements Person {
    private String name;
    private String password;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

则POJO数据：

```java
Person person = new PersonImpl(); 
person.setName("xxx"); 
person.setPassword("yyy");
```

可用下面Map表示：

```java
Map<String, Object> map = new HashMap<String, Object>(); 
map.put("class", "com.xxx.PersonImpl"); // 注意：如果参数类型是接口，或者List等丢失泛型，可通过class属性指定类型。
map.put("name", "xxx"); 
map.put("password", "yyy");
```

#### 泛化实现

泛接口实现方式主要用于服务器端没有API接口及模型类元的情况，参数及返回值中的所有POJO均用Map表示，通常用于框架集成，比如：实现一个通用的远程服务Mock框架，可通过实现GenericService接口处理所有服务请求。

```xml
<bean id="genericService" class="com.foo.MyGenericService" />
<dubbo:service interface="com.foo.BarService" ref="genericService" />
```

```java
package com.foo;
public class MyGenericService implements GenericService {
 
    public Object $invoke(String methodName, String[] parameterTypes, Object[] args) throws GenericException {
        if ("sayHello".equals(methodName)) {
            return "Welcome " + args[0];
        }
    }
}
```

```java
... 
GenericService xxxService = new XxxGenericService(); // 用com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口实现 
 
ServiceConfig<GenericService> service = new ServiceConfig<GenericService>(); // 该实例很重量，里面封装了所有与注册中心及服务提供方连接，请缓存
service.setInterface("com.xxx.XxxService"); // 弱类型接口名 
service.setVersion("1.0.0"); 
service.setRef(xxxService); // 指向一个通用服务实现 
 
// 暴露及注册服务 
service.export();
```

#### 回声测试

回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调用是否通畅，可用于监控。

所有服务自动实现EchoService接口，只需将任意服务引用强制转型为EchoService，即可使用。

```xml
<dubbo:reference id="memberService" interface="com.xxx.MemberService" />
```

```java
MemberService memberService = ctx.getBean("memberService"); // 远程服务引用
 
EchoService echoService = (EchoService) memberService; // 强制转型为EchoService
 
String status = echoService.$echo("OK"); // 回声测试可用性
 
assert(status.equals("OK"));
```

#### 上下文信息

上下文中存放的是当前调用过程中所需的环境信息。

所有配置信息都将转换为URL的参数，参见 [配置项一览表](user-guide-configuration-ref) 中的“对应URL参数”一列。

> **注意**
RpcContext是一个ThreadLocal的临时状态记录器，当接收到RPC请求，或发起RPC请求时，RpcContext的状态都会变化。
比如：A调B，B再调C，则B机器上，在B调C之前，RpcContext记录的是A调B的信息，在B调C之后，RpcContext记录的是B调C的信息。

##### (1) 服务消费方

```java
xxxService.xxx(); // 远程调用
boolean isConsumerSide = RpcContext.getContext().isConsumerSide(); // 本端是否为消费端，这里会返回true
String serverIP = RpcContext.getContext().getRemoteHost(); // 获取最后一次调用的提供方IP地址
String application = RpcContext.getContext().getUrl().getParameter("application"); // 获取当前服务配置信息，所有配置信息都将转换为URL的参数
// ...
yyyService.yyy(); // 注意：每发起RPC调用，上下文状态会变化
// ...
```

##### (2) 服务提供方

```java
public class XxxServiceImpl implements XxxService {
 
    public void xxx() { // 服务方法实现
        boolean isProviderSide = RpcContext.getContext().isProviderSide(); // 本端是否为提供端，这里会返回true
        String clientIP = RpcContext.getContext().getRemoteHost(); // 获取调用方IP地址
        String application = RpcContext.getContext().getUrl().getParameter("application"); // 获取当前服务配置信息，所有配置信息都将转换为URL的参数
        // ...
        yyyService.yyy(); // 注意：每发起RPC调用，上下文状态会变化
        boolean isProviderSide = RpcContext.getContext().isProviderSide(); // 此时本端变成消费端，这里会返回false
        // ...
    } 
}
```

#### 隐式传参

> 注：path,group,version,dubbo,token,timeout几个key有特殊处理，请使用其它key值。

[[/user-guide/images/context.png]]

##### (1) 服务消费方

```xml
RpcContext.getContext().setAttachment("index", "1"); // 隐式传参，后面的远程调用都会隐式将这些参数发送到服务器端，类似cookie，用于框架集成，不建议常规业务使用
xxxService.xxx(); // 远程调用
// ...
```

> 注: setAttachment设置的KV，在完成下面一次远程调用会被清空。即多次远程调用要多次设置。

##### (2) 服务提供方

```java
public class XxxServiceImpl implements XxxService {
 
    public void xxx() { // 服务方法实现
        String index = RpcContext.getContext().getAttachment("index"); // 获取客户端隐式传入的参数，用于框架集成，不建议常规业务使用
        // ...
    }
}
```

#### 异步调用

基于NIO的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小。2.0.6及其以上版本支持

[[/user-guide/images/future.jpg]]

配置声明：

**consumer.xml**

```xml
<dubbo:reference id="fooService" interface="com.alibaba.foo.FooService">
      <dubbo:method name="findFoo" async="true" />
</dubbo:reference>
<dubbo:reference id="barService" interface="com.alibaba.bar.BarService">
      <dubbo:method name="findBar" async="true" />
</dubbo:reference>
```

调用代码:

```java
fooService.findFoo(fooId); // 此调用会立即返回null
Future<Foo> fooFuture = RpcContext.getContext().getFuture(); // 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future。
 
barService.findBar(barId); // 此调用会立即返回null
Future<Bar> barFuture = RpcContext.getContext().getFuture(); // 拿到调用的Future引用，当结果返回后，会被通知和设置到此Future。
 
// 此时findFoo和findBar的请求同时在执行，客户端不需要启动多线程来支持并行，而是借助NIO的非阻塞完成。
 
Foo foo = fooFuture.get(); // 如果foo已返回，直接拿到返回值，否则线程wait住，等待foo返回后，线程会被notify唤醒。
Bar bar = barFuture.get(); // 同理等待bar返回。
 
// 如果foo需要5秒返回，bar需要6秒返回，实际只需等6秒，即可获取到foo和bar，进行接下来的处理。
```

你也可以设置是否等待消息发出：(异步总是不等待返回)

* `sent="true"` 等待消息发出，消息发送失败将抛出异常。
* `sent="false"` 不等待消息发出，将消息放入IO队列，即刻返回。

```xml
<dubbo:method name="findFoo" async="true" sent="true" />
```

如果你只是想异步，完全忽略返回值，可以配置 `return="false"`，以减少Future对象的创建和管理成本：

```xml
<dubbo:method name="findFoo" async="true" return="false" />
```

#### 本地调用

本地调用，使用了Injvm协议，是一个伪协议，它不开启端口，不发起远程调用，只在JVM内直接关联，但执行Dubbo的Filter链。

定义 injvm 协议:

```xml
<dubbo:protocol name="injvm" />
```

设置默认协议:

```xml
<dubbo:provider protocol="injvm" />
```

设置服务协议:

```xml
<dubbo:service protocol="injvm" />
```

优先使用 injvm:

```xml
<dubbo:consumer injvm="true" .../>
<dubbo:provider injvm="true" .../>
```

或

```xml
<dubbo:reference injvm="true" .../>
<dubbo:service injvm="true" .../>
```

> 注意：服务暴露与服务引用都需要声明 `injvm="true"`

##### 自动暴露、引用本地服务

从 dubbo 2.2.0 开始，每个服务默认都会在本地暴露。在引用服务的时候，默认优先引用本地服务。如果希望引用远程服务可以使用一下配置强制引用远程服务。

```xml
<dubbo:reference ... scope="remote" />
```

#### 参数回调

参数回调方式与调用本地callback或listener相同，只需要在Spring的配置文件中声明哪个参数是callback类型即可，Dubbo将基于长连接生成反向代理，这样就可以从服务器端调用客户端逻辑。

2.0.6及其以上版本支持，代码参见：https://github.com/alibaba/dubbo/tree/master/dubbo-test/dubbo-test-examples/src/main/java/com/alibaba/dubbo/examples/callback

##### (1) 共享服务接口

服务接口示例

**CallbackService.java**

```java
package com.callback;
 
public interface CallbackService {
    void addListener(String key, CallbackListener listener);
}
```

**CallbackListener.java**

```java
package com.callback;
 
public interface CallbackListener {
    void changed(String msg);
}
```

##### (2) 服务提供者

服务提供者接口实现示例

**CallbackServiceImpl.java**

```java
package com.callback.impl;
 
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
 
import com.callback.CallbackListener;
import com.callback.CallbackService;
 
public class CallbackServiceImpl implements CallbackService {
     
    private final Map<String, CallbackListener> listeners = new ConcurrentHashMap<String, CallbackListener>();
  
    public CallbackServiceImpl() {
        Thread t = new Thread(new Runnable() {
            public void run() {
                while(true) {
                    try {
                        for(Map.Entry<String, CallbackListener> entry : listeners.entrySet()){
                           try {
                               entry.getValue().changed(getChanged(entry.getKey()));
                           } catch (Throwable t) {
                               listeners.remove(entry.getKey());
                           }
                        }
                        Thread.sleep(5000); // 定时触发变更通知
                    } catch (Throwable t) { // 防御容错
                        t.printStackTrace();
                    }
                }
            }
        });
        t.setDaemon(true);
        t.start();
    }
  
    public void addListener(String key, CallbackListener listener) {
        listeners.put(key, listener);
        listener.changed(getChanged(key)); // 发送变更通知
    }
     
    private String getChanged(String key) {
        return "Changed: " + new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
    }
}
```

服务提供者配置示例

```xml
<bean id="callbackService" class="com.callback.impl.CallbackServiceImpl" />
<dubbo:service interface="com.callback.CallbackService" ref="callbackService" connections="1" callbacks="1000">
    <dubbo:method name="addListener">
        <dubbo:argument index="1" callback="true" />
        <!--也可以通过指定类型的方式-->
        <!--<dubbo:argument type="com.demo.CallbackListener" callback="true" />-->
    </dubbo:method>
</dubbo:service>
```

##### (3) 服务消费者

服务消费者配置示例

**consumer.xml**

```xml
<dubbo:reference id="callbackService" interface="com.callback.CallbackService" />
```

服务消费者调用示例

**CallbackServiceTest.java**

```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:consumer.xml");
context.start();
 
CallbackService callbackService = (CallbackService) context.getBean("callbackService");
 
callbackService.addListener("http://10.20.160.198/wiki/display/dubbo/foo.bar", new CallbackListener(){
    public void changed(String msg) {
        System.out.println("callback1:" + msg);
    }
});
```

#### 事件通知

在调用之前，调用之后，出现异常时，会触发oninvoke, onreturn, onthrow三个事件，可以配置当事件发生时，通知哪个类的哪个方法。支持版本：2.0.7之后

##### (1) 服务提供者与消费者共享服务接口

**IDemoService.java**

```java
interface IDemoService {
    public Person get(int id);
}
```

##### (2) 服务提供者实现

**DemoServiceImpl.java**

```java

class NormalDemoService implements IDemoService {
    public Person get(int id) {
        return new Person(id, "charles`son", 4);
    }
}
```

##### (3) 服务提供者配置

**provider.xml**

```xml
<dubbo:application name="rpc-callback-demo" />
<dubbo:registry address="http://10.20.160.198/wiki/display/dubbo/10.20.153.186" />
<bean id="demoService" class="com.alibaba.dubbo.callback.implicit.NormalDemoService" />
<dubbo:service interface="com.alibaba.dubbo.callback.implicit.IDemoService" ref="demoService" version="1.0.0" group="cn"/>
```

##### (4) 服务消费者Callback接口及实现

**Notify.java**

```java
interface Notify {
    public void onreturn(Person msg, Integer id);
    public void onthrow(Throwable ex, Integer id);
}
```

**NotifyImpl.java**

```java

class NotifyImpl implements Notify {
    public Map<Integer, Person>    ret    = new HashMap<Integer, Person>();
    public Map<Integer, Throwable> errors = new HashMap<Integer, Throwable>();
    public void onreturn(Person msg, Integer id) {
        System.out.println("onreturn:" + msg);
        ret.put(id, msg);
    }
    public void onthrow(Throwable ex, Integer id) {
        errors.put(id, ex);
    }
}
```

##### (5) 服务消费者Callback接口及实现

**consumer.xml**

```xml
<bean id ="demoCallback" class = "com.alibaba.dubbo.callback.implicit.NofifyImpl" />
<dubbo:reference id="demoService" interface="com.alibaba.dubbo.callback.implicit.IDemoService" version="1.0.0" group="cn" >
      <dubbo:method name="get" async="true" onreturn = "demoCallback.onreturn" onthrow="demoCallback.onthrow" />
</dubbo:reference>
```

callback与async功能正交分解：

* async=true，表示结果是否马上返回.
* onreturn 表示是否需要回调.

组合情况：(async=false 默认)

* 异步回调模式：async=true onreturn="xxx"
* 同步回调模式：async=false onreturn="xxx"
* 异步无回调 ：async=true
* 同步无回调 ：async=false

##### (6) TEST CASE

**Test.java**

```java
IDemoService demoService = (IDemoService) context.getBean("demoService");
NofifyImpl notify = (NofifyImpl) context.getBean("demoCallback");
int requestId = 2;
Person ret = demoService.get(requestId);
Assert.assertEquals(null, ret);
//for Test：只是用来说明callback正常被调用，业务具体实现自行决定.
for (int i = 0; i < 10; i++) {
    if (!notify.ret.containsKey(requestId)) {
        Thread.sleep(200);
    } else {
        break;
    }
}
Assert.assertEquals(requestId, notify.ret.get(requestId).getId());
```

#### 本地存根

远程服务后，客户端通常只剩下接口，而实现全在服务器端，但提供方有些时候想在客户端也执行部分逻辑，比如：做ThreadLocal缓存，提前验证参数，调用失败后伪造容错数据等等，此时就需要在API中带上Stub，客户端生成Proxy实例，会把Proxy通过构造函数传给Stub，然后把Stub暴露组给用户，Stub可以决定要不要去调Proxy。

Stub必须有可传入Proxy的构造函数。

[[/user-guide/images/stub.jpg]]

```xml
<dubbo:service interface="com.foo.BarService" stub="true" />
```

或

```xml
<dubbo:service interface="com.foo.BarService" stub="com.foo.BarServiceStub" />
```

api.jar:

```sh
com.foo.BarService
com.foo.BarServiceStub // 在API旁边放一个Stub实现，它实现BarService接口，并有一个传入远程BarService实例的构造函数
```

```java
package com.foo;
public class BarServiceStub implements BarService {
 
    private final BarService barService;
 
    // 构造函数传入真正的远程代理对象
    public (BarService barService) {
        this.barService = barService;
    }
 
    public String sayHello(String name) {
        // 此代码在客户端执行
        // 你可以在客户端做ThreadLocal本地缓存，或预先验证参数是否合法，等等
        try {
            return barService.sayHello(name);
        } catch (Exception e) {
            // 你可以容错，可以做任何AOP拦截事项
            return "容错数据";
        }
    }
}
```

#### 本地伪装

Mock通常用于服务降级，比如某验权服务，当服务提供方全部挂掉后，客户端不抛出异常，而是通过Mock数据返回授权失败。

Mock是Stub的一个子集，便于服务提供方在客户端执行容错逻辑，因经常需要在出现RpcException(比如网络失败，超时等)时进行容错，而在出现业务异常(比如登录用户名密码错误)时不需要容错，如果用Stub，可能就需要捕获并依赖RpcException类，而用Mock就可以不依赖RpcException，因为它的约定就是只有出现RpcException时才执行。

```xml
<dubbo:service interface="com.foo.BarService" mock="true" />
```

或

```xml
<dubbo:service interface="com.foo.BarService" mock="com.foo.BarServiceMock" />
```

api.jar:

```sh
com.foo.BarService
com.foo.BarServiceMock // 在API旁边放一个Mock实现，它实现BarService接口，并有一个无参构造函数
```

```java
package com.foo;
public class BarServiceMock implements BarService {
 
    public String sayHello(String name) {
        // 你可以伪造容错数据，此方法只在出现RpcException时被执行
        return "容错数据";
    }
}
```

如果服务的消费方经常需要try-catch捕获异常，如：

```java
Offer offer = null;
try {
    offer = offerService.findOffer(offerId);
} catch (RpcException e) {
   logger.error(e);
}
```

请考虑改为Mock实现，并在Mock中return null。

如果只是想简单的忽略异常，在2.0.11以上版本可用：

```xml
<dubbo:service interface="com.foo.BarService" mock="return null" />
```

#### 延迟暴露

如果你的服务需要warmup时间，比如初始化缓存，等待相关资源就位等，可以使用delay进行延迟暴露。

延迟5秒暴露服务

```xml
<dubbo:service delay="5000" />
```

延迟到Spring初始化完成后，再暴露服务：(基于Spring的ContextRefreshedEvent事件触发暴露)

```xml
<dubbo:service delay="-1" />
```

**Spring2.x初始化死锁问题**

在Spring解析到<dubbo:service />时，就已经向外暴露了服务，而Spring还在接着初始化其它Bean。
如果这时有请求进来，并且服务的实现类里有调用applicationContext.getBean()的用法。

1\. 请求线程的applicationContext.getBean()调用，先同步singletonObjects判断Bean是否存在，不存在就同步beanDefinitionMap进行初始化，并再次同步singletonObjects写入Bean实例缓存。

[[/user-guide/images/lock-get-bean.jpg]]

2\. 而Spring初始化线程，因不需要判断Bean的存在，直接同步beanDefinitionMap进行初始化，并同步singletonObjects写入Bean实例缓存。

[[/user-guide/images/lock-init-context.jpg]]

这样就导致getBean线程，先锁singletonObjects，再锁beanDefinitionMap，再次锁singletonObjects。
而Spring初始化线程，先锁beanDefinitionMap，再锁singletonObjects。反向锁导致线程死锁，不能提供服务，启动不了。

**规避办法**

1. 强烈建议不要在服务的实现类中有applicationContext.getBean()的调用，全部采用IoC注入的方式使用Spring的Bean。
2. 如果实在要调getBean()，可以将Dubbo的配置放在Spring的最后加载。
3. 如果不想依赖配置顺序，可以使用 `<dubbo:provider deplay=”-1” />`，使Dubbo在Spring容器初始化完后，再暴露服务。
4. 如果大量使用getBean()，相当于已经把Spring退化为工厂模式在用，可以将Dubbo的服务隔离单独的Spring容器。

#### 并发控制

限制com.foo.BarService的每个方法，服务器端并发执行（或占用线程池线程数）不能超过10个：

```xml
<dubbo:service interface="com.foo.BarService" executes="10" />
```

限制com.foo.BarService的sayHello方法，服务器端并发执行（或占用线程池线程数）不能超过10个：

```xml
<dubbo:service interface="com.foo.BarService">
    <dubbo:method name="sayHello" executes="10" />
</dubbo:service>
```

限制com.foo.BarService的每个方法，每客户端并发执行（或占用连接的请求数）不能超过10个：

```xml
<dubbo:service interface="com.foo.BarService" actives="10" />
```

或

```xml
<dubbo:reference interface="com.foo.BarService" actives="10" />
```

限制com.foo.BarService的sayHello方法，每客户端并发执行（或占用连接的请求数）不能超过10个：

```xml
<dubbo:service interface="com.foo.BarService">
    <dubbo:method name="sayHello" actives="10" />
</dubbo:service>
```

或

```xml
<dubbo:reference interface="com.foo.BarService">
    <dubbo:method name="sayHello" actives="10" />
</dubbo:service>
```

如果 `<dubbo:service>` 和 `<dubbo:reference>` 都配了actives，`<dubbo:reference>` 优先，参见：[配置的覆盖策略](user-guide-configuration#配置覆盖)。

#### Load Balance 均衡

配置服务的客户端的loadbalance属性为leastactive，此Loadbalance会调用并发数最小的Provider（Consumer端并发数）。

```xml
<dubbo:reference interface="com.foo.BarService" loadbalance="leastactive" />
```

或

```xml
<dubbo:service interface="com.foo.BarService" loadbalance="leastactive" />
```

#### 连接控制

限制服务器端接受的连接不能超过10个：（因为连接在Server上，所以配置在Provider上）

```xml
<dubbo:provider protocol="dubbo" accepts="10" />
```

```xml
<dubbo:protocol name="dubbo" accepts="10" />
```

限制客户端服务使用连接连接数：(如果是长连接，比如Dubbo协议，connections表示该服务对每个提供者建立的长连接数)

```xml
<dubbo:reference interface="com.foo.BarService" connections="10" />
```

或

```xml
<dubbo:service interface="com.foo.BarService" connections="10" />
```

如果 `<dubbo:service>`和 `<dubbo:reference>` 都配了connections，`<dubbo:reference>` 优先，参见：[配置的覆盖策略](user-guide-configuration#配置覆盖)

#### 延迟连接

延迟连接，用于减少长连接数，当有调用发起时，再创建长连接。

只对使用长连接的dubbo协议生效。

```xml
<dubbo:protocol name="dubbo" lazy="true" />
```

#### 粘滞连接

粘滞连接用于有状态服务，尽可能让客户端总是向同一提供者发起调用，除非该提供者挂了，再连另一台。

粘滞连接将自动开启延迟连接，以减少长连接数，参见：[延迟连接](user-guide-sample#延迟连接)

#### 令牌验证

[[/user-guide/images/dubbo-token.jpg]]

* 防止消费者绕过注册中心访问提供者
* 在注册中心控制权限，以决定要不要下发令牌给消费者
* 注册中心可灵活改变授权方式，而不需修改或升级提供者

可以全局设置开启令牌验证

```xml
<!--随机token令牌，使用UUID生成-->
<dubbo:provider interface="com.foo.BarService" token="true" />
```

```xml
<!--固定token令牌，相当于密码-->
<dubbo:provider interface="com.foo.BarService" token="123456" />
```

也可在服务级别设置：

```xml
<!--随机token令牌，使用UUID生成-->
<dubbo:service interface="com.foo.BarService" token="true" />
```

```xml
<!--固定token令牌，相当于密码-->
<dubbo:service interface="com.foo.BarService" token="123456" />
```

还可在协议级别设置：

```xml
<!--随机token令牌，使用UUID生成-->
<dubbo:protocol name="dubbo" token="true" />
```

```xml
<!--固定token令牌，相当于密码-->
<dubbo:protocol name="dubbo" token="123456" />
```

#### 路由规则

2.2.0以上版本支持, 路由规则扩展点：[路由扩展](dev-guide-spi-reference-manual#路由扩展)

向注册中心写入路由规则：(通常由监控中心或治理中心的页面完成)

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("condition://0.0.0.0/com.foo.BarService?category=routers&dynamic=false&rule=" + URL.encode("http://10.20.160.198/wiki/display/dubbo/host = 10.20.153.10 => host = 10.20.153.11") + "));
```

其中：

* condition://
表示路由规则的类型，支持条件路由规则和脚本路由规则，可扩展，必填。
* 0.0.0.0
表示对所有IP地址生效，如果只想对某个IP的生效，请填入具体IP，必填。
* com.foo.BarService
表示只对指定服务生效，必填。
* category=routers
表示该数据为动态配置类型，必填。
* dynamic=false
表示该数据为持久数据，当注册方退出时，数据依然保存在注册中心，必填。
* enabled=true
覆盖规则是否生效，可不填，缺省生效。
* force=false
当路由结果为空时，是否强制执行，如果不强制执行，路由结果为空的路由规则将自动失效，可不填，缺省为flase。
* runtime=false
是否在每次调用时执行路由规则，否则只在提供者地址列表变更时预先执行并缓存结果，调用时直接从缓存中获取路由结果。
如果用了参数路由，必须设为true，需要注意设置会影响调用的性能，可不填，缺省为flase。
* priority=1
路由规则的优先级，用于排序，优先级越大越靠前执行，可不填，缺省为0。
* rule=URL.encode("host = 10.20.153.10 => host = 10.20.153.11")
表示路由规则的内容，必填。

#### 条件路由规则

基于条件表达式的路由规则，如：`host = 10.20.153.10 => host = 10.20.153.11`

规则：

* "=>"之前的为消费者匹配条件，所有参数和消费者的URL进行对比，当消费者满足匹配条件时，对该消费者执行后面的过滤规则。
* "=>"之后为提供者地址列表的过滤条件，所有参数和提供者的URL进行对比，消费者最终只拿到过滤后的地址列表。
* 如果匹配条件为空，表示对所有消费方应用，如：=> host != 10.20.153.11
* 如果过滤条件为空，表示禁止访问，如：host = 10.20.153.10 =>

表达式：

* 参数支持：
    * 服务调用信息，如：method, argument 等 (暂不支持参数路由)
    * URL本身的字段，如：protocol, host, port 等
    * 以及URL上的所有参数，如：application, organization 等
* 条件支持：
    * 等号 `=` 表示"匹配"，如：host = 10.20.153.10
    * 不等号 `!=` 表示"不匹配"，如：host != 10.20.153.10
* 值支持：
    * 以逗号 `,` 分隔多个值，如：host != 10.20.153.10,10.20.153.11
    * 以星号 `*` 结尾，表示通配，如：host != 10.20.*
    * 以美元符 `$` 开头，表示引用消费者参数，如：host = $host

示例：

0. 排除预发布机：

    ```
    => host != 172.22.3.91
    ```
1. 白名单：
    注意：一个服务只能有一条白名单规则，否则两条规则交叉，就都被筛选掉了) 
    
    ```
    host != 10.20.153.10,10.20.153.11 =>
    ```
2. 黑名单：

    ```
    host = 10.20.153.10,10.20.153.11 =>
    ```
3. 服务寄宿在应用上，只暴露一部分的机器，防止整个集群挂掉：

    ```
    => host = 172.22.3.1*,172.22.3.2*
    ```
4. 为重要应用提供额外的机器：

    ```
    application != kylin => host != 172.22.3.95,172.22.3.96
    ```
5. 读写分离：

    ```
    method = find*,list*,get*,is* => host = 172.22.3.94,172.22.3.95,172.22.3.96
    method != find*,list*,get*,is* => host = 172.22.3.97,172.22.3.98
    ```
    
6. 前后台分离：

    ```
    application = bops => host = 172.22.3.91,172.22.3.92,172.22.3.93
    application != bops => host = 172.22.3.94,172.22.3.95,172.22.3.96
    ```
    
7. 隔离不同机房网段：

    ```
    host != 172.22.3.* => host != 172.22.3.*
    ```
    
8. 提供者与消费者部署在同集群内，本机只访问本机的服务：

    ```
    => host = $host
    ```
    
#### 脚本路由规则

支持JDK脚本引擎的所有脚本，比如：javascript, jruby, groovy 等，通过 type=javascript 参数设置脚本类型，缺省为javascript。

脚本没有沙箱约束，可执行任意代码，存在后门风险

```
"script://0.0.0.0/com.foo.BarService?category=routers&dynamic=false&rule=" + URL.encode("function route(invokers) { ... } (invokers)")
```

基于脚本引擎的路由规则，如：

```javascript
function route(invokers) {
    var result = new java.util.ArrayList(invokers.size());
    for (i = 0; i < invokers.size(); i ++) {
        if ("http://10.20.160.198/wiki/display/dubbo/10.20.153.10".equals(invokers.get(i).getUrl().getHost())) {
            result.add(invokers.get(i));
        }
    }
    return result;
} (invokers); // 表示立即执行方法
```

#### 配置规则

2.2.0 以上版本支持

向注册中心写入动态配置覆盖规则：(通常由监控中心或治理中心的页面完成)

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&timeout=1000"));
```

其中：

* override:// 表示数据采用覆盖方式，支持override和absent，可扩展，必填。
* 0.0.0.0 表示对所有IP地址生效，如果只想覆盖某个IP的数据，请填入具体IP，必填。
* com.foo.BarService 表示只对指定服务生效，必填。
* category=configurators 表示该数据为动态配置类型，必填。
* dynamic=false 表示该数据为持久数据，当注册方退出时，数据依然保存在注册中心，必填。
* enabled=true 覆盖规则是否生效，可不填，缺省生效。
* application=foo 表示只对指定应用生效，可不填，表示对所有应用生效。
* timeout=1000 表示将满足以上条件的timeout参数的值覆盖为1000。如果想覆盖其它参数，直接加在override的URL参数上。

示例：

1. 禁用提供者：(通常用于临时踢除某台提供者机器，相似的，禁止消费者访问请使用路由规则)

    ```
    override://10.20.153.10/com.foo.BarService?category=configurators&dynamic=false&disbaled=true
    ```
    
2. 调整权重：(通常用于容量评估，缺省权重为100)

    ```
    override://10.20.153.10/com.foo.BarService?category=configurators&dynamic=false&weight=200
    ```
    
3. 调整负载均衡策略：(缺省负载均衡策略为random)

    ```
    override://10.20.153.10/com.foo.BarService?category=configurators&dynamic=false&loadbalance=leastactive
    ```
    
4. 服务降级：(通常用于临时屏蔽某个出错的非关键服务)

    ```
    override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null
    ```
    
#### 服务降级

2.2.0以上版本支持, 参见：[配置规则](user-guide-sample#配置规则)

向注册中心写入动态配置覆盖规则：(通过由监控中心或治理中心的页面完成)

```java

RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中:

```sh
mock=force:return+null
```

* 表示消费方对该服务的方法调用都直接返回null值，不发起远程调用。
* 屏蔽不重要服务不可用时对调用方的影响。

还可以改为：

```sh
mock=fail:return+null
```

* 表示消费方对该服务的方法调用在失败后，再返回null值，不抛异常。
* 容忍不重要服务不稳定时对调用方的影响。

#### 优雅停机

Dubbo是通过JDK的ShutdownHook来完成优雅停机的，所以如果用户使用"kill -9 PID"等强制关闭指令，是不会执行优雅停机的，只有通过"kill PID"时，才会执行。

**原理**

服务提供方

* 停止时，先标记为不接收新请求，新请求过来时直接报错，让客户端重试其它机器。
* 然后，检测线程池中的线程是否正在运行，如果有，等待所有线程执行完成，除非超时，则强制关闭。

服务消费方

* 停止时，不再发起新的调用请求，所有新的调用在客户端即报错。
* 然后，检测有没有请求的响应还没有返回，等待响应返回，除非超时，则强制关闭。

设置优雅停机超时时间，缺省超时时间是10秒：(超时则强制关闭)

```xml

<dubbo:application ...>
    <dubbo:parameter key="shutdown.timeout" value="60000" /> <!-- 单位毫秒 -->
</dubbo:application>
```

如果ShutdownHook不能生效，可以自行调用：

```java
ProtocolConfig.destroyAll();
```

#### 主机绑定

缺省主机IP查找顺序：

* 通过LocalHost.getLocalHost()获取本机地址。
* 如果是127.*等loopback地址，则扫描各网卡，获取网卡IP。

注册的地址如果获取不正确，比如需要注册公网地址，可以：

1. 可以在/etc/hosts中加入：机器名 公网IP，比如：
    
    ```
    test1 205.182.23.201
    ```
    
2. 在dubbo.xml中加入主机地址的配置：

    ```xml
    <dubbo:protocol host="http://10.20.160.198/wiki/display/dubbo/205.182.23.201">
    ```

3. 或在dubbo.properties中加入主机地址的配置：

    ```
    dubbo.protocol.host=205.182.23.201
    ```
    
缺省主机端口与协议相关：

* dubbo: 20880
* rmi: 1099
* http: 80
* hessian: 80
* webservice: 80
* memcached: 11211
* redis: 6379

主机端口配置：

1. 在dubbo.xml中加入主机地址的配置：

    ```xml
    <dubbo:protocol name="dubbo" port="20880">
    ```

2. 或在dubbo.properties中加入主机地址的配置：

    ```
    dubbo.protocol.dubbo.port=20880
    ```
    
#### 日志适配

2.2.1以上版本支持，扩展点：[日志适配扩展](dev-guide-spi-reference-manual#日志适配扩展)

缺省自动查找：

* log4j
* slf4j
* jcl
* jdk

可以通过以下方式配置日志输出策略

```sh
java -Ddubbo.application.logger=log4j
```

**dubbo.properties**

```
dubbo.application.logger=log4j
```

**dubbo.xml**

```xml
<dubbo:application logger="log4j" />
```

#### 访问日志

如果你想记录每一次请求信息，可开启访问日志，类似于apache的访问日志。

此日志量比较大，请注意磁盘容量。

将访问日志输出到当前应用的log4j日志：

```xml
<dubbo:protocol accesslog="true" />
```

将访问日志输出到指定文件：

```xml
<dubbo:protocol accesslog="http://10.20.160.198/wiki/display/dubbo/foo/bar.log" />
```

#### 服务容器

服务容器是一个standalone的启动程序，因为后台服务不需要Tomcat或JBoss等Web容器的功能，如果硬要用Web容器去加载服务提供方，增加复杂性，也浪费资源。

服务容器只是一个简单的Main方法，并加载一个简单的Spring容器，用于暴露服务。

服务容器的加载内容可以扩展，内置了spring, jetty, log4j等加载，可通过Container扩展点进行扩展，参见：[Container](dev-guide-spi-reference-manual#容器扩展)。配置配在java命令-D参数或者dubbo.properties中

##### Spring Container

* 自动加载 META-INF/spring 目录下的所有 Spring 配置。
* 配置spring配置加载位置：`dubbo.spring.config=classpath*:META-INF/spring/*.xml`

##### Jetty Container

* 启动一个内嵌Jetty，用于汇报状态。
* 配置：
    * dubbo.jetty.port=8080 ----配置jetty启动端口
    * dubbo.jetty.directory=/foo/bar ----配置可通过jetty直接访问的目录，用于存放静态文件
    * dubbo.jetty.page=log,status,system ----配置显示的页面，缺省加载所有页面
        
##### Log4j Container

* 自动配置log4j的配置，在多进程启动时，自动给日志文件按进程分目录。
* 配置：
    * dubbo.log4j.file=/foo/bar.log ----配置日志文件路径
    * dubbo.log4j.level=WARN ----配置日志级别
    * dubbo.log4j.subdirectory=20880 ----配置日志子目录，用于多进程启动，避免冲突

##### 容器启动

如：(缺省只加载spring)

```sh
java com.alibaba.dubbo.container.Main
```

或：(通过main函数参数传入要加载的容器)

```sh
java com.alibaba.dubbo.container.Main spring jetty log4j
```

或：(通过JVM启动参数传入要加载的容器)

```sh
java com.alibaba.dubbo.container.Main -Ddubbo.container=spring,jetty,log4j
```

或：(通过classpath下的dubbo.properties配置传入要加载的容器)

**dubbo.properties**

```
dubbo.container=spring,jetty,log4j
```

#### Reference Config缓存

ReferenceConfig实例很重，封装了与注册中心的连接以及与提供者的连接，需要缓存，否则重复生成ReferenceConfig可能造成性能问题并且会有内存和连接泄漏。API方式编程时，容易忽略此问题。

Dubbo 2.4.0+版本，提供了简单的工具类ReferenceConfigCache用于缓存ReferenceConfig实例。

使用方式如下：

```java
ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>();
reference.setInterface(XxxService.class);
reference.setVersion("1.0.0");
......
 
ReferenceConfigCache cache = ReferenceConfigCache.getCache();
XxxService xxxService = cache.get(reference); // cache.get方法中会Cache Reference对象，并且调用ReferenceConfig.get方法启动ReferenceConfig
// 注意！ Cache会持有ReferenceConfig，不要在外部再调用ReferenceConfig的destroy方法，导致Cache内的ReferenceConfig失效！
 
// 使用xxxService对象
xxxService.sayHello();
```

消除Cache中的ReferenceConfig，销毁ReferenceConfig并释放对应的资源。

```java
ReferenceConfigCache cache = ReferenceConfigCache.getCache();
cache.destroy(reference);
```

缺省ReferenceConfigCache把相同服务Group、接口、版本的ReferenceConfig认为是相同，缓存一份。即以服务Group、接口、版本为缓存的Key。

可以修改这个策略，在ReferenceConfigCache.getCache时，传一个KeyGenerator。详见ReferenceConfigCache类的方法。

```java
KeyGenerator keyGenerator = new ...
 
ReferenceConfigCache cache = ReferenceConfigCache.getCache(keyGenerator );
```

#### 分布式事务

基于JTA/XA规范实现。

暂未实现。

两阶段提交：

[[/user-guide/images/jta-xa.jpg]]









    
