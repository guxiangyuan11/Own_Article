最近在学习的时候发现setTimeout/setInterval居然有第三个参数，以前我们在使用setTimeout/setInterval的时候我们一般都只是传一个回调函数再加上一个时间延迟。
我们先来看看MDN上是怎么解释的吧
###语法
>var timeoutID = scope.setTimeout(function[, delay, param1, param2, ...]);
var timeoutID = scope.setTimeout(function[, delay]);
var timeoutID = scope.setTimeout(code[, delay]);

参数中带 [ ] 的表示可传可不传
param1, param2, ... 附加参数，一旦定时器到期，它们会作为参数传递给 function 或 执行字符串（setTimeout参数中的code）

可以当作参数传给回调函数，上代码玩玩：
```
    function foo(x,y){
      console.log(x,y)
    }
    setTimeout(foo,2000,1,2) // 输出1  2
```
可以，不过要注意的一点就是兼容问题，MDN上面也提出了，它的描述:
>如果你需要向你的回调函数内传递一个参数, 而且还需要兼容IE9及以前的版本, 由于IE不支持传递额外的参数 (setTimeout() 或者 setInterval()都不可以) ,但你可以引入下面的兼容代码.该代码能让IE也支持符合HTML5标准的定时器函数.

*兼容旧IE代码太多，我就不在这贴了，想了解的可以去这里看看MDN[兼容旧IE代码](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout#%E5%85%BC%E5%AE%B9%E6%97%A7%E7%8E%AF%E5%A2%83%EF%BC%88polyfill%EF%BC%89)*

那么我们了解了有这个特性后，我们可以干点啥嘞？
相信很多小伙伴去面试的时候都会遇到这么一个问题:
请问这个会输出什么
```
    function foo(){
        for (var i=0; i <=5; i++) {
             setTimeout(function(){
                console.log(i)
             },0)
        };
     }
    foo();
```
当然是666666！至于为什么，这就涉及到了作用域以及setTimeou的异步性了--[解答](https://www.jb51.net/article/122489.htm),注意作者最后第四
条说因为setTimeout 不支持传带参数的函数，其实是因为他在第一个参数传了一个函数调用，而人家第一个参数
是需要一个函数，这里他的解答有误


那么解决方法我们一般这样做：
```
    function foo(){
        for (var i=0; i <=5; i++) {
            (function(x){
                setTimeout(function(){
                console.log(x)
             },0)
            }(i))
        };
     }
    foo(); // 0 1 2 3 4 5
```
利用闭包保存i值的方式传参，但是这样看起来很杂，我想接下来的解决方法，大家肯定都知道了，利用定时器的第三个参数来传参
```
    function foo(){
        for (var i=0; i <=5; i++) {
             setTimeout(function(x){
                console.log(x)
             },0,i)
        };
    }
    foo(); // 0 1 2 3 4 5
```
这样是不是就爽快多了