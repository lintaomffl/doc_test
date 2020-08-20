# Lambda表达式

参考文章链接:<br />深入探讨lambda表达式(上)[https://open.atatech.org/articles/159525](https://open.atatech.org/articles/159525)<br />深入探讨lambda表达式(下)[https://open.atatech.org/articles/163518](https://open.atatech.org/articles/163518)<br />函数式编程入门：[https://open.atatech.org/articles/149788](https://open.atatech.org/articles/149788)<br />

<a name="wdXSq"></a>
## 0.准备知识：
<a name="OAQh4"></a>
### 匿名内部类：
```
new 父类构造器（参数列表）|实现接口（）{
  
         //匿名内部类的类体部分 
 
    }
```
匿名内部类是对于一个抽象类或者接口进行实例化是用到的一个类，能够对于抽象方法或接口在类的内部对其进行实现。<br />明确的例子：
```
public abstract class Bird {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    
    public abstract int fly();
}

public class Test {
    
    public void test(Bird bird){
        System.out.println(bird.getName() + "能够飞 " + bird.fly() + "米");
    }
    
    public static void main(String[] args) {
        Test test = new Test();
        test.test(new Bird() {
            
            public int fly() {
                return 10000;
            }
            
            public String getName() {
                return "大雁";
            }
        });
    }
}
------------------
Output：
大雁能够飞 10000米
```
<a name="Cu2pN"></a>
### 函数式接口的定义：
函数式接口，一种仅有一个抽象方法的接口，可以添加@FunctionalInterface注解。<br />函数式接口虽然只能有一个方法，但是可以利用default对于方法设置一个默认的实现方式。<br />![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2020/png/304852/1593312212870-b27bed7e-e912-4e21-840a-f35a40825eb2.png#align=left&display=inline&height=486&margin=%5Bobject%20Object%5D&name=image.png&originHeight=972&originWidth=1598&size=1382014&status=done&style=none&width=799)上面截图中的信息量较大，分为两块内容。<br />第一块内容是使用 `@FunctionalInterface` 注解需满足的 2 个条件：

- 必须是接口，不能是注解、枚举或类，限定了使用的类型范围
- 被注解的接口，必须满足函数式接口的定义，即只能有一个抽象函数

第二块内容是 `@FunctionalInterface` 注解的功能已内置于编译器的处理逻辑中：**不管一个接口是否添加了 `@FunctionalInterface` 注解，只要该接口满足函数式接口的定义，编译器都会把它当做函数式接口**。<br />

<a name="3X5t1"></a>
## 1.Lambda表达式
<a name="OlR5i"></a>
### Lambda表达式是什么:
**若一个方法的形参是一个接口类型，且该接口是一个函数式接口（即只有一个抽象方法），那么就可以使用 Lambda 表达式来替代其对应的匿名类，达到易读、简化的目的。**<br />通常，Lambda 表达式的格式如下：
```
() -> {...}
或
(xxx) -> {...}
```
 <br />从前面的示例也可以看到，Lambda 表达式其实就代表了一个接口的实例对象，并且这个接口还得是一个函数式接口，即只能有一个抽象方法，这个抽象方法的具体实现，就是 Lambda 表达式中箭头的右侧 body 部分。<br />

<a name="sdMMB"></a>
### Lambda表达式特征：

- **特性 1：由箭头将表达式分为左、右两个部分**

必须是形如 `() -> {...}` 的形式。

- **特性 2：入参可为零个、一个、多个**

当为零个时，箭头左侧的括号不可省略：
```
() -> {System.out.println("test expression!");};
() -> 123;
```
 <br />当入参为 1 个时，箭头左侧的圆括号可省略：
```
(x) -> {System.out.println(x);};
x => x + 2;
```
 当入参为多个时，左侧括号不能省略：
```
(x, y, z) -> {
    System.out.println(x);
    System.out.println(y);
    System.out.println(z);
};
```
 以上都是合法表达式。但是，这并不意味着他们可以独立存在。若不给这些表达式赋左值，则编译器会报错：`Not s statement`。<br />前面我们也有提到，Lambda 表达式其实是一个实例对象，因此，赋左值，自然是赋值给某个特定类型的实例。它是如何赋值的呢？可手动指定，也可根据 IDE 自动生成（此时编译器会自动推断左值类型）。在正常使用过程中，我们往往都会有目的的手动赋左值。

- **特性 3：入参类型声明可省略，编译器会做自动类型推断**
```
List<String> strList = Arrays.asList("a", "b", "c", "d");
strList.forEach(str -> {
	System.out.println(str);
});
```
上方代码中，Lambda 表达式中的 `str`  局部变量，不需要再次声明类型，因为编译器会从 `strList` 变量中推断出 `str`  变量的类型为 `String`。

- **特性 4：表达式右侧的 body 中，只有一条语句，则可省略大括号，否则不可省略**

上面的 `strList` 变量的 `forEach()`  方式的遍历，可简化为如下形式：
```
strList.forEach(str -> System.out.println(str));
```

- **特性 5：表达式的返回值是可选的**

上面的 `forEach()` 方式，就是没有返回值的，也可认为是 void。<br />

<a name="fgzGd"></a>
### Lambda表达式的优势
<a name="eBwUa"></a>
#### 1.Lambda表达式是一种函数式编程，而函数式编程核心思想：通过函数来操作数据，复杂逻辑的实现是通过多个函数的组合来实现的。
相比声明式编程和命令式编程，它是一种更高级别的抽象：汇编语言要求我们如何用机器能理解的语言来写代码（指令）；高级语言如 Java、C++ 则是使用易于人理解的方式，但如何做，还需要我们来一步步设定，仍未逃脱指令式的思维模式；函数式编程，通过函数来操作数据，至于函数内部做了什么，交给其他函数来组合实现。
<a name="10"></a>
#### 2.使得代码更简洁，可读性强
<a name="11"></a>
#### 3.传递行为，而不止是传递值，更便于功能复用
[https://open.atatech.org/articles/159525#11](https://open.atatech.org/articles/159525#11) 这里有个直观说明利于功能复用的例子。
<a name="12"></a>
#### 4.流的并行化操作


<a name="oxSi0"></a>
## 2.Lambda表达式与匿名类的区别
Lambda表达式一定程度上可以代替匿名类，但是对于有多于一个抽象方法的接口，显然无法使用Lambda表达式完成，所以还是得用匿名类来实现。<br />有个例子：[https://open.atatech.org/articles/163518#0](https://open.atatech.org/articles/163518#0)<br />该例子说明，在面对非函数式接口时，Lambda表达式无法奏效，还是得使用匿名类。<br />

<a name="8F6p6"></a>
## 3.Lambda表达式的变量作用域
变量作用域规则：

- **规则 1：局部变量不可变，域变量或静态变量是可变的**
- **规则 2：表达式内的变量名不能与局部变量重名，域变量和静态变量不受限制**
- **规则 3：可使用 `this`、`super` 关键字，等同于在普通方法中使用**
- **规则 4：不能使用接口中的默认方法（default 方法）**

为何要使用final：

- **原因 1：引入的局部变量是副本，改变不了原本的值**
- **原因 2：局部变量存于栈中，多线程中使用有问题**
- **原因 3：线程安全问题**



<a name="7iY0r"></a>
## 4.Lambda表达式的函数式接口
Lambda主要的函数式接口在java.util.function包之下，其中包括以下四大核心接口

| 函数式接口 | 参数类型 | 返回类型 | 用途 |
| :--- | :--- | :--- | :--- |
| Consumer(消费型接口) | T | void | 对类型为T的对象应用操作。void accept(T t) |
| Supplier(供给型接口) | 无 | T | 返回类型为T的对象。 T get(); |
| Function(函数型接口) | T | R | 对类型为T的对象应用操作并返回R类型的对象。R apply(T t); |
| Predicate(断言型接口) | T | boolean | 确定类型为T的对象是否满足约束。boolean test(T t); |

其使用范例如下：[https://open.atatech.org/articles/149788#3](https://open.atatech.org/articles/149788#3) <br />

<a name="n8jit"></a>
## 5.函数式接口的使用
<a name="CmKx7"></a>
### 1.Stream类
Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序，当我们使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）→ 数据转换 → 执行操作获取想要的结果。每次转换原有Stream对象不改变，返回一个新的Stream对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道。<br />stream的创建与使用:[https://open.atatech.org/articles/149788#9](https://open.atatech.org/articles/149788#9)<br />

<a name="bDmWQ"></a>
### 2.Optional类
Optional类是Java8为了解决null值判断问题，借鉴google guava类库的Optional类而引入的一个同名Optional类，使用Optional类可以避免显式的null值判断（null的防御性检查），避免null导致的NPE<br />optional类的创建与使用：[https://open.atatech.org/articles/149788#23](https://open.atatech.org/articles/149788#23)<br />
<br />一个lambda综合利用Stream类的实例：[https://open.atatech.org/articles/149788#32](https://open.atatech.org/articles/149788#32)<br />

