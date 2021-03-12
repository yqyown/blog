---
title: Webpack实战(2)-居玉皓著
date: 2021-03-12 16:12:48
tags: ['React','Webpack']
categories: 技术读物
---
基于《Webpack实战 入门、进阶与调优》学习Webpack，搭建学习demo，共两个部分，本文是第二部分。

<br />
<br />

# ---七、代码分片

代码分片可以有效降低首屏加载资源的大小，同时也会带来新的问题，如哪些模块进行分片、分片后的资源如何处理等。

- CommonsChunkPlugin：Webpack 4之前内部自带的插件
- optimization.SplitChunks：是Webpack 4为了改进CommonsChunkPlugin而重新设计和实现的代码分片特性

<br />

当前Webpack版本 5.23.0，配置optimization.SplitChunks：

```javascript
/* eslint-disable no-undef */
let ENV = process.env.ENV;
let isProd = ENV === 'production';

module.exports = {
    entry: './src/index.js',
    output: {
        filename: isProd ? 'bundle@[chunkhash].js' : '[name].js', //:'bundle.js',
    },
    mode: ENV,
    optimization: {
        splitChunks: {
            chunks: 'all'
        }
    },
    module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', 'css-loader', 'postcss-loader'],
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

执行`npm run dev`，查看打包结果：

![optimization.SplitChunks终端输出](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/SplitChunks.png)

<br />
<br />

# 八、生产环境配置

## 8.1  配置生产和开发环境

1）使用相同的配置文件：

package.json：

```json
  "scripts": {
    "build": "set ENV=production&& webpack", //‘&&’前面不要加空格，否则获取的值包含空格
    "dev": "set ENV=development&& webpack-dev-server",
  },
```



webpack.config.js：

```javascript
const ENV = process.env.ENV;
const isProd =  ENV === 'production';

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
        }]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    }
}
```

执行`npm run build`，即可查看dist目录生成的文件；


    许多框架和库都采用process.env.NODE_ENV作为一个区别开发环境和生产环境的变量。process.env是Node.js用于保存当前进程环境变量的对象；而NODE_ENV则可以让开发者指定当前的运行环境。

<br />

2）为不同的环境创建各自的配置文件：

生产环境创建一个webpack.production.config.js，开发环境创建一个叫webpack.development.config.js，然后修改package.json：

```json
  "scripts": {
    "build": "webpack --config=webpack.production.config.js",
    "dev": "webpack-dev-server --config=webpack.development.config.js",
  },
```

将webpack.production.config.js和webpack.development.config.js的公共的配置提取出来，单独创建一个webpack.common.config.js；

<br />

让另外两个JS分别引用webpack.common.config.js，并添加自身环境的配置；也可以使用webpack-merge，它是一个专门用来做Webpack配置合并的工具；


## 8.2  ---source map

source map指的是将编译、打包、压缩后的代码映射回源代码的过程。Webpack打包压缩后的代码基本不具备可读性，若代码抛出一个错误，有了source map，再加上浏览器调试工具（devtools），就非常容易了。

<br />

JavaScript的source map配置很简单，只要在webpack.config.js中添加devtool即可：

```javascript
module.exports = {
    // ...
  devtool:'source-map',
}
```

App.js：

```javascript
import React from 'react';
import './App.css';

