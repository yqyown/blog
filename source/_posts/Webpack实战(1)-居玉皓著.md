---
title: Webpack实战(1)-居玉皓著
date: 2021-03-12 11:06:48
tags: ['React','Webpack']
categories: 技术读物
---
基于《Webpack实战 入门、进阶与调优》学习Webpack，搭建学习demo，共两个部分，本文是第一部分。

<br />
<br />

# 一、Webpack简介

Webpack 核心概念：

- Entry（入口）：Webpack 执行构建的第一步将从 Entry 开始，可抽象成输入
- Output（出口）：指示 webpack 如何去输出、以及在哪里输出
- Module（模块）：在 Webpack 里一切皆模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块
- Chunk（代码块）：一个 Chunk 由多个模块组合而成，用于代码合并与分割
- Loader（模块转换器）：用于把模块原内容按照需求转换成新内容
- Plugin（扩展插件）：在 Webpack 构建流程中的特定时机会广播出对应的事件，插件可以监听这些事件，并改变输出结果

<br />
<br />

# 二、安装

## 2.1  创建工程目录learn_react_webpack

<br />

## 2.2  进入该目录，执行npm初始化命令

```bash
npm init  #若使用yarn，则yarn init
```

    输入项目基本信息，最后在目录中生成了一个package.json文件，它相当于npm项目的说明书，里面记录了项目名称、版本、仓库地址等信息

<br />

## 2.3  执行安装webpack的命令

```bash
npm install webpack webpack-cli --save-dev
```

    1. webpack是核心模块，webpack-cli是命令行工具；
    2. 当前安装webpack 5.23.0，webpack-cli 3.3.12；

<br />

## 2.4  打包第一个应用

### 2.4.1  在工程目录下添加文件

文件目录如下：

```basic
learn_react_webpack/
dist/
    bundle.js
node_modules/
  public/
    index.html
  src/
    App.js
    index.js
  package.json
```

App.js：

```javascript
export default function(){
    document.write('Hello world!!');
}
```

index.js：

```javascript
import App from './App';
document.write('This is index file.<br/>');
App();
```

index.html：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>learn react webpack</title>
</head>
<body>
    <script src="/dist/bundle.js"></script>
</body>
</html>
```

### 2.4.2  使用npm scripts

编辑工程中的package.json文件，添加脚本命令：

```json
  "scripts": {
    "build":"webpack --entry=./src/index.js --output-filename=bundle.js --mode=development",
  },
```

    1. entry：资源打包入口，webpack默认的源代码入口就是src/index.js，因此可以省略entry配置
    2. output-filename：输出资源名
    3. mode：打包模式，development、production、none三种模式


### 2.4.3  执行打包

输入`npm build`，打开index.html页面验证


### 2.4.4  使用配置文件

使用`npx webpack -h`，查看webpack的配置项以及相对应的命令行参数。

<br />

当项目需要越来越多的配置时，将这些命令行参数改为对象的形式专门存放在一个配置文件里。Webpack的默认配置文件为webpack.config.js。

<br />

1）在工程根目录下创建webpack.config.js，添加如下代码：

```javascript
module.exports={
    entry:'./src/index.js',
    output:{
        filename:'bundle.js',
    },
    mode:'development'
}
```

2）去掉package.json中配置的打包参数：

```json
  "scripts": {
    "build":"webpack",
  },
```

3）为了验证效果，修改App.js内容

```javascript
export default function(){
    document.write('using a config file!!');
}
```

4）执行`npm run build`，Webpack就会预先读取webpack.config.js，然后进行打包，最后打开index.html进行验证



### 2.4.5  webpack-dev-server

每次修改完源代码都要执行npm run build更新bundle.js，然后刷新页面才能生效，有没有简便的方法呢？

本地开发工具——webpack-dev-server，可以提高开发调试效率。

```bash
npm install webpack-dev-server --save-dev
```

1）在package.json中添加dev命令

```json
  "scripts": {
    "build": "webpack",
    "dev":"webpack-dev-server",
  },
