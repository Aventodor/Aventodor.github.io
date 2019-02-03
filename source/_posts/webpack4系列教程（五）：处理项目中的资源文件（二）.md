---
title: webpack4系列教程（五）：处理项目中的资源文件（二）
category:
  - 学习笔记
  - webpack
abbrlink: ff9651be
date: 2019-02-02 21:23:03
---
# 1\. 在项目中使用 less 

在 src/assets/ 下新建 common.less ：

```
body{
  background: #fafafa;
  padding: 20px;
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-9020ad076d645408.gif?imageMogr2/auto-orient/strip) 

在 main.js 中引入 common.less ：

```
import './assets/style/common.less'
```

![](https://upload-images.jianshu.io/upload_images/2012934-07a809ec681f5cb9.gif?imageMogr2/auto-orient/strip) 

安装 less-loader：

```
npm i less-loader -D
```

![](https://upload-images.jianshu.io/upload_images/2012934-0d4314118bfca19e.gif?imageMogr2/auto-orient/strip) 

添加 rules：

```
     {
        test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          'less-loader'
        ]
      }
```

![](https://upload-images.jianshu.io/upload_images/2012934-d88bf39ae95cfa8b.gif?imageMogr2/auto-orient/strip) 

打包之后，在浏览器打开 dist/index.html，less文件中的样式已经通过 style 标签载入了：

![](http://upload-images.jianshu.io/upload_images/2012934-194b94439848478d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-e768c5d87d3c8956.gif?imageMogr2/auto-orient/strip) ​

# 2\. 使用MiniCssExtractPlugin

我们之前的样式代码都是通过 style 标签载入的，那么如何通过 link 引入CSS文件的方式实现呢？

这就需要使用一个插件，在webpack3中通常使用ExtractTextWebpackPlugin，但是在webpack4中已经不再支持ExtractTextWebpackPlugin的正式版，而测试版本又不够稳定，因此我们使用MiniCssExtractPlugin替代。首先安装：

```
npm install --save-dev mini-css-extract-plugin
```

![](https://upload-images.jianshu.io/upload_images/2012934-e9104b2ec3972890.gif?imageMogr2/auto-orient/strip) 

在webpack.config.js 中引入并添加 plugins ：

```
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
```

![](https://upload-images.jianshu.io/upload_images/2012934-ed26cea1f923a356.gif?imageMogr2/auto-orient/strip) 

```
new MiniCssExtractPlugin({
      filename: "[name].css"
    }),
```

![](https://upload-images.jianshu.io/upload_images/2012934-47f6a946b4f69719.gif?imageMogr2/auto-orient/strip) 

 修改 CSS 和 less 的 rules：

```
{
        test: /.css$/,
        use: [
          // 'style-loader',
          {
            loader: MiniCssExtractPlugin.loader
          },
          'css-loader'
        ]
      },
      {
        test: /.less$/,
        use: [
          // 'style-loader',
          {
            loader: MiniCssExtractPlugin.loader
          },
          'css-loader',
          'less-loader'
        ]
      }
```

![](https://upload-images.jianshu.io/upload_images/2012934-c7668816d7cd9001.gif?imageMogr2/auto-orient/strip) 

npm run build 之后，可见head中引入了一个 main.css 文件：

![](http://upload-images.jianshu.io/upload_images/2012934-ed039ee7ee623a49.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-a3b25b2ed841fce2.gif?imageMogr2/auto-orient/strip) ​

也正是我们在 common.less 和 modal.css 中的代码

![](http://upload-images.jianshu.io/upload_images/2012934-42b68e2aabc38723?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-48284d76011eaa5c.gif?imageMogr2/auto-orient/strip) ​

# 3\. postcss-loader

postcss-loader 可以帮助我们处理CSS，如自动添加浏览器前缀。

```
npm i -D postcss-loader autoprefixer
```

![](https://upload-images.jianshu.io/upload_images/2012934-07342baa1a4454e3.gif?imageMogr2/auto-orient/strip) 

在根目录下创建 postcss.config.js：

```
const autoprefixer = require('autoprefixer')

module.exports = {
  plugins: [
    autoprefixer({
      browsers: ['last 5 version']
    })
  ]
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-ee6337518867253b.gif?imageMogr2/auto-orient/strip) 

修改 css 和 less 的 rules：

```
{
        test: /\.css$/,
        use: [
          // 'style-loader',
          {
            loader: MiniCssExtractPlugin.loader
          },
          { loader: 'css-loader', options: { importLoaders: 1 } },
          'postcss-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          // 'style-loader',
          {
            loader: MiniCssExtractPlugin.loader
          },
          'css-loader',
          'postcss-loader',
          'less-loader'
        ]
      }
```

![](https://upload-images.jianshu.io/upload_images/2012934-e469db73dc41d618.gif?imageMogr2/auto-orient/strip) 

在 modal.css中加入：

```
.flex{
    display: flex;
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-afb154591decd67b.gif?imageMogr2/auto-orient/strip) 

打包之后打开 main.css，可见浏览器前缀已经加上了：

![](http://upload-images.jianshu.io/upload_images/2012934-041d73e7df49eb94.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-d9572fa1ae4660f1.gif?imageMogr2/auto-orient/strip) ​

