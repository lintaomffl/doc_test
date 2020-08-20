# SpringBoot之Bean

<a name="bWDbg"></a>
## Bean的定义：
被称作 bean 的对象是构成应用程序的支柱也是由 Spring IoC 容器管理的。bean 是一个被实例化，组装，并通过 Spring IoC 容器所管理的对象。<br />bean 定义包含称为配置元数据的信息，下述容器也需要知道配置元数据：<br />• 如何创建一个 bean<br />• bean 的生命周期的详细信息<br />• bean 的依赖关系<br />![bean元数据.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593333302250-c742e958-fed4-40c5-8be7-e7bc3efdaca0.png#align=left&display=inline&height=890&margin=%5Bobject%20Object%5D&name=bean%E5%85%83%E6%95%B0%E6%8D%AE.png&originHeight=890&originWidth=1760&size=1197673&status=done&style=none&width=1760)
<a name="L8R6C"></a>
### Spring 配置元数据
Spring IoC 容器完全由实际编写的配置元数据的格式解耦。有下面三个重要的方法把配置元数据提供给 Spring容器：<br />• 基于 XML 的配置文件。<br />• 基于注解的配置<br />• 基于 Java 的配置<br />Bean创建与加载：
<a name="fR9fQ"></a>
## Bean作用域：
<a name="fh0q8"></a>
### 五种作用域定义：
<a name="LUf7G"></a>
#### 1. singleton：
<bean id="userInfo" class="cn.lovepi.UserInfo" scope="singleton"></bean><br />当Bean的作用域为singleton的时候,Spring容器中只会存在一个共享的Bean实例，所有对Bean的请求只要id与bean的定义相匹配，则只会返回bean的同一实例。单一实例会被存储在单例缓存中，为Spring的缺省作用域。
<a name="RE5Bb"></a>
#### 2. prototype：
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" prototype "></bean><br />每次对该Bean请求的时候，Spring IoC都会创建一个新的作用域。<br />**对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton**
<a name="Eyggp"></a>
#### 3.request：
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" request "></bean><br />Request作用域针对的是每次的Http请求，Spring容器会根据相关的Bean的<br />定义来创建一个全新的Bean实例。而且该Bean只在当前request内是有效的。
<a name="zYgU9"></a>
#### 4.session：
<bean id="userInfo" class="cn.lovepi.UserInfo" scope=" session "></bean><br />针对http session起作用，Spring容器会根据该Bean的定义来创建一个全新的Bean的实例。而且该Bean只在当前http session内是有效的。
<a name="E0axU"></a>
#### 5.global session：
<bean id="userInfo" class="cn.lovepi.UserInfo"scope=“globalSession"></bean><br />类似标准的http session作用域，不过仅仅在基于portlet的web应用当中才有意义。Portlet规范定义了全局的Session的概念。他被所有构成某个portlet外部应用中的各种不同的portlet所共享。在global session作用域中所定义的bean被限定于全局的portlet session的生命周期范围之内。<br />

<a name="YRIsP"></a>
### 五种作用域细解：
<a name="H10lI"></a>
#### 1）singleton作用域
是指在Spring IoC容器中仅存在一个Bean的示例，Bean以单实例的方式存在，单实例模式是重要的设计模式之一，在Spring中对此实现了超越，可以对那些非线程安全的对象采用单实例模式。<br />接下来看一个示例：<br />1. <bean id="car" class="cn.lovepi.Car" scope="singleton"></bean><br />2. <bean id="boss1" class="cn.lovepi .Boss” p:car-ref=“car"></bean><br />3. <bean id="boss2" class="cn.lovepi .Boss” p:car-ref=“car"></bean><br />4. <bean id="boss3" class="cn.lovepi .Boss” p:car-ref=“car"></bean><br />在1中car这个Bean生命周期声明为了singleton模式，其他的bean如2，3，4引用了这个Bean。在容器中boss1、boss2、boss3的属性都指向同一个bean car。如下图所示：<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593333772300-1ef1ac2b-da45-4ecb-8070-09134fcdc07c.png#align=left&display=inline&height=176&margin=%5Bobject%20Object%5D&name=image.png&originHeight=262&originWidth=389&size=32785&status=done&style=none&width=261)<br />不仅在容器中通过对象引入配置注入引用相同的car bean，通过容器的getBean()方法返回的实例也指向同一个bean。<br />**在默认的情况下Spring的ApplicationContext容器在启动的时候，自动实例化所有的singleton的bean并缓存于容器当中。**<br />
<br />虽然启动时会花费一定的时间，但是他却带来了两个好处:<br />a.对bean提前的实例化操作，会及早发现一些潜在的配置的问题。<br />b.Bean以缓存的方式运行，当运行到需要使用该bean的时候，就不需要再去实例化了。加快了运行效率。<br />

<a name="pVHpG"></a>
#### 2）prototype作用域
是指每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行new Bean()的操作。在默认情况下，Spring容器在启动时不实例化prototype的Bean。<br />接下来看一个示例：<br />1. <bean id=“car" class="cn.lovepi.Car" scope=“prototype"></bean><br />2. <bean id=“boss1" class="cn.lovepi.Boss” p:car-ref=“car"></bean><br />3. <bean id=“boss2" class="cn.lovepi.Boss” p:car-ref=“car"></bean><br />4. <bean id=“boss3" class="cn.lovepi.Boss” p:car-ref=“car"></bean><br />通过以上的配置，Boss1、boss2、boss3所引用的都是一个新的car的实例，每次通过容器getBean()方法返回的也是一个新的car实例。如下图所示：<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593333829513-a4905071-06c1-4cec-b9c9-4665301813c4.png#align=left&display=inline&height=199&margin=%5Bobject%20Object%5D&name=image.png&originHeight=339&originWidth=466&size=47443&status=done&style=none&width=273)<br />**在默认情况下，Spring容器在启动时不实例化prototype这种bean。此外，Spring容器将prototype的实例交给调用者之后便不在管理他的生命周期了**。<br />3）当用户使用Spring的WebApplicationContext时，还可以使用另外3种Bean的作用域，即**request,session和globalSession**。<br />
<br />在使用Web应用环境相关的Bean作用域时，必须在Web容器中进行一些额外的配置：<br />
<br />对于低版本Web容器配置：<br />用户可以使用http过滤器来进行配置并在url-pattern中对所有的页面进行过滤。<br /><filter><br />   <filter-name>requestContextFilter</filter-name><br />   <filter-class>org.springframework.web.filter.RequestContextFilter</filterclass><br /></filter><br /><filter-mapping><br />   <filter-name>requestContextFilter</filter-name><br />   <url-pattern>/*</url-pattern><br /></filter-mapping><br />对于高版本Web容器配置：<br />使用http请求监听器来进行配置。<br /><listener> <listener-class><br />   org.springframework.web.context.request.RequestContextListener<br /></listener-class></listener><br />ServletContextListener只负责监听web容器启动和关闭的事件，<br />而RequestContextListener实现了ServletRequestListener监听器接口，该监听器监听http请求事件。<br />Web服务器接收到的每一次请求都会通知该监听器。<br />
<br />Spring容器的启动和关闭事件由web容器的启动和关闭事件来触发。<br />但如果Spring容器中的bean需要request、session和globalSession作用域的支持，Spring容器本身就必须获得web容器的http请求事件。以http请求事件来驱动bean的作用域的控制逻辑，也就是说通过配置RequestContextListener接口，Spring和web容器的结合就更紧密了。<br />
<br />那么接下来简单的介绍下这三种作用域。<br />**1.request作用域**<br />      对应一个http请求和生命周期，当http请求调用作用域为request的bean的时候,Spring便会创建一个新的bean，在请求处理完成之后便及时销毁这个bean。<br />**2.session作用域**<br />      Session中所有http请求共享同一个请求的bean实例。Session结束后就销毁bean。<br />**3.globalSession作用域**<br />      与session大体相同，但仅在portlet应用中使用。<br />Portlet规范定义了全局session的概念。请求的bean被组成所有portlet的自portlet所共享。<br />如果不是在portlet这种应用下，globalSession则等价于session作用域。<br />

