#     核心机制

## 面向抽象编程

面向抽象 => OCP => 可维护的代码

IOC/DI 思想



SpringBoot

徐徐图之



开闭原则（OCP）Open Closed Principle

软件、函数、类 扩展开放的 修改封闭

新增业务模块/类  代替原来的类

里式替换原则   迪米特法则  IOC DI	



面向抽象编程 interface abstract

三大特性 多态性





// LOL 40

// 140

// 频繁变动

// 地图 道具

// 平衡性



可选英雄数量

选择一个英雄



// 10年

### 重点理论：

1. 单纯 interface 可以统一方法的调用，但是不能统一对象的实例化
2. 面向对象 实例化对象 调用方法（完成业务逻辑）
3. 只有一段代码中没有 new 的出现，才能保持代码的相对稳定，才能逐步实现 OCP
4. 上面的话只是表象，实质是一段代码如果要保持稳定，就不应该负责对象的实例化
5. 对象实例化是不可能消除的
6. 把对象实例化的过程，转移到其他的代码块里
7. 代码中总是会存在不稳定，隔离这些不稳定，保证其它的代码是稳定的
8. 变化（控制代码）造成了不稳定
9. 配置文件属于系统外部，不属于代码本身



计算机代码

现实世界规律 业务 映射 投影





### IOC、DI、DIP



#### DIP

Denpendency Inversion Principle  依赖倒置

1. 高层模块不应该依赖底层模块，两者都应该依赖抽象
2. 抽象不应该依赖细节（具体）
3. 细节不应该依赖抽象



高层：抽象

底层：细节（具体）



#### DI

Denpendency Injection 依赖注入

1. 属性注入
2. 构造注入
3. 接口注入



#### IOC

IOC  抽象 模糊 实现

实现 DI

实现/应用



主控类 Container 而不是 A



### 变化（控制代码）



控制权

程序员 用户



控制代码/新增业务代码



控制权 由 产品经理或者用户 来 控制 代码

eg：更改配置文件



积木生产厂家（程序员）

一个一个积木

玩家/用户





## spring  与 springboot

问题 理论 解决方案

精选 

SSM： Spring + Spring MVC + MyBatis

Spring Framework

SpringBoot 是 Spring Framework 的 应用

 

核心优势？

自动配置 有什么用



### SpringBoot

OCP -> IOC



IOC实现：容器 加入容器 注入

灵活性  场景



目的：

抽象意义： 控制权交给用户

灵活的OCP 



#### XML



#### 注解

Stereotype annotations 模式注解

@Compoent  组件/类/bean  类的实例化 new



@Service	服务

@Controller	控制器

@Repository	仓储



@Configuration	在一个类里面加入到一组类



桥节点 IOC



IOC 对象 实例化 注入 时机

立即/提前 实例化

延迟实例化  @Lazy

在 @Compoent 下 添加 @Lazy 后 必须在 @Controller 下 也加入 @Lazy 才能完成延迟实例化





#### @Autowired

##### 被动注入

bytype

byname （Diana=> diana; DIana=>DIana）



bytype 默认注入方式

1. 找不到任何一个bean
2. 一个 直接注入
3. 找个多个 并不一定报错  按照字段名字推断选择哪个bean 



##### 主动注入

@Qualifier("")



字段注入 / 成员变量注入 

set注入

构造注入



### Configuration 配置类

编程模式





### 为什么 Spring 偏爱 配置

OCP 变化 隔离/反应 配置文件



### 为什么隔离到配置文件

1. 配置文件集中性
2. 清晰（没有业务逻辑）



### 配置分类

1. 常规配置 key:value
2. Xml 配置 类/对象





#### @ComponentScan

默认在 Application 的目录下扫描

可以添加@ComponentScan("xx.xx") 更改





### 面向对象变化的应对方案

1. 制定一个 interface，然后用多个类实现同一个 interface。

   策略模式

2. 一个类， 更改属性， 达到对应变化。interface （防御性）

   MySQL Class

   Port: 3306/3307 （配置文件）

   Url: 



#### 策略模式的变化方案

1. byname 切换 bean name

2. @Qualifier 指定 bean

3. 有选择的之注入一个 bean 注释掉某个bean上的@Compoent

4. 使用 @Primary

5. @Conditional 条件注解

   自定义条件注解

   @Conditional + Condition



#### 内置的成品条件注解

@ConditionOnProperty(value="hero.condition", havingValue="irelia", matchIfMissing = true)

matchIfMissing = true 当配置文件找不到hero.condition时，的默认值

如果有hero.condition 但没找到则报错



@ConditionalOnBean 当 SpringIoc 容器存在指定 Bean 的条件

@ConditionalOnMissingBean 当 SpringIoc 容器不存在指定 Bean 的条件







### 自动装配

1. 原理是什么
2. 为什么要有自动装配



#### SpringBoot SDK

1. Component Configuration
2. 加载第三方 SDK





#### @EnableXXXX

模块装配



MongoDB @Configuration 组合

Redis @Configuration

业务 @Configuration



根据自己业务



### SPI  机制/思想

SPI 的全名 Service Provider Interface



模块 实现方案

​												方案A

调用方 标准服务接口			方案B

​												方案C

基于interface + 策略模式 + 配置文件



解决 @Primary @条件注解

具体/粒度



整体解决方案的变化



### 总结

自动装配：  把变化 其他SDK	加入到 IOC	里面

OCP





# 框架机制

不直接使用框架

二次封装/开发



增强型 SpringBoot



异常反馈

资源不存在/参数错误/内部错误

反馈给客户端/前端



## UnifyResponse统一错误响应

json

```json
{
  code:10001,
  message:xxxx,
  request: GET url
}
```

@ControllerAdvice

@ExceptionHandler(value = Exception.class)





## 异常分类

Throwable

Error 错误

Exception 异常  可以处理

**CheckedException 编译时进行处理**

**RuntimeException 运行时异常**



HttpException extends RuntimeException

APIException extends Exception



异常 处理 CheckedException

​		无能为力 RuntimeException		unchecked



**已知异常、未知异常**

未知异常：对于前端开发者和用户 都是无意义的。服务端开发者代码逻辑有问题





## Spring特色

不需要手动 主动 注册到 spring 里面，没有手动注册的过程

自己去发现 通过扫描注解去发现  避免手动写很多代码





控制器里面不能写 业务逻辑



## lombok

```java
@Setter
@Getter
@ToString
// final 修饰 只有get 没有 set
//@Data // Get Set equals hashCode toString
@AllArgsConstructor
@NoArgsConstructor
@RequiredArgsConstructor // 和 @NonNull 搭配使用
@Builder
```



## JSR

Java Specification Requests

JSR-303 Bean Validation



Hibernate-Validator

@Min

@Max

@Positive

2.3.2 移除 添加

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
 </dependency>
```



@valid 和 @validated

```java
@Validated
public class BannerController {
public PersonDTO testP(@RequestBody @Validated PersonDTO person) 

@Valid
private SchoolDTO schoolDTO;
```

自定义注解

```java
@Documented	// 可以让注解里面的注释加入到文档里面
@Retention(RetentionPolicy.RUNTIME)	// 注解要被保留到什么阶段
@Target(ElementType.TYPE)	// 指定 注解 用在某些目标上
@Constraint(validatedBy = PasswordValidator.class)
public @interface PasswordEqual {
    String message() default "passwords are not equal";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

