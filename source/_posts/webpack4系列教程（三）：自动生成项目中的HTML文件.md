---
title: webpack4系列教程（三）：自动生成项目中的HTML文件
category:
  - 学习笔记
  - webpack
abbrlink: 6ead29be
date: 2019-02-02 21:20:44
tags: webpack
---
# 1. webpack中的CommonJS和ES Mudule 规范

## 1.1 CommonJs规范

CommonJs规范的出发点：JS没有模块系统、标准库较少、缺乏包管理工具；为了让JS可以在任何地方运行，以达到Java、PHP这些后台语言具备开发大型应用的能力。

在CommonJs规范中：

*   一个文件就是一个模块，拥有单独的作用域；
*   普通方式定义的变量、函数、对象都属于该模块内；
*   通过require来加载模块；
*   通过exports和modul.exports来暴露模块中的内容；

## 1.2 ES Mudule 规范

ES6在语言标准的层面上，实现了模块功能，基本特点如下：

*   每一个模块只加载一次， 每一个JS只执行一次， 如果下次再去加载同目录下同文件，直接从内存中读取；
*   每一个模块内声明的变量都是局部变量， 不会污染全局作用域；
*   模块内部的变量或者函数可以通过export导出；
*   一个模块可以导入别的模块；

模块功能主要由两个命令构成：export和import；export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能：

```
// esm.js
let firstName = 'Jack';
let lastName = 'Wang';

export {firstName, lastName}

// export命令除了输出变量，还可以输出函数
export function (a, b) {
  return a + b
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-7e8c6b23486d13a6.gif?imageMogr2/auto-orient/strip) 

使用export命令定义了模块的对外接口以后，其他 JS 文件就可以通过import命令加载这个模块，import命令接受一对大括号，里面指定要从其他模块导入的变量名，大括号里面的变量名，必须与被导入模块对外接口的名称相同。

```
// main.js
import {firstName, lastName} from './esm';

function say() {
  console.log('Hello , ' + firstName + ' ' + lastName)
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-52c7796bbd37e56f.gif?imageMogr2/auto-orient/strip) 

## 1.3 使用

现在，在src目录下新建 sum.js 和 minus.js

```
// sum.js ES Mudule 规范
// export default命令，为模块指定默认输出
export default function (a, b) {
  return a + b
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-af483dbef298f5fd.gif?imageMogr2/auto-orient/strip) 

```
// minus.js commonJS 规范

module.exports = function (a, b) {
  return a - b
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-4af3c2e670cf4290.gif?imageMogr2/auto-orient/strip) 

 修改 main.js 

```
import sum from './sum'
import minus from './minus'

console.log('sum(1, 2): ' + sum(1, 2))
console.log('minus(5, 2): ' + minus(5, 2))
```

![](https://upload-images.jianshu.io/upload_images/2012934-6dc4db91d2f8bf19.gif?imageMogr2/auto-orient/strip) 

执行 npm run build 之后，打开 index.html，在控制台中可以看到输出的结果。

![](http://upload-images.jianshu.io/upload_images/2012934-ea811a00ef59c294.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-8beaa5029e435a83.gif?imageMogr2/auto-orient/strip) ​

# 2. 自动生成项目中的HTML文件

在[前文](https://blog.csdn.net/qq_38286992/article/details/85538205)中我们为了演示打包好的 main.bundle.js ，在根目录下创建了一个 index.html ，并引入main.bundle.js。而在实际项目中，我们可以通过 webpack 的一个插件：[HtmlWebpackPlugin](https://www.webpackjs.com/plugins/html-webpack-plugin/) 来自动生成HTML文件并引入我们打包好的JS和CSS文件。

###  安装：

```
npm install --save-dev html-webpack-plugin
```

![](https://upload-images.jianshu.io/upload_images/2012934-1d9767cd9d4e3467.gif?imageMogr2/auto-orient/strip) 

 整理项目目录：

在根目录创建config文件夹，把webpack.config.js移入config，并修改webpack.config.js：

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

const config = {
  mode: 'none',
  entry: {
    main: path.join(__dirname, '../src/main.js')
  },
  output: {
    filename: '[name].bundle.js',
    path: path.join(__dirname, '../dist')
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.join(__dirname, '../index.html'),
      inject: true,
      minify: {
        removeComments: true
      }
    })
  ]
}

module.exports = config
```

![](https://upload-images.jianshu.io/upload_images/2012934-05953bae234d8a19.gif?imageMogr2/auto-orient/strip) 

*   template：模版文件的路径，这里使用根目录下的index.html文件；
*   inject：设为 true 表示把JS文件注入到body结尾，CSS文件注入到head中；
*   minify：removeComments: true 表示删除模版文件中的注释，minify还有很多配置可选请[自行参阅](https://github.com/jantimon/html-webpack-plugin#minification)；

下一步注释掉index.html 中我们手动引入的 script ：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="renderer" content="webkit"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
    <title>Title</title>
</head>
<body>

<!-- <script src="dist/main.bundle.js"></script> -->
</body>
</html>
```

![](https://upload-images.jianshu.io/upload_images/2012934-a7b707b7dd77631a.gif?imageMogr2/auto-orient/strip) 

执行 npm run build ，可以看到，dist 目录下多了一个 index.html，这就是通过 [HtmlWebpackPlugin ](https://www.webpackjs.com/plugins/html-webpack-plugin/)生成的文件，打开dist/index.html，已经自动引入了 main.bundle.js并且注释已被删除。

![](http://upload-images.jianshu.io/upload_images/2012934-85f88ef28d2e5c57?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-7a25248813e0df35.gif?imageMogr2/auto-orient/strip) ​

至此，我们已经成功实现自动生成项目中的HTML文件了。

# 3\. 清理/dist文件夹

每次执行npm run build 打包时，都会有上次的代码遗留下来，导致我们的 `/dist` 文件夹相当杂乱。通常，在每次构建前清理 `/dist` 文件夹，是比较推荐的做法。

[`clean-webpack-plugin`](https://www.npmjs.com/package/clean-webpack-plugin) 是一个比较普及的管理插件，让我们安装和配置下：

```
npm install clean-webpack-plugin --save-dev
```

![](https://upload-images.jianshu.io/upload_images/2012934-d0e67d3b2b5f260d.gif?imageMogr2/auto-orient/strip) 

在webpack.config.js 中使用：

```
const CleanWebpackPlugin = require('clean-webpack-plugin')
```

![](https://upload-images.jianshu.io/upload_images/2012934-b34f9e9bca564aba.gif?imageMogr2/auto-orient/strip) 

在 plugins 中加入：

```
new CleanWebpackPlugin(['dist'],{root: path.join(__dirname, '../')})
```

![](https://upload-images.jianshu.io/upload_images/2012934-2066640c0fbc2423.gif?imageMogr2/auto-orient/strip) 

第一个参数表示文件夹路径数组；第二个参数是 options 配置项，root 为到webpack根文件夹的绝对路径，默认为 __dirname，由于dist文件夹和webpack.config.js不再相同目录下，因此我们需要重新定义 root 路径，以免无法找到 dist 文件夹。

执行 npm run build ，在命令行中可见：

![](http://upload-images.jianshu.io/upload_images/2012934-1768f1081a850626.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-2044bf0d1b4ea8ec.gif?imageMogr2/auto-orient/strip) ​

dist 文件夹已被删除了。
