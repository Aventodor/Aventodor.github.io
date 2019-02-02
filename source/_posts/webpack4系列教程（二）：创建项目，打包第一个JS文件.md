---
title: webpack4系列教程（二）：创建项目，打包第一个JS文件
category:
  - 技术文章
  - webpack
abbrlink: 31e86db9
date: 2019-02-02 21:19:38
tags:
---
# 1. 创建项目

## 1.1 初始化一个项目

首先安装nodejs，打开 [nodeJs官网](https://nodejs.org/en/) 直接下载安装即可，安装完毕后打开命令行工具，进入你的项目文件夹，执行

npm init 进行项目的初始化：

![](http://upload-images.jianshu.io/upload_images/2012934-731bdbebf2b3af20?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


过程中会让你填写项目名、版本、描述、仓库地址、关键字等信息，可以不填一路回车，执行完毕后会在根目录下创建一个 package.json 文件，这样就初始化结束了。

## 1.2 安装webpack

由于在webpack4中已经不再默认安装 webpacl-cli，所以我们要手动安装，在命令行执行 npm i webpack webpack-cli -D 即可。对于大多数项目，建议本地安装。这可以使我们在引入破坏式变更(breaking change)的依赖时，更容易分别升级项目。

# 2\. 打包第一个JS文件 

首先，我们在根目录下创建一个 webpack.config.js 文件和一个src文件夹。然后在src中创建一个 main.js 文件，如下：

![](http://upload-images.jianshu.io/upload_images/2012934-9f53d80a28b4009f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 main.js 中写一行 

``` javascript
alert('hello world')
```

![](https://upload-images.jianshu.io/upload_images/2012934-2735c29e440e6834.gif?imageMogr2/auto-orient/strip) 

然后打开 webpack.config.js ，进行webpack的配置：

``` javascript
const path = require('path')

let config = {
  mode: 'none',
  entry: {
    main: path.join(__dirname, './src/main.js')
  },
  output: {
    filename: '[name].bundle.js',
    path: path.join(__dirname, './dist')
  }
}

module.exports = config
```

![](https://upload-images.jianshu.io/upload_images/2012934-0fd3a489b31c07da.gif?imageMogr2/auto-orient/strip) 

我们设置了一个名为 main 的入口，并以 src 下的 main.js 作为入口文件，然后输出到根目录下的 dist 文件夹中。

> 在webpack4中，我们需要设置 mode 属性，用来决定当前是development还是production环境，webpack会根据此值来进行一些默认操作，两种环境的不同配置后面的博文会详解，这里我们设置为 'none' ，来避免默认操作。前文已经说过，path 是 nodeJs中的核心模块用来操作路径，__dirname 表示文件的当前路径（此时为根路径）。而 output中的filename属性，[name] 表示入口的名称，此处就是 main。

接下来打开 package.json 文件，来编写一条命令执行webpack的打包。在 script 中添加：

``` javascript
"build": "webpack --config webpack.config.js --progress --colors"
```

![](https://upload-images.jianshu.io/upload_images/2012934-2dc5f57ea93dde5b.gif?imageMogr2/auto-orient/strip) 

webpack --config path/to/your/file/file.js 表示执行某个配置文件，--progress可以让我们看到打包的进度 ， --colors 开启命令行颜色显示，更多的webpack命令参数大家可以另行查阅。

![](http://upload-images.jianshu.io/upload_images/2012934-12ab16fb570d42f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



然后就可以在命令行执行：npm run build，执行完毕后，我们可以看到，在根目录下多了一个 dist 文件夹 并有一个 main.bundle.js文件，这就是webpack为我们打包出来的静态资源，而文件路径就是我们在 output 中设置的值。

为了演示打包好的 main.bundle.js ，我们在根目录下创建一个 index.html ，并引入main.bundle.js

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script src="dist/main.bundle.js"></script>
</body>
</html>
```


在浏览器中打开 index.html，可见main.js中的代码已经被执行了：

![](http://upload-images.jianshu.io/upload_images/2012934-6d606afa4a6ea576?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



在IDE中打开main.bundle.js，代码的最底部可以看到我们在main.js中写的代码。

![](http://upload-images.jianshu.io/upload_images/2012934-0f1c4b5fa1da29e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，我们的第一次 webpack 打包就成功了。
