[**HOME**](Home) > **常见问题解答**

### 常见问题解答

#### 1. 如果服务注册不上怎么办？

0. 检查dubbo的jar包有没有在classpath中，以及有没有重复的jar包
0. 检查有没有重复的dubbo.properties配置文件
0. 检查暴露服务的spring配置有没有加载
0. 检查beanId或beanName有没有重复
0. 查看有没有错误日志
    
    ```sh
    cat ~/output/logs/webx.log
    ```
0. 在服务提供者机器上测试与注册中心的网络是否通

    ```sh
    telnet 172.22.3.94 9090
    ```
0. 检查与注册中心的连接是否存在

    ```sh
    netstat -anp | grep 172.22.3.94
    ```
0. 如果是预发布机，检查hosts文件有没有正确绑定

    ```sh
    cat /etc/hosts
    ```
0. 实在不行，开启远程调试
    * 在服务器JVM参数中加入：`-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=7001,server=y,suspend=y`
注意线上只有7001和8080可以被线下访问，调试端口需用这两个之一，因注册是启动时行为，启动时必需挂起suspend=y
    * 在dubbo源码的DefaultRegistryService的registerService()方法中设置断点。
    * 在Eclipse的Debug按钮下拉菜单Debug Configurations中的Remote Java Applications中新增远程调试，并设置IP和端口，以及增加dubbo的源码，进行远程Debug调试。

#### 2. 出现RpcException: No provider available for remote service异常怎么办？

表示没有可用的服务提供者：

0. 检查连接的注册中心是否正确
0. 到注册中心查看相应的服务提供者是否存在
0. 检查服务提供者是否正常运行

#### 3. 出现调用超时com.alibaba.dubbo.remoting.TimeoutException异常怎么办？

通常是业务处理太慢，可在服务提供方执行：jstack PID > jstack.log 分析线程都卡在哪个方法调用上，这里就是慢的原因。如果不能调优性能，请将timeout设大。

#### 4. 出现hessian序列化失败com.caucho.hessian.client.HessianRuntimeException怎么办？

0. 检查服务方法的传入传出参数是否实现Serializable接口
0. 检查服务方法的传入传出参数是否继承了Number,Date,ArrayList,HashMap等hessian特殊化处理的类

#### 5. 出现Configuration problem: Unable to locate Spring NamespaceHandler for XML schema namespace [http://repo.alibaba-inc.com/schema/dubbo] 怎么办？

表示spring找不到 `<dubbo:...>` 配置的解析处理器。
通常是Dubbo的jar没有引入，请加入对Dubbo的依赖，或者是ClassLoader隔离，看是否有使用osgi或其它热加载机制。

#### 6. 出现"消息发送失败"异常怎么办？

通常是接口方法的传入传出参数未实现Serializable接口。

#### 7. 出现org.xml.sax.SAXParseException: cvc-elt.1: Cannot find the declaration of element 'beans'异常怎么办？

表示xsd加载失败

0. 检查spring版本，如果是spring2.0版本，因为该版本不能读取jar包内xsd，会读取外网的xsd，而线上环境通常不允许访问外网，
可修改/etc/hosts加入：(已将spring的xsd放在公司内部的maven仓库中)
    
    ```
    10.20.133.138  repo.alibaba-inc.com  www.springframework.org
    ```
spring2.5.x版本不存在此问题，可以考虑升级到2.5.x版本。
0. 检查有没有使用osgi的xsd，如果用了，需要将spring-osgi.jar及其依赖包加进来

#### 8. 项目依赖的三方库与Dubbo所依赖的版本冲突怎么办。

比如，项目使用的spring和commons.pool与dubbo冲突，dubbo使用的是spring2.5和commons.pool1.4，而项目中其它模块依赖的是spring2.0.1和commons.pool1.3。

