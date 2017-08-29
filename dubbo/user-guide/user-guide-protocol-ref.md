[**HOME**](Home) > **用户指南**

### 协议参考手册

推荐使用Dubbo协议

各协议的性能情况，请参见：[性能测试报告](user-guide-benchmark-report)

#### dubbo://

Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

Dubbo缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。

```xml
<dubbo:protocol name="dubbo" port="20880" />
```

设置默认协议：

```xml
<dubbo:provider protocol="dubbo" />
```

设置服务协议：

```xml
<dubbo:service protocol="dubbo" />
```

多端口：

```xml
<dubbo:protocol name=“dubbo” port=“9090” server=“netty” client=“netty” codec=“dubbo” serialization=“hessian2” charset=“UTF-8” threadpool=“fixed” threads=“100” queues=“0” iothreads=“9” buffer=“8192” accepts=“1000” payload=“8388608” />
```

[[/user-guide/images/dubbo-protocol.jpg]]

* Transporter: mina, netty, grizzy
* Serialization: dubbo, hessian2, java, json
* Dispatcher: all, direct, message, execution, connection
* ThreadPool: fixed, cached

Dubbo协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接。

```xml
<dubbo:protocol name="dubbo" connections="2" />
```

* `<dubbo:service connections=”0”>` 或 `<dubbo:reference connections=”0”>` 表示该服务使用JVM共享长连接。(缺省)
* `<dubbo:service connections=”1”>` 或 `<dubbo:reference connections=”1”>` 表示该服务使用独立长连接。
* `<dubbo:service connections=”2”>` 或`<dubbo:reference connections=”2”>` 表示该服务使用独立两条长连接。

为防止被大量连接撑挂，可在服务提供方限制大接收连接数，以实现服务提供方自我保护。

```xml
<dubbo:protocol name="dubbo" accepts="1000" />
```

缺省协议，使用基于mina1.1.7+hessian3.2.1的tbremoting交互。

* 连接个数：单连接
* 连接方式：长连接
* 传输协议：TCP
* 传输方式：NIO异步传输
* 序列化：Hessian二进制序列化
* 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串。
* 适用场景：常规远程服务方法调用

**为什么要消费者比提供者个数多?**

因dubbo协议采用单一长连接，假设网络为千兆网卡(1024Mbit=128MByte)，根据测试经验数据每条连接最多只能压满7MByte(不同的环境可能不一样，供参考)，理论上1个服务提供者需要20个服务消费者才能压满网卡。

**为什么不能传大包?**

因dubbo协议采用单一长连接，如果每次请求的数据包大小为500KByte，假设网络为千兆网卡(1024Mbit=128MByte)，每条连接最大7MByte(不同的环境可能不一样，供参考)，单个服务提供者的TPS(每秒处理事务数)最大为：128MByte / 500KByte = 262。单个消费者调用单个服务提供者的TPS(每秒处理事务数)最大为：7MByte / 500KByte = 14。如果能接受，可以考虑使用，否则网络将成为瓶颈。

**为什么采用异步单一长连接?**

因为服务的现状大都是服务提供者少，通常只有几台机器，而服务的消费者多，可能整个网站都在访问该服务，比如Morgan的提供者只有6台提供者，却有上百台消费者，每天有1.5亿次调用，如果采用常规的hessian服务，服务提供者很容易就被压跨，通过单一连接，保证单一消费者不会压死提供者，长连接，减少连接握手验证等，并使用异步IO，复用线程池，防止C10K问题。

(1) 约束：

* 参数及返回值需实现Serializable接口
* 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。
* Hessian序列化，只传成员属性值和值的类型，不传方法或静态变量，兼容情况：(由吴亚军提供)

    数据通讯  | 情况 | 结果
