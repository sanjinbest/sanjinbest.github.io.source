---
title: spring security 入门教程
date: 2018-02-24 12:31:03
comments: true
toc: true
categories: 
 - spring
tags: 
 - spring 
 - security
 - spring
 - 安全
 - 权限控制
---

本篇文章只作为spring security入门使用，具体深入内容请自行查询相关资料。
<!-- more -->

# spring security简介
 Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI（控制反转Inversion of Control ,DI:Dependency Injection 依赖注入）和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。当前版本为4.2.3。
# 接入方式
Spring Security的接入方式一共有两种：基于注解方式和基于xml配置方式。下面对两种接入方式作一下介绍。
首先是spring security依赖引入
pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>demo.security</groupId>
    <artifactId>security</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <!-- spring版本号 -->
        <spring.version>4.1.6.RELEASE</spring.version>
        <security.version>4.0.1.RELEASE</security.version>
        <!-- log4j日志文件管理包版本 -->
        <slf4j.version>1.7.7</slf4j.version>
        <log4j.version>1.2.17</log4j.version>

    </properties>

    <dependencies>
    <!-- springframework -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-oxm</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-core</artifactId>
        <version>${security.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-config</artifactId>
        <version>${security.version}</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-taglibs</artifactId>
        <version>${security.version}</version>
    </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>3.0-alpha-1</version>
            <scope>provided</scope>
        </dependency>
        <!-- 导入Mysql数据库链接jar包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.30</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>ROOT</finalName>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>config/${env}</directory>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>

        </project>
```
这里使用的是servlet3.0，自servlet3.0+规范后，允许servlet，filter，listener不必声明在web.xml中，而是以硬编码的方式存在，实现容器的零配置。
## 基于注解方式
第一步是要创建Spring Security的Java 配置类。
创建类SecurityConfiguration继承WebSecurityConfigurerAdapter，来对我们应用中所有的安全相关的事项（所有url，验证用户名密码，表单重定向等）进行控制。

SecurityConfiguration.java
```
package com.security.code;

import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

import javax.sql.DataSource;

/**
 * <li>security config</li>
 *
 * @author lixin
 * @create 17/3/22
 */
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter{

    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        
    }

    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        
    }

    /**
     * 拦截请求
     * @param web
     * @throws Exception
     */
    @Override
    public void configure(WebSecurity web) throws Exception {

	}
}
```
WebSecurityConfigurerAdapter共有三个configure方法。
```
configure(WebSecurity) 通过重载，配置Spring Security的Filter链
configure(HttpSecurity) 通过重载，配置如何通过拦截器保护请求
configure(AuthenticationManagerBuilder) 通过重载，配置user-detail服务
```
@EnableWebSecurity 注解将会启用Web安全功能。

第二步是初始化springSecurityFilter注册类，这里使用的方法是继承类AbstractSecurityWebApplicationInitializer
```
package com.security.code;

import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

/**
 * <li>注册springSecurityFilter</li>
 *
 * @author lixin
 * @create 17/3/22
 */
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer
{
}
```
这里可以没有任何实现。至此，基于注解方式的接入就完成了。下面介绍一下基于xml配置方式的接入。

## 基于xml配置方式接入
第一步，在web.xml加入配置
```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```
第二步，增加spring-security.xml，并引入到application.xml内

spring-security.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:security="http://www.springframework.org/schema/security"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/security
           http://www.springframework.org/schema/security/spring-security.xsd">

    <security:http auto-config="true" use-expressions="true">
        ...
    </security:http>

    <security:authentication-manager>
        ..
    </security:authentication-manager>

</beans>
```
具体配置后续会添加。基于xml配置方式就这么多，下面介绍一下用户的存储认证方式。

# 用户存储认证方式
spring security关于用户存储认证方面是非常灵活的，能够基于各种数据存储来认证用户。它内置了多种常见的用户存储场景，下面对各种场景进行下介绍。

## 使用基于内存的用户存储
从名称上可以知道这种方式是将用户名、密码、权限等数据存储在内存中，一般个人开发测试可以使用这种方式。

