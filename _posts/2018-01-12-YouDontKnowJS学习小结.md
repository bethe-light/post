---
layout: post
title: 'YouDontKnowJS学习小结'
date: 2018-01-12
author: 李振
cover: '../assets/img/JS.jpg'
tags: web
---

# YouDontKnowJS 学习小结

> **真正的理解闭包的原理与使用 更加透彻this绑定的四种规则机制**  
> 你不知道的JavaScript 人称小黄书，第一次看到这本书名 就想到了一句话 “You konw nothing! Jon snow”(你懂得)， 翻阅后感觉到很惊艳，分析的很透彻，学习起来也很快，Have fun！

## 块级作用域

* if语句

* for语句

* with

  ``` javascript
  with(obj) {
      a = 1;
      b = 2;
      c = 3;
  }
  ```

  相当于

  obj.a = 1;

  obj.b = 2;

  obj.c = 3;  (比较麻烦)

  > 而且用with在对象上创建的块作用域仅仅在with声明中有效

* try  / catch

  >  catch 事实上也会创建一个块作用域，且仅仅在catch内部有效

* let 

* const

## 块作用域的用处

  1. 垃圾回收
  2. let循环

## 声明提升

a = 2;  

var a;

console.log(a); // 2

> 这就是声明提升，先有蛋（声明），后有鸡（赋值）
>
> **只有声明被提升**，赋值或其他逻辑保留在原地
>
> 真正的顺序是 
>
> var a   (未执行前，在编译阶段)
>
> a = 2   （等待执行）
>
> console.log(a)

**函数声明会被提升，而函数表达式不会**

>  函数声明，变量声明同时出现（一般不会出现，出现情况是重复声明）, 则是 函数优先

## 作用域闭包

> **闭包实质**: <u>当函数可以记住并访问所在的词法作用域时</u>，<u>就产生了闭包</u>，<u>即使函数在当前词法作用域之外执行</u>
>
> 1. 将内部函数传递到词法作用域外，他都会有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包
>
> 2. 在定时器，时间监听器，Ajax请求，跨窗口通信，web workers 或者其他任何异步任务，只要使用了
>
>    **回调函数**，实际上都是在使用到了闭包
>
> **IIFE**：立即执行函数，典型的闭包例子，IIFE会通过声明并立即执行函数创建作用域 

> **经典栗子**

```javascript
for(var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i)
    },0);
}  // output: 5 5 5 5 5
// setTimeoiut延迟函数会在循环结束之后执行 ，而此时i为5 结果自然为5个5
//利用闭包解决,创建一个块作用域保存一个i的副本即可
@方法1
for(var i = 0; i < 5; i++) {
    (function() {
        var j = i;
        setTimeout(function() {
            console.log(j);
        },0);
    })();
} // output: 0 1 2 3 4 
@方法2
for(var i = 0; i < 5; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(j);
        },0);
    })(i);
} // output: 0 1 2 3 4 
@方法3 （let即可,简单实用）
for(let i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i)
    },0);
}  // output: 0 1 2 3 4
```



## 模块机制

> **现代模块**
>
> ```javascript
> var MyModules = (function manager() {
>     var modules = {};
>     function define(name, deps, imp) {
>         //...
>         modules[name] = imp.apply(imp, deps);
>     };
>     function get(name) {
>         return modules[name];
>     };
>     return {
>         define: define,
>         get: get
>     };
> })();
> ```
>
> **未来模块机制**
>
> ```javascript
> bar.js
> 	function add(a, b){
>     	return  a + b;
> 	}
> 	export add;
> 
> foo.js
> 	import bar from 'bar'
> 	console.log(bar.add());
> ```
>
> **特征**
>
> 1. 为创建一个内部作用域而调用一个包装的函数
> 2. 包装函数的返回值必须至少包含一个内部函数的引用， 这才会创建涵盖整个包装函数内部作用域的闭包

## 关于This

>  **为什么要用** **this**

> ​	1. this提供了一种更加优雅的方式来隐式传递 一个 对象的引用，使API设计的更加简洁与重复使用
>
> ​	2.this既不指向函数自身，也不指向函数的词法作用域
>
> ​	3.实质是在函数调用发生的绑定，指向完全取决于函数在哪里被调用