class App extends React.Component {
    render() {
        console.log(123);
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

执行`npm run dev`，F12打开chrome Dev Tool：

![引用source map开发者显示结果1](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/usesourcemap1.png)



![引用source map开发者显示结果2](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/usesourcemap2.png)



对于CSS、SCSS、Less来说，则需要添加额外的source map配置项：

```javascript
module.exports = {
    // ...
  module: {
        rules: [{
            test: /\.css$/,
            use: ['style-loader', {
                loader: 'css-loader',
                options: {
                    sourceMap: true,
                }
            }, 'postcss-loader'],
            exclude: /node_modules/, //node_modules中的模块不会执行这条规则 #/src\/pages/
        }]
  }
}
```


Webpack支持多种source map的形式。除了配置为devtool：'source-map'以外，还可以根据不同的需求选择cheap-source-map、eval-source-map等。

<br />

有了source map意味着任何人通过浏览器的开发者工具都可以看到工程源码，存在极大的安全性。如何保持功能的同时，防止暴露源码给用户呢？Webpack提供了hidden-source-map和nosources-source-map两种策略来提升source map的安全性。

<br />

hidden-source-map意味着Webpack仍然会产生完整的map文件，只不过不会在bundle文件中添加对于map文件的引用。若我们想要追溯源码，则需要利用一些第三方服务，将map文件上传到上面。目前流行的解决方案是Sentry。

<br />

nosources-source-map对于安全保护没那么强，但是使用方式相对简单，可以在浏览器开发者工具的sources选项卡中看到源码的目录结构，但是文件的具体内容会被隐藏起来。

<br />

在这些配置之外，我们还可以正常打包出source map，然后通过服务器的nginx设置将.map文件只对固定的白名单（比如公司内网）开放，这样我们仍然能看到源码，而一般用户的浏览器中无法获取。



## 8.3  资源压缩

### 8.3.1  压缩JS

Webpack 3开启压缩需调用webpack.optimize.UglifyJsPlugin；

<br />

Webpack 4配置移到config.optimization.minimize：

```javascript
module.exports = {
    // ...
      optimization: {
        minimize: true,
    },
}
```

terser-webpack-plugin插件配置。



### 8.3.2  压缩CSS

压缩CSS文件的前提是使用extract-text-webpack-plugin或mini-css-extract-plugin将样式提取出来，接着使用optimize-css-assets-webpack-plugin来进行压缩，这个插件本质上使用的是压缩器cssnano。


### 8.3.3  缓存

1）资源hash

一个常用的方法是在每次打包的过程中对资源的内容计算一次hash，并作为版本号存放在文件名中，如bundle\@2e0a691e769edb228e2.js。bundle是文件本身的名字，@后面跟的是文件内容hash值，每当代码发生变化时相应的hash也会变化。

<br />

2）输出动态HTML

资源名的改变意味着HTML中的引用路径的改变。每次更改后都要手动去维护很麻烦，理想的情况是打包结束后自动把最新的资源名同步过去。使用html-webpack-plugin插件：

```bash
npm install html-webpack-plugin --save-dev
```

    当前版本html-webpack-plugin 5.2.0；


webpack.config.js：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    // ...
    plugins: [
        new HtmlWebpackPlugin({
            template: './public/index.html'
        })
    ]
}
```

注释index.html中引用bundle.js的script标签；


执行`npm run dev`，验证结果；

<br />

3）使chunk id更稳定

理想状态下，对于缓存的应用是尽量让用户在启动时只更新代码变化的部分，而对没有变化的部分使用缓存。


之前我们介绍过使用CommonsChunkPlugin和SplitChunksPlugin来划分代码。通过它们来尽可能地将一些不常变动的代码单独提取出来，与经常迭代的业务代码区别开，这些资源就可以在客户端一直使用缓存。

<br />

4）bundle体积监控和分析

为了保证良好的用户体验，我们可以对打包输出的bundle体积进行持续的监控，以防止不必要的冗余模块被添加进来。


VS Code中有一个插件Import Cost可以帮助我们对引入模块的大小进行实时监测。每当我们在代码中引入一个新的模块（主要是node_modules中的模块）时，它都会为我们计算该模块压缩后及gzip过后将占多大体积：

![Import Cost插件](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/ImportCostPlugin.png)

当我们发现某些包过大时就可以采取一些措施，比如寻找一些更小的替代方案或只引用其中的某些子模块。


另外一个很有用的工具是webpack-bundle-analyzer，它能够帮助我们分析一个bundle的构成。



```bash
npm install webpack-bundle-analyzer --save-dev
```

    当前版本webpack-bundle-analyzer 4.4.0；



webpack.config.js：

```javascript
const Analyzer = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
    // ...
  plugins: [
     new Analyzer()
  ]
}
```

执行`npm run dev`，浏览器打开http:\//127.0.0.1:8888/，查看生成的bundle的模块组成结构图，每个模块所占的体积一目了然：

![bundle的模块组成结构图](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/bundleModuleImg.png)

最后我们还需要自动化地对资源体积进行监控，bundlesize这个工具包可以做到这一点。

```bash
npm install bundlesize --save-dev
```

    当前版本bundlesize 0.18.1；



package.json：

```json
{
    // ...
      "bundlesize": [
        {
            "path": "./dist/bundle.js",
            "maxSize": "50 kB"
        }
    ],
    // ... 
    "scripts": {
        "test:size": "bundlesize",
    },
    // ...
}
```

执行`npm run build`，dist文件夹生成bundle.js文件，再执行`npm run test:size`：

![验证bundle大小](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/testsize.png)


<br />
<br />


# 九、打包优化

主要介绍一些优化Webpack配置的方法，目的是让打包的速度更快，输出的资源更小。首先重述一条软件工程领域的经验——不要过早优化，在项目的初期不要看到任何优化点就拿来加到项目中，这样不但增加了复杂度，优化的效果也不会太理想。一般是当项目发展到一定规模后，性能问题随之而来，这时再去分析然后对症下药，才有可能达到理想的优化效果。

<br />

## 9.1  HappyPack

HappyPack是一个通过多线程来提升Webpack打包速度的工具。

<br />

## 9.2  缩小打包作用域

### 9.2.1  exclude和include

<br />

### 9.2.2  noParse

有些库我们是希望Webpack完全不要去进行解析的，即不希望应用任何loader规则。



```javascript
module.exports={
    // ...
  module:{
    noParse:/lodash/,
  }
}
```

上面的配置将会忽略所有文件名中包含lodash的模块，这些模块仍然会被打包进资源文件，只不过Webpack不会对其进行任何解析。

<br />

在Webpack 3及之后的版本还支持完整的路径匹配：

```javascript
module.exports={
    // ...
  module:{
    noParse:function(fullPath){
        // fullPath是绝对路径，如：/users/me/app/webpack-no-parse/lib/lodash.js
      return /lib/.test(fullPath);
    },
  }
}
```

上面的配置将会忽略所有lib目录下的资源解析。


### 9.2.3  IgnorePlugin

插件IgnorePlugin完全排除一些模块，被排除的模块即便被引用也不会被打包进资源文件中。


如，Moment.js是一个日期时间处理相关的库，为了做本地化它会加载很多语言包，对于我们来说一般用不到其他地区的语言包，但是它们会占很多体积，这时就可以用IgnorePlugin来去掉。


webpack.config.js：

```javascript
plugins:[
    new webpack.IgnorePlugin({
    resourceRegExp:/^\.\/locale$/, //匹配资源文件
    contextRegExp:/moment$/, //匹配检索目录
  })
]
```

### 9.2.4  Cache

有些loader会有一个cache配置项，用来在编译代码后同时保存一份缓存，在执行下一次编译前会先检查源码文件是否有变化，如果没有就直接采用缓存，也就是上次编译的结果。这样相当于实际编译的只有变化了的文件，整体速度会有一定提升。

<br />

在Webpack 5中添加了一个新的配置项“cache: { type: "filesystem" }”，它会在全局启用一个文件缓存。该特性仅仅在实验阶段，并且无法自动检测到缓存已经过期。


## 9.3  动态链接库与DllPlugin

动态链接库是早期Windows系统由于受限于当时计算机内存空间较小的问题而出现的一种内存优化方法。当一段相同的子程序被多个程序调用时，为了减少内存消耗，可以将这段子程序存储为一个可执行文件，当被多个程序调用时只在内存中生成和使用同一个实例。

<br />

DllPlugin借鉴了动态链接库的这种思路，将vendor完全拆出来，有自己的一整套Webpack配置并独立打包，在实际工程构建时就不用再对它进行任何处理，直接取用即可。


## 9.4  tree shaking

ES6 Module依赖关系的构建是在代码编译时而非运行时。基于这项特性Webpack提供了tree shaking功能，它可以在打包过程中帮助我们检测工程中没有被引用过的模块，这部分代码将永远无法被执行到，因此也被称为“死代码”。Webpack会对这部分代码进行标记，并在资源压缩时将它们从最终的bundle中去掉。

<br />

tree shaking只能对ES6 Module生效。

<br />

如果我们在工程中使用了babel-loader，那么一定要通过配置来禁用它的模块依赖解析。因为如果由babel-loader来做依赖解析，Webpack接收到的都是转化过的CommonJS形式的模块，无法进行tree shaking。


## 9.5  使用压缩工具去除死代码

tree shaking本身只是为死代码添加标记，真正去除死代码是通过压缩工具来进行的。使用terser-webpack-plugin即可。在Webpack 4之后的版本中，将mode设置为production也可以达到相同的效果。

<br />
<br />

# 十、开发环境调优

## 10.1  Webpack开发效率插件

### 10.1.1  webpack-dashboard

webpack-dashboard用来更好地展示每次构建结束后在控制台输出的一些打包相关的信息。


```bash
npm install webpack-dashboard --save-dev
```

    当前版本webpack-dashboard 3.3.1；



webpack.config.js：

```javascript
const DashboardPlugin=require('webpack-dashboard/plugin');