注解编码方式：
```
    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        //基于内存的用户存储、认证
        auth.inMemoryAuthentication()
                .withUser("admin").password("admin").roles("ADMIN","USER")
                .and()
                .withUser("user").password("user").roles("USER");
    }
```
代码里通过方法auth.inMemoryAuthentication()获取AuthenticationManagerBuilder对象，并设置了两个用户admin和user同时设置对应密码和所拥有的权限。

这里需要注意的是，roles()方法是authorities()方法的简写形式。roles()方法所给定的值都会加一个"ROLE_"前缀，并将其作为权限授予用户。实际上，如下用户配置与上面程序是一样的。
```
    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
         //基于内存的用户存储、认证
        auth.inMemoryAuthentication()
                .withUser("admin").password("admin").authorities("ROLE_AMIN","ROLE_USER")
                .and()
                .withUser("user").password("user").authorities("ROLE_USER");

    }
```
除了上面的方法还有一些其它方法可以配置用户的详细信息。
```
accountExpired(boolean) 定义账号是否已经过期
accountLocked(boolean) 定义账号是否已经锁定
and() 用来连接配置
authorities(GrantedAuthority...) 授予某个用户一项或多项权限
authorities(List<? extends GrantedAuthority>) 授予某个用户一项或多项权限
authorities(String...) 授予某个用户一项或多项权限
credentialsExpired(boolean) 定义凭证是否已经过期
disabled(boolean) 定义账号是否已被禁用
password(String) 定义用户的密码
roles(String...) 授予某个用户一项或多项角色
```
xml配置方式：
```
    <security:authentication-manager>
        <security:authentication-provider>
            <security:user-service>
                <security:user name="admin" authorities="ROLE_ADMIN" password="admin"/>
                <security:user name="user" authorities="ROLE_USER" password="user"/>
            </security:user-service>
        </security:authentication-provider>
    </security:authentication-manager>
```
## 基于数据库表用户存储认证
通常我们都会将用户数据存储在关系型数据库中，并通过jdbc进行访问。spring security使用以jdbc为支撑的用户存储，我们可以使用下面的方式进行配置。

基于注解编码方式：
```
    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        //基于数据库的用户存储、认证
        auth.jdbcAuthentication().dataSource(dataSource)
                .usersByUsernameQuery("select account,password,true from user where account=?")
                .authoritiesByUsernameQuery("select account,role from user where account=?")；
    }
```
第一个查询语句获取了用户的基本信息，第二个查询语句获取了用户权限数据。

基于xml配置方式：
```
<security:authentication-manager>
        <security:authentication-provider>
            <security:jdbc-user-service data-source-ref="dataSource"
                                        authorities-by-username-query="select account,role from user where account=?"
                                        users-by-username-query="select account,password,true from user where account=?"/>
            <security:password-encoder ref="bcryptEncoder"/>
    </security:authentication-manager>
```
## 基于LDAP进行用户存储认证
这种方式没有进行测试，如果感兴趣可以自行测试。

## 配置自定义的用户存储认证
这种方式更为灵活，更适合在生产环境使用。这种方式不在局限于存储环境。自定义的方式也很简单。只需要提供一个UserDetailService接口实现即可。
```
package com.blog.admin.security;

import com.blog.admin.dao.UserDao;
import db.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

/**
 * <li></li>
 *
 * @author lixin
 * @create 17/3/20
 */
@Service
public class BlogSecurityUserDetailsService implements UserDetailsService{

    @Autowired
    private UserDao userDao;

    /**
     * 校验用户
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userDao.queryByUserName(username);
        if(user == null)throw new UsernameNotFoundException("user not found");

        return new org.springframework.security.core.userdetails.User(user.getAccount(),user.getPassword(),userDao.getUserGrantedAuthoritys(user.getId()));
    }
}
```
自定义的方式只要实现接口方法loadUserByUsername（String username）即可，返回代表用户的UserDetails对象。调用方式也很简单：
```
    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        //自定义方式
        auth.userDetailsService(securityUserDetailsService);
    }
```
基于xml方式：
```
    <security:authentication-manager>
        <security:authentication-provider user-service-ref="blogUserDetailService">
        </security:authentication-provider>
    </security:authentication-manager>
```
# 密码加密策略
通常我们在存储密码的时候都是进行加密的，spring security默认提供了三种密码存储方式，同时也可以使用自定义的加密方式：

