---
title: js高级技巧
date: 2016-06-12 23:13:50
tags:
	- js高级函数
	- 节流函数
	- 函数柯里化
	- 数组分块

categories: javascript高级程序设计
	
---
### 1 . 高级函数
#### 安全的类型检测
- js内置的类型检测并非完全可靠，typeof操作符难以判断某个值是否为函数
- instanceof在多个frame的情况下，会出现问题。
例如：`var isArray = value instance of Array ; ` 
会由于存在多个window，而value与Array不属于同个window的情况而导致出错

对于这样的问题，最好的解决方法是通过调用Object的toString方法，例如：
```javascript
function isArray(){
    return Object.prototype.toString.call(value) == "[object Array]";
}
```
> 注意，这个技巧前提是Object.prototype.toString方法未被修改过

<!--more-->
#### 作用域安全的构造函数
调用构造函数的时候，如果忘记写new的话，构造函数中对this的赋值，则可能会赋值到window上成为全局变量，导致其他错误。
可以在构造函数增加判断来避免这种错误,如下：
```
function Person(name){
    if(this instanceof Person){
        this.name = name;
    }else{
        return new Person(name)
    }
}
```
#### 惰性载入函数
在程序中 ，类似对浏览器的检测等，进行一次检测过后，只要在当前环境下，其检测结果都不会改变。所以，我们的函数中 if 语句只需要判断一次就可以了，而不需要每次都执行。
对这种情况的解决方案便称为惰性载入。
惰性载入表示函数执行的分支仅会发生一次。
###### 惰性载入两种实现方法：
- 替换真正执行的方法，伪代码如下：
```
function foo(){
   if(someCheck){
      foo = function(){doSomeThing()};
   }else{
      foo = function(){doAnotherSomeThing()};
   }
}
```
- 声明函数时就指定适当的函数。通过匿名自执行函数
```
var foo = function(){
   if(someCheck){
      return function(){doSomeThing()};
   }else{
      return function(){doAnotherSomeThing()};
   }
}();
```
这样，在代码首次载入的时候，就已经得到对应的值了

#### 函数绑定
先看一个例子：
```
var handler = {
   message:"handler message",
   handlerClick:function(event){
      console.log(this.message);
   }
}
//用于将某个函数绑定到指定环境
var bind = function (fn, context) {
    return function () {
        fn.apply(context, arguments);
    }
};

var nomalBtn = document.getElementById("nomalBtn");
var bindBtn = document.getElementById("bindBtn");

nomalBtn.addEventListener("click", handler.handlerClick, false);//点击时候会打印 undefined
bindBtn.addEventListener("click", bind(handler.handlerClick,handler), false);//点击按钮会打印 handler.message的值

```
在上面的例子中，我们需要对象中的方法作为事件处理程序。但是，当事件触发时，this的指向却不是handler而是按钮本身。
解决方法可以使用匿名函数，但是，过多的匿名函数会令代码变的难于理解与调试，因此，推荐使用bind方法。
>ES5 为函数定义了一个原生的bind方法，也就是说，你不必自己实现bind方法，只需要直接在函数上调用即可`handler.handlerClick.bind(handler) ;`

#### 函数科里化
函数科里化是用于创建已经设置好一个或多个参数的函数。缩小了函数的适用范围，但提高函数的适性。
例如：
```
//普通的add版本
function add(num1, num2) {
    return num1 + num2;
}

//第一个参数为5的add版本
function curriedAdd5(num2) {
    return add(5, num2)
}
```
上面只是一个展示柯里化概念的例子。创建柯里化函数有一个通用方式：
```
function curry(fn, context) {
    //截取调用curry时候，除了fn,context,之后的所有参数
    var args = [].slice.call(arguments, 2);
    return function () {
        //获取调用fn的所有参数
        var totalArgs = args.concat([].slice.call(arguments));
        return fn.apply(context, totalArgs);
    }
}
```
这样，上面的例子中curriedAdd5可以用另一个方法来创建 `var curriedAdd5 = curry(add, null, 5) `

javascript中的柯里化函数和bind函数提供了强大的动态函数创建功能，但是两者都不应该滥用，因为每个函数都带来额外的开销

### 2. 防篡改对象
#### 不可扩展对象
不可扩展表示不能给该对象添加新属性和新方法，非严格情况下会静默失败，严格模式下会报错
主要有两个方法：
```
Object.preventExtensions(obj) //将对象定义为不可扩展的
Object.isExtensible(obj) // 判读对象是否可以扩展
```
#### 密封的对象
密封的对象表示不可扩展，且成员Configurable特性为false，即不能删除属性和方法，也不能修改属性的特性
主要有两个方法：
```
Object.seal(obj) //将对象定义为密封的
Object.isSealed(obj) // 判读对象是否密封
```
#### 冻结的对象
冻结的对象是最严格的反篡改级别。即不可扩展，又是密封的，且数据属性的writable特性为false。如果定义了[[set]]函数,访问器属性还是可写的
主要有两个方法：
```
Object.freeze(obj) //将对象定义冻结对象
Object.isFrozen(obj) // 判读对象被冻结
```
对于js库的作者而言，冻结对象很有用，因为js库最怕有人意外修改了库中的核心对象。

### 3.高级定时器
#### 重复的定时器
setInterval ： 创建的定时器确保了定时器代码规则的插入队列中。
但是会有两个问题：
1. 某些间隔会被跳过
2. 多个定时器的代码执行之间的间隔可能比预期的小

为了避免这两个问题，可以用如下模式使用链式setTimeout。
```
setTimeout(function(){
   setTimeout(arguments.callee,intervalTime)
},intervalTime)
```
这样可以确保不会有缺失的间隔，而且之间的间隔比预期的大
#### 数组分块
由于js是单线程的，有时候一个耗时的操作会阻塞线程，导致这段时间用户无法与界面交互。
如果是某个循环比较耗时，并且该循环并不必须同步完成，就可以使用***数组分块***的技术，小块小块的处理数组，给主线程有空闲的机会，就可以不影响用户的操作。
实现数组分块非常简单：
```
function chunk(array, process, context){
    setTimeout(function(){
        var item = array.shift();
        process.call(context, item);
        if (array.length > 0){
            setTimeout(arguments.callee, 100);
        }
    }, 100);
}
```
使用的时候直接将要分块的数组以及处理程序作为参数即可
>一旦某个函数需要花50ms以上的时间完成，就可以考虑能否分隔成多个小任务来完成

#### 函数节流
浏览器中，某些计算和处理要比其他的昂贵的多。如果在触发频率很高的事件中去执行这些操作，有可能会导致浏览器第崩溃。函数节流就是为了解决这个问题
函数节流背后的思想是指，某些代码不可以在没有间断的情况下重复执行。第一次调用函数，创建一个定时器，在指定的时间间隔之后运行代码。当第二次调用函数时，清除前一次的定时器，并设置另一个。这样只有在执行函数的请求停止了之后才执行。

#### 自定义事件
观察者模式
<p style="color:red">（尚未详细了解）</p>

### 小结
js中的函数非常强大，因为它们是第一类对象。使用闭包和函数环境切换，还可以有很多函数的强大方法。
善于使用高级函数来实现更好的功能。
善于使用观察者模式来使不同部分的代码之间解耦，让维护更容易