------------- | ------------- | -------------
A->B  | 类A多一种 属性（或者说类B少一种 属性）| 不抛异常，A多的那 个属性的值，B没有， 其他正常
A->B  | 枚举A多一种 枚举（或者说B少一种 枚举），A使用多 出来的枚举进行传输 | 抛异常
A->B | 枚举A多一种 枚举（或者说B少一种 枚举），A不使用 多出来的枚举进行传输 | 不抛异常，B正常接 收数据
A->B | A和B的属性 名相同，但类型不相同 | 抛异常
A->B | serialId 不相同 | 正常传输

    总结：会抛异常的情况：枚举值一边多一种，一边少一种，正好使用了差别的那种，或者属性名相同，类型不同

接口增加方法，对客户端无影响，如果该方法不是客户端需要的，客户端不需要重新部署。输入参数和结果集中增加属性，对客户端无影响，如果客户端并不需要新属性，不用重新部署。

输入参数和结果集属性名变化，对客户端序列化无影响，但是如果客户端不重新部署，不管输入还是输出，属性名变化的属性值是获取不到的。

总结：服务器端和客户端对领域对象并不需要完全一致，而是按照最大匹配原则。

(2) 配置：

**dubbo.properties**

```sh
dubbo.service.protocol=dubbo
```

#### rmi://

RMI协议采用JDK标准的java.rmi.*实现，采用阻塞式短连接和JDK标准序列化方式。

如果正在使用RMI提供服务给外部访问（公司内网环境应该不会有攻击风险），同时应用里依赖了老的common-collections包（dubbo不会依赖这个包，请排查自己的应用有没有使用）的情况下，存在反序列化安全风险。

请检查应用：

