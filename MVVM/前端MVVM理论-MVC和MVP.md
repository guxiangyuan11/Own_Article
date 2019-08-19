## 前端MVVM理论-MVC和MVP


最近在研究mvvm开发模式，仿照着vue写了一套简单的mvvm代码，也顺便来记录一下

在说mvvm模式之前先简单解释一下mvc以及mvp开发模式以便于理解mvvm模式，当然我这里写的比较简单，如果想要详细了解这两种开发模式可以在网上找找有很多也很详细。

我把MVVM模式的理解和解释写在了另外一篇文章里面，这篇文章主要介绍了MVC和MVP两种模式。
MVVM模式的讲解在篇文章里： [MVVM模式理论](https://www.jianshu.com/p/7088249276de)

## MVC --- Model View Controller

> model：应用程序中处理数据逻辑的一部分，通常用来模型对象对数据库的存存取等操作
view：视图部分，通常指html等用来对用户展示的一部分
controller：控制层通常用来处理业务逻辑，负责从试图读取数据，并向模型发送数据
##### 流程
MVC的一般流程是这样的：View（界面）触发事件--》Controller（业务）处理了业务，然后触发了数据更新--》不知道谁更新了Model的数据--》Model（带着数据）回到了View--》View更新数据
流程图的话大概是这样：
![流程图](https://upload-images.jianshu.io/upload_images/13892139-1dcef6ffe4c1598e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 优点
1.各施其职，互不干涉，在MVC模式中，三个层各施其职，所以如果一旦哪一层的需求发生了变化，就只需要更改相应的层中的代码而不会影响到其它层中的代码。
2.有利于开发中的分工，在MVC模式中，由于按层把系统分开，那么就能更好的实现开发中的分工。网页设计人员可以进行开发视图层中的JSP，对业务熟悉的开发人员可开发业务层，而其它开发人员可开发控制层。
3.有利于组件的重用，分层后更有利于组件的重用。如控制层可独立成一个能用的组件，视图层也可做成通用的操作界面。
##### 缺点
1. 开发者在代码中大量调用相同的 DOM API, 处理繁琐 ，操作冗余，使得代码难以维护。 
2. 大量的DOM 操作使页面渲染性能降低，加载速度变慢，影响用户体验。
3. 当 Model 频繁发生变化，开发者需要主动更新到View ；当用户的操作导致 Model 发生变化，开发者同样需要将变化的数据同步到Model 中，这样的工作不仅繁琐，而且很难维护复杂多变的数据状态。


## MVP --- Model View Presenter
其实就是Controller 改了个名称，叫做Presenter，把整体命名为MVP
> 切断View和Model的联系，让View只和Presenter（原Controller）交互，减少在需求变化中需要维护的对象的数量
View只知道Presenter, 不知道Model
View去调用Presenter， Presenter操作Model , Model 中进行业务计算。 关键点是，Presenter去更新View。
##### 流程图
![mvp流程图](https://upload-images.jianshu.io/upload_images/13892139-f84d737936b79d33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##### 优点
1.降低耦合度
2.Modle模块职责划分明显，层次清晰
3.隐藏数据
4.Presenter可以复用，一个Presenter可以用于多个View，而不需要更改Presenter的逻辑（当然是在View的改动不影响业务逻辑的前提下）
5.利于测试驱动开发。
6.View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。
7.代码灵活性

##### 缺点
1.由于对视图的渲染放在了Presenter中，所以视图和Presenter的交互会过于频繁。如果Presenter过多地渲染了视图，往往会使得它与特定的视图的联系过于紧密。一旦视图需要变更，那么Presenter也需要变更了。

其他文章导航：

> [前端MVVM理论-MVC和MVP](https://www.jianshu.com/p/e2ac3260c767)

> [前端MVVM理论-MVVM](https://www.jianshu.com/p/7088249276de)

> [前端MVVM实战-常用的几个方法和属性](https://www.jianshu.com/p/ca9404cf2f9b)

> [前端MVVM实战-数据代理](https://www.jianshu.com/p/56f859da7a7d)

> [前端MVVM实战-模板解析之双括号解析](https://www.jianshu.com/p/160c989e73c1)

> [前端MVVM实战-模板解析之事件指令和一般指令](https://www.jianshu.com/p/faff382af115)

> [前端MVVM实战-数据绑定(一)](https://www.jianshu.com/p/3bf0b4d76611)

> [前端MVVM实战-数据绑定(二)](https://www.jianshu.com/p/21592a132f67)