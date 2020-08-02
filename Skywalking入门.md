### **1、什么是Skywalking**

#### 1.1 Skywalking 简介

> FROM http://skywalking.apache.org/
>
> 分布式系统的应用程序性能监视工具，专为微服务、云原生架构和基于容器（Docker、K8s、Mesos）架构而设计。
>
> 提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案。
>
> - **采用javaagent无*侵入*的方式实现了应用调用信息的收集,原应用程序无需做任何改动。**
>
> - 多种监控手段。可以通过语言探针和 service mesh 获得监控是数据。
>
> - 多个语言自动探针。包括 Java，.NET Core 和 Node.JS。
>
> - 轻量高效。无需大数据平台，和大量的服务器资源。
>
> - 模块化。UI、存储、集群管理都有多种机制可选。
>
> - 支持告警。
>
> - 优秀的可视化解决方案。
>
>   

> 文档地址：https://github.com/SkyAPM/document-cn-translation-of-skywalking

微服务离不开调用链监控，我们可以利用调用链监控快速定位性能问题、故障分析，同时在项目设计文档不全的情况下，也能通过调用链了解业务的实现逻辑。

#### 1.2 架构

![Aaron Swartz](http://static.iocoder.cn/2d559e500ea828b2922ea75768d576a7)

> - 上部分 **Agent** ：负责从应用中，收集链路信息，发送给 SkyWalking OAP 服务器。目前支持 SkyWalking、Zikpin、Jaeger 等提供的 Tracing 数据信息。而我们目前采用的是，SkyWalking Agent 收集 SkyWalking Tracing 数据，传递给服务器。
> - 下部分 **SkyWalking OAP** ：负责接收 Agent 发送的 Tracing 数据信息，然后进行分析(Analysis Core) ，存储到外部存储器( Storage )，最终提供查询( Query )功能。
> - 右部分 **Storage** ：Tracing 数据存储。目前支持 ES、MySQL、Sharding Sphere、TiDB、H2 多种存储器。而我们目前采用的是 ES ，主要考虑是 SkyWalking 开发团队自己的生产环境采用 ES 为主。
> - 左部分 **SkyWalking UI** ：负责提供控台，查看链路等等。

#### 1.3  Skywalking拓扑效果

![Aaron Swartz](https://raw.githubusercontent.com/firedaemon-zz/image/master/skywalking拓扑图1.gif)

#### 1.4 Skywalking链路追踪效果

![Aaron Swartz](https://raw.githubusercontent.com/firedaemon-zz/image/master/skywalking_trace.png)

### **2、为什么选Skywalking**

#### 2.1 几个分布式链路追踪一览

> google的Drapper--未开源，最早的APM

> Zipkin是Twitter开源的调用链分析工具，目前基于springcloud sleuth得到了广泛的使用，特点是轻量，使用部署简单。

> Pinpoint是韩国人开源的基于字节码注入的调用链分析，以及应用监控分析工具。特点是支持多种插件，UI功能强大，接入端无代码侵入。

> SkyWalking是本土开源的基于字节码注入的调用链分析，以及应用监控分析工具。特点是支持多种插件，UI功能较强，接入端无代码侵入。目前已加入Apache孵化器。

> CAT是大众点评开源的基于编码和配置的调用链分析，应用监控分析，日志采集，监控报警等一系列的监控平台工具。

> uber-jaeger， Jaeger支持java/c++/go/node/php，在界面上较为完善（对比zipkin），但是也无告警功能。代码入侵度小。dubbo目前无插件支持，可二次开发。

#### 2.2 功能比较

|                     | pinpoint           | zipkin                   | jaeger                   | apache skywalking                 | **CAT**                            |
| ------------------- | ------------------ | ------------------------ | ------------------------ | --------------------------------- | ---------------------------------- |
| OpenTracing兼容     | 否                 | 是                       | 是                       | 是                                | 否                                 |
| 客户端支持语言      | java、php          | java,c#,go,php等         | java,c#,go,php等         | Java, .NET Core, NodeJS and PHP   | ava, C/C++, Node.js, Python, Go    |
| 存储                | hbase              | ES，mysql,Cassandra,内存 | ES，kafka,Cassandra,内存 | ES，H2,mysql,TIDB,sharding sphere | mysql,hdfs                         |
| 传输协议支持        | thrift             | http,MQ                  | udp/http                 | gRPC                              | http/tcp                           |
| ui丰富程度          | 高                 | 低                       | 中                       | 中                                | 高                                 |
| 全局调用统计        | 支持               | 不支持                   | 不支持                   | 支持                              | 支持                               |
| 实现方式-代码侵入性 | 字节码注入，无侵入 | 拦截请求，侵入           | 拦截请求，侵入           | 字节码注入，无侵入                | 代码埋点（拦截器，注解，过滤器等） |
| 跟踪粒度            | 方法级             | 接口级                   | 方法级                   | 方法级                            | 代码级                             |
| 扩展性              | 低                 | 高                       | 高                       | 中                                |                                    |
| trace查询           | 不支持             | 支持                     | 支持                     | 支持                              | 不支持                             |
| 告警支持            | 支持               | 不支持                   | 不支持                   | 支持                              | 支持                               |
| jvm监控             | 支持               | 不支持                   | 不支持                   | 支持                              | 支持                               |
| 性能损失            | 高                 | 中                       | 中                       | 低                                |                                    |
| 社区活跃度          | 10.6k              | 13.2k                    | 11.5k                    | 14.2k                             | 13.9k                              |

#### 2.3 性能比较

模拟了三种并发用户：500，750，1000。使用jmeter测试，每个线程发送30个请求，设置思考时间为10ms。使用的采样率为1，即100%，这边与生产可能有差别。pinpoint默认的采样率为20，即50%，通过设置agent的配置文件改为100%。zipkin默认也是1。组合起来，一共有12种。下面看下汇总表：

![Aaron Swartz](https://upload-images.jianshu.io/upload_images/11623845-0083ef911824cfc8.png)

从上表可以看出，在三种链路监控组件中，skywalking的探针对吞吐量的影响最小，zipkin的吞吐量居中。pinpoint的探针对吞吐量的影响较为明显，在500并发用户时，测试服务的吞吐量从1385降低到774，影响很大。然后再看下CPU和memory的影响，在内部服务器进行的压测，对CPU和memory的影响都差不多在10%之内。

### **3、Skywalking部署**

### **4、Skywalking配置和使用**

#### 4.1 SpringBoot 链路追踪 SkyWalking

> http://www.iocoder.cn/Spring-Boot/SkyWalking/

#### 4.2 SpringCloud 链路追踪 SkyWalking

> http://www.iocoder.cn/Spring-Cloud/SkyWalking
>
#### 4.3 Skywalking 插件

Java Agent是插件化、可插拔的。Skywalking的插件分为三种：

- 引导插件：在agent的 `bootstrap-plugins` 目录下
- 内置插件：在agent的 `plugins` 目录下
- 可选插件：在agent的 `optional-plugins` 目录下

Java Agent只会启用 `plugins` 目录下的所有插件，`bootstrap-plugins` 目录以及 `optional-plugins` 目录下的插件不会启用。如需启用引导插件或可选插件，只需将JAR包移到 `plugins` 目录下，如需禁用某款插件，只需从 `plugins` 目录中移除即可。

##### 4.3.1 引导插件

目前只有两款引导插件：

- `apm-jdk-http-plugin` 用来是监测HttpURLConnection；
- `apm-jdk-threading-plugin` 用来监测Callable以及Runnable；

有关引导插件的功能描述，可详见： 

> [https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/README.md#bootstrap-%E7%B1%BB%E6%8F%92%E4%BB%B6](https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/README.md#bootstrap-类插件)

##### 4.3.2 内置插件(含MQ、Redis)

内置插件主要用来为业界主流的技术与框架提供支持。所支持的技术&框架，详见 

> ##### https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/Supported-list.md

##### 4.3.3 可选插件(含Zookeeper、gateway、customize-enhance)

关于可选插件的功能描述，可详见 

> https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/README.md
>
> https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/

##### 4.3.4 插件扩展（Oracle)

Skywalking生态还有一些插件扩展，例如Oracle、Resin插件等。这部分插件主要是由于许可证不兼容/限制，Skywalking无法将这部分插件直接打包到Skywalking安装包内，于是托管在这个地址： https://github.com/SkyAPM/java-plugin-extensions，使用方式：

> - 前往  https://github.com/SkyAPM/java-plugin-extensions/releases， 下载插件JAR包
> - 将JAR包挪到 `plugins` 目录即可启用。

#### 4.4 自定义追踪

我们都是通过 SkyWalking 提供的插件，实现对指定框架的链路追踪。那么，如果我们希望对项目中的业务方法，实现链路追踪，方便我们排查问题，需要怎么做呢？SkyWalking 提供了两种方式：

> - 方式一，通过 SkyWalking `@Trace` 注解，可见[《Application-toolkit-trace.md》](https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/Application-toolkit-trace.md)文档。
> - 方式二，通过 SkyWalking XML `<enhanced />` 配置，可见[《Customize-enhance-trace.md》](https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/Customize-enhance-trace.md)文档。

##### 4.4.1 注解方式

POM引入依赖

```xml
<dependency>
		<groupId>org.apache.skywalking</groupId>
		<artifactId>apm-toolkit-trace</artifactId>
		<version>${skywalking.version}</version>
</dependency>
```

自己想加入的方法使用 @Trace 注解修饰

```java
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;
import org.apache.skywalking.apm.toolkit.trace.TraceContext;

@GetMapping("tractAnnotation")
 public User traceAnnotation(
       @RequestParam("name") String name
    ) {
        log.info("从前端接收到的参数:[{}]", name);
        User user = trace(name);
        ActiveSpan.tag("new-tag", user.toString());
        ActiveSpan.info("输出信息");
        log.info("tractId:[{}]", TraceContext.traceId());
        return user;
    }

    @Trace(operationName = "添加自定义的方法")
    @Tags({
            @Tag(key = "从方法参数中获取值", value = "arg[0]"),
            @Tag(key = "从返回值中获取值", value = "returnedObj.name")
    })
    private User trace(String name) {
        log.info("如果此方法没有被SkyWalking收集，但是又需要被收集到，可以加上@Trace注解");
        User user = new User();
        user.setName("创建的名字");
        return user;
    }
```

获取 traceid

```java
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;
import org.apache.skywalking.apm.toolkit.trace.TraceContex;

@RestController
public class PluginController {
    //获取traceid
  	@GetMapping("/getTraceId")
  	public String getTraceId(){
        // 当前链路定义为报错，并抛出错误信息给Skywalking ，在Skywalking UI 就可以看到异常信息
        ActiveSpan.error(new RuntimeException("test error Throwable"));
      	// 返回 traceid, 可以给前端页面
      	return TraceContext.traceId();
    }
}
```

> - 在你想追踪的方法上添加`@Trace`注解。添加后，你就可以在方法调用栈中查看到span的信息。
> - 在被追踪的方法的上下文周期内添加自定义tag。
> - `ActiveSpan.error()` 将当前 Span 标记为出错状态.
> - `ActiveSpan.error(String errorMsg)` 将当前 Span 标记为出错状态, 并带上错误信息.
> - `ActiveSpan.error(Throwable throwable)` 将当前 Span 标记为出错状态, 并带上 `Throwable`.
> - `ActiveSpan.debug(String debugMsg)` 在当前 Span 添加一个 debug 级别的日志信息.
> - `ActiveSpan.info(String infoMsg)` 在当前 Span 添加一个 info 级别的日志信息.

##### 4.4.2 customize-enhance 插件示例

有一个类，定义如下：

```java
public class TestService1 {
    public static void staticMethod(String str0, int count, Map m, List l, Object[] os) {
      // 业务逻辑
    }
  ...
}
123456
```

那么，想要对该方法进行监控，则可如下操作：

**移动jar包**
将 `optional-plugins/apm-customize-enhance-plugin-6.5.0.jar` 移动到 `plugins` 目录

**编写增强规则**
创建一个文件，名为例如 `customize_enhance.xml` ，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<enhanced>
    <class class_name="test.apache.skywalking.testcase.customize.service.TestService1">
        <method method="staticMethod(java.lang.String,int.class,java.util.Map,java.util.List,[Ljava.lang.Object;)" operation_name="/is_static_method_args" static="true">
            <operation_name_suffix>arg[0]</operation_name_suffix>
            <operation_name_suffix>arg[1]</operation_name_suffix>
            <operation_name_suffix>arg[3].[0]</operation_name_suffix>
            <tag key="tag_1">arg[2].['k1']</tag>
            <tag key="tag_2">arg[4].[1]</tag>
            <log key="log_1">arg[4].[2]</log>
        </method>
    </class>
</enhanced>
12345678910111213
```

规则说明：

| 配置                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| class_name            | 要被增强的类                                                 |
| method                | 类的拦截器方法                                               |
| operation_name        | 如果进行了配置，将用它替代默认的operation_name               |
| operation_name_suffix | 表示在operation_name后添加动态数据                           |
| static                | 方法是否为静态方法                                           |
| tag                   | 将在local span中添加一个tag。key的值需要在XML节点上表示。    |
| log                   | 将在local span中添加一个log。key的值需要在XML节点上表示。    |
| arg[x]                | 表示输入的参数值。比如args[0]表示第一个参数。                |
| .[x]                  | 当正在被解析的对象是Array或List，你可以用这个表达式得到对应index上的对象。 |
| .[‘key’]              | 当正在被解析的对象是Map, 你可以用这个表达式得到map的key。    |

特别需要注意的是method的写法：

- 基本类型： 基本类型.class ，例如： `int.class`
- 非基本类型： 类的完全限定名称 ，例如：`java.lang.String`
- 数组：可以写个数组打印一下，就知道格式了，例如：

```java
public static void main(String[] args) {
        String[] s = new String[]{};
        System.out.println(s);

        int [] x = new int []{};
        System.out.println(x);
    }
1234567
```

结果：

```
[Ljava.lang.String;@1b0375b3
[I@2f7c7260
12
```

因此，对于String类型的数组，就可以写成 `[Ljava.lang.String;`；对于int类型的数组，则写成 `[I`。

配置
`agent.config`中添加配置：

```apl
plugin.customize.enhance_file=customize_enhance.xml的绝对路径
```

##### 4.4.3 插件二次开发

插件二次开发，对com.ultrapower.* 的所有方法实现监控

- [x]    <u>*TODO*</u>

#### 4.5 跨线程追踪

在pom中添加依赖

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>${skywalking.version}</version>
</dependency>
```

使用注解`@TraceCrossThread`

```java
    @TraceCrossThread
    public static class MyCallable<String> implements Callable<String> {
        @Override
        public String call() throws Exception {
            return null;
        }
    }
...
    ExecutorService executorService = Executors.newFixedThreadPool(1);
    ExecutorService.submit(new MyCallable());
```

或使用插件提供的包装器`@CallableWrapper`, `@RunnableWrapper`, `@SupplierWrapper`执行多线程代码。

```java
    ExecutorService executorService = Executors.newFixedThreadPool(3);
    executorService.submit(CallableWrapper.of(new Callable<String>() {
        @Override public String call() throws Exception {
            return null;
        }
    }));
	
    executorService.execute(RunnableWrapper.of(new Runnable() {
     @Override 
      public void run() {
            //your code
        }
    }));
	
	CompletableFuture.supplyAsync(() ->{
    	System.out.println(Thread.currentThread().getId());
    	return userManager.getUsers();
	});
```

扒一扒包装器的源码，发现也是使用了`@TraceCrossThread`注解来实现的。

```java
@TraceCrossThread
public class RunnableWrapper implements Runnable {
    final Runnable runnable;

    public RunnableWrapper(Runnable runnable) {
        this.runnable = runnable;
    }

    public static RunnableWrapper of(Runnable r) {
        return new RunnableWrapper(r);
    }

    @Override
    public void run() {
        this.runnable.run();
    }
}
@TraceCrossThread
public class CallableWrapper<V> implements Callable<V> {
    final Callable<V> callable;

    public static <V> CallableWrapper of(Callable<V> r) {
        return new CallableWrapper<V>(r);
    }

    public CallableWrapper(Callable<V> callable) {
        this.callable = callable;
    }

    @Override
    public V call() throws Exception {
        return callable.call();
    }
}
@TraceCrossThread
public class SupplierWrapper<V> implements Supplier<V> {
    final Supplier<V> supplier;

    public static <V> SupplierWrapper of(Supplier<V> r) {
        return new SupplierWrapper<V>(r);
    }

    public SupplierWrapper(Supplier<V> supplier) {
        this.supplier = supplier;
    }

    @Override
    public V get() {
        return supplier.get();
    }
}
```

#### 4.6 日志文件中写入traceid

我们需要在日志中记录下来每次请求的唯一标识(trace-id)，这样就可以在SkyWalking定位到有问题的trace-id，然后通过这个trace-id我们就可以通过日志系统去定位到相关的日志，从而发现并解决问题。

修改pom

在pom中添加依赖, 这是一个skywalking的工具，通过这个工具可以得到trace-id

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>${skywalking.version}</version>
</dependency>
```

修改logback.xml

修改logback.xml中的Appender的Pattern

```xml
<encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
    <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level logger_name:%logger{36} - [%tid] - message:%msg%n</pattern>
    </layout>
</encoder>
```

> - 使用%tid 来占trace-id的位置，默认TID:N/A，当有请求调用时，会显示trace-id
> - 这种方式支持探针与注解

参考文档

> https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/Application-toolkit-logback-1.x.md
>
### **5、参考资料**
#### 5.1 哔哩哔哩 深入学习Skywalking  
> https://www.bilibili.com/video/BV1ZJ411s7Mn?p=18

####  5.2 SkyWalking 极简入门 
> http://www.iocoder.cn/SkyWalking/install/?self

#### 5.3 Skywalking源码分析
   > http://www.iocoder.cn/SkyWalking/
#### 5.4 Skywalking的增强与拦截机制
   > https://www.cnblogs.com/Java-Script/p/11518808.html
#### 5.5 OpenTracing官方标准-中文版
   > https://github.com/opentracing-contrib/opentracing-specification-zh
#### 5.6 Dapper，大规模分布式系统的跟踪系统
   > http://www.iocoder.cn/Fight/Dapper-translation/?self

#### 5.7 Skywalking DEMO 地址

> http://122.112.182.72:8080/ 