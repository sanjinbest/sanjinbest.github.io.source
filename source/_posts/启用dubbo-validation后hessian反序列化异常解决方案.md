title: 启用dubbo validation后hessian反序列化异常解决方案
author: 木子三金
tags: []
categories:
  - 问题记录
  - dubbo
date: 2019-04-26 10:44:00
---
## 现象
启用dubbo参数验证，当在provider端开启验证**validation="true"**。调用接口时，若接口参数为对象类型，对象属性验证失败，客户端会报org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl' could not be instantiated异常的解决方案。

<!-- more -->

```
2019-04-25T09:57:18.544+0800 [ERROR] business-service o.a.c.c.C.[.[.[.[dispatcherServlet] - - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.reflect.UndeclaredThrowableException] with root cause
com.alibaba.dubbo.remoting.RemotingException: com.alibaba.com.caucho.hessian.io.HessianFieldException: org.hibernate.validator.internal.engine.ConstraintViolationImpl.constraintDescriptor: 'org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl' could not be instantiated
com.alibaba.com.caucho.hessian.io.HessianFieldException: org.hibernate.validator.internal.engine.ConstraintViolationImpl.constraintDescriptor: 'org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl' could not be instantiated
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.logDeserializeError(JavaDeserializer.java:167)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer$ObjectFieldDeserializer.deserialize(JavaDeserializer.java:410)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.readObject(JavaDeserializer.java:276)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.readObject(JavaDeserializer.java:203)
	at com.alibaba.com.caucho.hessian.io.SerializerFactory.readObject(SerializerFactory.java:526)
... more
Caused by: com.alibaba.com.caucho.hessian.io.HessianProtocolException: 'org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl' could not be instantiated
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.instantiate(JavaDeserializer.java:316)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.readObject(JavaDeserializer.java:201)
	at com.alibaba.com.caucho.hessian.io.Hessian2Input.readObjectInstance(Hessian2Input.java:2818)
	at com.alibaba.com.caucho.hessian.io.Hessian2Input.readObject(Hessian2Input.java:2145)
	at com.alibaba.com.caucho.hessian.io.Hessian2Input.readObject(Hessian2Input.java:2074)
	at com.alibaba.com.caucho.hessian.io.Hessian2Input.readObject(Hessian2Input.java:2118)
	at com.alibaba.com.caucho.hessian.io.Hessian2Input.readObject(Hessian2Input.java:2074)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer$ObjectFieldDeserializer.deserialize(JavaDeserializer.java:406)
	... 41 more
Caused by: java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.alibaba.com.caucho.hessian.io.JavaDeserializer.instantiate(JavaDeserializer.java:312)
	... 48 more
Caused by: java.lang.NullPointerException
	at org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl.<init>(ConstraintDescriptorImpl.java:176)
	at org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl.<init>(ConstraintDescriptorImpl.java:233)
	... more
```

## 版本
```
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.7.1</version>
</dependency>
```
同时测试了2.6.5也有同样的问题。

## 分析

异常表象为'org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl' could not be instantiated，在继续跟踪栈信息是在下面的地方抛出了NullPointerException异常。
```
Caused by: java.lang.NullPointerException
	at org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl.<init>(ConstraintDescriptorImpl.java:176)
	at org.hibernate.validator.internal.metadata.descriptor.ConstraintDescriptorImpl.<init>(ConstraintDescriptorImpl.java:233)
```

进入ConstraintDescriptorImpl.java:176行看下，代码如下：
```
public ConstraintDescriptorImpl(ConstraintHelper constraintHelper,
			Member member,
			ConstraintAnnotationDescriptor<T> annotationDescriptor,
			ElementType type,
			Class<?> implicitGroup,
			ConstraintOrigin definedOn,
			ConstraintType externalConstraintType) {
this.annotationDescriptor = annotationDescriptor;
this.elementType = type;
this.definedOn = definedOn;
this.isReportAsSingleInvalidConstraint = annotationDescriptor.getType().isAnnotationPresent(ReportAsSingleViolation.class);

... more

```

