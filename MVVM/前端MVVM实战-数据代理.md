
### 前言
终于能抽点时间出来继续写这个系列的文章了（其实是因为懒╭(╯^╰)╮），接下来几节会开始进行实战（随性的更新），还有请看完我前两节再来看实战，不然有些代码会看的很蒙。

我会进行简单的包括数据代理绑定以及模板解析功能的实现，当然和VUE的源码还是有很大的区别的，只是去试着实现一下原理，虽然简单但是会让你体会到写框架你上你也行的感觉┗( ▔, ▔ )┛。


目标：
* 数据代理    
* 模板解析 
> * 解析双括号   
> * v-on绑定事件   
> * v-text绑定事件   
> * v-class绑定事件   
> * v-html绑定事件  
> * v-model绑定事件   
* 数据绑定 
> * 数据拦截  
> * 订阅发布  

大部分代码都是ES5，并且尽量每一行代码都写上注释，方便大家都能看懂

### 数据代理

先来看看vue是怎么处理的
~~~
var vm = new Vue({
    el:'#title',
    data:{
        a:'1'
    }
})
~~~
vue是遵守MVVM模式的，当改变a的数据时，view上的a数据也会跟着改变，再来看看vue的实例,在最外层的对象和_data对象上都绑定的有一个a数据，把鼠标放在（...）上时，会显示一个invoke property getter的提示，意思是这个数值会通过getter来进行获取，在实例初始化时data里面定义的数据被存储到了_data对象里面，至于为什么，后面会讲到

![](https://upload-images.jianshu.io/upload_images/13892139-773c86140a52f466.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


往下看在这里有两个函数分别对应a数据的代理set和代理get函数

![](https://upload-images.jianshu.io/upload_images/13892139-6c33021c54303300.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


~~~
console.log(vm.a) //其实是在调用a的代理getter函数
vm.a=2 // 其实在调用a的代理setter函数
~~~
在这里有一个疑问，明明把a数据定义在data对象里面的，但为什么我们可以通过vm直接对a进行读写操作呢？

这就是所谓的数据代理，通过vue对象对data里面的数据进行代理操作，所有的数据获取和设置都由vue的实例去操作。

接下来开始模仿实现数据代理
~~~
var mv = new MvvmVue({
    el:'#title',
    data:{
        a:'1'
    }
})
~~~
然后新建一个dataProxy.js文件，目录如下

![](https://upload-images.jianshu.io/upload_images/13892139-45dae47ef112d69a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


新建一个MvvmVue对象，并模仿vue存放数据
~~~
// dataProxy.js
function MvvmVue(options) {
    this.$options = options // 得到传过来的配置,并存到$options对象
    var data = this._data = this.$options.data // 得到配置里面的data对象，并存到_data
    var _self = this // 保存this对象
    // 遍历属性对象JSON
    Object.keys(data).forEach(function (key) {
        // 实现属性代理
        ...
    })
}
~~~
上面的代码都很好理解，比较重要的一点就是把初始化的data存储到了_data里面。
数据都存放完毕，将需要代理的数据，也就是传入到MvvmVue对象里的数据，使用Object.keys的方式循环调用代理函数来进行数据代理
~~~
// dataProxy.js
function MvvmVue(options) {
    this.$options = options // 得到传过来的配置,并存到$options对象
    var data = this._data = this.$options.data // 得到配置里面的data对象，并存到_data
    var _self = this // 保存this对象
    // 遍历属性对象JSON
    Object.keys(data).forEach(function (key) {
        // 实现属性代理
        _self._proxy(key) // 调用_proxy函数，进行数据代理
    })
}
~~~
_proxy函数的实现，这里说一下，在ES6/7之前JS没有私有属性和函数的说法，所以一般默认名字前加 _ 线的为私有属性或函数。

~~~
// dataProxy.js
//这里使用原型链添加函数
MvvmVue.prototype._proxy = function (key) {
    var _self = this // 保存this对象，即MvvmVue对象
    // 这里用到了之前我重点说的defineProperty函数，数据代理就全靠它来实现
    // 将需要代理的对象名添加到对象实例上，并定义得到方式
    Object.defineProperty(_self, key, {
        configurable: false, // 不能再重新定义
        enumerable: true, // 可以枚举
        // 当读取对象此属性值时自动调用, 将函数返回的值作为属性值, this为实例对象
        // 如实例去获取data中的某属性时
        get() { //getter
            // 返回的是实例内部_data中的对象值，拿取得是_data中的数据
            return _self._data[key]
        },
        // 当修改了对象的当前属性值时自动调用, 监视当前属性值的变化, 修改相关的属性, this为实例对象
        // 如实例去设置data中的某属性值时
        set(newValue) { //setter
            // 设置的是实例内部_data中的对象值
            _self._data[key] = newValue
        }
    })
}
~~~

这个时候就已经有了数据代理，当我们对数据进行取值或设置值得时候都会走getter和setter函数，如下
~~~
console.log(mv.a) //其实是在调用a的代理getter函数
mv.a=2 // 其实在调用a的代理setter函数
~~~

如果还不够直观的话，最佳观察方式是打开F12，开发者模式下在代码上打断点，然后跟着一步一步走，你就会很清楚代码的运行顺序了

![](https://upload-images.jianshu.io/upload_images/13892139-72be5ccb88552577.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/13892139-71c6ac769217dcc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> [前端MVVM理论-MVC和MVP](https://www.jianshu.com/p/e2ac3260c767)

> [前端MVVM理论-MVVM](https://www.jianshu.com/p/7088249276de)

> [前端MVVM实战-常用的几个方法和属性](https://www.jianshu.com/p/ca9404cf2f9b)

> [前端MVVM实战-数据代理](https://www.jianshu.com/p/56f859da7a7d)

> [前端MVVM实战-模板解析之双括号解析](https://www.jianshu.com/p/160c989e73c1)

> [前端MVVM实战-模板解析之事件指令和一般指令](https://www.jianshu.com/p/faff382af115)

> [前端MVVM实战-数据绑定(一)](https://www.jianshu.com/p/3bf0b4d76611)

> [前端MVVM实战-数据绑定(二)](https://www.jianshu.com/p/21592a132f67)