0. 在Maven中，使用项目根pom.xml中的dependencyManagement版本仲裁解决：

    ```xml
<dependencyManagement>
  <dependencies>
    <dependency>
        <groupId>com.alibaba.external</groupId>
        <artifactId>sourceforge.spring</artifactId>
        <version>2.0.1</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba.external</groupId>
        <artifactId>jakarta.commons.poolg</artifactId>
        <version>1.3</version>
    </dependency>
  </dependencies>
</dependencyManagement>    
```

0. 在Antx中，使用项目根project.xml中版本仲裁解决：

    ```xml
<projects name="thirdpart">
    <project id="sourceforge/spring" version="2.0.1"/>
    <project id="jakarta/commons/pool" version="1.3"/>
</projects> 
```

#### 9. 出现java.util.concurrent.RejectedExecutionException或者Thread pool exhausted怎么办？

RejectedExecutionException表示线程池已经达到最大值，并且没有空闲连，拒绝执行了一些任务。
Thread pool exhausted通常是min和max不一样大时，表示当前已创建的连接用完，进行了一次扩充，创建了新线程，但不影响运行。
原因可能是连接池不够用，请调整dubbo.properites中的：

```sh
// 设成一样大，减少线程池收缩开销
dubbo.service.min.thread.pool.size=200
dubbo.service.max.thread.pool.size=200
```

配置项说明请参见：配置参考手册
如果线程池已经有200，还不够，通常是业务处理占用线程时间过长，需优化业务，可通过运行：

```sh
jstack 进程号 > jstack.txt
```

分析当前大多数线程都在干什么，从而分析出哪个地方是瓶颈，比如，如果大部分线程都在处理SQL，可能是数据库连接不够，或数据源配置错误，或SQL没走索引等。

#### 10. 出现com.alibaba.dubbo.registry.internal.rpc.exception.RpcLocalExceptionIoTargetIsNotConnected怎么办？

0. 检查注册中心是否开启白名单功能，如果开启，当IP不在白名单列表中，注册中心将拒绝连接。
0. 检查端口是否正确，注册中心有两个端口，一个为控制台HTTP端口，用于管理员查看数据，一个为程序注册服务用的TCP端口。

#### 11. 出现Remote server returns error: [6], Got invocation exception怎么办？

此异常表示Dubbo框架调用服务提供者的实现方法失败，并且不是方法本身的业务异常。
通常是服务消费者和服务提供者的API签名不一致引起，或者提供方比消费方少此函数。
一般是服务增加方法，或修改了方法签名，而双方用的服务API的jar包不一致。

#### 12. 出现Error closing connection/tbr-client java.lang.NullPointerException怎么办？

如果服务提供者先关闭，当注册中心通知服务消费者后，服务消费者会再次关闭与服务提供者的连接，而此时连接早已不存在，TBRemoting没有判断null，直接调用了close方法，所以会抛出空指针异常，由于TBRemoting源码由taobao管理，暂时未解决此BUG，但不影响使用，可忽略。Dubbo1.0.11-3以后版本已hack了taobao的代码，不存在此问题。

#### 13. 出现org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'xxxService': Initialization of bean failed; nested exception is java.lang.IllegalArgumentException: Method must not be null怎么办？

通常是classpath下存在spring多个版本的jar包，排除掉不需要的spring包即可。

#### 14. 出现Error setting property values; nested exception is org.springframework.beans.NotWritablePropertyException: Invalid property 'applicationName' of bean class [com.alibaba.dubbo.registry.internal.DefaultRegistryService]: Bean property 'applicationName' is not writable or has an invalid setter method.怎么办？

出现类似的dubbo某个类的属性没有setter方法的异常，通常是classpath下有多个不同版本的dubbo的jar包，导致配置文件与类不匹配。可以在程序中运行下面的代码发现重复的类或jar包：(代码中的类名视具体冲突而定)

```java
Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources("com/alibaba/dubbo/registry/internal/DefaultRegistryService.class");
while (urls.hasMoreElements()) {
    URL url = urls.nextElement();
    System.out.println(">>>>>>>>>>>>>>>>>>>>>>" + url.getFile());
}
```

