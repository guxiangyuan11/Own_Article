## 前端MVVM模式从理论到实战 （五）

### 模板解析 
> * 解析双括号   
> * v-on绑定事件   
> * v-text绑定事件   
> * v-class绑定事件   
> * v-html绑定事件  
> * v-model绑定事件 

#### 解析双括号 

先初始化代码

~~~
...
<div id="title">
    <span>{{title}}</span>
</div>
<script>
var mv = new MvvmVue({
    el:'#title',
    data:{
        title:'Hello'
    }
})
</script>
...
~~~

对于双括号的解析，从上面可以看出需要的是对节点的判断和节点文本值的解析，这样前面定义的el:'#title'就起到了关键作用，通过所传入的id能快速的找到的模板。

新建一个complie.js文件，创建一个模板编译类，用来专门处理模板的解析,这个构造函数需要接收模板的id，以及MvvmVue实例，后者是用来进行数据获取的，毕竟最后是要把数据渲染到HTML上

~~~
// compile.js
function Compile(el, vm) {
    var _self = this
    _self._vm = vm // 保存MvvmVue实例对象
    // 得到element，
    _self._el = _self.isElement(el) ? el:document.querySelector(el)
}
// 判断传过来的是否是node节点
Compile.prototype.isElement = function (node){
    return node.nodeType == 1 // 如果是1的话就是元素
}
// dataProxy.js
function MvvmVue(options) {
    this.$options = options // 得到传过来的配置
    var data = this._data = this.$options.data // 得到配置里面的data对象
    var _self = this // 保存this对象
    // 遍历属性对象JSON
    Object.keys(data).forEach(function (key) {
        // 实现属性代理
        _self._proxy(key)
    })
    // 编译HTML模板
    this.$compile = new Compile(_self.$options.el || document.body, _self)
}
~~~

为了提升性能，不直接去操作DOM，而是使用fragment片段，前面也有讲

~~~
// compile.js
function Compile(el, vm) {
    var _self = this
    _self._vm = vm // 保存MvvmVue实例对象
    // 得到element，
    _self._el = _self.isElement(el) ? el:document.querySelector(el)

    if(_self._el){
        // 通过得到的element去遍历所有的子元素并添加到_fragment片段
        _self._fragment =  _self.getFragment(_self._el)
        
        //最重要也是最复杂的一步
        ... // 进行_fragment里面的模板编译
        
        // 最后一步
        // 把编译好的fragment片段添加到对应的DOM节点中
        _self._el.appendChild(_self._fragment)
    }
}
// 判创建fragment片段
Compile.prototype.getFragment=function (el){
    var fragment = document.createDocumentFragment()
    var child
    // 添加子节点到fragment
    while (child=el.firstChild) {
        fragment.appendChild(child)
    }
    return fragment
}
...

~~~

接下来就是激动人心的模板解析函数，已经把开始和结束部分写完了，就差这中间的模板解析部分

~~~
// 模板编译函数
Compile.prototype.compileFrag=function (el){
    // 拿到节点中的第一层所有子节点
    var childNodes = el.childNodes
    var _self = this
}
~~~

接下来就是对所有子节点的分析，需要对节点类型进行判断，目前只考虑文本类型，由于childNodes是伪数组需要转换成数组才能用到数组的一些函数，然后使用正则表达式去匹配文本是否符合要求

~~~
...
// 进行模板编译
Compile.prototype.compileFrag=function (el){
    // 拿到节点中的第一层所有子节点
    var childNodes = el.childNodes
    var _self = this
    Array.prototype.slice.apply(childNodes).forEach(function (node) {
        // 正则找 {{}} 需要拿到里面的字符串
        var reg = /\{\{(.*)\}\}/
        // 判断是否是文本且节点的内容能否匹配上正则
        if(_self.isText(node) && reg.test(node.textContent)){ 
            // 对双括号模板进行解析和编译
            ...
        }
        // 如果子节点还有子节点
        if (node.childNodes && node.childNodes.length) {
            // 调用实现所有层次节点的编译
            _self.compileFrag(node); // 由于需要拿到所有的子节点，所以需要用到递归调用
        }
    })
}
// 判断传过来的是否是文本
Compile.prototype.isText = function (node){
    return node.nodeType == 3 // 如果是3的话就是文本
}
...
~~~
对双括号的解析另外起一个函数，专门来处理文本模板

~~~
Compile.prototype.compileFrag=function (el){
    // 拿到节点中的第一层所有子节点
    var childNodes = el.childNodes
    var _self = this
    Array.prototype.slice.apply(childNodes).forEach(function (node) {
        // 正则找 {{}} 需要拿到里面的字符串
        var reg = /\{\{(.*)\}\}/
        // 判断是否是文本且节点的内容能否匹配上正则
        if(_self.isText(node) && reg.test(node.textContent)){ 
            // 对双括号模板赋值
            _self.compileText(node, RegExp.$1)
        }
        // 如果子节点还有子节点
        if (node.childNodes && node.childNodes.length) {
            // 调用实现所有层次节点的编译
            _self.compileFrag(node); // 由于需要拿到所有的子节点，所以需要用到递归调用
        }
    })
}
// 对文本节点进行赋值
// node就是含有模板的那个节点
// regStr就是在{{}}里定义的属性名
Compile.prototype.compileText = function (node, regStr){
    // 这里需要去获取在MvvmVue中所定义的data
    // 去定义一个工具对象来获取具体的数据
    node.textContent = this.vUtil.getDataValue(this._vm, regStr)
}
//工具函数--之后对于模板的数据处理都会用到这个函数
Compile.prototype.vUtil = {
    // 得到{{}}中指定的数据，从实例中得到
    getDataValue: function (vm, regStr){
        var val = vm._data
        // 这里考虑到模板中可能出现a.b.c的获取方式，所以需要把所有的对象通过这种方式来获取到最终的值。
        // 其实在编码最开始的时候是不会考虑这么多的，为了节省篇幅我就先把这坑给填了
        regStr = regStr.split('.')
        regStr.forEach(function (k) {
            val = val[k]
        })
        return val
    },
}
~~~

这里说一下关于正则RegExp对象，在正则中有个叫子匹配的概念，即正则表达式中每个小括号内的部分表达式就是一个子表达式，RegExp.$1...$9属性用于返回正则表达式模式中某个子表达式匹配的文本。这里的RegExp是全局对象，RegExp.$1...$9是全局属性。当执行任意正则表达式匹配操作时，JavaScript会自动更新全局对象RegExp上的全局属性，用以存储此次正则表达式模式的匹配结果。当再次执行正则表达式匹配时，RegExp上的全局属性又会更新，覆盖掉之前的数据，以反映本次正则表达式模式的匹配结果。

这样大括号解析就算是解析成功了，当然目前只是能够解析成功，还没有实现数据绑定。

如下图所示

![](../img/MVVM/6.png)

#### 
