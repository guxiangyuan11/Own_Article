### 回顾

上一节对数据进行了劫持，和了解Dep和watcher是什么，以及他们之间关系

目前已知在Dep对象中有通知功能和存储watcher功能，在Watcher对象中有存储dep和视图更新功能，接下来开始写代码

### Dep
在observer.js文件中声明Dep对象
~~~
// observer.js
// 对Dep设置id
var countId = 0
function Dep() {
    // 每次实例化Dep的时候就使id+1
    this.id = countId++
    // 用于存储watch对象的数组
    this.watchs = []
}
Dep.prototype = {
    // 添加watcher存储到dep上
    addWatchs(watch) {
        this.watchs.push(watch)
    },
    // 发布消息，通知watch更新
    notify(){
    // 通知更新
    ....
    }
}
~~~
Dep实例化的时机

之前已经写过，每一个数据对应一个Dep对象，那么Dep实例化的时机应该是在对数据进行defineProperty的时候，我们需要Dep对象，在setter的时候通知watcher进行更新

在defineReactive函数中创建属于当前数据的Dep，然后在setter的时候通过notify函数发布通知
~~~
// observer.js
...
    defineReactive(data, key,val){
        // 创建该对象的dep对象
        var dep = new Dep()
        // 间接性的递归，监测更深层次的数据
        var childObj = observe(val);
        // 给data重新定义属性(添加set/get)
        Object.defineProperty(data,key,{
            enumerable: true, // 可枚举
            configurable: false, // 不能再define
            get(){
                return val
            },
            set(newVal){
                if(newVal === val){
                    return
                }
                val = newVal
                // 如果新值是一个对象的话就再进行监测
                childObj = observe(newVal)
                // 通知更新
                dep.notify()
            }
        })
    }
...
~~~

### Watcher

上一章中有写到watcher对象的创建时机应该是在模板编译阶段

那么在编译阶段什么地方创建watcher对象呢？

由于watcher对象需要对模板更新数据，那么更新模板的函数自然由指令处理函数也就是解析模板数据的函数来解决

在compile.js内的vUtil对象中，我们对指令进行了统一的处理，并且模板更新的函数也是由他来提供的，所以watcher的更新回调写在这里是再好不过的


~~~
// compile.js
Compile.prototype.vUtil = {
...
deal: function (vm, node, value, type) {
        var _self = this;
        // 这里需要处理不同的指令
        this.dealTypeFn[type+'Updata'] && this.dealTypeFn[type+'Updata'](node, this.getDataValue(vm,value))

        // 进行数据监听
        // 接收三个值：表达式，实例对象，回调函数
        // 表达式是用来辅助获取_data中的数据
        // vm是为了方便拿取其绑定的_data中的数据
        // 回调函数使用dealTypeFn函数处理页面更新
        new Watcher(value, vm, function (val) {
            _self.dealTypeFn[type+'Updata'] && _self.dealTypeFn[type+'Updata'](node, val)
        })
}
...
}
~~~

新建一个watcher.js文件，创建watcher对象

~~~
// watcher.js
function Watcher(exp, vm, cb) {
    this.exp = exp // 记录传过来的表达式
    this.vm = vm // 实例对象
    this.cb = cb // 函数回调
    this.depIds = {} // 用来存储dep对象，key是dep的id
}
~~~

每一个dep对象中有存储当前watcher对象，在watcher对象中有存储当前dep对象，但是如何把他们串联在一起是一个问题

首先知道的是每一个数据都有getter函数，在数据被获取的时候触发这个函数，那么就从这个函数去着手解决以上问题

在dep类中创建一个target属性用来获取当前watcher对象，***注意，这里target应该是作为Dep类的一个属性，而不是实例的属性***
~~~
// observer.js
...
Dep.target = null
...
~~~


watcher类新增一个addDep函数

~~~
// watcher.js
Watcher.prototype = {
    // 添加dep，存储在Watcher对象里
    // 参数是当前数据对应的dep对象
    addDep(dep){

        // 如果存在就不用再添加
        if(!this.depIds.hasOwnProperty(dep.id)){
            // 调用dep的addWatchs函数添加当前watcher对象
            dep.addWatchs(this)
            // 存储dep对象到depIds
            this.depIds[dep.id] = dep
        }
    }
}
~~~
以上做了两件很重要的事添加dep和watcher。

由于在获取某个数据的时候会去调用该数据的getter函数，且应该在创建watcher实例的时候（因为这个时候会对模板进行编译和解析）就应该对dep和watcher对象都进行各自的存储，所以addDep函数调用时机最好是放在getter函数触发的时机


~~~
// watcher.js
function Watcher(exp, vm, cb) {
    ...
    // get函数返回当前数据值
    // this.value之后在数据更新时用来判断是否是旧值
    this.value = this.get()  
}
Watcher.prototype = {
    get(){
        // 将target指向该Watcher对象
        Dep.target = this
        // 这里在获取value值时会去调用对应数据的getter函数
        var value = this.getDataValue()
        // 置空
        Dep.target = null
        return value
    },
    // 同compile.js中的getDataValue一样
    getDataValue(){
        var vals = this.exp.split(".")
        var val = this.vm._data
        vals.forEach(function (key) {
            val = val[key]
        })
        return val
    }
}
~~~

