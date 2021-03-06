# Node.js、npm包管理工具和 webpack打包工具
## Node.js —— 运行在服务端的 JavaScript
* Node.js 应用是由哪几部分组成的
    >* 引入 required 模块：我们可以使用 require 指令来载入 Node.js 模块。
    >* 创建服务器：服务器可以监听客户端的请求，类似于 Apache 、Nginx 等 HTTP 服务器。
    >* 接收请求与响应请求 服务器很容易创建，客户端可以使用浏览器或终端发送 HTTP 请求，服务器接收请求后返回响应数据。
## npm包管理工具
>* npm是前端开发中常用的一种工具，对于普通开发者来说，便于管理依赖。往大了说，便于共享代码。写完代码，使用npm发布以后，然后别人用npm可以方便地共享到你的代码。
>* NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
>>* 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
>>* 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
>>* 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。
### npm的使用：
 >* mac环境下的安装：brew install node //node自带npm
 >在前端工程的根文件下，npm init --yes 会在该文件夹下生成package.json//package.json 声明了该工程的名称、版本、依赖等信息。
 >在该工程的根文件夹下，npm install 依赖名 --save，会安装依赖，同时把改依赖信息写入package.json当中
```javascript
npm install 依赖名 --save-dev 安装开发测试环境的依赖
npm update //更新依赖
npm uninstall 依赖名 //在本地去除依赖
npm uninstall --save 依赖名 //在package.json中删除该依赖的信息
npm install -g 依赖名 //全局安装（全局指该主机的全局），可以直接在命令行中使用
npm update -g 依赖名 //依赖的更新
npm uninstall -g 依赖名 //依赖的卸载
```
## Node.js REPL(Read Eval Print Loop:交互式解释器):  读取 执行  打印  循环
>* Node.js 异步编程的直接体现就是回调。
>* 异步编程依托于回调来实现，但不能说使用了回调后程序就异步化了。 回调大大提高了Node.js的性能

## Node.js 事件循环
>* Node.js 是单进程单线程应用程序，但是通过事件和回调支持并发，所以性能非常高。
>* Node.js 的每一个 API 都是异步的，并作为一个独立线程运行，使用异步函数调用，并处理并发。
>* Node.js 基本上所有的事件机制都是用设计模式中观察者模式实现。
>* Node.js 单线程类似进入一个while(true)的事件循环，直到没有事件观察者退出，每个异步事件都生成一个事件观察者，如果有事件发生就调用该回调函数.

## Node.js模块系统
>* 为了让Node.js的文件可以相互调用，Node.js提供了一个简单的模块系统。
>* 模块是Node.js 应用程序的基本组成部分，文件和模块是一一对应的。换言之，一个 Node.js 文件就是一个模块，这个文件可能是JavaScript 代码、JSON 或者编译过的C/C++ 扩展。

## WebPack
### 高效的打包工具，可以把一堆js合成为一个js文件，同时生成html
>使用方法：
>>安装：
```javascript
npm install webpack -g
```
>>配置文件webpack.config.js

 ```javascript
var path = require('path');

module.exports = {
    entry: './app/index.js',     //入口文件
    output: {
        filename: 'bundle.js',   //生成的文件名
        path: path.resolve(__dirname, 'dist') //生成文件的路径
    }
};
```

>>通过npm使用webpack在package.json内配置scripts build

```javascript
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"  //关键代码
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.23.1",
    "babel-loader": "^6.3.2",
    "babel-preset-es2015": "^6.22.0",
    "html-webpack-plugin": "^2.28.0",
    "webpack": "^2.2.1"
  },
  "dependencies": {
    "lodash": "^4.17.4"
  }
}
```
>>然后, npm run build就开始使用webpack进行打包
 
>>webpack把app/index.js为入口的一堆js打包合成为一个 bundle.js
更厉害的是，webpack可以将一个html模板和bundle.js合成在一起，生成一个引用了bundle.js的html文件。
生成html的方法：
>>安装
```
npm install html-webpack-plugin --save-dev 
```
>> 修改webpack.config.js
```
var path = require('path');
var HtmlWebpackPlugin= require('html-webpack-plugin'); //声明下插件

module.exports = {
    entry: './app/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins:[                              //使用插件
        new HtmlWebpackPlugin({
            filename :'index.html',    //要生成的html文件
            template:'index.html'     //html文件的模板
        })
    ]
};
```

>>修改后，npm run build 就会在dist文件夹下生成一个html文件，bundle.js文件已经引入好了。
 
 
>> 另外前段开发过程中存在一个这样的问题：
es6（es2015和es2016）已经出了很久了，但是市面上的浏览器还不能完全兼容。为了解决这个问题，babel应运而生。babel就是能把你es6写的代码转成es5，方便浏览器兼容。
 
>> babel使用方法：
>> 安装
```
npm install babel-core babel-loader --save-dev
```
>> 修改webpack.config.js的配置文件
```javascript
var path = require('path');
var HtmlWebpackPlugin= require('html-webpack-plugin');

module.exports = {
    entry: './app/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    plugins:[
        new HtmlWebpackPlugin({
            filename :'index.html',
            template:'index.html'
        })
    ],
    module:{        //添加loader
        loaders:[
            {
                test: /.js$/,   //那些文件要转化
                exclude: /node_modules/, //屏蔽哪些
                loader:"babel-loader"       //指定loader
            }
        ]
    }
};    

```