- NoOpPasswordEncoder      明文方式保存
- BCtPasswordEncoder       强hash方式加密
- StandardPasswordEncoder  SHA-256方式加密
- 实现PasswordEncoder接口       自定义加密方式
注解编码方式：
```
    /**
     * 配置user-detail服务
     * @param auth
     * @throws Exception
     */
    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        //基于数据库的用户存储、认证
        auth.jdbcAuthentication().dataSource(dataSource)
                .usersByUsernameQuery("select account,password,true from user where account=?")
                .authoritiesByUsernameQuery("select account,role from user where account=?")
                .passwordEncoder(NoOpPasswordEncoder.getInstance());
    }
```
通过方法passwordEncoder传入对应的加密实例即可。

xml配置方式：
```
<security:authentication-manager>
        <security:authentication-provider>
            <security:jdbc-user-service data-source-ref="dataSource"
                                        authorities-by-username-query="select account,role from user where account=?"
                                        users-by-username-query="select account,password,true from user where account=?"/>
            <security:password-encoder ref="bcryptEncoder"/>
        </security:authentication-provider>
    </security:authentication-manager>

    <bean id="bcryptEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
```
# 请求拦截策略
下面开始介绍一下spring security的重要功能，请求拦截策略。spring security的请求拦截匹配有两种风格，ant风格和正则表达式风格。编码方式是通过重载configure(HttpSecurity)方法实现。
```
    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/","/css/**","/js/**").permitAll()   //任何人都可以访问
                .antMatchers("/admin/**").access("hasRole('ADMIN')")     //持有user权限的用户可以访问
                .antMatchers("/user/**").hasAuthority("ROLE_USER");
    }
```
上面使用的是ant风格的匹配，可以通过http.regexMatcher()方法使用正则表达式风格。

对应的xml配置方式：
```
    <security:http auto-config="true" use-expressions="true">
        <security:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <security:intercept-url pattern="/user/*" access="hasRole('ROLE_USER')"/>
    </security:http>
```
除了上面的请求保护方法外，还有一些方法能够用来定义如何保护请求。
```
access(String)     如果给定的SpEL表达式计算结果为true，就允许访问
anonymous()        允许匿名用户访问
authenticated()    允许认证过的用户访问
denyAll()          无条件拒绝所有访问
fullyAuthenticated()   如果用户是完整认证的话（不是通过Remember-me功能认证的），就允许访问
hasAnyAuthority(String...)   如果用户具备给定权限中的某一个的话，就允许访问
hasAnyRole(String...)   如果用户具备给定角色中的某一个的话，就允许访问
hasAuthority(String)   如果用户具备给定权限的话，就允许访问
hasIpAddress(String)   如果请求来自给定IP地址的话，就允许访问
hasRole(String)   如果用户具备给定角色的话，就允许访问
not()   对其他访问方法的结果求反
permitAll()   无条件允许访问
rememberMe()   如果用户是通过Remember-me功能认证的，就允许访问
```
# 强制安全性通道
通常我们都是使用http发送数据，这种方式是不安全的。对于敏感信息我们通常都是通过https进行加密发送。spring security对于安全性通道也提供了一种方式。我们可以在配置中添加requiresChannel()方法使url强制使用https。
```
    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/","/css/**","/js/**").permitAll()   //任何人都可以访问
                .antMatchers("/admin/**").access("hasRole('ADMIN')")     //持有user权限的用户可以访问
                .antMatchers("/user/**").hasAuthority("ROLE_USER")
                .and()
                .requiresChannel().antMatchers("/admin/info").requiresSecure();
    }
```
不论何时，只要是对“/admin/info”的请求，spring security都认为需要安全性通道，并自动将请求重定向到https上。

与之相反，如果有些请求不需要https传送，可以使用requiresInsecure()替代requiresSecure()，将请求声明为始终使用http传送。