~~~
// observer.js
    defineReactive(data, key,val){
    ...
        Object.defineProperty(data,key,{
            enumerable: true, // 可枚举
            configurable: false, // 不能再define
            get(){
                // 在get方法中添加watche对象到该dep上
                // 如果是通过watch触发的get方法就进行watch存储到该dep上
                if(Dep.target){
                    // 在watcher那边添加dep对象，以及dep存储当前watcher
                    // 将当前数据的dep对象做参数传过去
                    Dep.target.addDep(dep)
                }
                return val
            },
    ...
        })
    }
~~~

以上调用顺序应该是:

创建watcher对象 -> 获取每一个数据 -> getter函数中调用addDep函数 -> addDep函数进行各自的对象存储


### 数据更新

接下来就是数据更新了，cb回调函数还没有用，watcher类中新增update函数,然后在Dep类中的notify函数添加函数体

~~~
// watcher.js
Watcher.prototype = {
...
    update(){
        var oldValue = this.value
        // 在_data中获取修改后的值
        var newValue = this.get()
        // 判断是否和原值一样
        if(newValue === oldValue){
            return
        }
        // 调用更新视图,这里需要把this指向改为mvvm对象实例
        this.cb.call(this.vm ,newValue)
        this.value = newValue
    }
...
}

// observer.js
Dep.prototype = {
    ...
    // 发布消息，通知watch更新
    notify(){
        this.watchs.forEach(function (watch) {
            watch.update()
        })
    }
}
~~~

这样在数据发生改变时，就会调用该数据的setter函数，然后在函数内调用notify函数通知更新。

### v-model

很简单只需要在工具函数中加一个model指令解析，然后监听input事件，通过input事件来修改相应的数据，然后触发setter更新就行了

~~~
// compile.js

Compile.prototype.vUtil = {
...
    model: function (vm, node, value) {
        var _self = this,
            val = this.getDataValue(vm, value);
        if(value){
            node.value = val
        }
        // 绑定input事件
        node.addEventListener('input', function (e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }
            // 修改原值，触发setter
            _self._setDataValue(vm, value, newValue);
            val = newValue;
        });
    },
    _setDataValue:function (vm, exp, value) {
        var val = vm._data
        exp = exp.split('.')
        exp.forEach(function (k, i) {
            // 非最后一个key，更新val的值
            if (i < exp.length - 1) {
                val = val[k]
            } else {
                val[k] = value
            }
        });
    }
...
}
~~~


最后来看一下效果

~~~
// index.html
<div id="app">
    <div>{{name}}</div>
    <div>{{age}}</div>
    <div>{{eat.a}}</div>
    <div>{{eataab}}</div>
    <div data-as="asd" v-html="htmlT"></div>
    <div data-as="asd2" v-text="textT"></div>
    <div data-as="asd2" v-text="msg"></div>
    <div class="demo" v-class="classT">哈哈</div>
    <button style="width: 100px;height: 30px" @click="show">点我</button>
    <button style="width: 100px;height: 30px" @click="update">更新</button>
    <input type="text" v-model="msg">
    <div>{{msg}}</div>
</div>
<script src="js/compile.js"></script>
<script src="js/watcher.js"></script>
<script src="js/observer.js"></script>
<script src="js/dataProxy.js"></script>
<script>
    var vm = new MvvmVue({
        el:'#app',
        data: {
            name: 'asd',
            age: 12,
            eat:{
                a: 1,
                b: 2
            },
            htmlT:'<a>haha</a>',
            textT: 'demodemodemo',
            classT: 'test',
            msg: '12'
        },
        methods:{
            show: function () {
                alert(this.eat.a)
            },
            update: function () {
                this.name = "1111"
            }
        }
    })
    console.log(vm)
</script>
~~~

![11.png](https://upload-images.jianshu.io/upload_images/13892139-3eea7bb767934307.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


完成！！


> [前端MVVM理论-MVC和MVP](https://www.jianshu.com/p/e2ac3260c767)

> [前端MVVM理论-MVVM](https://www.jianshu.com/p/7088249276de)

> [前端MVVM实战-常用的几个方法和属性](https://www.jianshu.com/p/ca9404cf2f9b)

> [前端MVVM实战-数据代理](https://www.jianshu.com/p/56f859da7a7d)

> [前端MVVM实战-模板解析之双括号解析](https://www.jianshu.com/p/160c989e73c1)

> [前端MVVM实战-模板解析之事件指令和一般指令](https://www.jianshu.com/p/faff382af115)

> [前端MVVM实战-数据绑定(一)](https://www.jianshu.com/p/3bf0b4d76611)

> [前端MVVM实战-数据绑定(二)](https://www.jianshu.com/p/21592a132f67)