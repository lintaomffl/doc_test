# Spring IoC2依赖注入

参考学习文章：<br />[https://open.atatech.org/articles/176192](https://open.atatech.org/articles/176192)
<a name="1"></a>
## 依赖注入(Dependency Injection)
Spring的核心模块之一IoC通过控制反转，将Bean的创建与注入交由容器负责。容器在构造Bean时，会自动分析该Bean的相关依赖项，并将依赖项依次装配填充——所谓「依赖注入」；<br />本节梳理下Spring依赖注入的不同实现与注意点。

---

<a name="2"></a>
## 依赖注入的实现途径
Spring解决依赖注入的工程思想分为：

- 类构造器方式(like Constructor-based)：仿效Java构造器形式；
- 类Setter方式(like Setter-based)：仿效Setter方法形式；
| 实现方式 | JavaConfig | 注解 | 侧重点 |
| :--- | :--- | :--- | :--- |
| 类构造器(Constructor-based) | ✅ | ⭕ | 侧重于注入**必需依赖项** |
| 类Setter(Setter-based) | ✅ | ✅ | 侧重于注入**可选依赖项** |

<a name="3"></a>
## 依赖注入—基于注解的实现
在Bean内，对依赖项加以注解实现注入；
<a name="4"></a>
### 相关注解一览
|  | 适用目标 | 作用于成员变量 | 作用于构造器 | 作用于setter方法 | 其他 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `@Value` | 基本数据类型，配置数据 | ✅ | ✅ | ✅ |  |
| `@Autowired` | 复杂数据类型，byType注入 | ✅ | ✅ | ✅，支持多个入参对象注入 |  |
| `@Resource` | 复杂数据类型，byName注入 | ✅ | ⭕ | ✅，只支持单个入参对象注入 |  |
| ~~`@Inject`~~ | ~~JSR-330标准，暂不讨论~~ | ~~暂不讨论~~ | ~~暂不讨论~~ | ~~暂不讨论~~ |  |

<a name="5"></a>
### 注入配置数据: @Value
`@Value("${属性名}")`用于加载**配置属性**；使用`@PropertySource`加载指定配置文件，否则Spring Boot默认从`application.properties`来获取配置的属性值；<br />配置文件`application.properties`：
```
config.name=tom
config.desc=boy
```
测试Bean`TestBean`：
```
@Componet
public class TestBean {
    @Value("${config.name}")
    private String name;
    private String desc;
    public TestBean(@Value("${config.desc}") String desc) {
        this.desc = desc;
    }
}
```
指定加载配置文件`com/property/user.properties`,
```
@PropertySource("classpath:com/property/user.properties")
@Componet
public class TestBean {
    @Value("${user.name}")
    private String name;
    // ...
}
```
此外`@Value`[还支持SpEL表达式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-value-annotations)；
<a name="6"></a>
### 按类型依赖注入: @Autowired
`@Autowired`对依赖项按照其类型(byType)进行注入；
```
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
    boolean required() default true;
}
```
`@Autowired`的`required`属性表示作用依赖项是否必需，**默认必需**；当显式指定`required=false`时，对应依赖项不在IoC中时，不会进行对应注入操作；
```
public class TestObject {
    /**
     * direct injection
     */
    @Autowired
    private DependentObject dependency1;
    /**
     * inject all beans of AbstractService which is a abstract class or interface,
     * the keys contains all bean names,
     * the value contains the corresponding specific bean;
     */
    @Autowired
    private Map<String, AbstractService> serviceSet;
    /**
     * inject all beans of AbstractService
     */
    @Autowired
    private Set<AbstractService> serviceSet;
    /**
     * inject all beans of AbstractService
     */
    @Autowired
    private Set<AbstractService> serviceList;
    /**
     * inject all beans of AbstractService
     */
     /
    @Autowired
    private AbstractService[] serviceArray;
    private DependentObject1 dependency2;
    private DependentObject1 dependency3;
    /**
     * constructor-based injection
     * the annotated constructor does not have to be public
     */
    @Autowired
    public TestObject(DependentObject dependency2) {
        this.dependency2 = dependency2;
    }
    /**
     * setter-based injection
     */
    @Autowired
    public void setDependency3(DependentObject dependency3) {
        this.dependency3 = dependency3;
    }
}
```
<a name="7"></a>
### 按名称依赖注入: @Resource
`@Resource`根据`name`属性对拥有对应名称的bean进行注入，**默认为成员变量名称或者setter方法的入参名称**；
```
public class TestObject {
    /**
     * 默认按照name = dependency1注入
     */
    @Resource
    private Dependency dependency1
    private Dependency dependency2;
    private Dependency dependency3;
    /**
     * 默认按照name = dependency2注入
     */
    @Resource
    public void setDependency(Dependency dependency2) {
        this.dependency2 = dependency2
    }
    /**
     * 显式指定bean名称，按照specificDependency查找并注入dependency3
     */
    @Resource(name = "specificDependency")
    public void setDependency(Dependency dependency3) {
        this.dependency3 = dependency3
    }
}
```
<a name="8"></a>
### 多依赖项选择问题
当依赖项在IoC存在多个待选Bean时，如何选择？
<a name="9"></a>
#### @Autowired

- 对于多个候选bean，`@Autowired`隐式按照bean id去匹配，建议使用`Qualifier`显式指定；
- `@Autowired`可通过额外的`@Qualifier`指定名称进行唯一注入；
<a name="10"></a>
#### @Resource
@Resource有两个属性`name`与`type`(默认Object类型)，装配顺序如下：

- 同时指定name与type，Spring从上下文中唯一确定bean进行装配，找不到抛出异常；
- 仅指定name，从上下文中查找bean ID匹配的bean进行装配，找不到抛出异常；
- 仅指定type，从上下文中找到类型匹配的的唯一bean进行装配，找不到或不唯一抛出异常；
- 如果name与type均未指定，默认按照name进行装配，如果没有匹配则回退到原始类型进行匹配，匹配成功则进行自动装配；

---

<a name="11"></a>
## 依赖注入—基于JavaConfig的实现
<a name="12"></a>
### 基本注入实现
依赖项如果是

- 基本数据类型：显式按照`@Value`限定名称从配置文件里加载值；
- 非基本类型：**隐式**按照Autowired注入，可使用`@Qualifier`限定名称；
```
/**
* 基于JavaConfig
* Constructor-based方式依赖注入
* @author yuanjie
*/
@Configuration
public class AppConfig {
/**
 * 假定Bean TransferService无任何依赖项
 */
@Bean
public TransferService transferServiceWithoutDependency() {
    return new TransferServiceImpl();
}
/**
 *  accountRepository按照Autowired方式注入，但无需@Autowired
 */
@Bean
public TransferService transferServiceOne(AccountRepository accountRepository) {
    return new TransferServiceImpl(accountRepository);
}
@Bean
public TransferService transferServiceTwo(@Value("${service.name}")String name,
@Qualifier("account1") AccountRepository accountRepository) {
    return new TransferServiceImpl(name, accountRepository);
}
}
```
<a name="13"></a>
### 导入其他不同方式下的依赖Bean
如[Spring IoC开发笔记(1)](https://yzwall.co/2020/07/05/spring-ioc-1/#%E5%AF%BC%E5%85%A5%E5%85%B6%E4%BB%96%E4%B8%8D%E5%90%8C%E6%96%B9%E5%BC%8F%E4%B8%8B%E7%9A%84bean%E5%85%83%E6%95%B0%E6%8D%AE)所谈，当依赖项的元数据已经在其他Config类/XML中定义完毕，可以使用以下方式导入：

- 基于注解方式：使用`@Autowired`或`@Resource`以成员变量形式注入；
- 基于JavaConfig方式：使用`@Import`导入其他config类定义好的bean，使用基于注解方式显式注入;
- 基于XML方式：使用`@ImportResource`扫描xml中定义好的bean，使用基于注解方式显式注入；
> 🔔 推荐使用基于注解方式显式注入依赖项，而不是前文所述仿效构造器形式，入参由Spring黑盒处理注入

假定TransferService分别依赖dependency1，dependency2和dependency3，其中：

- dependency1在xml中装配完毕；
- dependency2和dependency4在其他JavaConfig中装配完毕；
- dependency3直接使用@Componet标识bean；

dependency1: 基于xml，`beans-another.xml`:
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <bean id="dependency1" class="com.xxx.Dependency1">
        <!-- ... -->
    </bean>
</beans>
```
dependency2和dependency4: 基于JavaConfig
```
package com.xxx;
@Configuration
public class AnotherConfig {
    @Bean
    public Dependency2 dependency2() {
        Dependency2 dependency2 = new Dependency2();
        return dependency2;
    }
    @Bean
    public Dependency4 dependency4() {
        Dependency2 dependency4 = new Dependency4();
        return dependency4;
    }
}
```
dependency3: 基于注解形式
```
@Component
public class Dependency3 {
    // ...
}
```
```
@Configuration
@Import({Dependency2.class, Dependency4.class})
@ImportResource("classpath:/com/config/beans-another.xml")
public class BeanConfig {
    @Autowired
    Dependency1 dependency1;
    @Autowired
    Dependency2 dependency2;
    @Autowired
    Dependency3 dependency3;
    @Bean
    public TransferService transferService() {
        TransferService service = new TransferServiceImpl();
        service.setDependency1(dependency1);
        service.setDependency2(dependency2);
        service.setDependency3(dependency3);
        return service;
    }
}
```
<a name="14"></a>
### 同属一个Config类下的Bean之间的依赖注入
如下边代码所示，`beanOne`构造时，引用了`beanTwo()`构造方法，Spring会在构造`beanOne`时，先将`beanTwo`构造完毕后注入到`beanOne`中；
```
@Configuration
public class AppConfig {
    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }
    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```
<a name="15"></a>
### 从配置文件中导入数据
`@PropertySource`搭配`@Value`实现；
<a name="16"></a>
### 多依赖项选择问题
当注入时出现多个候选Bean时，可使用`@Primary`指定具体Bean为选择第一顺位，否则抛出异常：
```
@Configuration
public class AppConfig {
    /**
     * 指定该Bean为注入TransferService的第一选择
     */
    @Primary
    @Bean
    public TransferService transferServiceWithoutDependency() {
        return new TransferServiceImpl();
    }
}
```
<a name="17"></a>
### 条件注入—@Conditional
Spring提供注解`@Conditional`和`@ConditionalOnXXX`，支持**满足自定义条件**才进行Bean创建；<br />对于日常开发，一般Spring内置的`ConditionalOnXXX`注解即可满足大部分要求；<br />这里[有篇文章](https://www.javazhiyin.com/44985.html)汇总的比较全面，可以参考;

---

<a name="18"></a>
## 规定Bean依赖关系—@DependsOn
> 理论上Spring IoC会按照依赖关系进行递归注入

在编程层面，Spring提供注解`@DependsOn`保证bean的依赖项全部注入完毕后，当前bean才开始注入；<br />假定`TransferService`依赖`Dependency1`，`Dependency2`。
```
@Component("dependency1")
public class Dependency1 {
    // ...
}
```
```
@Component("anotherDependency2")
public class Dependency2 {
    // ...
}
```
`@DependsOn`与`@Component`搭配：
```
@DependsOn({"dependency1", "anotherDependency2"})
@Component
public class TransferService {
    // ...
}
```
`@DependsOn`在JavaConfig中的应用：
```
@Configuration
public class AppConfig {
    @DependsOn({"dependency1", "anotherDependency2"})
    @Bean
    public TransferService transferService {
        // ...
    }
}
```
值得注意的是`@DependsOn`只能保证依赖bean已被创建，对应成员变量已被填充，**不保证**依赖bean的`@PostConstruct`方法已被执行。

---

<a name="19"></a>
## JSR标准注解
在JSR标准中，

- `@Inject`(JSR-330)等同于Spring中的`@Autowired`;
- `@Named`(JSR-330)或者`@ManagedBean`(JSR-250)等同于`@Component`；

除此之外，Spring提供了标准之外的其他实现，比如`@Value`等；JSR标准注解的局限[戳这里](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-standard-annotations)<br />从标准与实现的关系来讲，应该尽可能使用标准而非实现细节，这样当存在一个比Spring更优越的框架出现时，业务代码由于是框架无关的，因此变更成本较小。但是目前JavaEE领域几乎早已是Sprin全家桶的天下，Spring的实现已逐渐成为JavaEE开发的实际标准😅
