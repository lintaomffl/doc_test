# Spring IoC3Bean生命周期相关

参考学习文章：<br />[https://open.atatech.org/articles/176193](https://open.atatech.org/articles/176193)
<a name="1"></a>
## 回调处理—Bean的初始化和销毁
<a name="2"></a>
### Bean的元数据定义实现
Spring支持在定义bean的元数据时，显式指定初始化/销毁的回调方法；<br />基于xml形式：
```
<bean id="testBean" class="com.TestBean" init-method="diyInitial", destroy-method="diyDestroy" />
```
基于javaConfig形式：
```
public class TestBean {
    public void diyInitial() {
        System.out.println("invoke diyInitial()");
    }
    public void diyDestroy() {
        System.out.println("invoke diyDestroy()");
    }
}
```
```
@Configuration
public class BeanConfig {
    @Bean(initMethod = "diyInitial", destroyMethod = "diyDestroy")
    public TestBean testBean() {
        System.out.println("start injecting testBean");
        return new TestBean();
    }
}
```
<a name="3"></a>
### Spring内置接口实现
Spring内置提供两种情况：

- `InitializingBean`：bean实现`afterPropertiesSet()`方法完成初始化回调；
- `DisposableBean`：bean实现`destroy()`方法完成销毁前回调；
<a name="4"></a>
### JSR-250标准实现：@PostConstruct和@PreDestroy
`@PostConstruct`和`@PreDestroy`是JSR-250标准引入注解，作用于**无参方法**上，顾名思义表示创建bean之前，销毁bean之后的回调方法；Spring在代码层面使用`CommonAnnotationBeanPostProcessor`负责解析执行上述注解。
```
public class TestBean {
    @PostConstruct
    public void postConstruct() {
        System.out.println("invoke @PostConstruct annotated method");
    }
    @PreDestroy
    public void preDestroy() {
        System.out.println("invoke @PreDestroy annotated method");
    }
}
```
<a name="5"></a>
### 各类回调方式执行顺序
本小节总结下同时使用以上回调机制时，各类方法的执行先后顺序：<br />测试bean:
```
public class TestBean implements InitializingBean, DisposableBean {
    public void diyInitial() {
        System.out.println("invoke diyInitial()");
    }
    public void diyDestroy() {
        System.out.println("invoke diyDestroy()");
    }
    @PostConstruct
    public void postConstruct() {
        System.out.println("invoke @PostConstruct annotated method");
    }
    @PreDestroy
    public void preDestroy() {
        System.out.println("invoke @PreDestroy annotated method");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("invoke afterPropertiesSet()");
    }
    @Override
    public void destroy() throws Exception {
        System.out.println("invoke destroy()");
    }
}
```
配置类：
```
@Configuration
public class BeanConfig {
    @Bean(initMethod = "diyInitial", destroyMethod = "diyDestroy")
    public TestBean testBean() {
        System.out.println("start injecting testBean");
        return new TestBean();
    }
}
```
测试类：
```
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = { BeanConfig.class })
public class BeanInjectTest {
    @Test
    public void printCallBackInfoWhenInjectTestBean() {
        // 仅测试bean注入回调方法执行顺序
    }
}
```
打印日志：
```
start injecting testBean
invoke @PostConstruct annotated method
invoke afterPropertiesSet()
invoke diyInitial()
invoke @PreDestroy annotated method
invoke destroy()
invoke diyDestroy()
```

- **初始化顺序(按时间先后)**：`@PostConstruct` -> `InitializingBean#afterPropertiesSet()` -> Bean元数据中自定义初始化方法
- **销毁执行顺序(按时间先后)**：`@PreDestroy` -> `DisposableBean#destroy()` -> Bean元数据中自定义销毁方法

---

<a name="6"></a>
## Bean的懒加载机制
<a name="7"></a>
### 预先加载bean的局限
**预先Bean初始化(eager bean initial)**：默认情况下，Spring容器在启动阶段创建Bean；
> 默认情况下，Spring IoC容器(ApplicationContext)会自动实例化所有singleton bean并且以缓存形式保存，不自动实例化prototype bean

**预先初始化存在问题**：一些Bean只需在使用时初始化，预先初始化不必要的bean会造成时空开销；针对预先Bean初始化问题，<br />针对预先加载的问题，Spring支持Bean懒加载机制(lazy Bean initialization)，让Bean在真实起作用时才被加载；
> 延迟初始化存在问题：无法在服务启动前发现Bean配置错误；

在基于注解和基于Java的配置中，使用`org.springframework.context.annotation.Lazy`注解定义延迟创建，
<a name="8"></a>
### 基于注解配置实现懒加载
```
@Component
@Lazy(true)
public class TestBean {
 // ...
}
```
<a name="9"></a>
### 基于JavaConfig实现懒加载
```
@Configuration
public class BeanConfig {
    @Bean
    @Lazy(true) // 懒加载
    public TestBean testBean() {
        return new TestBean();
    }
}
```
<a name="10"></a>
### 使用懒加载解决循环依赖问题
假定存在循环依赖关系`TestA->TestB->TestA`；使用懒加载机制可循环依赖问题，`@Lazy`在代码层面作用于`Beandefinition`的`setLazyInit()`方法，当注入TestA时，Spring不是直接注入依赖项testB，而是由于TestB启用了懒加载机制而**临时注入一个代理对象**，当testB被真正调用时，才从容器中取出真正的对象进行替换执行；<br />bean懒加载完成注入时，注入的实际上是一个临时的代理对象；
```
@Componet
public class TestA {
    @Autowired
    private TestB testB;
    // ...
}
```
```
@lazy
@Componet
public class TestB {
    @Autowired
    private TestA testA;
    // ...
}
```
