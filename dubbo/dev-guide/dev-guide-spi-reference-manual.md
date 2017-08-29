[**HOME**](Home) > **开发指南**

### SPI参考手册

> **SPI使用范围**
扩展接口仅用于系统集成，或Contributor扩展功能插件。

* [协议扩展](#协议扩展)
* [调用拦截扩展](#调用拦截扩展)
* [引用监听扩展](#引用监听扩展)
* [暴露监听扩展](#暴露监听扩展)
* [集群扩展](#集群扩展)
* [路由扩展](#路由扩展)
* [负载均衡扩展](#负载均衡扩展)
* [合并结果扩展](#合并结果扩展)
* [注册中心扩展](#注册中心扩展)
* [监控中心扩展](#监控中心扩展)
* [扩展点加载扩展](#扩展点加载扩展)
* [动态代理扩展](#动态代理扩展)
* [编译器扩展](#编译器扩展)
* [消息派发扩展](#消息派发扩展)
* [线程池扩展](#线程池扩展)
* [序列化扩展](#序列化扩展)
* [网络传输扩展](#网络传输扩展)
* [信息交换扩展](#信息交换扩展)
* [组网扩展](#组网扩展)
* [Telnet命令扩展](#Telnet命令扩展)
* [状态检查扩展](#状态检查扩展)
* [容器扩展](#容器扩展)
* [页面扩展](#页面扩展)
* [缓存扩展](#缓存扩展)
* [验证扩展](#验证扩展)
* [日志适配扩展](#日志适配扩展)


#### 协议扩展

##### 1. 扩展说明：

RPC协议扩展，封装远程调用细节。

契约：

* 当用户调用 `refer()` 所返回的 `Invoker` 对象的 `invoke()` 方法时，协议需相应执行同 URL 远端 `export()` 传入的 `Invoker` 对象的 `invoke()` 方法。
* 其中，`refer()` 返回的 `Invoker` 由协议实现，协议通常需要在此 `Invoker` 中发送远程请求，`export()` 传入的 `Invoker` 由框架实现并传入，协议不需要关心。

注意：

* 协议不关心业务接口的透明代理，以 `Invoker` 为中心，由外层将 `Invoker` 转换为业务接口。
* 协议不一定要是 TCP 网络通讯，比如通过共享文件，IPC 进程间通讯等。

##### 2. 扩展接口：

* `com.alibaba.dubbo.rpc.Protocol`
* `com.alibaba.dubbo.rpc.Exporter`
* `com.alibaba.dubbo.rpc.Invoker`

```java
public interface Protocol {
    /**
     * 暴露远程服务：<br>
     * 1. 协议在接收请求时，应记录请求来源方地址信息：RpcContext.getContext().setRemoteAddress();<br>
     * 2. export()必须是幂等的，也就是暴露同一个URL的Invoker两次，和暴露一次没有区别。<br>
     * 3. export()传入的Invoker由框架实现并传入，协议不需要关心。<br>
     * 
     * @param <T> 服务的类型
     * @param invoker 服务的执行体
     * @return exporter 暴露服务的引用，用于取消暴露
     * @throws RpcException 当暴露服务出错时抛出，比如端口已占用
     */
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;
 
    /**
     * 引用远程服务：<br>
     * 1. 当用户调用refer()所返回的Invoker对象的invoke()方法时，协议需相应执行同URL远端export()传入的Invoker对象的invoke()方法。<br>
     * 2. refer()返回的Invoker由协议实现，协议通常需要在此Invoker中发送远程请求。<br>
     * 3. 当url中有设置check=false时，连接失败不能抛出异常，需内部自动恢复。<br>
     * 
     * @param <T> 服务的类型
     * @param type 服务的类型
     * @param url 远程服务的URL地址
     * @return invoker 服务的本地代理
     * @throws RpcException 当连接服务提供方失败时抛出
     */
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;
 
}
```

##### 3. 扩展配置：

```xml
<dubbo:protocol id="xxx1" name="xxx" /> <!-- 声明协议，如果没有配置id，将以name为id -->
<dubbo:service protocol="xxx1" /> <!-- 引用协议，如果没有配置protocol属性，将在ApplicationContext中自动扫描protocol配置 -->
<dubbo:provider protocol="xxx1" /> <!-- 引用协议缺省值，当<dubbo:service>没有配置prototol属性时，使用此配置 -->
```

##### 4. 已知扩展：

* `com.alibaba.dubbo.rpc.injvm.InjvmProtocol`
* `com.alibaba.dubbo.rpc.dubbo.DubboProtocol`
* `com.alibaba.dubbo.rpc.rmi.RmiProtocol`
* `com.alibaba.dubbo.rpc.http.HttpProtocol`
* `com.alibaba.dubbo.rpc.http.hessian.HessianProtocol`

##### 5. 扩展示例：

Maven项目结构

```

src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxProtocol.java (实现Protocol接口)
                |-XxxExporter.java (实现Exporter接口)
                |-XxxInvoker.java (实现Invoker接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.Protocol (纯文本文件，内容为：xxx=com.xxx.XxxProtocol)
```

XxxProtocol.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.Protocol;
 
public class XxxProtocol implements Protocol {
    public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
        return new XxxExporter(invoker);
    }
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        return new XxxInvoker(type, url);
    }
}
```

XxxExporter.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.support.AbstractExporter;
 
public class XxxExporter<T> extends AbstractExporter<T> {
    public XxxExporter(Invoker<T> invoker) throws RemotingException{
        super(invoker);
        // ...
    }
    public void unexport() {
        super.unexport();
        // ...
    }
}
```

XxxInvoker.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.support.AbstractInvoker;
 
public class XxxInvoker<T> extends AbstractInvoker<T> {
    public XxxInvoker(Class<T> type, URL url) throws RemotingException{
        super(type, url);
    }
    protected abstract Object doInvoke(Invocation invocation) throws Throwable {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.Protocol

```
xxx=com.xxx.XxxProtocol
```

#### 调用拦截扩展

##### 1. 扩展说明

服务提供方和服务消费方调用过程拦截，Dubbo 本身的大多功能均基于此扩展点实现，每次远程方法执行，该拦截都会被执行，请注意对性能的影响。

约定：

* 用户自定义 filter 默认在内置 filter 之后。
* 特殊值 `default`，表示缺省扩展点插入的位置。比如：`filter="xxx,default,yyy"`，表示 `xxx` 在缺省 filter 之前，`yyy` 在缺省 filter 之后。
* 特殊符号 `-`，表示剔除。比如：`filter="-foo1"`，剔除添加缺省扩展点 `foo1`。比如：`filter="-default"`，剔除添加所有缺省扩展点。
* provider 和 service 同时配置的 filter 时，累加所有 filter，而不是覆盖。比如：`<dubbo:provider filter="xxx,yyy"/>` 和 `<dubbo:service filter="aaa,bbb" />`，则 `xxx`,`yyy`,`aaa`,`bbb` 均会生效。如果要覆盖，需配置：`<dubbo:service filter="-xxx,-yyy,aaa,bbb" />`

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.Filter`

##### 3. 扩展配置

```xml
<dubbo:reference filter="xxx,yyy" /> <!-- 消费方调用过程拦截 -->
<dubbo:consumer filter="xxx,yyy"/> <!-- 消费方调用过程缺省拦截器，将拦截所有reference -->
<dubbo:service filter="xxx,yyy" /> <!-- 提供方调用过程拦截 -->
<dubbo:provider filter="xxx,yyy"/> <!-- 提供方调用过程缺省拦截器，将拦截所有service -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.filter.EchoFilter`
* `com.alibaba.dubbo.rpc.filter.GenericFilter`
* `com.alibaba.dubbo.rpc.filter.GenericImplFilter`
* `com.alibaba.dubbo.rpc.filter.TokenFilter`
* `com.alibaba.dubbo.rpc.filter.AccessLogFilter`
* `com.alibaba.dubbo.rpc.filter.CountFilter`
* `com.alibaba.dubbo.rpc.filter.ActiveLimitFilter`
* `com.alibaba.dubbo.rpc.filter.ClassLoaderFilter`
* `com.alibaba.dubbo.rpc.filter.ContextFilter`
* `com.alibaba.dubbo.rpc.filter.ConsumerContextFilter`
* `com.alibaba.dubbo.rpc.filter.ExceptionFilter`
* `com.alibaba.dubbo.rpc.filter.ExecuteLimitFilter`
* `com.alibaba.dubbo.rpc.filter.DeprecatedFilter`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxFilter.java (实现Filter接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.Filter (纯文本文件，内容为：xxx=com.xxx.XxxFilter)
```

XxxFilter.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.Filter;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.Invocation;
import com.alibaba.dubbo.rpc.Result;
import com.alibaba.dubbo.rpc.RpcException;
 
 
public class XxxFilter implements Filter {
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // before filter ...
        Result result = invoker.invoke(invocation);
        // after filter ...
        return result;
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.Filter

```
xxx=com.xxx.XxxFilter
```

#### 引用监听扩展

##### 1. 扩展说明

当有服务引用时，触发该事件。

##### 2.  扩展接口

`com.alibaba.dubbo.rpc.InvokerListener`

##### 3. 扩展配置

```xml
<dubbo:reference listener="xxx,yyy" /> <!-- 引用服务监听 -->
<dubbo:consumer listener="xxx,yyy" /> <!-- 引用服务缺省监听器 -->
```

##### 4. 已知扩展

`com.alibaba.dubbo.rpc.listener.DeprecatedInvokerListener`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxInvokerListener.java (实现InvokerListener接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.InvokerListener (纯文本文件，内容为：xxx=com.xxx.XxxInvokerListener)
```

XxxInvokerListener.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.InvokerListener;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.RpcException;
 
 
public class XxxInvokerListener implements InvokerListener {
    public void referred(Invoker<?> invoker) throws RpcException {
        // ...
    }
    public void destroyed(Invoker<?> invoker) throws RpcException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.InvokerListener

```
xxx=com.xxx.XxxInvokerListener
```

#### 暴露监听扩展

##### 1. 扩展说明

当有服务暴露时，触发该事件。

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.ExporterListener`

##### 3. 扩展配置

```xml
<dubbo:service listener="xxx,yyy" /> <!-- 暴露服务监听 -->
<dubbo:provider listener="xxx,yyy" /> <!-- 暴露服务缺省监听器 -->
```

##### 4. 已知扩展

`com.alibaba.dubbo.registry.directory.RegistryExporterListener`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxExporterListener.java (实现ExporterListener接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.ExporterListener (纯文本文件，内容为：xxx=com.xxx.XxxExporterListener)
```

XxxExporterListener.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.ExporterListener;
import com.alibaba.dubbo.rpc.Exporter;
import com.alibaba.dubbo.rpc.RpcException;
 
 
public class XxxExporterListener implements ExporterListener {
    public void exported(Exporter<?> exporter) throws RpcException {
        // ...
    }
    public void unexported(Exporter<?> exporter) throws RpcException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.ExporterListener

```
xxx=com.xxx.XxxExporterListener
```

#### 集群扩展

##### 1. 扩展说明

当有多个服务提供方时，将多个服务提供方组织成一个集群，并伪装成一个提供方。

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.cluster.Cluster`

##### 3. 扩展配置

```xml
<dubbo:protocol cluster="xxx" />
<dubbo:provider cluster="xxx" /> <!-- 缺省值配置，如果<dubbo:protocol>没有配置cluster时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.cluster.support.FailoverCluster`
* `com.alibaba.dubbo.rpc.cluster.support.FailfastCluster`
* `com.alibaba.dubbo.rpc.cluster.support.FailsafeCluster`
* `com.alibaba.dubbo.rpc.cluster.support.FailbackCluster`
* `com.alibaba.dubbo.rpc.cluster.support.ForkingCluster`
* `com.alibaba.dubbo.rpc.cluster.support.AvailableCluster`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxCluster.java (实现Cluster接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.cluster.Cluster (纯文本文件，内容为：xxx=com.xxx.XxxCluster)
```

XxxCluster.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.cluster.Cluster;
import com.alibaba.dubbo.rpc.cluster.support.AbstractClusterInvoker;
import com.alibaba.dubbo.rpc.cluster.Directory;
import com.alibaba.dubbo.rpc.cluster.LoadBalance;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.Invocation;
import com.alibaba.dubbo.rpc.Result;
import com.alibaba.dubbo.rpc.RpcException;
 
public class XxxCluster implements Cluster {
    public <T> Invoker<T> merge(Directory<T> directory) throws RpcException {
        return new AbstractClusterInvoker<T>(directory) {
            public Result doInvoke(Invocation invocation, List<Invoker<T>> invokers, LoadBalance loadbalance) throws RpcException {
                // ...
            }
        };
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.cluster.Cluster

```
xxx=com.xxx.XxxCluster
```

#### 路由扩展

##### 1. 扩展说明

从多个服务提者方中选择一个进行调用。

##### 2. 扩展接口

* `com.alibaba.dubbo.rpc.cluster.RouterFactory`
* `com.alibaba.dubbo.rpc.cluster.Router`

##### 3. 扩展配置

```xml
<dubbo:protocol router="xxx" />
<dubbo:provider router="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置loadbalance时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.cluster.router.ScriptRouterFactory`
* `com.alibaba.dubbo.rpc.cluster.router.FileRouterFactory`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxRouterFactory.java (实现LoadBalance接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.cluster.RouterFactory (纯文本文件，内容为：xxx=com.xxx.XxxRouterFactory)

```

> XxxRouterFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.cluster.RouterFactory;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.Invocation;
import com.alibaba.dubbo.rpc.RpcException;
 
public class XxxRouterFactory implements RouterFactory {
    public <T> List<Invoker<T>> select(List<Invoker<T>> invokers, Invocation invocation) throws RpcException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.cluster.RouterFactory

```
xxx=com.xxx.XxxRouterFactory
```

#### 负载均衡扩展

##### 1. 扩展说明

从多个服务提者方中选择一个进行调用

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.cluster.LoadBalance`

##### 3. 扩展配置

```xml
<dubbo:protocol loadbalance="xxx" />
<dubbo:provider loadbalance="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置loadbalance时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.cluster.loadbalance.RandomLoadBalance`
* `com.alibaba.dubbo.rpc.cluster.loadbalance.RoundRobinLoadBalance`
* `com.alibaba.dubbo.rpc.cluster.loadbalance.LeastActiveLoadBalance`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxLoadBalance.java (实现LoadBalance接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.cluster.LoadBalance (纯文本文件，内容为：xxx=com.xxx.XxxLoadBalance)
```

XxxLoadBalance.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.cluster.LoadBalance;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.Invocation;
import com.alibaba.dubbo.rpc.RpcException; 
 
public class XxxLoadBalance implements LoadBalance {
    public <T> Invoker<T> select(List<Invoker<T>> invokers, Invocation invocation) throws RpcException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.cluster.LoadBalance

```
xxx=com.xxx.XxxLoadBalance
```

#### 合并结果扩展

##### 1. 扩展说明

合并返回结果，用于分组聚合。

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.cluster.Merger`

##### 3. 扩展配置

```xml
<dubbo:method merger="xxx" />
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.cluster.merger.ArrayMerger`
* `com.alibaba.dubbo.rpc.cluster.merger.ListMerger`
* `com.alibaba.dubbo.rpc.cluster.merger.SetMerger`
* `com.alibaba.dubbo.rpc.cluster.merger.MapMerger`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxMerger.java (实现Merger接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.cluster.Merger (纯文本文件，内容为：xxx=com.xxx.XxxMerger)
```

XxxMerger.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.cluster.Merger;
 
public class XxxMerger<T> implements Merger<T> {
    public T merge(T... results) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.cluster.Merger

```
xxx=com.xxx.XxxMerger
```

#### 注册中心扩展

##### 1. 扩展说明

负责服务的注册与发现。

##### 2. 扩展接口

* `com.alibaba.dubbo.registry.RegistryFactory`
* `com.alibaba.dubbo.registry.Registry`

##### 3. 扩展配置

```xml
<dubbo:registry id="xxx1" address="xxx://ip:port" /> <!-- 定义注册中心 -->
<dubbo:service registry="xxx1" /> <!-- 引用注册中心，如果没有配置registry属性，将在ApplicationContext中自动扫描registry配置 -->
<dubbo:provider registry="xxx1" /> <!-- 引用注册中心缺省值，当<dubbo:service>没有配置registry属性时，使用此配置 -->
```

##### 4. 扩展契约

RegistryFactory.java

```java
public interface RegistryFactory {
    /**
     * 连接注册中心.
     * 
     * 连接注册中心需处理契约：<br>
     * 1. 当设置check=false时表示不检查连接，否则在连接不上时抛出异常。<br>
     * 2. 支持URL上的username:password权限认证。<br>
     * 3. 支持backup=10.20.153.10备选注册中心集群地址。<br>
     * 4. 支持file=registry.cache本地磁盘文件缓存。<br>
     * 5. 支持timeout=1000请求超时设置。<br>
     * 6. 支持session=60000会话超时或过期设置。<br>
     * 
     * @param url 注册中心地址，不允许为空
     * @return 注册中心引用，总不返回空
     */
    Registry getRegistry(URL url); 
}
```

RegistryService.java

```java
public interface RegistryService { // Registry extends RegistryService 
    /**
     * 注册服务.
     * 
     * 注册需处理契约：<br>
     * 1. 当URL设置了check=false时，注册失败后不报错，在后台定时重试，否则抛出异常。<br>
     * 2. 当URL设置了dynamic=false参数，则需持久存储，否则，当注册者出现断电等情况异常退出时，需自动删除。<br>
     * 3. 当URL设置了category=overrides时，表示分类存储，缺省类别为providers，可按分类部分通知数据。<br>
     * 4. 当注册中心重启，网络抖动，不能丢失数据，包括断线自动删除数据。<br>
     * 5. 允许URI相同但参数不同的URL并存，不能覆盖。<br>
     * 
     * @param url 注册信息，不允许为空，如：dubbo://10.20.153.10/com.alibaba.foo.BarService?version=1.0.0&application=kylin
     */
    void register(URL url);
 
    /**
     * 取消注册服务.
     * 
     * 取消注册需处理契约：<br>
     * 1. 如果是dynamic=false的持久存储数据，找不到注册数据，则抛IllegalStateException，否则忽略。<br>
     * 2. 按全URL匹配取消注册。<br>
     * 
     * @param url 注册信息，不允许为空，如：dubbo://10.20.153.10/com.alibaba.foo.BarService?version=1.0.0&application=kylin
     */
    void unregister(URL url);
 
    /**
     * 订阅服务.
     * 
     * 订阅需处理契约：<br>
     * 1. 当URL设置了check=false时，订阅失败后不报错，在后台定时重试。<br>
     * 2. 当URL设置了category=overrides，只通知指定分类的数据，多个分类用逗号分隔，并允许星号通配，表示订阅所有分类数据。<br>
     * 3. 允许以interface,group,version,classifier作为条件查询，如：interface=com.alibaba.foo.BarService&version=1.0.0<br>
     * 4. 并且查询条件允许星号通配，订阅所有接口的所有分组的所有版本，或：interface=*&group=*&version=*&classifier=*<br>
     * 5. 当注册中心重启，网络抖动，需自动恢复订阅请求。<br>
     * 6. 允许URI相同但参数不同的URL并存，不能覆盖。<br>
     * 7. 必须阻塞订阅过程，等第一次通知完后再返回。<br>
     * 
     * @param url 订阅条件，不允许为空，如：consumer://10.20.153.10/com.alibaba.foo.BarService?version=1.0.0&application=kylin
     * @param listener 变更事件监听器，不允许为空
     */
    void subscribe(URL url, NotifyListener listener);
 
    /**
     * 取消订阅服务.
     * 
     * 取消订阅需处理契约：<br>
     * 1. 如果没有订阅，直接忽略。<br>
     * 2. 按全URL匹配取消订阅。<br>
     * 
     * @param url 订阅条件，不允许为空，如：consumer://10.20.153.10/com.alibaba.foo.BarService?version=1.0.0&application=kylin
     * @param listener 变更事件监听器，不允许为空
     */
    void unsubscribe(URL url, NotifyListener listener);
 
    /**
     * 查询注册列表，与订阅的推模式相对应，这里为拉模式，只返回一次结果。
     * 
     * @see com.alibaba.dubbo.registry.NotifyListener#notify(List)
     * @param url 查询条件，不允许为空，如：consumer://10.20.153.10/com.alibaba.foo.BarService?version=1.0.0&application=kylin
     * @return 已注册信息列表，可能为空，含义同{@link com.alibaba.dubbo.registry.NotifyListener#notify(List<URL>)}的参数。
     */
    List<URL> lookup(URL url);
 
}
```

NotifyListener.java

```java
public interface NotifyListener { 
    /**
     * 当收到服务变更通知时触发。
     * 
     * 通知需处理契约：<br>
     * 1. 总是以服务接口和数据类型为维度全量通知，即不会通知一个服务的同类型的部分数据，用户不需要对比上一次通知结果。<br>
     * 2. 订阅时的第一次通知，必须是一个服务的所有类型数据的全量通知。<br>
     * 3. 中途变更时，允许不同类型的数据分开通知，比如：providers, consumers, routes, overrides，允许只通知其中一种类型，但该类型的数据必须是全量的，不是增量的。<br>
     * 4. 如果一种类型的数据为空，需通知一个empty协议并带category参数的标识性URL数据。<br>
     * 5. 通知者(即注册中心实现)需保证通知的顺序，比如：单线程推送，队列串行化，带版本对比。<br>
     * 
     * @param urls 已注册信息列表，总不为空，含义同{@link com.alibaba.dubbo.registry.RegistryService#lookup(URL)}的返回值。
     */
    void notify(List<URL> urls);
 
}
```

##### 5. 已知扩展

`com.alibaba.dubbo.registry.support.dubbo.DubboRegistryFactory`

##### 6. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxRegistryFactoryjava (实现RegistryFactory接口)
                |-XxxRegistry.java (实现Registry接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.registry.RegistryFactory (纯文本文件，内容为：xxx=com.xxx.XxxRegistryFactory)
```

XxxRegistryFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.registry.RegistryFactory;
import com.alibaba.dubbo.registry.Registry;
import com.alibaba.dubbo.common.URL;
 
public class XxxRegistryFactory implements RegistryFactory {
    public Registry getRegistry(URL url) {
        return new XxxRegistry(url);
    }
}
```

XxxRegistry.java

```java
package com.xxx;
 
import com.alibaba.dubbo.registry.Registry;
import com.alibaba.dubbo.registry.NotifyListener;
import com.alibaba.dubbo.common.URL;
 
public class XxxRegistry implements Registry {
    public void register(URL url) {
        // ...
    }
    public void unregister(URL url) {
        // ...
    }
    public void subscribe(URL url, NotifyListener listener) {
        // ...
    }
    public void unsubscribe(URL url, NotifyListener listener) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.registry.RegistryFactory

```
xxx=com.xxx.XxxRegistryFactory
```

#### 监控中心扩展

##### 1. 扩展说明

负责服务调用次和调用时间的监控。

##### 2. 扩展接口

* `com.alibaba.dubbo.monitor.MonitorFactory`
* `com.alibaba.dubbo.monitor.Monitor`

##### 3. 扩展配置

```xml
<dubbo:monitor address="xxx://ip:port" /> <!-- 定义监控中心 -->
```

##### 4. 已知扩展

com.alibaba.dubbo.monitor.support.dubbo.DubboMonitorFactory

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxMonitorFactoryjava (实现MonitorFactory接口)
                |-XxxMonitor.java (实现Monitor接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.monitor.MonitorFactory (纯文本文件，内容为：xxx=com.xxx.XxxMonitorFactory)
```

XxxMonitorFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.monitor.MonitorFactory;
import com.alibaba.dubbo.monitor.Monitor;
import com.alibaba.dubbo.common.URL;
 
public class XxxMonitorFactory implements MonitorFactory {
    public Monitor getMonitor(URL url) {
        return new XxxMonitor(url);
    }
}
```

XxxMonitor.java

```java
package com.xxx;
 
import com.alibaba.dubbo.monitor.Monitor;
 
public class XxxMonitor implements Monitor {
    public void count(URL statistics) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.monitor.MonitorFactory

```
xxx=com.xxx.XxxMonitorFactory
```

#### 扩展点加载扩展

##### 1. 扩展说明

扩展点本身的加载容器，可从不同容器加载扩展点。

##### 2. 扩展接口

`com.alibaba.dubbo.common.extension.ExtensionFactory`

##### 3. 扩展配置

```xml
<dubbo:application compiler="jdk" />
```

##### 4. 已知扩展

* `com.alibaba.dubbo.common.extension.factory.SpiExtensionFactory`
* `com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxExtensionFactory.java (实现ExtensionFactory接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.extension.ExtensionFactory (纯文本文件，内容为：xxx=com.xxx.XxxExtensionFactory)
```

XxxExtensionFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.extension.ExtensionFactory;
 
public class XxxExtensionFactory implements ExtensionFactory {
    public Object getExtension(Class<?> type, String name) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.common.extension.ExtensionFactory

```
xxx=com.xxx.XxxExtensionFactory
```

#### 动态代理扩展

##### 1. 扩展说明

将 `Invoker` 接口转换成业务接口。

##### 2. 扩展接口

`com.alibaba.dubbo.rpc.ProxyFactory`

##### 3. 扩展配置

```xml
<dubbo:protocol proxy="xxx" />
<dubbo:provider proxy="xxx" /> <!-- 缺省值配置，当<dubbo:protocol>没有配置proxy属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.rpc.proxy.JdkProxyFactory`
* `com.alibaba.dubbo.rpc.proxy.JavassistProxyFactory`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxProxyFactory.java (实现ProxyFactory接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.rpc.ProxyFactory (纯文本文件，内容为：xxx=com.xxx.XxxProxyFactory)
```

XxxProxyFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.rpc.ProxyFactory;
import com.alibaba.dubbo.rpc.Invoker;
import com.alibaba.dubbo.rpc.RpcException;
 
 
public class XxxProxyFactory implements ProxyFactory {
    public <T> T getProxy(Invoker<T> invoker) throws RpcException {
        // ...
    }
    public <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.rpc.ProxyFactory

```
xxx=com.xxx.XxxProxyFactory
```

#### 编译器扩展

##### 1. 扩展说明

Java 代码编译器，用于动态生成字节码，加速调用。

##### 2. 扩展接口

`com.alibaba.dubbo.common.compiler.Compiler`

##### 3. 扩展配置

自动加载

##### 4. 已知扩展

* `com.alibaba.dubbo.common.compiler.support.JdkCompiler`
* `com.alibaba.dubbo.common.compiler.support.JavassistCompiler`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxCompiler.java (实现Compiler接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.compiler.Compiler (纯文本文件，内容为：xxx=com.xxx.XxxCompiler)
```

XxxCompiler.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.compiler.Compiler;
 
public class XxxCompiler implements Compiler {
    public Object getExtension(Class<?> type, String name) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.common.compiler.Compiler

```
xxx=com.xxx.XxxCompiler
```

#### 消息派发扩展

##### 1. 扩展说明

通道信息派发器，用于指定线程池模型。

##### 2. 扩展接口

`com.alibaba.dubbo.remoting.Dispatcher`

##### 3. 扩展配置

```xml
<dubbo:protocol dispatcher="xxx" />
<dubbo:provider dispatcher="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置dispatcher属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.remoting.transport.dispatcher.all.AllDispatcher`
* `com.alibaba.dubbo.remoting.transport.dispatcher.direct.DirectDispatcher`
* `com.alibaba.dubbo.remoting.transport.dispatcher.message.MessageOnlyDispatcher`
* `com.alibaba.dubbo.remoting.transport.dispatcher.execution.ExecutionDispatcher`
* `com.alibaba.dubbo.remoting.transport.dispatcher.connection.ConnectionOrderedDispatcher`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxDispatcher.java (实现Dispatcher接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.remoting.Dispatcher (纯文本文件，内容为：xxx=com.xxx.XxxDispatcher)
```

XxxDispatcher.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.Dispatcher;
 
public class XxxDispatcher implements Dispatcher {
    public Group lookup(URL url) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.remoting.Dispatcher

```
xxx=com.xxx.XxxDispatcher
```

#### 线程池扩展

##### 1. 扩展说明

服务提供方线程程实现策略，当服务器收到一个请求时，需要在线程池中创建一个线程去执行服务提供方业务逻辑。

##### 2. 扩展接口

`com.alibaba.dubbo.common.threadpool.ThreadPool`

##### 3. 扩展配置

```xml
<dubbo:protocol threadpool="xxx" />
<dubbo:provider threadpool="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置threadpool时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.common.threadpool.FixedThreadPool`
* `com.alibaba.dubbo.common.threadpool.CachedThreadPool`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxThreadPool.java (实现ThreadPool接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.threadpool.ThreadPool (纯文本文件，内容为：xxx=com.xxx.XxxThreadPool)
```

XxxThreadPool.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.threadpool.ThreadPool;
import java.util.concurrent.Executor;
 
public class XxxThreadPool implements ThreadPool {
    public Executor getExecutor() {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.common.threadpool.ThreadPool

```
xxx=com.xxx.XxxThreadPool
```

#### 序列化扩展

##### 1. 扩展说明

将对象转成字节流，用于网络传输，以及将字节流转为对象，用于在收到字节流数据后还原成对象。

##### 2. 扩展接口

* `com.alibaba.dubbo.common.serialize.Serialization`
* `com.alibaba.dubbo.common.serialize.ObjectInput`
* `com.alibaba.dubbo.common.serialize.ObjectOutput`

##### 3. 扩展配置

```xml
<dubbo:protocol serialization="xxx" /> <!-- 协议的序列化方式 -->
<dubbo:provider serialization="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置serialization时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.common.serialize.dubbo.DubboSerialization`
* `com.alibaba.dubbo.common.serialize.hessian.Hessian2Serialization`
* `com.alibaba.dubbo.common.serialize.java.JavaSerialization`
* `com.alibaba.dubbo.common.serialize.java.CompactedJavaSerialization`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxSerialization.java (实现Serialization接口)
                |-XxxObjectInput.java (实现ObjectInput接口)
                |-XxxObjectOutput.java (实现ObjectOutput接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.serialize.Serialization (纯文本文件，内容为：xxx=com.xxx.XxxSerialization)
```

XxxSerialization.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.serialize.Serialization;
import com.alibaba.dubbo.common.serialize.ObjectInput;
import com.alibaba.dubbo.common.serialize.ObjectOutput;
 
 
public class XxxSerialization implements Serialization {
    public ObjectOutput serialize(Parameters parameters, OutputStream output) throws IOException {
        return new XxxObjectOutput(output);
    }
    public ObjectInput deserialize(Parameters parameters, InputStream input) throws IOException {
        return new XxxObjectInput(input);
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.common.serialize.Serialization

```
xxx=com.xxx.XxxSerialization
```

#### 网络传输扩展

##### 1. 扩展说明

远程通讯的服务器及客户端传输实现。

##### 2. 扩展接口

* `com.alibaba.dubbo.remoting.Transporter`
* `com.alibaba.dubbo.remoting.Server`
* `com.alibaba.dubbo.remoting.Client`

##### 3. 扩展配置

```xml
<dubbo:protocol transporter="xxx" /> <!-- 服务器和客户端使用相同的传输实现 -->
<dubbo:protocol server="xxx" client="xxx" /> <!-- 服务器和客户端使用不同的传输实现 -->
<dubbo:provider transporter="xxx" server="xxx" client="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置transporter/server/client属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.remoting.transport.transporter.netty.NettyTransporter`
* `com.alibaba.dubbo.remoting.transport.transporter.mina.MinaTransporter`
* `com.alibaba.dubbo.remoting.transport.transporter.grizzly.GrizzlyTransporter`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxTransporter.java (实现Transporter接口)
                |-XxxServer.java (实现Server接口)
                |-XxxClient.java (实现Client接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.remoting.Transporter (纯文本文件，内容为：xxx=com.xxx.XxxTransporter)
```

XxxTransporter.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.Transporter;
 
public class XxxTransporter implements Transporter {
    public Server bind(URL url, ChannelHandler handler) throws RemotingException {
        return new XxxServer(url, handler);
    }
    public Client connect(URL url, ChannelHandler handler) throws RemotingException {
        return new XxxClient(url, handler);
    }
}
```

XxxServer.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.transport.transporter.AbstractServer;
 
public class XxxServer extends AbstractServer {
    public XxxServer(URL url, ChannelHandler handler) throws RemotingException{
        super(url, handler);
    }
    protected void doOpen() throws Throwable {
        // ...
    }
    protected void doClose() throws Throwable {
        // ...
    }
    public Collection<Channel> getChannels() {
        // ...
    }
    public Channel getChannel(InetSocketAddress remoteAddress) {
        // ...
    }
}
```

XxxClient.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.transport.transporter.AbstractClient;
 
public class XxxClient extends AbstractClient {
    public XxxServer(URL url, ChannelHandler handler) throws RemotingException{
        super(url, handler);
    }
    protected void doOpen() throws Throwable {
        // ...
    }
    protected void doClose() throws Throwable {
        // ...
    }
    protected void doConnect() throws Throwable {
        // ...
    }
    public Channel getChannel() {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.remoting.Transporter

```
xxx=com.xxx.XxxTransporter
```

#### 信息交换扩展

##### 1. 扩展说明

基于传输层之上，实现Request-Response信息交换语义。

##### 2. 扩展接口

* `com.alibaba.dubbo.remoting.exchange.Exchanger`
* `com.alibaba.dubbo.remoting.exchange.ExchangeServer`
* `com.alibaba.dubbo.remoting.exchange.ExchangeClient`

##### 3. 扩展配置

```xml
<dubbo:protocol exchanger="xxx" />
<dubbo:provider exchanger="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置exchanger属性时，使用此配置 -->
```

##### 4. 已知扩展

`com.alibaba.dubbo.remoting.exchange.exchanger.HeaderExchanger`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxExchanger.java (实现Exchanger接口)
                |-XxxExchangeServer.java (实现ExchangeServer接口)
                |-XxxExchangeClient.java (实现ExchangeClient接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.remoting.exchange.Exchanger (纯文本文件，内容为：xxx=com.xxx.XxxExchanger)
```

XxxExchanger.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.exchange.Exchanger;
 
 
public class XxxExchanger implements Exchanger {
    public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
        return new XxxExchangeServer(url, handler);
    }
    public ExchangeClient connect(URL url, ExchangeHandler handler) throws RemotingException {
        return new XxxExchangeClient(url, handler);
    }
}
```

XxxExchangeServer.java

```java

package com.xxx;
 
import com.alibaba.dubbo.remoting.exchange.ExchangeServer;
 
public class XxxExchangeServer impelements ExchangeServer {
    // ...
}
```

XxxExchangeClient.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.exchange.ExchangeClient;
 
public class XxxExchangeClient impelments ExchangeClient {
    // ...
}
```

META-INF/dubbo/com.alibaba.dubbo.remoting.exchange.Exchanger

```
xxx=com.xxx.XxxExchanger
```

#### 组网扩展

##### 1. 扩展说明

对等网络节点组网器。

##### 2. 扩展接口

`com.alibaba.dubbo.remoting.p2p.Networker`

##### 3. 扩展配置

```xml
<dubbo:protocol networker="xxx" />
<dubbo:provider networker="xxx" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置networker属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.remoting.p2p.support.MulticastNetworker`
* `com.alibaba.dubbo.remoting.p2p.support.FileNetworker`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxNetworker.java (实现Networker接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.remoting.p2p.Networker (纯文本文件，内容为：xxx=com.xxx.XxxNetworker)
```

XxxNetworker.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.p2p.Networker;
 
public class XxxNetworker implements Networker {
    public Group lookup(URL url) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.remoting.p2p.Networker

```
xxx=com.xxx.XxxNetworker
```

#### Telnet命令扩展

##### 1. 扩展说明

所有服务器均支持 telnet 访问，用于人工干预。

##### 2. 扩展接口

`com.alibaba.dubbo.remoting.telnet.TelnetHandler`

##### 3. 扩展配置

```xml
<dubbo:protocol telnet="xxx,yyy" />
<dubbo:provider telnet="xxx,yyy" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置telnet属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.remoting.telnet.support.ClearTelnetHandler`
* `com.alibaba.dubbo.remoting.telnet.support.ExitTelnetHandler`
* `com.alibaba.dubbo.remoting.telnet.support.HelpTelnetHandler`
* `com.alibaba.dubbo.remoting.telnet.support.StatusTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.ListTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.ChangeTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.CurrentTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.InvokeTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.TraceTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.CountTelnetHandler`
* `com.alibaba.dubbo.rpc.dubbo.telnet.PortTelnetHandler`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxTelnetHandler.java (实现TelnetHandler接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.remoting.telnet.TelnetHandler (纯文本文件，内容为：xxx=com.xxx.XxxTelnetHandler)
```

XxxTelnetHandler.java

```java
package com.xxx;
 
import com.alibaba.dubbo.remoting.telnet.TelnetHandler;
 
@Help(parameter="...", summary="...", detail="...")
 
public class XxxTelnetHandler implements TelnetHandler {
    public String telnet(Channel channel, String message) throws RemotingException {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.remoting.telnet.TelnetHandler

```
xxx=com.xxx.XxxTelnetHandler
```

用法

```sh
telnet 127.0.0.1 20880
dubbo> xxx args
```

#### 状态检查扩展

##### 1. 扩展说明

检查服务依赖各种资源的状态，此状态检查可同时用于 telnet 的 status 命令和 hosting 的 status 页面。

##### 2. 扩展接口

`com.alibaba.dubbo.common.status.StatusChecker`

##### 3. 扩展配置

```xml
<dubbo:protocol status="xxx,yyy" />
<dubbo:provider status="xxx,yyy" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置status属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.common.status.support.MemoryStatusChecker`
* `com.alibaba.dubbo.common.status.support.LoadStatusChecker`
* `com.alibaba.dubbo.rpc.dubbo.status.ServerStatusChecker`
* `com.alibaba.dubbo.rpc.dubbo.status.ThreadPoolStatusChecker`
* `com.alibaba.dubbo.registry.directory.RegistryStatusChecker`
* `com.alibaba.dubbo.rpc.config.spring.status.SpringStatusChecker`
* `com.alibaba.dubbo.rpc.config.spring.status.DataSourceStatusChecker`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxStatusChecker.java (实现StatusChecker接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.status.StatusChecker (纯文本文件，内容为：xxx=com.xxx.XxxStatusChecker)
```

XxxStatusChecker.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.status.StatusChecker;
 
public class XxxStatusChecker implements StatusChecker {
    public Status check() {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.common.status.StatusChecker

```
xxx=com.xxx.XxxStatusChecker
```

#### 容器扩展

##### 1. 扩展说明

服务容器扩展，用于自定义加载内容。

##### 2. 扩展接口

`com.alibaba.dubbo.container.Container`

##### 3. 扩展配置

```sh
java com.alibaba.dubbo.container.Main spring jetty log4j
```

##### 4. 已知扩展

* `com.alibaba.dubbo.container.spring.SpringContainer`
* `com.alibaba.dubbo.container.spring.JettyContainer`
* `com.alibaba.dubbo.container.spring.Log4jContainer`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxContainer.java (实现Container接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.container.Container (纯文本文件，内容为：xxx=com.xxx.XxxContainer)
```

XxxContainer.java

```java
package com.xxx;
 
com.alibaba.dubbo.container.Container;
 
 
public class XxxContainer implements Container {
    public Status start() {
        // ...
    }
    public Status stop() {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.container.Container

```
xxx=com.xxx.XxxContainer
```

#### 页面扩展

##### 1. 扩展说明

对等网络节点组网器。

##### 2. 扩展接口

`com.alibaba.dubbo.container.page.PageHandler`

##### 3. 扩展配置

```xml
<dubbo:protocol page="xxx,yyy" />
<dubbo:provider page="xxx,yyy" /> <!-- 缺省值设置，当<dubbo:protocol>没有配置page属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.container.page.pages.HomePageHandler`
* `com.alibaba.dubbo.container.page.pages.StatusPageHandler`
* `com.alibaba.dubbo.container.page.pages.LogPageHandler`
* `com.alibaba.dubbo.container.page.pages.SystemPageHandler`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxPageHandler.java (实现PageHandler接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.container.page.PageHandler (纯文本文件，内容为：xxx=com.xxx.XxxPageHandler)
```

XxxPageHandler.java

```java
package com.xxx;
 
import com.alibaba.dubbo.container.page.PageHandler;
 
public class XxxPageHandler implements PageHandler {
    public Group lookup(URL url) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.container.page.PageHandler

```
xxx=com.xxx.XxxPageHandler
```

#### 缓存扩展

##### 1. 扩展说明

用请求参数作为key，缓存返回结果。

##### 2. 扩展接口

`com.alibaba.dubbo.cache.CacheFactory`

##### 3. 扩展配置

```xml
<dubbo:service cache="lru" />
<dubbo:service><dubbo:method cache="lru" /></dubbo:service> <!-- 方法级缓存 -->
<dubbo:provider cache="xxx,yyy" /> <!-- 缺省值设置，当<dubbo:service>没有配置cache属性时，使用此配置 -->
```

##### 4. 已知扩展

* `com.alibaba.dubbo.cache.support.lru.LruCacheFactory`
* `com.alibaba.dubbo.cache.support.threadlocal.ThreadLocalCacheFactory`
* `com.alibaba.dubbo.cache.support.jcache.JCacheFactory`


##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxCacheFactory.java (实现StatusChecker接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.cache.CacheFactory (纯文本文件，内容为：xxx=com.xxx.XxxCacheFactory)
```

XxxCacheFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.cache.CacheFactory;
 
public class XxxCacheFactory implements CacheFactory {
    public Cache getCache(URL url, String name) {
        return new XxxCache(url, name);
    }
}
```

XxxCacheFactory.java

```java
package com.xxx;
 
import com.alibaba.dubbo.cache.Cache;
 
public class XxxCache implements Cache {
    public Cache(URL url, String name) {
        // ...
    }
    public void put(Object key, Object value) {
        // ...
    }
    public Object get(Object key) {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.cache.CacheFactory

```
xxx=com.xxx.XxxCacheFactory
```

##### 验证扩展

##### 1. 扩展说明

参数验证扩展点。

##### 2. 扩展接口

`com.alibaba.dubbo.validation.Validation`

##### 3. 扩展配置

```xml
<dubbo:service validation="xxx,yyy" />
<dubbo:provider validation="xxx,yyy" /> <!-- 缺省值设置，当<dubbo:service>没有配置validation属性时，使用此配置 -->
```

##### 4. 已知扩展

`com.alibaba.dubbo.validation.support.jvalidation.JValidation`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxValidation.java (实现Validation接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.validation.Validation (纯文本文件，内容为：xxx=com.xxx.XxxValidation)
```

> XxxValidation.java

```java
package com.xxx;
 
import com.alibaba.dubbo.validation.Validation;
 
public class XxxValidation implements Validation {
    public Object getValidator(URL url) {
        // ...
    }
}
```

XxxValidator.java

```java
package com.xxx;
 
import com.alibaba.dubbo.validation.Validator;
 
public class XxxValidator implements Validator {
    public XxxValidator(URL url) {
        // ...
    }
    public void validate(Invocation invocation) throws Exception {
        // ...
    }
}
```

META-INF/dubbo/com.alibaba.dubbo.validation.Validation

```
xxx=com.xxx.XxxValidation
```

#### 日志适配扩展

##### 1. 扩展说明

日志输出适配扩展点。

##### 2. 扩展接口

`com.alibaba.dubbo.common.logger.LoggerAdapter`

##### 3. 扩展配置

```xml
<dubbo:application logger="xxx" />
```

```sh
-Ddubbo:application.logger=xxx
```

##### 4. 已知扩展

* `com.alibaba.dubbo.common.logger.slf4j.Slf4jLoggerAdapter`
* `com.alibaba.dubbo.common.logger.jcl.JclLoggerAdapter`
* `com.alibaba.dubbo.common.logger.log4j.Log4jLoggerAdapter`
* `com.alibaba.dubbo.common.logger.jdk.JdkLoggerAdapter`

##### 5. 扩展示例

Maven项目结构

```
src
 |-main
    |-java
        |-com
            |-xxx
                |-XxxLoggerAdapter.java (实现LoggerAdapter接口)
    |-resources
        |-META-INF
            |-dubbo
                |-com.alibaba.dubbo.common.logger.LoggerAdapter (纯文本文件，内容为：xxx=com.xxx.XxxLoggerAdapter)
```

XxxLoggerAdapter.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.logger.LoggerAdapter;
 
public class XxxLoggerAdapter implements LoggerAdapter {
    public Logger getLogger(URL url) {
        // ...
    }
}
```

XxxLogger.java

```java
package com.xxx;
 
import com.alibaba.dubbo.common.logger.Logger;
 
public class XxxLogger implements Logger {
    public XxxLogger(URL url) {
        // ...
    }
    public void info(String msg) {
        // ...
    }
    // ...
}
```

META-INF/dubbo/com.alibaba.dubbo.common.logger.LoggerAdapter

```
xxx=com.xxx.XxxLoggerAdapter
```