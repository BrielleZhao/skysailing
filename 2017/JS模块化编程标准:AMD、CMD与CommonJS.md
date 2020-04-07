## 1.前言
#### 本世纪以来,随着网页应用复杂化的趋势逐渐明显，需要一个团队分工协作、进度管理、单元测试等等。开发者不得不使用软件工程的方法，管理网页的业务逻辑。Javascript模块化编程应运而生。但是ES6标准以前的javascript本身不是支持模块化的语言，没有类（Class）和模块(Module)的概念。
参考阮一峰老师的文章，js的模块化，除了常见的原始写法，对象写法，立即执行函数写法之外，还有以下几个写法值得了解：
* 放大模式
>如果一个模块很大，必须分成几个部分，或者一个模块需要继承另一个模块，这时就有必要采用"放大模式"（augmentation）
```javascript
var module1 = (function (mod){
　　　　mod.m3 = function () {
　　　　　　//...
　　　　};
　　　　return mod;
　　})(module1);
```
> 上面的代码为module1模块添加了一个新方法m3()，然后返回新的module1模块。

* 宽放大模式（Loose augmentation）
> 在浏览器环境中，模块的各个部分通常都是从网上获取的，有时无法知道哪个部分会先加载。如果采用上一节的写法，第一个执行的部分有可能加载一个不存在空对象，这时就要采用"宽放大模式"。
```javascript
var module1 = ( function (mod){
　　　　//...
　　　　return mod;
　　})(window.module1 || {});
```
> 与"放大模式"相比，＂宽放大模式＂就是"立即执行函数"的参数可以是空对象。

* 输入全局变量
> 独立性是模块的重要特点，模块内部最好不与程序的其他部分直接交互。为了在模块内部调用全局变量，必须显式地将其他变量输入模块。
```javascript
　var module1 = (function ($, YAHOO) {
　　　　//...
　　})(jQuery, YAHOO);
```
上面的module1模块需要使用jQuery库和YUI库，就把这两个库（其实是两个模块）当作参数输入module1。这样做除了保证模块的独立性，还使得模块之间的依赖关系变得明显。这方面更多的讨论，参见Ben Cherry的著名文章《JavaScript Module Pattern: In-Depth》

## 2.模块化的规范
#### 考虑到模块化的前提，就是大家必须以同样的方式编写模块目前。所以必须制定统一的模块化标准。 通行的Javascript模块规范共有两种：CommonJS和AMD。我主要介绍AMD，但是要先从CommonJS讲起。
###  2.1 CommonJS规范
2009年，美国程序员Ryan Dahl创造了node.js项目，将javascript语言用于服务器端编程。这标志"Javascript模块化编程"正式诞生。
node.js的模块系统，就是参照CommonJS规范实现的。在CommonJS中，有一个全局性方法require()，用于加载模块。
一个单独文件就是一个模块，通过require方法来同步加载要依赖的模块，然后通过extports或则module.exports来导出需要暴露的接口。
```javascript
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```
**优点**：
>* 服务器端模块重用，NPM中模块包多，有将近20万个。

**缺点**：
>* 加载模块是同步的，只有加载完成后才能执行后面的操作，也就是当要用到该模块了，现加载现用，不仅加载速度慢，而且还会导致性能、可用性、调试和跨域访问等问题。
>* Node.js主要用于服务器编程，加载的模块文件一般都存在本地硬盘，加载起来比较快，不用考虑异步加载的方式，因此,CommonJS规范比较适用。
>* 然而，这并**不适合在浏览器环境**，同步意味着阻塞加载，浏览器端的模块，不能采用"同步加载"（synchronous），只能采用"异步加载"（asynchronous）。因此有了AMD CMD解决方案。

**实现**:
>* 服务器端的 Node.js；
>* Browserify，浏览器端的 CommonJS 实现，可以使用 NPM 的模块，但是编译打包后的 文件体积可能很大；
>* modules-webmake，类似Browserify，还不如 Browserify 灵活；
>* wreq，Browserify 的前身；


