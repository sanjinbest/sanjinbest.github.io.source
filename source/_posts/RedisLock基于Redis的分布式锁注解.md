title: '@RedisLock基于Redis的分布式锁注解'
author: 木子三金
tags:
  - redis
  - lock
  - 分布式锁
categories:
  - 实用工具
date: 2019-12-16 10:38:00
---
>背景：日常开发中分布式锁是一个常用的工具，通常我们都是手动在代码里获取-释放锁，我们每次都需要写重复的代码，且易出现锁忘记释放的情况。因此封装了一个基于Redis的分布式锁实现。

<!-- more -->

# 使用方式及约束
## 使用方式
示例代码：
```
    /**
     * 分布式锁示例
     * @param key
     */
    @RedisLock(prefixKey = "redisLockKey")
    public void redisLockDemo(String key){

    }
```

## 约束
- @RedisLock注解作用于**方法**之上。
- 要求方法的返回值为 **void**
- 且方法的**第一个参数为锁的key**
- 注解强制提供**prefixKey**参数，防止key重复
- 获取锁失败会抛出**LockedFailException**异常，可自行捕获并处理
- 由于依赖Redis，固需要自行注入redisClient

# 参详详解
|参数名称|必要|含义|默认值|
|----|----|----|----|
|prefixKey|Y|分布式锁key前缀|无|
|expire|N|锁过期时间|5 * 60 * 1_000L|
|retryTimes|N|获取锁失败重试次数|3|
|sleepMillis|N|获取锁失败重试间隔时间|100|

# 源码
## @RedisLock
```
/**
 *
 * 基于redis的锁工具
 *
 * 约定：使用方法的第一个参数为key的扩展变量
 *
 * 注解会隐式抛出2类异常：
 * @throws LockedFailException  获取锁失败
 * @throws Throwable  目标方法执行异常
 *
 * @Author lixin
 * @Date 2019-04-26 22:04
 */
@Component
@Target(value = {ElementType.METHOD})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface RedisLock {

    /**
     * key prefix
     * @return
     */
    String prefixKey();

    /**
     * expire. default = 5 * 60 * 1_000L
     * @return
     */
    long expire() default 5 * 60 * 1_000L;

    /**
     * retyr times. default = 3
     * @return
     */
    int retryTimes() default 3;

    /**
     * sleep millis. default = 60 * 100
     * @return
     */
    long sleepMillis() default 100;

}
```
## RedisLockAspect.java
```
/**
 * redis锁切面实现
 *
 * @Author lixin
 * @Date 2019-04-26 22:11
 */
@Aspect
@Slf4j
public class RedisLockAspect {

    private RedisUtil redisUtil;

    @Pointcut("@annotation(redisLock)")
    public void locked(RedisLock redisLock){}

    public RedisUtil getRedisUtil() {
        return redisUtil;
    }

    public void setRedisUtil(RedisUtil redisUtil) {
        this.redisUtil = redisUtil;
    }

    /**
     *
     * @param joinPoint
     * @param redisLock
     * @throws LockedFailException  获取锁失败
     * @throws Throwable  目标方法执行异常
     */
    @Around("locked(redisLock)")
    public void around(ProceedingJoinPoint joinPoint, RedisLock redisLock) throws Throwable {
        String key = redisLock.prefixKey() + joinPoint.getArgs()[0];
        String value = UUID.randomUUID().toString();
        boolean locked = redisUtil.tryGetDistributedLock(
                key,
                value,
                redisLock.expire(),
                redisLock.retryTimes(),
                redisLock.sleepMillis());

        if(!locked){
            log.error("RedisLockAspect try get lock fail.key:{},value:{}",key,value);
            throw new LockedFailException("try get distributed lock fail.key:"+key);
        }

        log.info("RedisLockAspect try get lock success. key : {},value : {}",key,value);

        try{
            joinPoint.proceed();
        }finally {
            redisUtil.releaseDistributedLock(key,value);
            log.info("RedisLockAspect unlock. key : {},value : {}",key,value);
        }
    }
}
```
## LockedFailException.java
```
/**
 * @ClassName LockedFailException
 * @Description
 * @Author lixin
 * @Date 2019-04-28 15:09
 */
public class LockedFailException extends RuntimeException {

    public LockedFailException(String msg){
        super(msg);
    }
}
```