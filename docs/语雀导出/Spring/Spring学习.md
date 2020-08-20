# Spring学习

JavaEE体系结构包括四层，从上到下分别是应用层、Web层、业务层、持久层。Struts和SpringMVC是Web层的框架，Spring是业务层的框架，Hibernate和MyBatis是持久层的框架。
<a name="6B6dk"></a>
## 什么是MVC？
MVC 模式代表 Model-View-Controller（模型-视图-控制器） 模式。这种模式用于应用程序的分层开发。

- **Model（模型）** - 模型代表一个存取数据的对象或 JAVA POJO。它也可以带有逻辑，在数据变化时更新控制器。
- **View（视图）** - 视图代表模型包含的数据的可视化。
- **Controller（控制器）** - 控制器作用于模型和视图上。它控制数据流向模型对象，并在数据变化时更新视图。它使视图与模型分离开。

![](https://yuque.antfin.com/attachments/lark/0/2020/png/304852/1591170078074-176aa9a8-0b83-4970-adb4-e07247336fa2.png#align=left&display=inline&height=548&margin=%5Bobject%20Object%5D&originHeight=548&originWidth=1200&size=0&status=done&style=none&width=1200)