#### 15. 服务提供者没挂，但在注册中心里看不到怎么办？

首先，确认服务提供者是否连接了正确的注册中心，不只是检查配置中的注册中心地址，而且要检查实际的网络连接。
其次，看服务提供者是否非常繁忙，比如压力测试，以至于没有CPU片段向注册中心发送心跳，这种情况，减小压力，将自动恢复。

#### 16. 出现ERROR monitor.StatLog -拒绝连接 java.net.ConnectException:拒绝连接 com.alibaba.dubbo.monitor.StatLog.sendStatData怎么办？

监控中心不可用，发送统计信息失败，不影响调用，但将丢失统计信息。

#### 17. 服务地址出现127.0.0.1怎么办？

Dubbo1.0.7以后版本不存在此问题，当发现本机IP为127.0.0.1时，将遍历所有网卡查找有效IP。

之前版本处理方式：

正确配置的IP映射，Linux下为/etc/hosts，Windows下为C:/WINDOWS/system32/drivers/etc/hosts

假设：ifconfig命令行结果为10.20.130.230，hostname命令行结果为test2，
则配置为：

```
127.0.0.1 localhost
10.20.130.230 test2
```

#### 18. 通过netstat -anp看到连接的注册中心和配置的不一样怎么办？

检查classpath下是否存在两个dubbo.properties文件：

```java
Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources("dubbo.properties");
while (urls.hasMoreElements()) {
   URL url = urls.nextElement();
   System.out.println(">>>>>>>>>>>>>>>>>>>>>>" + url.getFile());
}
```

#### 19. 客户端的异常信息里的errorcode是什么意思?

如Remote server returns error: [6], Got invocation exception

0. 收到消息的时候线程池拒绝处理
0. 服务提供者端未能根据服务名找到相应服务
0. 该服务调用时，服务提供者端不能加载参数类型对应的class
0. 参数不能被正确的反序列化
0. 不能正确从Class中create该调用所指示的方法
0. 不能正确调用该方法

#### 20. 出现expected string at 0×33 java.lang.String 怎么办？

这是Hessian3.2.1的一个BUG，Dubbo内部使用Hessian3.2.1做序列化，升级到Dubbo1.0.14以上版本，已解决此问题。
具体原因参见：http://pt.alibaba-inc.com/wp/experience_929/hessian-big-string-serialize-problems.html

#### 21. 预发布环境，在本地的/etc/hosts文件作了对注册中心的绑定，为什么服务还是注册到生产环境的注册中心？

antx.properties中配置的 dubbo.registry.address = dubbo-reg1.hst.xyi.cn.alidc.net dubbo-reg2.hst.xyi.cn.alidc.net dubbo-reg3.hst.xyi.cn.alidc.net dubbo-reg4.hst.xyi.cn.alidc.net

而/etc/hosts里的绑定如下：
172.22.14.13 dubbo-reg1.hst.xyi.cn.alidc.net dubbo-reg2.hst.xyi.cn.alidc.net

两边的不一致，导致该问题出现。

将绑定修改为：172.22.14.13 dubbo-reg1.hst.xyi.cn.alidc.net dubbo-reg2.hst.xyi.cn.alidc.net dubbo-reg3.hst.xyi.cn.alidc.net dubbo-reg4.hst.xyi.cn.alidc.net 即可！

#### 22. 注册中心上服务是存在的，为什么报找不到服务的错误？

报错信息： Caused by: com.alibaba.dubbo.rpc.RpcException: No invoker available for remote service com.alibaba.china.album.service.IBankNewPicService:1.0.0, servers: []

注册中心上看到的服务提供者提供的服务地址是：dubbo://172.29.61.76:55372?version=1.0.0&group=ibank&dubbo=1.0.0&application=ibank

原因：服务提供者配置了group属性，默认的路由规则是服务名= group/serviceName。

对这个服务，根据默认的路由规则，消费者消费的服务名应该是 ibank/com.alibaba.china.album.service.IBankNewPicService:1.0.0

