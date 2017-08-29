[**HOME**](Home) > **用户指南**

### API参考手册

Dubbo的常规功能，都保持零侵入，但有些功能不得不用API侵入才能实现。

Dubbo中除这里声明以外的接口或类，都是内部接口或扩展接口，普通用户请不要直接依赖，否则升级版本可能出现不兼容。

API汇总如下：

#### 配置API

* com.alibaba.dubbo.config.ServiceConfig
* com.alibaba.dubbo.config.ReferenceConfig
* com.alibaba.dubbo.config.ProtocolConfig
* com.alibaba.dubbo.config.RegistryConfig
* com.alibaba.dubbo.config.MonitorConfig
* com.alibaba.dubbo.config.ApplicationConfig
* com.alibaba.dubbo.config.ModuleConfig
* com.alibaba.dubbo.config.ProviderConfig
* com.alibaba.dubbo.config.ConsumerConfig
* com.alibaba.dubbo.config.MethodConfig
* com.alibaba.dubbo.config.ArgumentConfig 

> 参见：[API配置](user-guide-configuration#api配置)

#### 注解API

* com.alibaba.dubbo.config.annotation.Service
* com.alibaba.dubbo.config.annotation.Reference

> 参见：[注解配置](user-guide-configuration#注解配置)

#### 模型API

* com.alibaba.dubbo.common.URL
* com.alibaba.dubbo.rpc.RpcException

### 上下文API

* com.alibaba.dubbo.rpc.RpcContext

> 参见：[上下文信息](user-guide-sample#上下文信息) & [对方地址]() & [隐式传参](user-guide-sample#隐式传参) & [异步调用](user-guide-sample#异步调用)

#### 服务API

* com.alibaba.dubbo.rpc.service.GenericService
* com.alibaba.dubbo.rpc.service.GenericException

> 参见：[泛化引用](user-guide-sample#泛化引用) & [泛化实现](user-guide-sample#泛化实现)

* com.alibaba.dubbo.rpc.service.EchoService

> 参见：[回声测试](user-guide-sample#回声测试)