###  2.2 AMD规范（ Asynchronous Module Definition 异步模块定义规范）
[官方规范](https://github.com/amdjs/amdjs-api/wiki/AMD)
AMD是"Asynchronous/eɪˈsɪŋkrənəs/ Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。
* define(id?, dependencies?, factory)  定义模块
> 它要在声明模块的时候指定所有的依赖 dependencies ，并且还要当做形参传到factory 中，对于依赖的模块提前执行，依赖前置。
*require(dependencies?,callback) 加载模块
> require函数接受两个参数。第一个参数是一个数组，表示所依赖的模块，上例就是['moduleA', 'moduleB', 'moduleC']，即主模块依赖这三个模块；第二个参数是一个回调函数，当前面指定的模块都加载成功后，它将被调用。加载的模块会以参数形式传入该函数，从而在回调函数内部就可以使用这些模块。
```javascript
define("module", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
require(["module", "../file"], function(module, file) { /* ... */ });
```
>举例，使用require.js的引入方式
```html
<script src="js/require.js" data-main="js/main"></script>
```
>main.js为主文件.假定主模块依赖jquery、underscore和backbone这三个模块，main.js就可以这样写
```javascript
require(['jquery', 'underscore', 'backbone'], function ($, _, Backbone){
　　　　// some code here
　　});
  
```
>require.config()方法，可以对模块的加载行为进行自定义
```javascript
require.config({
　　　　baseUrl: "js/lib",
　　　　paths: {
　　　　　　"jquery": "jquery.min",
　　　　　　"underscore": "underscore.min",
　　　　　　"backbone": "backbone.min"
　　　　}
　　});
```
> 一个依赖于myLib的math.js
```
define('math',['myLib'], function(myLib){
　　　　function foo(){
　　　　　　myLib.doSomething();
　　　　}
　　　　return {
　　　　　　foo : foo
　　　　};
　　});
  ```
 > 几个例子
 Using require and exports
```javascript
 //Sets up the module with ID of "alpha", that uses require, exports and the module with ID of "beta":

   define("alpha", ["require", "exports", "beta"], function (require, exports, beta) {
       exports.verb = function() {
           return beta.verb();
           //Or:
           return require("beta").verb();
       }
   });
//An anonymous module that returns an object literal: 返回对象字面量的匿名模块

   define(["alpha"], function (alpha) {
       return {
         verb: function(){
           return alpha.verb() + 2;
         }
       };
   });
//A dependency-free module can define a direct object literal:  用对象定义独立模块

   define({
     add: function(x, y){
       return x + y;
     }
   });
//A module defined using the simplified CommonJS wrapping: 用简化CommonJS包裹器定义的模块

   define(function (require, exports, module) {
     var a = require('a'),
         b = require('b');

     exports.action = function () {};
   });
```
**优点**：

>在浏览器环境中异步加载模块；并行加载多个模块;管理模块之间的依赖性，便于代码的编写和维护。；

**缺点**：

>开发成本高，代码的阅读和书写比较困难，模块定义方式的语义不顺畅；不符合通用的模块化思维方式，是一种妥协的实现；

**实现**：

>RequireJS； curl；

###  2.3 CMD规范 （Common Module Definition 公共模块定义规范）
 Common Module Definition 规范和 AMD 很相似，尽量保持简单，并与 CommonJS 和 Node.js 的 Modules 规范保持了很大的兼容性。他来源于[Sea.js](https://github.com/seajs/seajs)
规范见[CMD](https://github.com/cmdjs/specification/blob/master/draft/module.md)
> 关键函数 define(factory);
* define 函数接收单个参数即模式工厂
* factory 为对象、字符串时，表示模块的接口就是该对象、字符串
* factory 为函数时，表示是模块的构造方法。执行该构造方法，可以得到模块向外提供的接口
* factory 方法在执行时，默认会传入三个参数：require、exports 和 module
```javascript
define(function(require, exports, module) {
  // 模块代码
});
```
* define(id?, deps?, factory) define可接受两个以上参数
* define.cmd 可用于判断当前页面有无CMD 模块加载器
> require **Function**  接受*模块标识*作为唯一参数，用来获取其他模块提供的接口
```javascript
define(function(require, exports) {
  // 获取模块 a 的接口
  var a = require('./a');
  // 调用模块 a 的方法
  a.doSomething();
});
```
> require.async require.async(id, callback?)
  require.async 方法用来在模块内部异步加载模块，并在加载完成后执行指定回调。callback 参数可选。
  
**优点**：
```javascript
define(function(require, exports, module) {
  // 异步加载一个模块，在加载完成时，执行回调
  require.async('./b', function(b) {
    b.doSomething();
  });
  // 异步加载多个模块，在加载完成时，执行回调
  require.async(['./c', './d'], function(c, d) {
    c.doSomething();
    d.doSomething();
  });
});
```
**注意**：*require 是同步往下执行，require.async 则是异步回调执行。require.async 一般用来加载可延迟异步加载的模块。*
> require.resolve require.resolve(id)
使用模块系统内部的路径解析机制来解析并返回模块路径。该函数不会加载模块，只返回解析后的绝对路径.
这可以用来获取模块路径，一般用在插件环境或需动态拼接模块路径的场景下。
```javascript
define(function(require, exports) {
  console.log(require.resolve('./b'));
  // ==> http://example.com/path/to/b.js

});
```
> exports **Object** 是一个对象用来向外提供模块接口
* 模块对外接口一般使用export实现
```javascript
define(function(require, exports) {
  // 对外提供 foo 属性
  exports.foo = 'bar';
  // 对外提供 doSomething 方法
  exports.doSomething = function() {};

});
```
* 使用 return 直接向外提供接口（公司的项目模块大多使用这种模式）
```javascript
define(function(require) {
  // 通过 return 直接提供接口
  return {
    foo: 'bar',
    doSomething: function() {}
  };

});
//简化
define({
  foo: 'bar',
  doSomething: function() {}
});
```
* 注意，不能直接赋值exports
```javascript
  // 错误用法！！!
  exports = {
    foo: 'bar',
    doSomething: function() {}
  };
```
**提示**：*exports 仅仅是 module.exports 的一个引用。在 factory 内部给 exports 重新赋值时，并不会改变 module.exports 的值。因此给 exports 赋值是无效的，不能用来更改模块接口。*

> module *Object* 是一个对象，上面存储了与当前模块相关联的一些属性和方法
* module.id  模块的唯一标识
* module.uri  根据模块系统的路径解析规则得到的模块绝对路径
* module.dependencies  数组Array 表示当前模块的依赖
* module.exports 当前模块对外提供的接口
只通过 exports 参数来提供接口，有时无法满足开发者的所有需求。 比如当模块的接口是某个类的实例时，需要通过 module.exports 来实现：
```javascript
define(function(require, exports, module) {

  // exports 是 module.exports 的一个引用
  console.log(module.exports === exports); // true

  // 重新给 module.exports 赋值
  module.exports = new SomeClass();

  // exports 不再等于 module.exports
  console.log(module.exports === exports); // false

});
```
**注意** ：对 module.exports 的赋值需要同步执行，不能放在回调函数里。

*此部分参考[https://github.com/seajs/seajs/issues/242](https://github.com/seajs/seajs/issues/242)


**优点**
>依赖就近，延迟执行 可以很容易在 Node.js 中运行； 

**缺点**：
>依赖 SPM 打包，模块的加载逻辑偏重； 

**实现**：
>Sea.js ；coolie 