#### 23. 获取版本号出现java.lang.NullPointerException怎么办？

java.lang.NullPointerException
at com.alibaba.dubbo.classic.DubboVersion.getVersion

这个只有1.0.14和1.0.14-2存在的问题，在获取版本时静态字段初始化顺序不对，不影使用，可忽略，升级为1.0.15以上版本不再会有该问题。

#### 24. 以及配置中如何使用占位符？

> 注：此为Spring的标准功能，仅在此提示使用方式，不属于Dubbo范畴。

使用Spring自带的PropertyPlaceholderConfigurer实现properties配置：

```
xxx=10.20.130.230:9090
yyy=morgan
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://repo.alibaba-inc.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://repo.alibaba-inc.com/schema/dubbohttp://repo.alibaba-inc.com/schema/dubbo/dubbo-component.xsd">
 
    <!-- 使用Spring自带的占位符替换功能 -->
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <!-- 指定properties配置所在位置 -->
        <property name="location" value="classpath:xxx.properties" />
    </bean>
 
    <!-- 使用${}引用配置项 -->
    <dubbo:registry address="${xxx}" application="${yyy}" />
 
</beans>
```

#### 25. 使用多个进程启动服务，端口冲突怎么办？

> 注：此为Spring的标准功能，仅在此提示使用方式，不属于Dubbo范畴。

使用Spring自带的PropertyPlaceholderConfigurer的SYSTEM_PROPERTIES_MODE_OVERRIDE实现通过-D参数设置端口：

```sh
java -Ddubbo.service.server.port=20881
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://repo.alibaba-inc.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans.xsdhttp://repo.alibaba-inc.com/schema/dubbohttp://repo.alibaba-inc.com/schema/dubbo/dubbo-component.xsd">
 
    <!-- 使用Spring自带的占位符替换功能 -->
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <!-- 系统-D参数覆盖 -->
        <property name="systemPropertiesModeName" value="SYSTEM_PROPERTIES_MODE_OVERRIDE" />
        <!-- 指定properties配置所在位置 -->
        <property name="location" value="classpath:xxx.properties" />
    </bean>
 
    <!-- 使用${}引用配置项 -->
    <dubbo:provider port="${dubbo.service.server.port}" />
 
</beans>
```

#### 26. 如何加载Spring？

> 注：此为Spring的标准功能，仅在此提示使用方式，不属于Dubbo范畴。

0. 基于ClassPath加载：

    ```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[}{"http://10.20.160.198/wiki/display/dubbo/service.xml"});
context.start(); 
```
0. 基于文件系统加载：
    
    ```java
FileSystemXmlApplicationContext context = new FileSystemXmlApplicationContext(new String[}{"http://10.20.160.198/home/xxx/service.xml"});
context.start(); 
```
0. 基于Web容器加载：(WEB-INF/web.xml)
    
    ```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_4.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_4.xsd"
    id="appication" version="2.4">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:service.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app> 
```

#### 27. 出现org.xml.sax.SAXParseException: cvc-complex-type.2.4.c: The matching wildcard is strict, but no declaration can be found for element怎么办？

通常是在用Dubbo1.0的jar包，却用了Dubbo2.0才支持的`<dubbo:registry>` `<dubbo:application>` `<dubbo:provider>` `<dubbo:consumer>`或2.0才支持的属性。

#### 28. 出现Could not deserialize parameter instance, error is: readObject: unexpected end of file怎么办？

通常是消费方或提供方的内存不足，导致buffer不能分配，使发送到一半的请求被中断了。
也可能是网络抖动，导致传输流被中断。

#### 29. 出现java.net.SocketException: Invalid argument: sun.nio.ch.Net.setIntOption怎么办？

通常是Windows Vista和Windows7的JDK1.6的部分版本存在BUG：
https://issues.apache.org/jira/browse/DIRMINA-379
可以换换JDK版本试试。