四种绑定方法：

**1.默认绑定**

```js
function foo（）{
    console.log(this.a);
}
var a = 2;
foo(); //2
// this默认指向window
```

**2.隐式绑定**

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
}
obj.foo(); //2
```

> 会存在隐式丢失绑定对象，丢失后就成为 默认绑定（指向全局对象）

**3.显示绑定**

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
}
foo.call( obj ); //2
```

> 通过call(),apply() 显示绑定，也会存在绑定丢失问题 call(this,参数列表)，apply(this,args)，效果相同，参数不同

>  **硬绑定**可以解决，但是一旦绑定，则不能再修改this

**4.new绑定**

> javascript 的 new操作与其他语言的new操作 很大的不同
>
> new 的四个实质过程
>
> 1. 创一个全新的对象
> 2. 新对象会被执行[[Prototype]]链接
> 3. 新对象会被绑定到函数调用的this
> 4. 若函数没有返回其他对象，new表达式的函数调用会自动返回这个新对象

```javascript
function foo() {
    this.a = a;
}
var bar = new foo(2);
console.log(bar.a); // 2
```

#### **上面四种绑定的优先级**

> > new绑定  >  显示绑定  >  隐式绑定  >  默认绑定
> >
> > 默认绑定优先级最低；
>
> > 显示绑定比隐式绑定优先级更高
>
> ```javascript
> function foo() {
>     console.log(this.a);
> }
> var obj1 = {
>     a: 2,
>     foo: foo
> };
> var obj2 = {
>     a: 4,
>     foo: foo
> };
> obj1.foo(); //隐式 2
> obj2.foo(); //隐式 4
> 
> obj1.foo.call( obj2 ); // 隐式,显示 同在  显示优先 4
> obj2.foo.call( obj1 ); // 2 
> 
> ```
>
> > new绑定比隐式绑定优先级更高  
>
> ```javascript
> function foo(some) {
>     this.a = some;
> } 
> var obj1 = {
>     foo: foo
> };
> var obj2 = {};
> obj1.foo(2);  
> console.log(obj1.a); //2
> obj1.foo.call(obj2, 3);
> console.log(obj2.a); //3
> 
> var bar = new obj1.foo(4);
> console.log(obj1.a); // 隐式绑定 2
> console.log(bar.a);  // new绑定 4
> ```
>
> >  new绑定 与 显示绑定比较
>
> ```javascript
> function foo(some) {
>     this.a = some;
> }
> var obj = {};
> var bar = foo.bind(obj1);
> bar(2);
> console.log(obj1.a); // 2
> 
> var baz = new bar(3);
> console.log(obj1.a); // 2
> console.log(baz.a);  // 3
> // new 修改了硬绑定 bind()中的this
> ```

#### 箭头函数的特殊 this

> ES6中特殊函数类型 箭头函数 并不适用于上面四中绑定方式规则
>
> **箭头函数 根据函数或则全局作用域决定this**
>
> ```javascript
> function foo() {
>     return (a) => {
>        // this继承foo
>        console.log(this.a);
>     };
> }
> var ojb1 = {
>     a: 2
> }
> var obj2 = {
>     a: 3
> }
> var bar = foo.call(obj1); 
> bar.call(obj2); // 2 
> ```
>
> > foo()的this绑定到obj1，而且bar 的this也会绑定到obj1 ，箭头函数的绑定无法被修改，new 也不能将其修改

##### 两种代码风格的使用

> 1. var that = this
> 2. 箭头函数
>
> ```javascript
> 箭头函数
> function foo() {
>     setTimeout(() => {
>         console.log(this.a); //若不适用箭头函数，则绑定丢失，this默认绑定到去全局 为 1
>     }, 1000);
> }
> var obj = {
>     a: 2
> };
> var a = 1;
> foo.call(obj);// 2
> ******************************************************************************
> 使用 that = this
> function foo() {
>     var that = this;
>     setTimeout(function() {
>         console.log(that.a); //用that传递原this的词法作用域
>     },1000);
> }
> var obj = {
>     a: 2
> };
> foo.call(obj); //2 
> ```

> **tips:** 两种风格最好不要混合使用，不然很难维护，笔者本人还是比较喜欢使用 that 而不是箭头函数

