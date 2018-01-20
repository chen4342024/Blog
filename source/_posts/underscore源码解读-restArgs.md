---
title: underscore源码解读 ==> restArgs
date: 2016-09-16 20:07:58
tags: 
	- js
	- underscore
categories: underscore源码解读
---
### underscore源码分析
#### 如何实现不定参数
使用过underscore.js 的人，肯定都使用过以下几个方法：
```
_.without(array, *values) //返回一个删除所有values值后的array副本
_.union(*arrays) //返回传入的arrays（数组）并集
_.difference(array, *others)//返回来自array参数数组，并且不存在于other 数组
...
```
这些方法都有一个共同点，就是可以传入不定数量的参数，例如，我想删除掉array中的value1，value2，可以这样使用,`_.without(array,value1,value2);`
##### 那么，这个需要怎样才能做到呢？
我们知道，js中function里面，有一个arguments参数，它是一个类数组，里面包含着调用这个方法的所有参数，所以可以这样处理：
```
_.without = function(){
   if(arguments.length > 0){
      var array = arguments[0];
      var values = [];
	  for(var i = 1 ; i < arguments.length; i++){
	     values.push(arguments[i])
	  }
	  //这样得到了array，和values数组，便可以进一步处理了
   }
}
```
<!--more-->
上面只是打个比方，**想要支持不定参数，我们要做的就是把固定参数和动态参数从arguments中分离出来。**
但是，我们这样写的话，需要在每个支持不定参数的函数里，都copy这样一段代码，这样实在不是很优雅。所以需要封装成一个通用的函数。
我们直接看看underscore是封装的好了。
#### restArgs源码
```
var restArgs = function (func, startIndex) {
    //startIndex ,表示几个参数之后便是动态参数
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function () {
        var length = Math.max(arguments.length - startIndex, 0);
		//处理arguments，将动态参数保存进rest数组
        var rest = Array(length);
        for (var index = 0; index < length; index++) {
            rest[index] = arguments[index + startIndex];
        }
		//处理0,1,2三种情况，这里要单独处理，是想优先使用call，因为，call的性能比apply要好一点
        switch (startIndex) {
            case 0:
                return func.call(this, rest);
            case 1:
                return func.call(this, arguments[0], rest);
            case 2:
                return func.call(this, arguments[0], arguments[1], rest);
        }
		//如果startIndex不是0,1,2三种情况，则使用apply调用方法，将args作为参数，args将为数组[固定参数 ,rest];
        var args = Array(startIndex + 1);
        for (index = 0; index < startIndex; index++) {
            args[index] = arguments[index];
        }
        args[startIndex] = rest;
        return func.apply(this, args);
    };
};
//这里without主要的逻辑处理方法，作为参数，传给restArgs，在restArgs中处理完参数后，使用call或apply调用逻辑处理方法，这时候接受到参数otherArrays，已经是一个数组了，包含了之前的动态参数。
_.without = restArgs(function (array, otherArrays) {
    //处理without具体事件
});
```
underscore.js中利用js高级函数的特性，巧妙的实现了动态参数，如果要使某函数支持不定参数，只需要将该函数作为参数，传入restArgs中即可,例如：
```
function addByArray(values){
   var sum = 0;
   for (var i = 0 ; i < values.length; i++) {
      sum += values[i];
   }
   return sum;
}
var add = restArgs(addByArray);
//调用：
addByArray([2,5,3,6]);//16
add(2,5,3,6) //16
```
#### ES6 不定参数
ES6引入了rest参数，（形式为"...变量名"），用于获取多余参数，这样就不需要使用arguments对象来实现了
```
function add(...values) {
   let sum = 0;
   for (var val of values) {
      sum += val;
   }
   return sum;
}
add(2, 5, 3) // 10
```
#### 总结
在underscore中，restArgs只是为了支持不定参数。实际使用中，也许我们都是直接使用ES6，或用babel将ES6转成ES5来支持不定参数，不过，能了解下是如何实现的，对我们来说，想必也是极好的。：）


