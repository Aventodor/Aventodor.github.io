---
title: webpack4系列教程（一）：初识webpack
date: 2019-02-02 21:12:55
category:
    - 技术文章
    - webpack
tags:
---
# 1. 什么是webpack

先来看看[官网](https://www.webpackjs.com/concepts/)对webpack的介绍 ：

> 本质上，*webpack*是一个现代 JavaScript 应用程序的*静态模块打包器(module bundler)*。当 webpack 处理应用程序时，它会递归地构建一个*依赖关系图(dependency graph)*，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个*bundle*。

简单来说webpack就是一个JavaScript的打包器，将各种模块（module）打包成资源文件；还可以通过 Code Spliting 来把代码分离到不同的 bundle 中，然后可以按需加载或并行加载这些文件；webpack 可以使用 loader 来预处理文件，这允许你打包除了JavaScript 之外的任何静态资源。

[官网](https://www.webpackjs.com/concepts/)首页很清晰的展示了webpack的主要功能：

​

![image](http://upload-images.jianshu.io/upload_images/2012934-6beaaa872608b868?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到，一堆 modules 经过 webpack 打包处理成了各种静态资源。这就是 webpack

# 2. webpack核心概念

在开始学习 webpack 之前，你需要了解 webpack 的四个核心概念： 

- 入口（entry）
- 出口（output）
- loader
- 插件（plugins） 

## 2.1 入口（entry）

入口指示 webpack 应该使用哪个模块，来开始构建其内部依赖。进入入口后，webpack 会找出有哪些模块和库是与入口相依赖的。

我们可以在webpack配置中配置entry属性，来设置一个或多个入口起点。以下是一个简单的entry配置：
```javascript
const config = {
  entry: {
    main: 'path/to/your/entry/index.js'
  }
}
module.exports = config
```
## 2.2 出口（output）

 设置output是为了告诉webpack要在哪里输出其创建的bundle，并且可以对bundle命名。示例：

``` javascript
const path = require('path')
const config = {
  entry: {
    main: 'path/to/your/entry/index.js'
  }，  
  output: {
    filename:'[name].bundle.js',
    path: path.join(__dirname,'./dist')  
  }
}
module.exports = config

```

我们通过 output.filename 来设置输出bundle的文件名， output.path 来设置 bundle 的输出路径 

 >path 是 nodeJs 中的核心模块，用来处理项目中的路径。

## 2.3 loader

由于 webpack 只认识 JavaScript 代码，因此就需要借助其他方法来处理那些非 JavaScript 文件，如 css、image、font等。而 loader 可以将所有类型的文件处理成 webpack 能够识别的有效模块，然后再对其进行处理。

##### loader 中有两个重要属性：

1. test属性，用于标识出应该被对应的 loader 进行转换的某个或某些文件，通常是一个正则表达式；
2. use属性，表示进行转换时，应该使用哪个 loader；
```javascript
const path = require('path')
 
const config = {
  entry: {
    main: 'path/to/your/entry/index.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.join(__dirname, './dist')
  },
  module: {
    loaders: [
      {
        test: /\.ejs$/,
        use: ['ejs-loader']
      }
    ]
  }
}
 
module.exports = config
```

以上示例中loader的配置相当于告诉webpack在遇到 .ejs 的文件时，在打包之前先用 ejs-loader 装换一下。

## 2.4 插件（plugins）

loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。想要使用一个插件，你只需要require()它，然后把它添加到plugins数组中。多数插件可以通过选项(option)自定义。
```javascript
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin') // 通过 npm 安装
 
const config = {
  entry: {
    main: 'path/to/your/entry/index.js'
  },
  output: {
    filename: '[name].bundle.js',
    path: path.join(__dirname, './dist')
  },
  module: {
    loaders: [
      {
        test: /\.ejs$/,
        use: ['ejs-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: 'webpack demo',
      template: path.join(__dirname, './index.html')
    })
  ]
}
 
module.exports = config
```

HtmlWebpackPlugin 将为你生成一个 HTML5 文件， 其中包括使用script标签的 body 中的所有 webpack 包，webpack 提供提供了许多功能强大的插件，查阅[插件列表](https://www.webpackjs.com/plugins)获取更多插件的使用方法。