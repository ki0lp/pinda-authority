# pinda-authority

## 介绍

品达权限管理系统

## 软件架构

软件架构说明

## pd-tools-user

> 2023年03月05日23:35:08

实现方式：拦截器`HandlerInterceptorAdapter`和参数解析器`HandlerMethodArgumentResolver`

使用方式：在需要注入当前用户信息的controller方法上添加`@LoginUser`注解

```java
/**
 * 请求的方法参数SysUser上添加该注解，则注入当前登录人信息
 * 例1：public void test(@LoginUser SysUser user) // 取BaseContextHandler中的 用户id、账号、姓名、组织id、岗位id等信息
 * 例2：public void test(@LoginUser(isRoles = true) SysUser user) //能获取SysUser对象的实时的用户信息和角色信息
 * 例3：public void test(@LoginUser(isOrg = true) SysUser user) //能获取SysUser对象的实时的用户信息和组织信息
 * 例4：public void test(@LoginUser(isStation = true) SysUser user) //能获取SysUser对象的实时的用户信息和岗位信息
 * 例5：public void test(@LoginUser(isFull = true) SysUser user) //能获取SysUser对象的所有信息
 *
 */
```

### 建造者模式

`https://www.runoob.com/design-pattern/builder-pattern.html`

### EnableLoginArgResolver

通过再启动类上添加注解`com.itheima.pinda.user.annotation.EnableLoginArgResolver`直接激活配置

```markdown
这里想不明白的是，为什么需要通过拦截器来解析请求头中的token然后再来获取用户信息，为什么不用参数解析器直接来处理

还有所有的当前信息都是从请求头中获取的，那么是不是请求头中的带的信息太多了，为什么不获取到请求头中的token之后通过获取Redis中的用户信息来填充用户信息

如果说是为了达到一个通用性，但是既然都上微服务了，那多少是有Redis的吧
```

## pd-tools-validator

> 2023年03月05日23:36:16

实现方式：spring boot中自己已经带了`spring boot`里面的`spring-boot-starter-web`是默认应用了`Hibernate-validator`包，差个配置和激活，这里配置，然后在启动类添加注解激活

### hibernate-validator

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

`https://blog.csdn.net/millery22/article/details/123566489`

`https://www.jianshu.com/p/0bfe2318814f`

开启快速失败，不会进入到controller的方法调用中，通过全局异常捕捉来给前端提示

在启动类上添加该注解来启动表单验证功能`com.itheima.pinda.validator.config.EnableFormValidator`

## pd-tools-xss

> 2023年03月06日00:02:22

实现方式：`javaweb`原生的`Filter`方式过滤解析出请求头、请求报文中的攻击脚本，然后过滤（通过配置文件），通过`EnableAutoConfiguration`配置引入项目，也就是说只要是`maven`中依赖该包就立马生效

```xml
<!-- XSS -->
<dependency>
    <groupId>org.owasp.antisamy</groupId>
    <artifactId>antisamy</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>commons-logging</artifactId>
            <groupId>commons-logging</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

上面的两个都是可以通过注解来达到开关开启的方式，这个只要是引入就会直接生效

`src/main/resources/META-INF/spring.factories`中的`EnableAutoConfiguration`指向配置文件

## pd-tools-swagger2

> 2023年03月06日08:37:37

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-models</artifactId>
        </exclusion>
        <exclusion>
            <artifactId>swagger-annotations</artifactId>
            <groupId>io.swagger</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>io.swagger</groupId>
    <artifactId>swagger-models</artifactId>
</dependency>
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-bean-validators</artifactId>
</dependency>
```

### knife4j

`https://doc.xiaominfo.com/docs/quick-start`

## pd-tools-log

> 2023年03月06日08:56:37

### spring event

`https://mp.weixin.qq.com/s?__biz=MzIzMzgxOTQ5NA==&mid=2247580192&idx=1&sn=7cdb5fab108c978b9bb54e8eae6e1111&chksm=e8fc2629df8baf3fc6e527ee8d6a820cb5c94601ef0c98b34adc1c85647c0bacae8b07b1a2cd&scene=27`

`Spring Event（Application Event）`其实就是一个观察者设计模式，一个 `Bean` 处理完成任务后希望通知其它 `Bean` 或者说一个 `Bean` 想观察监听另一个`Bean` 的行为。

### Consumer

`https://blog.csdn.net/weixin_39773239/article/details/114839214`

`com.itheima.pinda.log.aspect.SysLogAspect.tryCatch` 中使用Consumer，只需要执行目标方法，不需要返回值，只是为了通用包裹异常的代码

### @Pointcut

`com.itheima.pinda.log.aspect.SysLogAspect.sysLogAspect`与`com.itheima.pinda.log.util.LogUtil.getControllerMethodDescription`说明`Aspect`并非是只能用在service层作为切面切入，只是我们惯性这么使用，任何有加上`@SysLog`注解的地方都会被切到

## pd-tools-jwt

> 2023年03月06日09:27:21

`https://www.cnblogs.com/sstu/p/15523887.html#10-pd-tools-jwt`

### JJWT

`https://blog.csdn.net/qq_45332753/article/details/113833777`

## pd-tools-j2cache

缓存中间件，作为数据库和业务层之间的缓存需求，存储地点：内存、Redis

## pd-tools-dozer

对象转换