title: Feign官方文档翻译
author: 木子三金
tags:
  - Feign
  - spring cloud
categories:
  - spring
date: 2019-07-04 17:12:00
---
# Feign 让编写java http客户端变简单

Fegin是一个java调用HTTP的客户端binder，其灵感来自于[Retrofit](https://github.com/square/retrofit)、[JAXRS-2.0](https://jax-rs-spec.java.net/nonav/2.0/apidocs/index.html)和[WebSocket](http://www.oracle.com/technetwork/articles/java/jsr356-1937161.html)。Feign的首要目标是降低将[Denominator](https://github.com/Netflix/Denominator)统一的绑定到HTTP的复杂性，而无需关心是否为RESTfull.

<!-- more -->

### Why Feign and not X?

Feign的使用方式与Jersey和CXF编写java client来调用REST或SOAP服务类似。此外，Feign还允许编写基于Apache HC之类的类库的代码。Feign通过自定义decoder和error处理以最小的成本将你的代码连接到http，并可以使用到任何基于文本的http API上。

### Feign如何工作?

Feign基于将注解转换成请求模板的方式工作。参数会简单直接的应用到模板上。尽管Feign只支持基于文本的API，但是它的简化了类似于请求重放等系统层面的开发。此外，Feign让单元测试更加方便。

### Java 版本兼容

Feign 10.x版本基于java 1.8同时可以在java9，10和11版本上工作。如果要在JDK6上使用，请使用Feign 9.x

### 基本使用方式

下面是一个典型的用法，适配Retrofit示例。

```java
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("POST /repos/{owner}/{repo}/issues")
  void createIssue(Issue issue, @Param("owner") String owner, @Param("repo") String repo);

}

public static class Contributor {
  String login;
  int contributions;
}

public static class Issue {
  String title;
  String body;
  List<String> assignees;
  int milestone;
  List<String> labels;
}

public class MyApp {
  public static void main(String... args) {
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  
    // Fetch and print a list of the contributors to this library.
    List<Contributor> contributors = github.contributors("OpenFeign", "feign");
    for (Contributor contributor : contributors) {
      System.out.println(contributor.login + " (" + contributor.contributions + ")");
    }
  }
}
```

### 接口注解

Feign的注解定义了接口及其client如何工作的 `Contract`，Feign默认`Contract` 如下表。

Annotation | Interface Target | Usage
---|---|---
`@RequestLine` | Method | 用来定义request 的 `HttpMethod`和`UriTemplate`。表达式（expression）使用大括号{}括起来，表达式的值会由被`@Param`注解的参数提供。
`@Param` | Parameter | 定义模板变量，变量的值会根据参数名解析到`Expression`内。
`@Headers` | Method或Interface | 定义`HeaderTemplate`，表达式的值会由被`@Param`注解的参数提供。`@Header`可以使用到`Method`或`Interface`上，当作用到`Interface`上时`Interface`内的所有方法都会使用该`Header`，如果作用到`Method`上那只会对该方法使用`Header`。
`@QueryMap` | Parameter | 可以修饰K-V结构的`Map`或POJO，用以扩展查询参数。
`@HeaderMap` | Parameter | 可以修饰K-V结构的`Map`或POJO，用以扩展`Http Header`。
`@Body` | Method | 与`UriTemplate`和`HeaderTemplate`类似，用来定义一个`Template`，表达式的值会由被`@Param`注解的参数提供。

### 表达式（Templates）与模板（Expressions）

Feign的`Expressions`遵循[URI Template - RFC 6570](https://tools.ietf.org/html/rfc6570) level 1规范。`Expressions`需要配合`Param`所修饰的方法参数进行扩展。

*Example*

```java
public interface GitHub {
  
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repository);
  
  class Contributor {
    String login;
    int contributions;
  }
}

public class MyApp {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
    
    /* The owner and repository parameters will be used to expand the owner and repo expressions
     * defined in the RequestLine.
     * 
     * the resulting uri will be https://api.github.com/repos/OpenFeign/feign/contributors
     */
    github.contributors("OpenFeign", "feign");
  }
}
```
- 表达式(Expression)：被花括号括起来的内容，可以使用正则表达式，表达式名称与正则表达式用冒号`:`分隔，比如{owner:[a-zA-Z]*}。表达式的值由被`@Param`修饰的变量提供。
- 模板(Template)：多个表达式的集合，如`GET /repos/{owner}/{repo}/contributors`


#### 请求参数说明

`RequestLine` 和 `QueryMap` 的模板遵循[URI Template - RFC 6570](https://tools.ietf.org/html/rfc6570) level 1规范，具体的规范如下：
- 忽略掉无法解析的表达式
- 如果没有对字符串或变量进行编码或未通过`@Param`注解期编码方式，默认会使用pct-encoded编码。

#### 未定义的值和空值

**未定义的表达式也是合法的表达式**，如果明确指定表达式的值为`Null`或者为空，遵循[URI Template - RFC 6570](https://tools.ietf.org/html/rfc6570)规范，则会提供一个空的值给表达式。
当Feign解析表达式时，会优先判断是否定义了表达式的值来决定是否要保留查询参数名。如果未定义表达式的值，查询参数名将会被去除，看下面的示例。

*Empty String*
```java
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   parameters.put("param", "");
   this.demoClient.test(parameters);
}
```
Result
```
http://localhost:8080/test?param=
```

*Missing*
```java
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   this.demoClient.test(parameters);
}
```
Result
```
http://localhost:8080/test
```

*Undefined*
```java
public void test() {
   Map<String, Object> parameters = new LinkedHashMap<>();
   parameters.put("param", null);
   this.demoClient.test(parameters);
}
```
Result
```
http://localhost:8080/test
```

 [高级用法](#advanced-usage) 提供了更多的示例.

> **关于斜杠`/`**
>
>默认情况下`@RequestLine` 和 `@QueryMap` 模板不会对`/`编码，如果要对其编码可以设置`@RequestLine`的`decodeSlash`值为`false`。

> **关于加号 `+`**
>
>根据URI规范，请求地址和请求参数是允许使用加号`+`的，但是在查询的处理上可能会不一致。在一些旧的系统上`+`可能会被当作空白字符。Feign根据现代系统的做法不会将`+`当作空白字符，而是编码为`%2B`。
>
>如果要将`+`处理成空白字符，可以使用空格或者直接将其编码为`%20`

##### 自定义扩展

`@Param`可以通过可选属性`expander`对某一参数进行扩展处理。`expander`属性需要接收一个实现了Expander接口的实现类。

```java
public interface Expander {
    String expand(Object value);
}
```

方法的返回值规则与上述规则相同。忽略掉无法解析的表达式;
如果没有对字符串或变量进行编码或未通过@Param注解期编码方式，默认会使用pct-encoded编码。更多使用示例参考[Custom @Param Expansion](#custom-param-expansion)


#### 请求头扩展

`Headers` 与 `HeaderMap` 模板规则与[Request Parameter Expansion](#request-parameter-expansion) 规则相同
- 如果返回值为null或空，则移除整个header
- 如果未进行pct编码，则会进行pct编码

 [Headers](#headers) 示例代码.

> **请注意`@Param` 参数和参数名称**: 
>
> 如果`@RequestLine`, `@QueryMap`, `@BodyTemplate`, 或 `@Headers`拥有相同的表达式名称，那么这些表达式会被赋予相同的值。如下示例中的`contentType`将会被同时解析到`@RequestLine`和`@Headers`的表达式中

>
> ```java
> public interface ContentService {
> @RequestLine("GET /api/documents/{contentTåype}")
> @Headers("Accept: {contentType}")
> String getDocumentByType(@Param("contentType") String type);
> }
> ```
>
> 在设计接口时需要额外注意。

#### 请求体扩展

`Body`模板的规则与[Request Parameter Expansion](#request-parameter-expansion) 相同：

- 忽略掉无法解析的表达式
- 扩展内容在被传递到请求体前不会被编码
- 必须在header内指定`Content-Type`。参考示例代码 [Body Templates](#body-templates)。

---
### 定制化服务

Feign同时也提供了一些定制化的能力,你可以使用`Feign.builder()`来构建自定义的接口。

```java
interface Bank {
  @RequestLine("POST /account/{id}")
  Account getAccountInfo(@Param("id") String id);
}

public class BankService {
  public static void main(String[] args) {
    Bank bank = Feign.builder().decoder(
        new AccountDecoder())
        .target(Bank.class, "https://api.examplebank.com");
  }
}
```

### 多态

Feign支持创建多个api接口。通过继承接口`Target<T>`(默认实现为`HardCodedTarget<T>`)实现，在程序运行request前会动态的发现并包装请求。

例如，下面的场景对当前使用的url增加`CloudIdentityTarget`进行auth验证。

```java
public class CloudService {
  public static void main(String[] args) {
    CloudDNS cloudDNS = Feign.builder()
      .target(new CloudIdentityTarget<CloudDNS>(user, apiKey));
  }
  
  class CloudIdentityTarget extends Target<CloudDNS> {
    /* implementation of a Target */
  }
}
```

### 更多示例

Feign包含了[GitHub](./example-github) 和 [Wikipedia](./example-wikipedia)的客户端示例代码，以供大家作为开发参考。特殊的示例请参考[example daemon](https://github.com/Netflix/denominator/tree/master/example-daemon).

---
### 集成能力

Feign设计原则就是能够与其它开源工具友好的工作，也欢迎将您喜欢的项目集成进来。

### Gson
[Gson](./gson) 包含了可以与JSON API共同使用的编码器和解码器

使用`Feign.Builder`添加`GsonEncoder` 或 `GsonDecoder`

```java
public class Example {
  public static void main(String[] args) {
    GsonCodec codec = new GsonCodec();
    GitHub github = Feign.builder()
                         .encoder(new GsonEncoder())
                         .decoder(new GsonDecoder())
                         .target(GitHub.class, "https://api.github.com");
  }
}
```

### Jackson
[Jackson](./jackson)包含了可以与JSON API共同使用的编码器和解码器

使用`Feign.Builder`添加`JacksonEncoder` 或 `JacksonDecoder`

```java
public class Example {
  public static void main(String[] args) {
      GitHub github = Feign.builder()
                     .encoder(new JacksonEncoder())
                     .decoder(new JacksonDecoder())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Sax
[SaxDecoder](./sax) 可以在标准的JVM或Android环境下解码XML。

使用Sax对响应数据进行解析
```java
public class Example {
  public static void main(String[] args) {
      Api api = Feign.builder()
         .decoder(SAXDecoder.builder()
                            .registerContentHandler(UserIdHandler.class)
                            .build())
         .target(Api.class, "https://apihost");
    }
}
```

### JAXB

[JAXB](./jaxb)包含了可以与XML API共同使用的编码器和解码器

使用`Feign.Builder`添加`JAXBEncoder` 或 `JAXBDecoder`
```java
public class Example {
  public static void main(String[] args) {
    Api api = Feign.builder()
             .encoder(new JAXBEncoder())
             .decoder(new JAXBDecoder())
             .target(Api.class, "https://apihost");
  }
}
```

### JAX-RS

[JAXRSContract](./jaxrs)对标准的JAX-RS进行了重新处理，目前基于1.1规范。

```java
interface GitHub {
  @GET @Path("/repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@PathParam("owner") String owner, @PathParam("repo") String repo);
}

public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                       .contract(new JAXRSContract())
                       .target(GitHub.class, "https://api.github.com");
  }
}
```

### OkHttp

使用[OkHttpClient](./okhttp)直接将Feign请求交由OkHttp处理，OkHttp使用了SPDY协议，可以更好的对网络进行控制

Feign使用OkHttp只需要将OkHttp加入到classpath，然后配置Feign使用OkHttpClient。

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .client(new OkHttpClient())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Ribbon

[RibbonClient](./ribbon)为Feign client重写URL解析，同时提供了智能路由和弹性能力。

集成ribbon需要将url改为ribbon的客户端名称，如`myAppProd`
```java
public class Example {
  public static void main(String[] args) {
    MyService api = Feign.builder()
          .client(RibbonClient.create())
          .target(MyService.class, "https://myAppProd");
  }
}
```

### Java 11 Http2

[Http2Client](./java11)直接将Feign http请求交由Java11 [New HTTP/2 Client](http://www.javamagazine.mozaicreader.com/JulyAug2017#&pageSet=39&page=0)处理以实现HTTP/2。

若为Feign使用New HTTP/2客户端需要JDK11,然后配置Feign使用Http2Client。

```java
GitHub github = Feign.builder()
                     .client(new Http2Client())
                     .target(GitHub.class, "https://api.github.com");
```

### Hystrix

[HystrixFeign](./hystrix)使用[Hystrix](https://github.com/Netflix/Hystrix)提供了断路器的支持。

若要使用Hystrix，需要将Hystrix模块加入classpath。然后使用`HystrixFeign` builder。

```java
public class Example {
  public static void main(String[] args) {
    MyService api = HystrixFeign.builder().target(MyService.class, "https://myAppProd");
  }
}
```

### SOAP

[SOAP](./soap)包含了可以与XML API共同使用的编码器和解码器

该模块通过JAXB和SOAPMessage对SOAP Body提供编码和解码。同时将SOAPFault解码能力包装到了原生的`javax.xml.ws.soap.SOAPFaultException`，所以使用时只需要捕获`SOAPFaultException`来处理SOAPFault。

Add `SOAPEncoder` and/or `SOAPDecoder` to your `Feign.Builder` like so:

使用`Feign.Builder`来添加`SOAPEncoder` 或 `SOAPDecoder`。

```java
public class Example {
  public static void main(String[] args) {
    Api api = Feign.builder()
	     .encoder(new SOAPEncoder(jaxbFactory))
	     .decoder(new SOAPDecoder(jaxbFactory))
	     .errorDecoder(new SOAPErrorDecoder())
	     .target(MyApi.class, "http://api");
  }
}
```

注意：如果SOAP的错误响应包含http code (4xx、5xx、…)，则需要添加`SOAPErrorDecoder`对错误进行处理。

### SLF4J

[SLF4JModule](./slf4j)直接将Feign's logging交由[SLF4J](http://www.slf4j.org/)处理，你可以方便的选择logging的后端实现。

若要使用SLF4J，需要添加SLF4J和对应的后端模块到classpath内，然后使用`Feign.builder`配置logging。

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .logger(new Slf4jLogger())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

### Decoders

`Feign.builder()`可以指定Decoder对响应数据进行解码。

如果接口内某些方法反回了除`Response`, `String`, `byte[]` 或 `void`之外的返回值，则需要配置一个自定义的 `Decoder`。

使用gson对json进行解码

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .decoder(new GsonDecoder())
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

如果你需要在解码器解码之前作一些前置操作，可以使用`mapAndDecode`。
举个例子，现在要求API只处理jsonp，可以在传输之前不对jsonp进行包装。

```java
public class Example {
  public static void main(String[] args) {
    JsonpApi jsonpApi = Feign.builder()
                         .mapAndDecode((response, type) -> jsopUnwrap(response, type), new GsonDecoder())
                         .target(JsonpApi.class, "https://some-jsonp-api.com");
  }
}
```

### Encoders
The simplest way to send a request body to a server is to define a `POST` method that has a `String` or `byte[]` parameter without any annotations on it. You will likely need to add a `Content-Type` header.

一个最简单的请求方式就是使用`POST`方法传递一个`String`或`byte[]`类型参数到服务端，有时可能需要在header内指定`Content-Type`，如下方式

```java
interface LoginClient {
  @RequestLine("POST /")
  @Headers("Content-Type: application/json")
  void login(String content);
}

public class Example {
  public static void main(String[] args) {
    client.login("{\"user_name\": \"denominator\", \"password\": \"secret\"}");
  }
}
```

在这里可以使用`Encoder`配置方式，这样就可以发送一些类型安全的请求体。

```java
static class Credentials {
  final String user_name;
  final String password;

  Credentials(String user_name, String password) {
    this.user_name = user_name;
    this.password = password;
  }
}

interface LoginClient {
  @RequestLine("POST /")
  void login(Credentials creds);
}

public class Example {
  public static void main(String[] args) {
    LoginClient client = Feign.builder()
                              .encoder(new GsonEncoder())
                              .target(LoginClient.class, "https://foo.com");
    
    client.login(new Credentials("denominator", "secret"));
  }
}
```

### @Body 模板

`@Body`注解用来声明一个请求体模板，并配合`@Param`注解对请求参数进行扩展。在使用时需要指定`Content-Type`。

```java
interface LoginClient {

  @RequestLine("POST /")
  @Headers("Content-Type: application/xml")
  @Body("<login \"user_name\"=\"{user_name}\" \"password\"=\"{password}\"/>")
  void xml(@Param("user_name") String user, @Param("password") String password);

  @RequestLine("POST /")
  @Headers("Content-Type: application/json")
  // json curly braces must be escaped!
  @Body("%7B\"user_name\": \"{user_name}\", \"password\": \"{password}\"%7D")
  void json(@Param("user_name") String user, @Param("password") String password);
}

public class Example {
  public static void main(String[] args) {
    client.xml("denominator", "secret"); // <login "user_name"="denominator" "password"="secret"/>
    client.json("denominator", "secret"); // {"user_name": "denominator", "password": "secret"}
  }
}
```

### Headers

Feign支持API和单独客户端2个层面的headers配置。

#### API层面配置

如果接口内的多个方法都需要包含相同headers配置，可以将headers配置在API接口上。

同时在API接口和接口方法上使用`@Headers` 。

```java
@Headers("Accept: application/json")
interface BaseApi<V> {
  @Headers("Content-Type: application/json")
  @RequestLine("PUT /api/{key}")
  void put(@Param("key") String key, V value);
}
```
`@Headers`根据根据表达式的值，动态的从`@Param`内获取对应的内容。

```java
public interface Api {
   @RequestLine("POST /")
   @Headers("X-Ping: {token}")
   void post(@Param("token") String token);
}
```

有时可能需要动态的指定headers内的key和value。这类场景可以使用`HeaderMap` 来动态传入headers的key和value。

```java
public interface Api {
   @RequestLine("POST /")
   void post(@HeaderMap Map<String, Object> headerMap);
}
```

#### 动态header

如果一个API接口会被不同的目标端使用，同时需要指定不同的headers，或者在发送请求时作特殊处理。可以通过`RequestInterceptor` 或者
`Target`来实现

使用`RequestInterceptor`的例子可以参考`Request Interceptors`章节。下面是使用`Target`的实现方式。

```java
  static class DynamicAuthTokenTarget<T> implements Target<T> {
    public DynamicAuthTokenTarget(Class<T> clazz,
                                  UrlAndTokenProvider provider,
                                  ThreadLocal<String> requestIdProvider);
    
    @Override
    public Request apply(RequestTemplate input) {
      TokenIdAndPublicURL urlAndToken = provider.get();
      if (input.url().indexOf("http") != 0) {
        input.insert(0, urlAndToken.publicURL);
      }
      input.header("X-Auth-Token", urlAndToken.tokenId);
      input.header("X-Request-ID", requestIdProvider.get());

      return input.request();
    }
  }
  
  public class Example {
    public static void main(String[] args) {
      Bank bank = Feign.builder()
              .target(new DynamicAuthTokenTarget(Bank.class, provider, requestIdProvider));
    }
  }
```

上述这些实现是在Feign client构建和使用时，通过自定义`RequestInterceptor` 或 `Target` 来设置headers，这种方式可以配置在每一个Feign client上，同时也会对所有的API接口生效。这是一个很实用的功能，比如为所有client的每一个请求header增加auth token。这些扩展方法与API方法调用使用同一个线程，所以在API方法运行时可以根据上下文动态的指定headers的内容，比如，可以使用ThreadLocal为每个请求线程存储不同的header内容，对于实现特殊的请求线程链路追踪是比较有用的方式。

### 高级用法

#### 基础api

有时，某些服务的api会使用相同的约束。Feign可以使用接口单继承的方式进行配置。

```java
interface BaseAPI {
  @RequestLine("GET /health")
  String health();

  @RequestLine("GET /all")
  List<Entity> all();
}
```
通过继承来扩展基础api的能力。
```java
interface CustomAPI extends BaseAPI {
  @RequestLine("GET /custom")
  String custom();
}
```

也可以通过泛型，约束API接口的操作对象类型。

```java
@Headers("Accept: application/json")
interface BaseApi<V> {

  @RequestLine("GET /api/{key}")
  V get(@Param("key") String key);

  @RequestLine("GET /api")
  List<V> list();

  @Headers("Content-Type: application/json")
  @RequestLine("PUT /api/{key}")
  void put(@Param("key") String key, V value);
}

interface FooApi extends BaseApi<Foo> { }

interface BarApi extends BaseApi<Bar> { }
```

#### Logging

通过`logger`可以对API接口的调用日志作一些特殊处理。

```java
public class Example {
  public static void main(String[] args) {
    GitHub github = Feign.builder()
                     .decoder(new GsonDecoder())
                     .logger(new Logger.JavaLogger().appendToFile("logs/http.log"))
                     .logLevel(Logger.Level.FULL)
                     .target(GitHub.class, "https://api.github.com");
  }
}
```

#### 拦截器

你可以使用`RequestInterceptor`拦截所有的请求，比如作为中间请求代理你可以设置`X-Forwarded-For`。

```java
static class ForwardedForInterceptor implements RequestInterceptor {
  @Override public void apply(RequestTemplate template) {
    template.header("X-Forwarded-For", "origin.host.com");
  }
}

public class Example {
  public static void main(String[] args) {
    Bank bank = Feign.builder()
                 .decoder(accountDecoder)
                 .requestInterceptor(new ForwardedForInterceptor())
                 .target(Bank.class, "https://api.examplebank.com");
  }
}
```

或是使用`BasicAuthRequestInterceptor`来实现一个身份验证拦截器。
```java
public class Example {
  public static void main(String[] args) {
    Bank bank = Feign.builder()
                 .decoder(accountDecoder)
                 .requestInterceptor(new BasicAuthRequestInterceptor(username, password))
                 .target(Bank.class, "https://api.examplebank.com");
  }
}
```

#### 自定义 @Param

被`Param`注解的参数最终都是通过`toString`返回参数值，通过`Param.Expander`设置自定义扩展，比如格式化日期。

```java
public interface Api {
  @RequestLine("GET /?since={date}") Result list(@Param(value = "date", expander = DateToMillis.class) Date date);
}
```

#### 动态查询参数

可以对Map类型参数使用`QueryMap`注解，Map类型参数的K-V就是查询的参数名及参数值

```java
public interface Api {
  @RequestLine("GET /find")
  V find(@QueryMap Map<String, Object> queryMap);
}
```

`QueryMapEncoder`同样可以通过JAVA BEAN生成查询参数。

```java
public interface Api {
  @RequestLine("GET /find")
  V find(@QueryMap CustomPojo customPojo);
}
```

当使用上面的方式，如果没有自定义`QueryMapEncoder`，那么查询参数名将默认使用对象的成员变量名生成。使用JAVA BEAN生成如下参数"/find?name={name}&number={number}"(如果成员变量的值为null，则参数会被忽略)

```java
public class CustomPojo {
  private final String name;
  private final int number;

  public CustomPojo (String name, int number) {
    this.name = name;
    this.number = number;
  }
}
```

设置自定义的`QueryMapEncoder`

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .queryMapEncoder(new MyCustomQueryMapEncoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

当使用`@QueryMap`注解对象时，默认编码器通过反射方式查找对象属性，并将属性转换成string类型。如果你希望使用getter和setter来构建查询字符串，你需要在Java Bean内定义getter和setter方法，并使用`BeanQueryMapEncoder`

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .queryMapEncoder(new BeanQueryMapEncoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

### 错误处理

通过Feign.builder为Feign实例注册一个自定义的`ErrorDecoder`，可以对异常的响应作额外的处理。

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .errorDecoder(new MyErrorDecoder())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

所有HTTP code为非2xx的响应都会触发 `ErrorDecoder`的`decode`方法，可以在该方法内对响应进行处理，包装自定义的异常或执行一些额外逻辑。

如果想要对失败请求进行重试，可以抛出一个`RetryableException`，则会使用已注册的`Retryer`进行重试.

### 重试

Feign默认会自动重试所有抛出了`IOException`的方法，也会自动重试`ErrorDecoder`抛出的`RetryableException`请求。可以通过builder注册自定义的`Retryer`来实现自定义的重试策略。

```java
public class Example {
  public static void main(String[] args) {
    MyApi myApi = Feign.builder()
                 .retryer(new MyRetryer())
                 .target(MyApi.class, "https://api.hostname.com");
  }
}
```

`Retryer`根据方法`continueOrPropagate(RetryableException e)`返回`true` 或者`false`来决定是否进行重试，`Retryer`在每个`Client`创建时指定，可以根据不同的场景使用特定的重试策略。
如果重试失败最终的抛出一个`RetryException`异常。可以通过为Feign client配置`exceptionPropagationPolicy()`选项，会输出请求失败的原始异常信息。

#### 静态方式和默认方法

Feign的目标接口可以包含静态方法或者默认方法（jdk版本需要1.8及以上）。可以用来定义不包含其它逻辑的Feign client基础api。比如，使用静态方法创建一个普通client；默认方法则可以被用来组装查询或定义默认参数。

```java
interface GitHub {
  @RequestLine("GET /repos/{owner}/{repo}/contributors")
  List<Contributor> contributors(@Param("owner") String owner, @Param("repo") String repo);

  @RequestLine("GET /users/{username}/repos?sort={sort}")
  List<Repo> repos(@Param("username") String owner, @Param("sort") String sort);

  default List<Repo> repos(String owner) {
    return repos(owner, "full_name");
  }

  /**
   * Lists all contributors for all repos owned by a user.
   */
  default List<Contributor> contributors(String user) {
    MergingContributorList contributors = new MergingContributorList();
    for(Repo repo : this.repos(owner)) {
      contributors.addAll(this.contributors(user, repo.getName()));
    }
    return contributors.mergeResult();
  }

  static GitHub connect() {
    return Feign.builder()
                .decoder(new GsonDecoder())
                .target(GitHub.class, "https://api.github.com");
  }
}
```