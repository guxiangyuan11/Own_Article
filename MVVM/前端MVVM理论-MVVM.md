## 前端MVVM理论-MVVM

OK来介绍一下现在很火的MVVM模式，其实这个并不是新的开发模式，最主要还是在MVC模式上做了改进，早在WPF上就已经实现了这种模式。

MVVM最早由微软提出来，它借鉴了桌面应用程序的MVC思想，在前端页面中，把Model用纯JavaScript对象表示，View负责显示，两者做到了最大限度的分离。
MVC和MVP设计模式的介绍：  [MVC和MVP](https://www.jianshu.com/p/e2ac3260c767)

## MVVM --- Model View Controller
![MVVM设计模式](https://upload-images.jianshu.io/upload_images/13892139-52e91e6603193a6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照以上流程图来解释每个层代表的意思
* view: 应于Activity和xml，负责View的绘制以及与用户交互,例如当前我们所看到的网页这个界面视图。
* viewmodel：它是View的抽象，将view和model联系在一起，起到桥梁的作用，负责完成View于Model间的交互,负责业务逻辑。
* model: 就是数据访问层，Model用于封装与应用程序的业务逻辑相关的数据以及对数据的处理方法。它具有对数据直接访问的权利，例如对数据库的访问，Model不依赖于View和ViewModel，也就是说，模型不关心会被如何显示或是如何被操作，模型也不能包含任何用户使用的与界面相关的逻辑。Model在实际开发中根据实际情况可以进行细分。

*****
简单的来说View没有大量代码逻辑，也不用频繁的去操作DOM，而将大量代码逻辑、状态转到ViewModel，它的思想和MVP模式有些类似，ViewModel大致上就是MVP的Presenter和MVC的Controller了，而View和ViewModel间没有了MVP的界面接口，而是直接交互，比起MVP，MVVM不仅简化了业务与界面的依赖关系，还优化了数据频繁更新的解决方案，甚至可以说提供了一种有效的解决模式。
*****
* 数据驱动： 在以前的模式中，总是先处理业务数据，然后根据的数据变化，去获取DOM的引用然后更新DOM，也是通过DOM来获取用户输入，最后再进行数据的更新。而在MVVM中数据和业务逻辑处于一个独立且抽象的View Model中，ViewModel只要关注数据和业务逻辑，不需要和DOM之间打交道。由数据自动去驱动DOM去自动更新DOM，DOM的改变又同时自动反馈到数据，数据成为主导因素，这样使得在业务逻辑处理只要关心数据，方便且简单。
* 低耦合： 三个层都是独立，ViewModel只负责处理和提供数据，DOM想怎么处理数据都由DOM自己决定，也就是可以完全分离。如果是团队协作完全可以分工来做，一个做view，一个写ViewModel，效率更高。
* 复用性： 一个View Model复用到多个View中，同样的一份数据，用不同的UI去做展示，对于版本迭代频繁的view改动，只要更换View层就行，对于如果想在view上的做测试更是方便的多。
* 单元测试：  View Model里面是数据和业务逻辑，View中关注的是UI，这样的做测试是很方便的，完全没有彼此的依赖，不管是UI的单元测试还是业务逻辑的单元测试，都是低耦合的。
***
写了这么多的理论还不如来一顿实操，目前前端主流的MVVM框架 angular reactjs vuejs这三大框架，其中vue是比较容易上手的，所以来模仿vue来实现一套数据双向绑定的简单MVVM设计模式，更容易理解MVVM。
首先看看实现的流程图
![原理图](https://upload-images.jianshu.io/upload_images/13892139-a32b95ccb79526d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

模仿以上流程图来进行一个代码编写，主要去实现数据代理，模板解析，数据绑定，其中比较难的地方在发布订阅那块。由于模块比较多所以会分为几个章节来写。

> [前端MVVM理论-MVC和MVP](https://www.jianshu.com/p/e2ac3260c767)

> [前端MVVM理论-MVVM](https://www.jianshu.com/p/7088249276de)

> [前端MVVM实战-常用的几个方法和属性](https://www.jianshu.com/p/ca9404cf2f9b)

> [前端MVVM实战-数据代理](https://www.jianshu.com/p/56f859da7a7d)

> [前端MVVM实战-模板解析之双括号解析](https://www.jianshu.com/p/160c989e73c1)

> [前端MVVM实战-模板解析之事件指令和一般指令](https://www.jianshu.com/p/faff382af115)

> [前端MVVM实战-数据绑定(一)](https://www.jianshu.com/p/3bf0b4d76611)

> [前端MVVM实战-数据绑定(二)](https://www.jianshu.com/p/21592a132f67)