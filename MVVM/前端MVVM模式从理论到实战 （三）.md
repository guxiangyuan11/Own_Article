##前端MVVM模式从理论到实战 （三）

在进行实战之前先简述一下用到的比较重要的两个东西:
Object.defineProperties(obj, props)函数 和 DocumentFragment对象
>  ##### Object.defineProperties(obj, props)：
> 它是实现数据代理最重要的函数，所以理解它的工作原理和使用很总要

> ######参数：
>  * obj
在其上定义或修改属性的对象。
>  * props
要定义其可枚举属性或修改的属性描述符的对象。对象中存在的属性描述符主要有两种：数据描述符和访问器描述符（更多详情，请参阅Object.defineProperty()）。描述符具有以下键：
>  * configurable
true 当且仅当该属性描述符的类型可以被改变并且该属性可以从对应对象中删除。默认为 false
>  * enumerable
true 当且仅当在枚举相应对象上的属性时该属性显现。默认为 false
>  * value
与属性关联的值。可以是任何有效的JavaScript值（数字，对象，函数等）。默认为 undefined.
>  * writable
true当且仅当与该属性相关联的值可以用assignment operator（赋值运算符）改变时。默认为 false
> ###### 返回值： 就是被修改的对象obj

##### eg1:
```
var obj = {};
var obj2 = Object.defineProperties(obj, { // 使用一个空对象{}来接收修改的对象
  'property1': {
    value: true,
    writable: true // 可以被赋值操作改变
  },
  'property2': {
    value: 'Hello',
    writable: false // 不能被赋值操作改变  
  }
  // etc. etc.
});
console.log(obj2 )
obj2.property2 = 'yes~~'  //注意：可以对该属性进行赋值操作，但不会报错，诚然也不会修改成功
console.log(obj2 )
```
(复制以上代码到调试框便可以看到打印)
其他的参数大家可以自己试试，很好玩。

然后在配置项中除了这些参数外还有两个重要参数：
>  * get
作为该属性的 getter 函数，如果没有 getter 则为undefined。函数返回值将被用作属性的值。默认为 undefined
>  * set
作为属性的 setter 函数，如果没有 setter 则为undefined。函数将仅接受参数赋值给该属性的新值。默认为 undefined
```
var obj = {
    firstName: 'A',
    lastName: 'B'
  }
 Object.defineProperty(obj, 'fullName', {
    //访问描述符
    // 当读取对象此属性值时自动调用, 将函数返回的值作为属性值, this为obj
    get () {
      return this.firstName + "-" + this.lastName
    },
    // 当修改了对象的当前属性值时自动调用, 监视当前属性值的变化, 修改相关的属性, this为obj
    set (value) {
      const names = value.split('-')
      this.firstName = names[0]
      this.lastName = names[1]
    }
  })
  console.log(obj.fullName) // A-B
  obj.fullName = 'C-D'
  console.log(obj.firstName, obj.lastName) // C D
```
(复制以上代码到调试框便可以看到打印)
**再次注意：上面的this指向的是obj，也就是第一个被修改的对象**
从上面的打印可以看出，在第一次进行 fullName获取的时候打印出来是走了get函数里面的操作，所以打印出来是A-B。
下面是对fullName属性进行赋值操作，然后走了set函数，在函数内部是对firstName和lastName进行了修改，所以取值后两个属性都跟着更改了。

由上面的函数可以实现数据代理，这个在后面会提到

#####DocumentFragment
接下来是DocumentFragment: 文档碎片(高效批量更新多个节点)
首先我们来了解下document和DocumentFragment最直观的区别
>document: 对应显示的页面, 包含n个elment  一旦更新document内部的某个元素后，界面就会直接更新
  documentFragment: 内存中保存n个element的容器对象(不与界面关联), 如果更新framgmet中的某个element, 界面不会改变

对于documentFragment记住一点就行了，通过它来进行复杂的节点操作或创建时，可以避免大量的[重绘回流](https://www.css88.com/archives/4996)，由此来提高整个页面的性能。
以下是示例代码：
```
  const ul = document.getElementById('fragment_test')
  // 1. 创建fragment
  const fragment = document.createDocumentFragment()
  // 2. 取出ul中所有子节点取出保存到fragment
  let child
  while(child=ul.firstChild) { // 一个节点只能有一个父亲
    fragment.appendChild(child)  // 先将child从ul中移除, 添加为fragment子节点
  }

  // 3. 更新fragment中所有li的文本
  Array.prototype.slice.call(fragment.childNodes).forEach(node => {
    if (node.nodeType===1) { // 元素节点 <li>
      node.textContent = 'atguigu'
    }
  })

  // 4. 将fragment插入ul
  ul.appendChild(fragment)
```

> 上面fragment可以看成是<fragment></fragment> 的节点对象，但是注意这里fragment并没有实际意义对于document来说，在添加它到document中的时候只会添加它的子孙节点，而忽略其本身，它只是一个容器。

在后面的章节里面会用到此对象来进行界面实时更新，和模板解析