module.exports = {
  // ...
  plugins: [
    new DashboardPlugin(),
  ]
}
```

package.json：

```json
"scripts": {
   //"dev": "set ENV=development&& webpack-dev-server",
   "dev": "set ENV=development&& webpack-dashboard -- webpack-dev-server",
},
```



### 10.1.2  webpack-merge

假设我们的项目有3种不同的配置，分别对应本地环境、测试环境和生产环境。每一个环境对应的配置都不同，但也有一些公共的部分，那么将公共的部分提取出来，创建一个webpack.common.config.js：

```javascript
/* eslint-disable indent */
/* eslint-disable no-undef */
const HtmlWebpackPlugin = require('html-webpack-plugin');
const Analyzer = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const DashboardPlugin=require('webpack-dashboard/plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
    },
    devtool: 'source-map',
    optimization: {
        minimize: true,
    },
    module: {
        rules: [{
            test: /\.css$/,
                use: ['style-loader', {
                    loader: 'css-loader',
                    options: {
                        sourceMap: true,
                    }
                }, 'postcss-loader'],
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
                        presets: [
                            [
                                '@babel/env', {
                                    modules: false, //禁用模块语句的转化, 将ES6 Module的语法交给Webpack本身处理
                                }
                            ],
                            [
                                '@babel/preset-react', {
                                    modules: false,
                                }
                            ]
                        ],
                    }
                },
                exclude: /node_modules/
            },
            {
                test: /\.js$/,
                use: 'eslint-loader',
                enforce: 'pre',
                exclude: /node_modules/,
            }
        ]
    },
    devServer: {
        // publicPath: '/', //用于确定bundle的来源，并具有优先级高于contentBase
        contentBase: './public', //页面打开的url是以devServer中的contentBase作为当前查询目录，只要文档不在contentBase所指定的目录中，就只会显示cannot get
        hot: true
    },
    plugins: [
        ////输出动态HTML
        new HtmlWebpackPlugin({
            template: './public/index.html'
        }),
        ////生成bundle模块组成结构图
        new Analyzer(),
        ////更好地展示打包信息
        new DashboardPlugin(),
    ]
}
```

每一个环境都有一个相应的配置文件，对于生产环境可以专门创建一个webpack.prod.config.js：

```javascript
/* eslint-disable no-undef */
const commonConfig = require('./webpack.common.config.js');