对应xml配置：
```
    <security:http auto-config="true" use-expressions="true">
        <security:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <security:intercept-url pattern="/user/*" access="hasRole('ROLE_USER')" />
        <security:intercept-url pattern="/admin/info" access="hasRole('ROLE_ADMIN')" requires-channel="https"/>
    </security:http>
```
# 防止CSRF
spring security从版本3.2开始，默认就会启用CSRF防护。spring security通过一个同步token的方式来实现CSRF防护功能。它会拦截状态变化的请求，并检查CSRF token。如果请求中不包含CSRF token的话，或者token不能与服务器端的token匹配，请求就会失败，并抛出CsrfException异常。

如果使用JSP作为页面模板的话，需要作的非常简单。
```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
```
这样就spring security就会自动生成csrf token。如果想关闭csrf防护，需要作的也很简单，只需要调用一下csrf().disable();即可。

   编码方式：
```
    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/","/css/**","/js/**").permitAll()   //任何人都可以访问
                .antMatchers("/admin/**").access("hasRole('ADMIN')")     //持有user权限的用户可以访问
                .antMatchers("/user/**").hasAuthority("ROLE_USER")
                .and().csrf().disable();
    }
```
对应xml方式：
```
    <security:http auto-config="true" use-expressions="true">
        <security:csrf disabled="true" />
        <security:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <security:intercept-url pattern="/user/*" access="hasRole('ROLE_USER')" />
    </security:http>
```
# remember-me功能
remember-me是一个很重要的功能，用户肯定不希望每次都输入用户名密码进行登录。spring security提供的remember-me功能使用起来非常简单。启用这个功能只需要调用rememberMe()方法即可。
```
    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/","/css/**","/js/**").permitAll()   //任何人都可以访问
                .antMatchers("/admin/**").access("hasRole('ADMIN')")     //持有user权限的用户可以访问
                .antMatchers("/user/**").hasAuthority("ROLE_USER")
                .and().rememberMe().key("abc").rememberMeParameter("remember_me").rememberMeCookieName("my-remember-me").tokenValiditySeconds(86400)；
    }
```
同时可以设置参数名称，cookie的name和过期时间。对应的xml配置：
```
    <security:http auto-config="true" use-expressions="true">
        <security:remember-me />
        <security:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <security:intercept-url pattern="/user/*" access="hasRole('ROLE_USER')" />
    </security:http>
```
remember-me有多种参数可以选择配置。

这里有个需要注意的地方 ，在登录页面内的remember-me的input标签内不要设置value的值，否则remember-me功能将不会生效。

# 自定义登录页面
spring security会提供一个默认的登录页面，如果你想使用自己的登录页面，可以这样设置。

编码方式：
```
    /**
     * 拦截请求
     * @param http
     * @throws Exception
     */
    @Override
    public void configure(HttpSecurity http)throws Exception{
        http.authorizeRequests()
                .antMatchers("/","/css/**","/js/**").permitAll()   //任何人都可以访问
                .antMatchers("/admin/**").access("hasRole('ADMIN')")     //持有user权限的用户可以访问
                .antMatchers("/user/**").hasAuthority("ROLE_USER")
                .and().formLogin()
                .loginPage("/login").usernameParameter("username").passwordParameter("password")
                .and().exceptionHandling().accessDeniedPage("/loginfail");
    }
```
通过formLogin()方法来设置使用自定义登录页面，loginPage是登录页面地址，accessDeniePage登录失败跳转地址。

对应的xml配置：
```
    <security:http auto-config="true" use-expressions="true">
        <security:intercept-url pattern="/admin/*" access="hasRole('ROLE_ADMIN')"/>
        <security:intercept-url pattern="/user/*" access="hasRole('ROLE_USER')" />
        <security:form-login login-page="/login"
                             username-parameter="username"
                             password-parameter="password"
                             authentication-failure-url="/loginfail"
                             default-target-url="/"/>
    </security:http>
 ```

到此spring security的一些常用功能就介绍完了，写的比较粗糙，可能会存在一些错误。欢迎大家反馈交流。