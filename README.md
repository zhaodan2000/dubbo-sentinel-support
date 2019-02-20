# Sentinel Dubbo Adapter

> Note: 中文文档请见[此处](https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E6%B5%81%E6%A1%86%E6%9E%B6%E7%9A%84%E9%80%82%E9%85%8D#dubbo)。

Sentinel Dubbo Adapter provides service consumer filter and provider filter
for [Dubbo](http://dubbo.io/) services. 

To use Sentinel Dubbo Adapter, you can simply add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-dubbo-adapter</artifactId>
    <version>x.y.z</version>
</dependency>
```

> Note: currently this adapter is not compatible with Dubbo 2.7.x due to its package renaming.
We are working to support the latest Dubbo version.

The Sentinel filters are **enabled by default**. Once you add the dependency,
the Dubbo services and methods will become protected resources in Sentinel,
which can leverage Sentinel's flow control and guard ability when rules are configured.
Demos can be found in [sentinel-demo-dubbo](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-dubbo).

If you don't want the filters enabled, you can manually disable them. For example:

```xml
<dubbo:consumer filter="-sentinel.dubbo.consumer.filter"/>

<dubbo:provider filter="-sentinel.dubbo.provider.filter"/>
```

For more details of Dubbo filter, see [here](https://dubbo.incubator.apache.org/#/docs/dev/impls/filter.md?lang=en-us).

## Dubbo resources

The resource for Dubbo services has two granularities: service interface and service method.

- Service interface：resourceName format is `interfaceName`，e.g. `com.alibaba.csp.sentinel.demo.dubbo.FooService`
- Service method：resourceName format is `interfaceName:methodSignature`，e.g. `com.alibaba.csp.sentinel.demo.dubbo.FooService:sayHello(java.lang.String)`

## Flow control based on caller

In many circumstances, it's also significant to control traffic flow based on the **caller**.
For example, assuming that there are two services A and B, both of them initiate remote call requests to the service provider.
If we want to limit the calls from service B only, we can set the `limitApp` of flow rule as the identifier of service B (e.g. service name).

Sentinel Dubbo Adapter will automatically resolve the Dubbo consumer's *application name* as the caller's name (`origin`),
and will bring the caller's name when doing resource protection.
If `limitApp` of flow rules is not configured (`default`), flow control will take effects on all callers.
If `limitApp` of a flow rule is configured with a caller, then the corresponding flow rule will only take effect on the specific caller.

> Note: Dubbo consumer does not provide its Dubbo application name when doing RPC,
so developers should manually put the application name into *attachment* at consumer side,
then extract it at provider side. Sentinel Dubbo Adapter has implemented a filter (`DubboAppContextFilter`)
where consumer can carry application name information to provider automatically.
If the consumer does not use Sentinel Dubbo Adapter but requires flow control based on caller, developers can manually put the application name into attachment with the key `dubboApplication`.

## Global fallback

Sentinel Dubbo Adapter supports global fallback configuration.
The global fallback will handle exceptions and give replacement result when blocked by
flow control, degrade or system load protection. You can implement your own `DubboFallback` interface
and then register to `DubboFallbackRegistry`. If no fallback is configured, Sentinel will wrap the `BlockException`
then directly throw it out.

Besides, we can also leverage [Dubbo mock mechanism](http://dubbo.apache.org/#!/docs/user/demos/local-mock.md?lang=en-us) to provide fallback implementation of degraded Dubbo services.