module.exports = Object.assign(commonConfig, {
    mode: 'production'
});
```

现在本地测试使用webpack-dev-server，file-loader的publicPath设置为根目录“/”，而执行`npm run build`打包的图片保存在dist文件夹下，因此需要在webpack.prod.js中重新设置。

<br />

如果通过Object.assign无法准确找到CSS的规则并进行替换，所以必须替换整个module的配置。

<br />

但我们可以通过webpack-merge解决这个问题。它在合并module.rules的过程中会以test属性作为标识符，当发现有相同项出现的时候会以后面的规则覆盖前面的规则，这样就不必添加冗余代码了。



```bash
npm install webpack-merge --save-dev
```

    当前版本webpack-merge 5.7.3；



webpack.prod.config.js：

```javascript
/* eslint-disable no-undef */
const commonConfig = require('./webpack.common.config.js');
const { merge }=require('webpack-merge');

module.exports = merge(commonConfig, {
    mode: 'production',
    module:{
        rules:[
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: {
                    loader: 'file-loader',
                    options: {
                        name: '[name].[ext]',
                        publicPath: '/dist/',
                    }
                }
            }
        ]
    }
});
```

webpack.dev.config.js：

```javascript
/* eslint-disable no-undef */
const commonConfig = require('./webpack.common.config.js');
// webpack-merge v5 (and later)
const { merge }=require('webpack-merge');
// webpack-merge v4 (and earlier)
//const merge = require('webpack-merge');

module.exports = merge(commonConfig, {
    mode: 'development',
    module:{
        rules:[
            {
                test: /\.(png|svg|jpg|gif)$/,
                use: {
                    loader: 'file-loader',
                    options: {
                        name: '[name].[ext]',
                        publicPath: '/',
                    }
                }
            }
        ]
    }
});
```

webpack.common.config.js：

```javascript
module.exports = {
  // ...
  module:{
    rules:[{
                ////配置在开发和生产环境各自的配置文件中(注释以下内容)
                    //     test: /\.(png|svg|jpg|gif)$/,
            //     use: {
            //         loader: 'file-loader',
            //         options: {
            //             name: '[name].[ext]',
            //             publicPath: '/dist/',
            //         }
            //     }
    }]
  }
};
```

package.json：

```json
    "scripts": {
        "build": "webpack --config=webpack.prod.config.js",
        "dev": "webpack-dev-server --config=webpack.dev.config.js",
    },
