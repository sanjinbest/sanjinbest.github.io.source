title: 自定义注解@Validated（兼容dubbo方法与普通方法）
author: 木子三金
tags:
  - dubbo
  - validator
categories:
  - dubbo
date: 2019-05-31 18:11:00
---
** 背景：最近尝试在项目内使用dubbo validation，但是遇到了一些问题，具体描述请看：[启用dubbo validation后hessian反序列化异常解决方案](http://www.sanjinbest.com/%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95/dubbo/%E5%90%AF%E7%94%A8dubbo-validation%E5%90%8Ehessian%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%BC%82%E5%B8%B8%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)**

<!-- more -->

文章内的解决方案适用于新启用的项目，对于已经在运行的dubbo项目，升级成本比较高。
所以，写了一个基于Hibernate validator的注解@Validated


#### 优点：
- 基于Hibernate validator实现，基于aop编程方式。

- 可作用在方法和class级别，支持分组校验（方法参数分组不支持）

- 友好的响应，dubbo方法校验失败返回DubboResult对象，普通方法抛出ConstraintViolationException异常

- 结合@Valid使用，可实现嵌套校验。

- 支持dubbo2.6.X及dubbo2.7.X，后续dubbo升级该注解无需升级

- 服务提供方升级调用方无感知。

#### 不足：
- 方法级别校验目前groups参数不生效，Bean级别groups可正常使用。

- 主要针对dubbo方法及普通方法，controller层方法未测试，建议使用spring 的@Validated

#### 代码实现
- Validated

```
package validation;

import org.springframework.stereotype.Component;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @ClassName Validated
 * 参数校验，可使用到方法、类级别
 * 功能：
 *  1.基于Hibernate Validator实现。
 *  2.自动识别dubbo方法和普通方法，dubbo方法校验不通过返回DubboResult对象，普通方法抛出ConstraintViolationException异常。
 *  3.结合<a href="javax.validation.Valid">@Valid</a>使用，可实现嵌套校验。
 *
 * 问题：1.方法级别校验目前groups参数不生效，Bean级别groups可正常使用。
 *
 * @Description
 * @Author lixin
 * @Date 2019-05-20 15:42
 */
@Component
@Target(value = {ElementType.METHOD,ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Validated {

    Class<?>[] value() default {};

    Class<?>[] groups() default {};
}

```

- ValidationAspect

```
package validation;

import com.example.demo.dubbo.DubboResult;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.core.annotation.AnnotationUtils;
import org.springframework.stereotype.Component;

import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.Objects;
import java.util.function.Predicate;

/**
 * @ClassName ValidationAspect
 * @Description
 * @Author lixin
 * @Date 2019-05-20 15:43
 */
@Component
@Aspect
@Slf4j
public class ValidationAspect {

    private final Class<? extends Annotation> ANNOTATION_TYPE = Validated.class;

    private final String DUBBO_2_6_X = "com.alibaba.dubbo.config.annotation.Service";
    private final String DUBBO_2_7_X = "org.apache.dubbo.config.annotation.Service";


    @Pointcut("@annotation(validation.Validated)")
    public void methodValid(){}

    @Pointcut("@within(validation.Validated)")
    public void classValid(){}

    /**
     * valid parameters
     * @param joinPoint
     */
    @Around(value = "methodValid() || classValid()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] args = joinPoint.getArgs();
        if(Objects.isNull(args) || args.length == 0){
            return joinPoint.proceed();
        }

        Method method = ((MethodSignature) joinPoint.getSignature()).getMethod();

        Annotation annotation = obtainMethodAnnotation(method);
        if(Objects.isNull(annotation)){
            log.info("no method annotation.");
            return joinPoint.proceed(args);
        }

        Class<?>[] groups = ((Validated)annotation).groups();

        Validator.ValidateResult validateResult = Validator.validateMethod(joinPoint.getThis(), method, args, groups);

        if(validateResult.isHasErrors()){
            if(dubboService(method.getDeclaringClass())){
                return DubboResult.newFailure(validateResult.toString());
            }else{
                throw new ConstraintViolationException(validateResult.toString());
            }
        }

        return joinPoint.proceed(args);
    }

    /**
     * abtain method or class annotation
     * @param method
     * @return
     */
    private Annotation obtainMethodAnnotation(Method method){
        Annotation annotation = AnnotationUtils.findAnnotation(method, ANNOTATION_TYPE);
        if(Objects.isNull(annotation)){
            annotation = AnnotationUtils.findAnnotation(method.getDeclaringClass(),ANNOTATION_TYPE);
        }
        return annotation;
    }

    /**
     * is dubbo service
     * @param clazz
     * @return
     */
    private boolean dubboService(Class clazz){
        return Arrays.asList(clazz.getAnnotations())
                .stream()
                .filter(dubboServicePredicate)
                .count() > 0;
    }

    private final Predicate<Annotation> dubboServicePredicate = annotation -> {
            String annotationName = annotation.annotationType().getName();
            return DUBBO_2_6_X.equals(annotationName) || DUBBO_2_7_X.equals(annotationName);
    };

}

```

- Validator

```
package validation;

import lombok.Data;
import org.hibernate.validator.HibernateValidator;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.ValidatorFactory;
import javax.validation.executable.ExecutableValidator;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * Validation tool
 *
 * @author lixin
 * @date 2019-05-20 14:20:21
 * @since jdk1.8
 */
public class Validator {

    private final static ValidatorFactory factory = Validation
            .byProvider(HibernateValidator.class)
            .configure()
            .failFast(false)
            .buildValidatorFactory();

    /**
     * object validator
     */
    private static javax.validation.Validator objectValidator = null;

    /**
     * executable method validator
     */
    private static ExecutableValidator methodValidator = null;

    static{
        objectValidator = factory.getValidator();
        methodValidator = factory.getValidator().forExecutables();
    }

    /**
     * validate bean or with groups
     *
     * @param t bean
     * @param groups
     * @return ValidResult
     */
    public static <T> ValidateResult validateBean(T t,Class<?>...groups) {
        return buildValidateResult(objectValidator.validate(t,groups));
    }
    /**
     * validate a property of bean
     *
     * @param t
     * @param propertyName
     * @return ValidResult
     */
    public static <T> ValidateResult validateProperty(T t, String propertyName) {
        return buildValidateResult(objectValidator.validateProperty(t, propertyName));
    }

    /**
     * validate method parameters
     *
     * @param t
     * @param method
     * @param parameterValues
     * @param groups
     * @return
     */
    public static <T> ValidateResult validateMethod(T t,
                                                Method method,
                                                Object[] parameterValues,
                                                Class<?>... groups){
        return buildValidateResult(methodValidator.validateParameters(t, method, parameterValues, groups));
    }


    /**
     * build validate result
     * @param violations
     * @param <T>
     * @return
     */
    public static <T> ValidateResult buildValidateResult(Set<ConstraintViolation<T>> violations){
        ValidateResult result = new Validator().new ValidateResult();
        result.setHasErrors(Objects.nonNull(violations) && violations.size() > 0);

        if(result.isHasErrors()){
            violations.forEach(violation -> result.addError(violation.getPropertyPath().toString(),violation.getMessage()));
        }

        return result;
    }

    /**
     * validate result
     */
    @Data
    public class ValidateResult {

        /**
         * has error
         */
        private boolean hasErrors;

        /**
         * error message
         */
        private List<ErrorMessage> errors;

        public ValidateResult() {
            this.errors = new ArrayList<>();
        }

        /**
         * all errors
         * @return
         */
        @Override
        public String toString() {
            return errors.stream()
                    .map( e -> e == null ? "null" : e.getPath() + ": " + e.getMessage() )
                    .collect( Collectors.joining( ", " ) );
        }

        /**
         * add one error
         * @param propertyName
         * @param message
         */
        public void addError(String propertyName, String message) {
            this.errors.add(new ErrorMessage(propertyName, message));
        }
    }

    @Data
    public class ErrorMessage {

        private String path;

        private String message;

        public ErrorMessage(String path, String message) {
            this.path = path;
            this.message = message;
        }
    }

}
```

- ConstraintViolationException

```
package validation;

import javax.xml.bind.ValidationException;
import java.io.Serializable;

/**
 * @ClassName ConstraintViolationException
 * @Description
 * @Author lixin
 * @Date 2019-05-21 17:47
 */
public class ConstraintViolationException extends ValidationException implements Serializable {
    public ConstraintViolationException(String message) {
        super(message);
    }
}

```