```

2）编辑webpack.config.js文件，配置webpack-dev-server

```javascript
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
    },
    mode: 'development',
    devServer: {
        // publicPath: '/', //用于确定 bundle 的来源，并具有优先级高于 contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

3）修改App.js内容

```javascript
export default function(){
    document.write('using webpack-dev-server!!');
}
```

4）修改index.html内容

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>learn react webpack</title>
</head>
<body>
    <script src="bundle.js"></script> //如果设置了publicPath，要以publicPath作为路径
</body>
</html>
```

5）执行`npm run dev`，打开http:\//localhost:8080/

![终端执行结果](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmRunDev.png)

![启动页面](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmRunDevStartIndex.png)

webpack-dev-server配置时遇到的问题：

Q1：Cannot get /

执行`npm run dev`后，页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就会显示cannot get /，因此，contentBase设置的值是index.html所在的文件目录。

<br />

Q2：没有展示页面内容，只显示了一个目录视图

如果contentBase参数设置不对的话，会展示一个以该参数指定目录作为根目录的路由窗口，contentBase设置到为index.html。如果设置了publicPath，index.html中的src也要以publicPath作为路径。

<br />

=====》由于文件直接打包在/dist文件夹根目录下，因此publicPath不用设置，最终配置如下：

![webpack.config.js配置](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/webpackDevServerWebpackConfig.png)

![index.html页面配置](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/webpackDevServerIndex.png)


<br />
<br />


# 三、类型模块


CommonJS

ES6 Module

AMD

UMD


<br />


## 3.1  CommonJS和ES6 Module形式

### 3.1.1  模块

- CommonJS：

```javascript
//app.js
var name='app.js';

//index.js
var name='index.js';
require('./app.js');
console.log(name); //index.js, app.js中的变量声明不会影响index.js, 每个模块拥有各自的作用域
```



- ES6 Module：

```javascript
//app.js
export default{
    name:'app.js',
  add:function(a, b){
    return a + b;
  }
}

//index.js
import app from './app.js';
const sum=app.add(2,3);
console.log(sum); //5
```



### 3.1.2  导入

- CommonJS：

```javascript
//app.js
module.exports={
    add:function(a,b){
    return a + b;
  }
}

//index.js
const app=require('./app.js');
const sum=app.add(2,3);
console.log(sum); //5
```

    注意：当我们require一个模块时会有两种情况：
    1. require的模块是第一次被加载。这时会首先执行该模块，然后导出内容；
    2. require的模块曾被加载过。这时该模块的代码不会再次执行，而是直接导出上次执行后得到的结果


- ES6 Module：

```javascript
//app.js
const name='app.js';
const add=function(a,b){ return a + b };
export { name , add };

//index.js
import {name,add as calculateSum} from './app.js';
calculateSum(2,3);
```



### 3.1.3  导出

- CommonJS：

```javascript
module.exports={
  name:'app.js',
    add:function(a,b){
    return a + b;
  }
}

或

exports.name='app.js';
exports.add=function(a, b){
    return a + b;
}
```

    注意：module.exports 和 exports 不能混用



- ES6 Module：

```javascript
export const name='app.js';
export const add=function(a,b){return a + b};

或

const name='app.js';
const add=function(a,b){return a + b};
export { name,add as getSum };
```



## 3.2  CommonJS和ES6 Module区别

### 3.2.1  动态与静态


CommonJS和ES6 Module最本质的区别在于前者对模块依赖的解决是“动态的”，而后者是“静态的”。这里的“动态”是模块依赖关系的建立发生在代码运行阶段；而“静态”则是模块依赖关系的建立发生在代码编译阶段。  

<br />

require的模块路径可以动态指定，支持传入一个表达式，甚至通过if语句判断是否加载某个模块，因此CommonJS模块被执行前，没办法确定明确的依赖关系。

<br />

ES6 Module的导入、导出语句都是声明式的，不支持导入的路径是一个表达式，并且导入、导出语句必须位于模块的顶层作用域，相比于CommonJS具备以下优势：

- 死代码检测和排除。可以用静态分析工具检测出哪些模块没有被调用过。
- 模块变量类型检查。确保模块之间传递的值或接口类型是正确的。
- 编译器优化。在CommonJS等动态模块系统中，本质上导入的都是一个对象，而ES6 Module支持直接导入变量，减少引用层级。


<br />


### 3.2.2  值拷贝与动态映射

CommonJS是一份导出值的拷贝；ES6 Module则是值的动态映射，这个映射是只读的。


- CommonJS：

```javascript
//app.js
var count = 0;
module.exports = {
    count: count,
    add: function (a, b) {
        return a + b;
    }
} 

//index.js
var count=require('./App.js').count;
var add=require('./App.js').add;

console.log(count); //0，对App.js中count值的拷贝
add(2,3);
console.log(count); //0，App.js中变量值的改变不会对这里的拷贝值有影响

count+=1;
console.log(count); //1，拷贝值可以更改
```



ES6 Module：

```javascript
//app.js
let count = 0;
const add = function (a, b) {
    count += 1;
    return a + b;
};
export { count, add };


//index.js
import { count, add } from './App';

console.log(count); //0, 对app.js中count值的映射
add(2, 3);
console.log(count); //1, 实时反映app.js中count值的变化

count += 1; //Uncaught TypeError: Cannot set property count of #<Object> which has only a getter
```

### 2.3  循环依赖



<br />


## 3.3  AMD（异步模块定义）

使用define函数定义模块，接受3个参数：

- 第1个参数当前模块的id，相当于模块名；
- 第2个参数是当前模块的依赖，如getSum模块需要引入app模块作为依赖；
- 第3个参数用来描述模块的导出值，可以是函数或对象。如果是函数则导出的是函数的返回值；如果是对象则直接导出对象本身。

```javascript
define('getSum',['app'],function(math){
    return function(a,b){
    console.log('sum:'+app.add(a,b));
  }
});
```

## 3.4  UMD（通用模块标准）


<br />
<br />


# 四、资源输入输出

## 4.1  资源处理流程

Webpack会从入口文件开始检索，将具有依赖关系的模块生成一棵依赖树，最终得到一个chunk。由这个chunk得到的打包产物称之为bundle（束）。entry、chunk、bundle的关系如图：

![资源输出关系图](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/output.png)



在工程中可以定义多个入口，每个入口都会产生一个结果资源，entry和bundle存在着对应关系：

![entry和bundle关系](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/entryBundle.png)



## 4.2  配置资源

```javascript
module.exports={
    context:path.join(__dirname,'./src'),
  entry:{
    app:'app.js',
    page:'page.js', //演示多入口, 每个页面都有一个独立的bundle
    vendor:['react','react-dom','react-router'],
  },
  output:{
    filename:'[name].js',
    path:path.join(__dirname,'./dist'),
    publicPath:'/assets/',
  } 
}
```

    配置资源入口：
    1. context：资源入口的路径前缀；可以省略，默认值为当前工程的根目录；
    2. entry：支持多种形式：字符串、数组、对象、函数；
    3. vendor：一般指的是工程所使用的库、框架等第三方模块集中打包而产生的bundle；


    配置资源出口：
    1. filename：输出资源的文件名，形式为字符串，不仅可以是bundle的名字，还可以是一个相对路径。路径中的目录不存在，Webpack会自动创建该目录；模板变量支持[hash]、[chunkhash]、[id]、[query]；
    2. path：指定资源输出的位置，Webpack 4之后，output.path默认为dist目录；
    3. publicPath：由JS或CSS所请求的间接资源路径；


<br />
<br />


# 五、预处理器

Webpack本身只认识JavaScript，对于其他类型的资源(CSS、图片、字体等)必须预先定义一个或多个loader对其进行转译，输出为Webpack能够接收的形式再继续进行，因此loader实际上是一个预处理的工作。

<br />

## 5.1  css-loader：处理CSS的各类加载语法


将css-loader添加到工程中：

```bash
npm install css-loader style-loader
```

    1. css-loader的作用仅仅是处理CSS的各种加载语法（@import和url()函数等），如果要使样式起作用还需要style-loader将样式插入页面；
    2. 当前安装css-loader 5.0.2，style-loader 2.0.0；



App.css：

```css
.app{
    background: red;
}
```

App.js：

```javascript
import './App.css';

export default function(){
    document.write('<div class="app">Hello world!!</div>');
}
```

index.js：

```javascript
import App from './App';

document.write('This is index file.<br/>');
App();
```

webpack.config.js：

```javascript
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
    },
    mode: 'development',
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定 bundle 的来源，并具有优先级高于 contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

    1. css-loader与style-loader通常是配合在一起使用，style-loader用来将样式字符串包装成style标签插入到页面；
    2. 因为Webpack打包时是按照数组**从后往前**的顺序将资源交给loader处理的，因此style-loader加到css-loader前面；
    3. exclude和include同时存在时，exclude的优先级更高；（*[*配置参考*](https://survivejs.com/webpack/loading/loader-definitions/)*）
    4. resource和issuer，resource是被加载模块，issuer是加载者；
    5. enforce用来指定一个loader的种类，只接收“pre”和“post”两种字符串类型的值；

<br />

————————————————————————————————————————————————

eslint：开源的JavaScript的linting工具，用来检查代码质量问题和统一代码风格。

<br />


安装命令：

```bash
npm install eslint eslint-loader --save-dev

# or

yarn add eslint eslint-loader --dev
```

    当前版本eslint 7.20.0，eslint-loader 4.0.2，eslint-plugin-react 7.22.0；



初始化命令：

```bash
npx eslint --init

# or

yarn run eslint --init
```

![生成eslintrc.json文件](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/eslintrcJson.png)

执行完成（这里选择当前项目不使用typescript），选择json格式，生成.eslintrc.json文件；



.eslintrc.json：

```json
{
    "env": {
        "browser": true,
        "es2021": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:react/recommended"
    ],
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "ecmaVersion": 12,
        "sourceType": "module"
    },
    "plugins": [
        "react"
    ],
    "rules": {
        "indent": ["error", 4] //打开规则作为一个错误; 首字母缩进4个字符
    }
}
```

webpack.config.js：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : 'bundle.js',
    },
    mode: ENV,
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: 'file-loader',
        }, {
            test: /\.js$/,
            use: 'eslint-loader',
            enforce: 'pre',
            exclude: /node_modules/,
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

App.js：

```javascript
import './App.css';

export default function () {
    document.write('<div class="app">Hello world!!</div>');
}
```

<br />

————————————————————————————————————————————————

设置的规则是“首字母缩进4个字符”，故意删除App.js中的首字母缩进，有两种方式解决：

===》VS Code安装ESLint插件，快捷键`shift+Alt+f`格式化；

===》eslint --fix命令：

```json
  "scripts": {
    "esfix":"yarn eslint --fix --ext .js ./src"
  },
```

执行`yarn run esfix`，查看App.js代码格式是否修复


<br />


## 5.2  babel-loader：处理ES6+编译为ES5

安装react命令：

```bash
npm install react react-dom --save
```

index.html：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>learn react webpack</title>
</head>

<body>
    <div id="webapp">
        hello webpack
    </div>
    <script src="bundle.js"></script>
</body>

</html>
```

index.js：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import hexoimg from './images/hexo-deploy.png';

ReactDOM.render(
    <>
        <div>Hello React!</div>
        <img src={hexoimg} />
        <App />
    </>,
    document.getElementById('webapp')
);
```

App.js：

```javascript
import React from 'react';
import './App.css';

class App extends React.Component {
    render() {
        return (
            <div className="app">
                这是App组件！
            </div>
        );
    }
}

export default App;
```

执行`npm run dev`，出现错误：

![没有引用babel终端显示结果](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/noBabel.png)

因为使用了react，react是使用jsx语法实现的，而jsx不能直接被浏览器识别和执行，所以这里需要引入Babel库进行转码。

```bash
npm install babel-loader @babel/core @babel/preset-env @babel/preset-react
```

    1. 当前版本babel-loader 8.2.2，@babel/core 7.13.1，@babel/preset-env 7.13.5，@babel/preset-react 7.12.13；
    2. 各个模块的作用：
    babel-loader：它会使Babel与Webpack协同工作的模块；
    @babel/core：它是Babel编译器的核心模块；
    @babel/preset-env：它是Babel官方推荐的预置器，可根据用户设置的目标环境自动添加所需的插件和补丁来编译ES6+代码；



webpack.config.js：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : 'bundle.js',
    },
    mode: ENV,
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    publicPath: '/',
                }
            }
        }, {
            test: /\.(js|jsx)$/,
            use: {
                loader: 'babel-loader',
                options: {
                    cacheDirectory: true, //缓存机制, 重复打包未改变的模块防止二次编译
                    presets: [[
                        '@babel/env', {
                            modules: false, //禁用模块语句的转化, 将ES6 Module的语法交给Webpack本身处理
                        }
                    ], [
                        '@babel/preset-react', {
                            modules: false,
                        }
                    ]],
                }
            },
            exclude: /node_modules/
        },
        {
            test: /\.js$/,
            use: 'eslint-loader',
            enforce: 'pre',
            exclude: /node_modules/,
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

执行`npm run dev`，验证结果：

![引用babel页面结果](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/hasBabelStartIndex.png)


## 5.3  file-loader：打包文件类型的资源，并返回其publicPath

1）安装命令

```bash
npm install file-loader
```

    当前版本file-loader 6.2.0；



文件目录结构：

```basic
learn_react_webpack/
dist/
    bundle.js
node_modules/
  public/
    index.html
  src/
        images/
            hexo-deploy.png
    App.js
    index.js
  package.json
```



index.js：

```javascript
import App from './App';
import hexoimg from './images/hexo-deploy.png';

document.write('This is index file.<br/>');
document.write('<img src=' + hexoimg + '>');
App();
```



webpack.config.js：

```javascript
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
    },
    mode: 'development',
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: 'file-loader',
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定 bundle 的来源，并具有优先级高于 contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

2）执行`npm run dev`，打开index.html验证结果；



以上配置没有指定output.publicPath，因此打印出的图片路径只是文件名，默认为文件的hash值加上文件后缀。


<br />


===》添加output.publicPath：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : 'bundle.js',
        publicPath: '/'
    },
    mode: ENV,
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: 'file-loader',
        }, {
            test: /\.js$/,
            use: 'eslint-loader',
            enforce: 'pre',
            exclude: /node_modules/,
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

index.js：

```javascript
import App from './App';
import hexoimg from './images/hexo-deploy.png';

document.write('This is index file.<br/>');
document.write('<img src=' + hexoimg + '>');
console.log(hexoimg);
App();
```

执行`npm run dev`，此时图片路径变成以下形式：

![引用file-loader](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmFileloader.png)





===》file-loader支持配置文件名以及publicPath：

```javascript
rule:[
  {
    test: /\.(png|svg|jpg|gif)$/,
    use:{
        loader:'file-loader',
      options:{
        name:'[name].[ext]',
        publicPath:'/',
      }
    }
  }
]
```

index.js：

```javascript
import App from './App';
import hexoimg from './images/hexo-deploy.png';

document.write('This is index file.<br/>');
document.write('<img src=' + hexoimg + '>');
console.log(hexoimg);
App();
```

执行`npm run dev`，此时图片路径变成以下形式：

![引用file-loader输出保留文件名](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/fileloaderUseFileName.png)


## 5.4  url-loader：作用与file-loader类似

不同在于设置一个文件大小的阈值，当大于该阈值时与file-loader一样返回publicPath，而小于该阈值时返回文件base64形式编码。



```bash
npm install url-loader
```

    当前版本url-loader 4.1.1；



webpack.config.js：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : 'bundle.js',
    },
    mode: ENV,
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        },{
            test: /\.(png|svg|jpg|gif)$/,
            use:{
                loader:'url-loader',
                options:{
                    limit:102400,
                    name:'[name].[ext]',
                    publicPath:'/',
                }
            }
        }, {
            test: /\.js$/,
            use: 'eslint-loader',
            enforce: 'pre',
            exclude: /node_modules/,
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

index.js：

```javascript
import App from './App';
import hexoimg from './images/hexo-deploy.png';

document.write('This is index file.<br/>');
document.write('<img src=' + hexoimg + '>');
console.log(hexoimg);
App();
```

执行`npm run dev`，此时图片路径变成以下形式：

![引用url-loader](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmUrlloader.png)

## 5.5  自定义loader

比如我们将实现一个loader，它会为所有JS文件启用严格模式，也就是说它会在文件头部加上如下代码：

```javascript
'use strict'
```

在开发一个loader时，我们可以借助npm/yarn的软链功能进行本地调试（可以考虑发布到npm等）。下面初始化这个loader并配置到工程中。



创建一个force-strict-loader目录，然后在该目录下执行npm初始化命令。

```bash
npm init -y
```

接着创建index.js，也就是loader的主体：

```javascript
module.exports = function(content){
    var useStrictPrefix = '\'use strict\';\n\n';
  return useStrictPrefix + content;
}
```

现在可以在Webpack工程中安装并使用这个loader：

```bash
npm install <path-to-loader>/force-strict-loader
```

    在Webpack工程目录下使用相对路径安装，会在项目的node_modules中创建一个指向实际force-strict-loader目录的软链，也就是之后可以随时修改loader源码并且不需要重复安装。



修改Webpack配置：

```javascript
module: {
    rules: [
    {
        test:'/\.js$/',
      use:'force-strict-loader',
    }
  ]
}
```

这个loader设置为对所有JS文件生效。此时对该工程进行打包，可以看到JS文件的头部都已经加上了启用严格模式的语句。


<br />
<br />


# 六、样式处理

## 6.1  分离样式文件

使用style-loader和css-loader打包JS引用CSS的样式，如果使用style标签方式引入样式，如何输出单独的CSS文件呢？

- extract-text-webpack-plugin：适用于Webpack 4之前版本
- mini-css-extract-plugin：适用于Webpack 4及以上版本

<br />


## 6.2  样式预处理

样式预编译语言：

SCSS

Less


<br />


## 6.3  PostCSS

PostCSS是一个用JS工具和插件转换 CSS 代码的工具。它的工作模式是接收样式源代码并交由编译插件处理，最后输出CSS。



```bash
npm install postcss-loader
```

    当前版本postcss-loader 5.0.0；



webpack.config.js：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : 'bundle.js',
    },
    mode: ENV,
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader','postcss-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }, {
            test: /\.(png|svg|jpg|gif)$/,
            use: {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    publicPath: '/',
                }
            }
        }, {
            test: /\.(js|jsx)$/,
            use: {
                loader: 'babel-loader',
                options: {
                    cacheDirectory: true, //缓存机制, 重复打包未改变的模块防止二次编译
                    presets: [[
                        '@babel/env', {
                            modules: false, //禁用模块语句的转化, 将ES6 Module的语法交给Webpack本身处理
                        }
                    ], [
                        '@babel/preset-react', {
                            modules: false,
                        }
                    ]],
                }
            },
            exclude: /node_modules/
        },
        {
            test: /\.js$/,
            use: 'eslint-loader',
            enforce: 'pre',
            exclude: /node_modules/,
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

在项目的根目录下创建一个postcss.config.js：

```javascript
/* eslint-disable no-undef */
module.exports={};
```



### 6.3.1  Autoprefixer自动前缀

根据caniuse.com上的数据，自动决定是否要为某一特性添加厂商前缀，由开发者为其指定支持浏览器的范围。



```bash
npm install autoprefixer
```

    当前版本autoprefixer 10.2.4；



在postcss.config.js中添加autoprefixer：

```javascript
/* eslint-disable no-undef */
const autoprefixer = require('autoprefixer');

module.exports = {
    plugins: [
        autoprefixer({
            grid: true,
            overrideBrowserslist: [
                '>1%',
                'last 3 versions',
                'android 4.2',
                'ie 8'
            ]
        })
    ]
};
```

App.css：

```css
.container {
  display: grid;
}

.app {
  background: red;
}
```

App.js：

```javascript
import React from 'react';
import './App.css';

class App extends React.Component {
    render() {
        return (
            <div className="container">
                <div className="app">
                    这是App组件！
                </div>
            </div>
        );
    }
}

export default App;
```

执行`npm run dev`，进行验证：

![引用autoprefixer](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmAutoprefixer.png)

    当前测试浏览器 Chrome 88.0.4324.182（正式版本） （64 位）



### 6.3.2  stylelint质量检查工具

stylelint是一个CSS的质量检测工具，就像eslint一样，为其添加各种规则，来统一项目的代码风格，确保代码质量。


```bash
npm install stylelint
```

    当前版本stylelint 13.11.0；



postcss.config.js：

```javascript
/* eslint-disable no-undef */
const autoprefixer = require('autoprefixer');
const stylelint = require('stylelint');

module.exports = {
    plugins: [
        autoprefixer({
            grid: true,
            overrideBrowserslist: [
                '>1%',
                'last 3 versions',
                'android 4.2',
                'ie 8'
            ]
        }),
        stylelint({
            config: {
                rules: {
                    'declaration-no-important': true
                }
            }
        })
    ]
};
```

App.css：

```css
.container {
  display: grid;
}

.app {
  background: red !important;
}
```

App.js：

```javascript
import React from 'react';
import './App.css';

class App extends React.Component {
    render() {
        return (
            <div className="container">
                <div className="app">
                    这是App组件！
                </div>
            </div>
        );
    }
}

export default App;
```

执行`npm run dev`，进行验证：

![引用stylelint终端显示结果](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmStylelint.png)



### 6.3.3  CSSNext

PostCSS与CSSNext结合使用，在应用中使用最新的CSS语法特性。


```bash
npm install postcss-cssnext
```

    当前版本postcss-cssnext 3.1.0；



postcss.config.js：

```javascript
/* eslint-disable no-undef */
const autoprefixer = require('autoprefixer');
const stylelint = require('stylelint');
const postcssCssnext=require('postcss-cssnext');

module.exports = {
    plugins: [
        autoprefixer({
            grid: true,
            overrideBrowserslist: [
                '>1%',
                'last 3 versions',
                'android 4.2',
                'ie 8'
            ]
        }),
        stylelint({
            config: {
                rules: {
                    'declaration-no-important': true
                }
            }
        }),
        postcssCssnext({
            overrideBrowserslist: [
                '>1%',
                'last 2 versions',
            ]
        })
    ]
};
```

App.css：

```css
:root {
  --fontFamily: "Times New Roman";
}

.container {
  display: grid;
}

.app {
  background: red; /* !important; */
  font-family: var(--fontFamily);
}
```

执行`npm run dev`，验证结果：

![引用CSSNext](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/npmCSSNext.png)

<br />

备注：执行`npm run dev`，终端显示：

![cssnext与autoprefixer冲突](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/cssnextautoprefixer.png)

因此注释postcss.config.js中的autoprefixer配置，重新执行：

![webpack.config注释autoprefixer配置](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/annotationAutoprefixer.png)



## 6.4  CSS Modules

CSS Modules将CSS模块化：

- 每个CSS文件中的样式都拥有单独的作用域，不会和外界发生命名冲突；
- 对CSS进行依赖管理，可以通过相对路径引入CSS文件；
- 可以通过composes轻松复用其他CSS模块；



使用CSS ModulesCSS文件会导出一个对象：

```javascript
//style.css
.title{
    color:red;
}

//app.js
import styles from './style.css';
document.write('<h1 class=`${styles.title}`>my webpack app.</h1>');
```

<br />

（第一部分完）