* 将commons-collections3 请升级到 [3.2.2版本](https://commons.apache.org/proper/commons-collections/release_3_2_2.html)
* 将commons-collections4 请升级到 [4.1版本](https://commons.apache.org/proper/commons-collections/release_4_1.html)

新版本的commons-collections解决了该问题

如果服务接口继承了java.rmi.Remote接口，可以和原生RMI互操作，即：

* 提供者用Dubbo的RMI协议暴露服务，消费者直接用标准RMI接口调用，
* 或者提供方用标准RMI暴露服务，消费方用Dubbo的RMI协议调用。

如果服务接口没有继承java.rmi.Remote接口

* 缺省Dubbo将自动生成一个com.xxx.XxxService$Remote的接口，并继承java.rmi.Remote接口，并以此接口暴露服务，
* 但如果设置了 `<dubbo:protocol name="rmi" codec="spring" />`，将不生成$Remote接口，而使用Spring的RmiInvocationHandler接口暴露服务，和Spring兼容。

定义 RMI 协议：

```xml
<dubbo:protocol name="rmi" port="1099" />
```

设置默认协议：

```xml
<dubbo:provider protocol="rmi" />
```

设置服务协议：

```xml
<dubbo:service protocol="rmi" />
```

多端口：

```xml
<dubbo:protocol id="rmi1" name="rmi" port="1099" />
<dubbo:protocol id="rmi2" name="rmi" port="2099" />
 
<dubbo:service protocol="rmi1" />
```

Spring 兼容性：

```xml
<dubbo:protocol name="rmi" codec="spring" />
```

Java标准的远程调用协议

* 连接个数：多连接
* 连接方式：短连接
* 传输协议：TCP
* 传输方式：同步传输
* 序列化：Java标准二进制序列化
* 适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。
* 适用场景：常规远程服务方法调用，与原生RMI服务互操作

(1) 约束：

* 参数及返回值需实现Serializable接口
* dubbo配置中的超时时间对rmi无效，需使用java启动参数设置：-Dsun.rmi.transport.tcp.responseTimeout=3000，参见下面的RMI配置

(2) 配置：

**dubbo.properties**

```
dubbo.service.protocol=rmi
```

(3) RMI配置：

```sh
java -Dsun.rmi.transport.tcp.responseTimeout=3000
```

更多RMI优化参数请查看：
http://download.oracle.com/docs/cd/E17409_01/javase/6/docs/technotes/guides/rmi/sunrmiproperties.html

#### hessian://

Hessian协议用于集成Hessian的服务，Hessian底层采用Http通讯，采用Servlet暴露服务，Dubbo缺省内嵌Jetty作为服务器实现。

[Hessian](http://hessian.caucho.com) 是Caucho开源的一个 RPC 框架，其通讯效率高于WebService和Java自带的序列化。

依赖：

```xml
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.7</version>
</dependency>
```

可以和原生Hessian服务互操作，即：

* 提供者用Dubbo的Hessian协议暴露服务，消费者直接用标准Hessian接口调用，
* 或者提供方用标准Hessian暴露服务，消费方用Dubbo的Hessian协议调用。

基于Hessian的远程调用协议。

* 连接个数：多连接
* 连接方式：短连接
* 传输协议：HTTP
* 传输方式：同步传输
* 序列化：Hessian二进制序列化
* 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
* 适用场景：页面传输，文件传输，或与原生hessian服务互操作

(1) 约束：

* 参数及返回值需实现Serializable接口
* 参数及返回值不能自定义实现List, Map, Number, Date, Calendar等接口，只能用JDK自带的实现，因为hessian会做特殊处理，自定义实现类中的属性值都会丢失。

(2) 配置：

定义 hessian 协议：

```xml
<dubbo:protocol name="hessian" port="8080" server="jetty" />
```

设置默认协议：

```xml
<dubbo:provider protocol="hessian" />
```

设置 service 协议：

```xml
<dubbo:service protocol="hessian" />
```

多端口：

```xml
<dubbo:protocol id="hessian1" name="hessian" port="8080" />
<dubbo:protocol id="hessian2" name="hessian" port="8081" />
```

直连：

```xml
<dubbo:reference id="helloService" interface="HelloWorld" url="hessian://10.20.153.10:8080/helloWorld" />
```

#### thrift://

2.3.0以上版本支持。

[Thrift](http://thrift.apache.org) 是Facebook捐给Apache的一个RPC框架

当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如service name，magic number等。使用dubbo thrift协议同样需要使用thrift的idl compiler编译生成相应的java代码，后续版本中会在这方面做一些增强。

示例：https://github.com/alibaba/dubbo/tree/master/dubbo-rpc/dubbo-rpc-thrift/src/test/java/com/alibaba/dubbo/rpc/protocol/thrift/examples

依赖：

```xml
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.8.0</version>
</dependency>
```

所有服务共用一个端口：(与原生Thrift不兼容)

```xml
<dubbo:protocol name="thrift" port="3030" />
```

Thrift不支持数据类型：null值 (不能在协议中传递null值)

#### memcached://

2.3.0以上版本支持。[Memcached](http://memcached.org/) 是一个高效的KV缓存服务器。

可以通过脚本或监控中心手工填写表单注册memcached服务的地址：

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("memcached://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));
```

然后在客户端使用时，不需要感知Memcached的地址：

```xml
<dubbo:reference id="cache" interface="http://10.20.160.198/wiki/display/dubbo/java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="cache" interface="http://10.20.160.198/wiki/display/dubbo/java.util.Map" url="memcached://10.20.153.10:11211" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" />
```

方法名建议和memcached的标准方法名相同，即：get(key), set(key, value), delete(key)。

如果方法名和memcached的标准方法名不相同，则需要配置映射关系：(其中"p:xxx"为spring的标准p标签)

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />
```

#### redis://

2.3.0以上版本支持。[Redis](http://redis.io) 是一个高效的KV存储服务器。

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("redis://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));
```

然后在客户端使用时，不需要感知Redis的地址：

```xml
<dubbo:reference id="store" interface="http://10.20.160.198/wiki/display/dubbo/java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="store" interface="http://10.20.160.198/wiki/display/dubbo/java.util.Map" url="redis://10.20.153.10:6379" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="store" interface="com.foo.StoreService" url="redis://10.20.153.10:6379" />
```

方法名建议和redis的标准方法名相同，即：get(key), set(key, value), delet(key)。

如果方法名和redis的标准方法名不相同，则需要配置映射关系：(其中"p:xxx"为spring的标准p标签)

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />
```
