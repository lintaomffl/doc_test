# Spring IoC1bean元数据实现

参考学习文章：<br />[https://open.atatech.org/articles/176191#5](https://open.atatech.org/articles/176191#5)<br />[https://open.atatech.org/articles/176192](https://open.atatech.org/articles/176192)<br />[https://open.atatech.org/articles/176193](https://open.atatech.org/articles/176193)<br />[https://open.atatech.org/articles/176194](https://open.atatech.org/articles/176194)
<a name="AcMf8"></a>
# bean元数据实现：
<a name="Qbzqb"></a>
## 控制反转：
对于一个类依赖另一个类的情况，如若调用依赖类的构造方法或者共产来进行实例化，会存在的大量耦合。所以利用一个统一的第三方容器来进行实例化，则能一定程度地解耦。所以组件类将依赖项的控制权交由一个容器，由容器来实现依赖注入，即可消除调用类对依赖组件的依赖。
<a name="w1xLf"></a>
## 实现方式：
主要的实现方式包括：基于XML方式、基于JavaConfig方式、基于类扫描方式。
<a name="0vmVE"></a>
### 基于JavaConfig方式

- `@Configuration`，等同于XML中的`<beans/>`，自身已注解`@Component`，作用是声明当前类负责在代码层面对Bean进行实例化，在ComponetScan时，类中所有`@Bean`方法会被全部调用生成`Beandefinition`，交给Spring IoC容器管理；
- `@Bean`等同于XML中的`<bean/>`，作用是将配置类中的成员方法标记为指定Bean的实例化逻辑；
   - Bean类型：Bean的类型与方法返回值类型相同；
   - Bean默认名称：Bean名称**默认和方法名相同**；
   - Bean指定名称：通过@Bean的`name`属性显式指定名称；
- 使用`@ImportResource`导入XML中的bean的元数据：
- 使用`@ComponetScan`导入指定包路径下的bean的元数据；

所有Bean方法在执行过程中会被CGLIB进行代理，会对方法进行override，因此`@Bean`方法的权限修饰符不能出现`private`和`final`；

<a name="VYDIF"></a>
### 基于路径扫描方式
基于路径扫描方式的思想是：<br />1. 通过`@Component`及其衍生注解对Bean进行显式标识；<br />2. 通过扫描上述Bean的包路径，让Spring IoC容器(`ApplicationContext`)可以检测并管理对应Bean(`BeanDefinition`)；
<a name="FVJEE"></a>
#### @Componet及其衍生品
Spring提供注解`@Component`直接作用于Bean类，显式表明该类为一个Bean；此外，Spring还提供基于`@Component`，根据业务诉求不同的其他注解：

- `@Repository`：作用于数据持久层；
- `@Service`：作用于业务架构中的Service类；`@Componet`同样可以作用于Service类，但是从明确业务语义的角度，应当选择`@Service`；
- `@Controller`：作用于MVC的Controller类；

除此之外，以上注解由于应用场景不同，Spring内置不同的切面进行处理；<br />@Compnent作用的Bean名称默认为所在类的驼峰命名；如果有自定义Bean名称需求，可通过`@Componet`指定默认名称，一般用于注入接口/抽象类的指定实现中：
```
@Service("ConcretService")
public class ConcretServiceImpl implement AbstractService {
    @Override
    public Boolean validate(String name) {
        // ...
    }
}
```
```
public interface AbstractService {
    Boolean validate(String name);
}
```
```
@Componet
public class AnotherService {
    @Qulifier("ConcretService")
    @Autowired
    private AbstractService ConcretService;
    // ...
}
```