报空指针的位置就是
```
this.isReportAsSingleInvalidConstraint = annotationDescriptor.getType().isAnnotationPresent(ReportAsSingleViolation.class);
```

## 结论
dubbo使用默认序列化协议**hessian**，在反序列化时根据**构造函数参数个数优先级来取参数最少的构造器**，由于ConstraintDescriptorImpl该类只一个构造器，debug进入发现在初始化时传入的参数全部为null，这就造成了NullPointerException异常，导致序列化失败。
从ConstraintViolationException继承结构可以看到其继承自ValidationException，并增加了一个参数private final Set<ConstraintViolation<?>> constraintViolations;用来将验证信息传递到consumer，反序列化的问题就是出在了Set<ConstraintViolation<?>>上。

![upload successful](/images/pasted-88.png)


## 解决方案
知道了问题出在哪里，那解决起来还是很方便的。既然Set<ConstraintViolation<?>> constraintViolations是为了将校验信息传递到consumer，那我们自己自定义一个校验信息体就OK啦。
使用dubbo[验证扩展](http://dubbo.apache.org/zh-cn/docs/dev/impls/validation.html)自定义一个validation。


具体实现如下：
实现一个Validation接口，返回一个自定义的Validator
```
package com.example.demo.dubbo.validation;

import org.apache.dubbo.common.URL;
import org.apache.dubbo.validation.Validation;
import org.apache.dubbo.validation.Validator;

/**
 * @ClassName CustomValidation
 * @Description
 * @Author lixin
 * @Date 2019-04-23 17:34
 */
public class CustomValidation implements Validation{

    @Override
    public Validator getValidator(URL url) {
        return new CustomValidator(url);
    }
}

```

CustomValidation实现，这里只需要简单的捕获一下ConstraintViolationException异常，将violations取出重新格式化即可，同时还保持了dubbo validation的原生逻辑支持
```
package com.example.demo.dubbo.validation;

import org.apache.dubbo.common.URL;
import org.apache.dubbo.common.json.JSONArray;
import org.apache.dubbo.common.json.JSONObject;
import org.apache.dubbo.common.utils.ReflectUtils;
import org.apache.dubbo.validation.support.jvalidation.JValidator;
import org.apache.dubbo.validation.Validator;

import javax.validation.*;
import java.util.*;

/**
 * 继承自{@link JValidator}保证了dubbo原生的vidation逻辑。
 * 修改点：1、捕获ConstraintViolationException异常，将验证未通过异常改为ValidationException，解决dubbo hessian反序列化失败问题
 *       2、格式化验证提示，使用json格式响应
 *
 * @Author lixin
 * @Date 2019-04-23 17:35
 */
public class CustomValidator extends JValidator implements Validator {

    private final Class<?> clazz;

    @SuppressWarnings({"unchecked", "rawtypes"})
    public CustomValidator(URL url) {
        super(url);
        this.clazz = ReflectUtils.forName(url.getServiceInterface());
    }

    @Override
    public void validate(String methodName, Class<?>[] parameterTypes, Object[] arguments) throws Exception {

        try{
            super.validate(methodName,parameterTypes,arguments);
        }catch (ConstraintViolationException e){
            throw new ValidationException(constraintMessage(clazz.getName(),methodName,e.getConstraintViolations()));
        }
    }

    private String constraintMessage(String className,String methodName,Set<ConstraintViolation<?>> violations){
        JSONObject json = new JSONObject();
        json.put("service",className);
        json.put("method",methodName);

        JSONArray details = new JSONArray();
        for(ConstraintViolation violation : violations){
            JSONObject detail = new JSONObject();
            detail.put("bean",violation.getRootBean().getClass().getName());
            detail.put("property",violation.getPropertyPath().toString());
            detail.put("message",violation.getMessage());
            details.add(detail);
        }
        json.put("details",details);
        return json.toString();
    }
}

```


## 使用方式
provider端启用(注解方式)
```
@Service(validation = "xxx")
```
consumer端启用(注解方式)
```
@Reference(validation = "xxx")
```
## 建议
1、在consumer端开启验证，以减少无用的请求打到provider端。
2、自定义的model类，增加默认构造器。