```

分别执行`npm run dev`或`npm run build`，进行验证；



### 10.1.3  speed-measure-webpack-plugin

觉得Webpack构建很慢但又不清楚如何优化？可以试试speed-measure-webpack-plugin插件（简称SMP）。SMP可以分析出Webpack整个打包过程中在各个loader和plugin上耗费的时间，这将会有助于找出构建过程中的性能瓶颈。



### 10.1.4  size-plugin

size-plugin这个插件会帮助我们监控资源体积的变化，会输出本次构建的资源体积，以及与上次构建相比体积变化了多少。

<br />

## 10.2  模块热替换HMR

一些Web开发框架和工具只要检测到代码改动就会自动重新构建，然后触发网页刷新，这种一般称为live reload。

<br />

Webpack则在live reload的基础上又进一步，可以让代码在网页不刷新的前提下得到最新的改动，甚至不需要重新发起请求就能看到更新后的效果，这就是模块热替换功能（Hot Module Replacement，HMR）。


### 10.2.1  开启HMR

HMR是需要手动开启的，并且有一些必要条件。

<br />

首先我们要确保项目是基于webpack-dev-server或webpack-dev-middle进行开发的，Webpack本身的命令行并不支持HMR。

<br />

调用HMR API有两种方式，一种是手动添加这部分代码；另一种是借助一些现成的工具，如react-hot-loader、vue-loader等。

<br />

===》一种是，手动添加代码：

```javascript
import { add } from 'util.js';
add(2,3);

if(module.hot){
    module.hot.accept();
}
```

<br />

===》另一种是，第三方提供的HMR，react组件的热更新由react-hot-loader处理

<br />

### 10.2.2  HMR原理     

> 当你对代码修改并保存后，webpack将会对代码进行重新打包，并将改动的模块发送到浏览器端，浏览器用新的模块替换掉旧的模块，去实现局部更新页面而非整体刷新页面。

<br />

在本地开发环境下，浏览器是客户端，webpack-dev-server（WDS）相当于我们的服务端。HMR的核心就是客户端从服务端拉取更新后的资源（准确地说，HMR拉取的不是整个资源文件，而是chunk diff，即chunk需要更新的部分）

<br />

第一步，浏览器什么时候去拉取这些更新，这需要WDS对本地源文件进行监听。实际上WDS与浏览器之间维护了一个websocket，当本地资源发生变化时WDS会向浏览器推送更新事件，并带上这次构建的hash，让客户端与上一次资源进行比对。通过hash的比对可以防止冗余更新的出现。

<br />

下一步，要知道拉取什么。在刚刚的websocket中，客户端已经知道新的构建结果和当前有差别，就会向WDS发起一个请求（这个请求名字为[hash].hot-update）来获取更新文件的列表，即哪些模块有改动。


<br />
<br />

# 十一、其他JavaScript打包工具

## 11.1  Rollup

如果当前的项目需求仅仅是打包Javascript，Rollup是我们的第一选择。

<br />

## 11.2  Parcel



<br />
<br />


# 十二、(附加)自动清除上一次打包的文件


```bash
npm install clean-webpack-plugin --save-dev
```

    当前版本clean-webpack-plugin 3.0.0；



webpack.common.config.js：

```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    // ...
  plugins: [
    new CleanWebpackPlugin(),
  ]
}
```

执行`npm run build`，验证结果，出现以下两个问题：

<br />

Q1：clean-webpack-plugin: options.output.path not defined. Plugin disabled...打包清除不了之前的文件

```javascript
// webpack.common.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    // ...
  output: {
        filename: 'bundle.js',
        path: path.resolve(process.cwd(), 'dist'), //必须配置这行代码，否则无法删除/dist/目录中的文件(写法不止一种)
  },
  plugins: [
    new CleanWebpackPlugin(),
  ]
}
```

<br />

Q2：Error: EPERM: operation not permitted, lstat 'D:\LearnCode\learn_react_webpack\dist\images\hexo-deploy.png'

```javascript
// webpack.common.config.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    // ...
  output: {
        filename: 'bundle.js',
        path: path.resolve(process.cwd(), 'dist'), //必须配置这行代码，否则无法删除/dist/目录中的文件(写法不止一种)
  },
  plugins: [
    new CleanWebpackPlugin({
        cleanOnceBeforeBuildPatterns: ['./dist/*'] //配置./dist/*或./dist/images/*,测试无差别(20210302)
    }),
  ]
}
```

<br />
<br />

# 十三、代码备份

github链接地址：https://github.com/YuliaScott/learn_react_webpack

```bash
下载压缩包或克隆项目
解压
执行 npm install（安装项目包文件）
执行 npm run dev 或 npm run build
```
<br />

（第二部分完）