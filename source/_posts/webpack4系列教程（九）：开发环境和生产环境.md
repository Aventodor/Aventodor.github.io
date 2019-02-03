---
title: webpack4系列教程（九）：开发环境和生产环境
category:
  - 学习笔记
  - webpack
abbrlink: 8d0369c8
date: 2019-02-02 21:26:25
tags: webpack
---
# 1. 构建开发环境
如果你一直跟随我前面的博文，那么你对webpack的基础知识已经有比较深刻的理解了。之前，我们一直执行着：
```bash
npm run build
```
来打包编译输出我们的代码，本文我们来看看如何构建一个开发环境，来使我们的开发变得方便些。
## 1.1 webpack-dev-server
webpack-dev-server是一个简单的小型的web服务器，并且能够实时重载，配置也很简单，首先安装：
```bash
npm install --save-dev webpack-dev-server
```
配置webpack.config.js:
```javascript
devServer: {
    port: 8080,  // 端口号
    host: '0.0.0.0', // 主机名，设为该值可通过IP访问
    overlay: {
      errors: true // 错误提示
    }
  }
```
在package.json中添加命令：
```bash
    "dev": "cross-env NODE_ENV=development webpack-dev-server --config config/webpack.config.js"
```
执行：
```bash
npm run dev
```
可见我们的服务已经跑起来了：
![Image 1.png](https://upload-images.jianshu.io/upload_images/2012934-2964c9a355cbd06d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


  

## 1.2 source-map
在webpack打包源码时，我们会很难找到错误的出现位置，比如将源文件 sum.js、minus.js打包到bundle.js中，其中一个源文件出现了错误，仅仅会追踪到bundle.js中，这对我们来说并不理想。因此为了更加便捷的找到错误的原始位置，JavaScript为我们提供了 source-map的功能，将编译后的代码映射回原始源代码。如果一个错误来自于 sum.js，source map 就会明确的告诉你。  
  
我们来测试一下，在sum.js中输出一个错误：
```javascript
// ES Mudule 规范
export default function (a, b) {
  console.error('this is test') // 输出错误
  return a + b
}

```
在没有devtool配置的情况下 npm run dev，会发现错误提示的行数并不准确，
![Image 2.png](https://upload-images.jianshu.io/upload_images/2012934-694038175799b6a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原因是我们的代码是被编译过的
![Image 3.png](https://upload-images.jianshu.io/upload_images/2012934-d81ed03516a54a42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后在webpack.config.js中加入配置：
```javascript
  devtool: 'inline-source-map', // 加入devtool配置
```
当配置文件改动时需要重新执行 npm run dev：
![Image 4.png](https://upload-images.jianshu.io/upload_images/2012934-432ed6eac8cb2797.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Image 5.png](https://upload-images.jianshu.io/upload_images/2012934-0b4cd3f4cf8a2015.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

错误提示行数以及源码映射都是正确的。devtool的取值有很多，大家可根据需要自行配置
![Image 6.png](https://upload-images.jianshu.io/upload_images/2012934-66866fa91860766f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 1.3 模块热替换
>模块热替换(Hot Module Replacement)是 webpack 提供的最有用的功能之一。它允许在运行时更新各种模块，而无需进行完全刷新。

使用非常简单，在webpack.config.js中引入webpack:
```javascript
  const webpack = require('webpack')
```
在plugins数组中添加：
```javascript
  new webpack.NamedModulesPlugin(),
  new webpack.HotModuleReplacementPlugin()
```
给devServer中的hot属性设为true：
```javascript
devServer: {
    port: 8080,  // 端口号
    host: '0.0.0.0', // 主机名，设为该值可通过IP访问
    overlay: {
      errors: true // 错误提示
    },
    hot: true
  }
```
这样我们修改代码的时候就可以局部刷新模块而不是刷新整个页面了。
  
# 2.构建生产环境
>开发环境(development)和生产环境(production)的构建目标差异很大。在开发环境中，我们需要具有强大的、具有实时重新加载(live reloading)或热模块替换(hot module replacement)能力的 source map 和 localhost server。而在生产环境中，我们的目标则转向于关注更小的 bundle，更轻量的 source map，以及更优化的资源，以改善加载时间。由于要遵循逻辑分离，我们通常建议为每个环境编写彼此独立的 webpack 配置。

>虽然，以上我们将生产环境和开发环境做了略微区分，但是，请注意，我们还是会遵循不重复原则(Don't repeat yourself - DRY)，保留一个“通用”配置。为了将这些配置合并在一起，我们将使用一个名为 webpack-merge 的工具。通过“通用”配置，我们不必在环境特定的配置中重复代码。

我们先从安装 webpack-merge 开始，用来合并webpack配置项：
```bash
npm install --save-dev webpack-merge
```
在config文件夹下创建 webpack.dev.js 和 webpack.build.js 并修改 webpack.config.js，将开发与生产环境的公共配置放在webpack.config.js中：
```javascript
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

const isDev = process.env.NODE_ENV === 'development'

const config = {
  entry: {
    main: path.join(__dirname, '../src/main.js')
  },
  output: {
    filename: '[name].bundle.js',
    path: path.join(__dirname, '../dist')
  },
  module: {
    rules: [
      {
        test: /\.(vue|js|jsx)$/,
        loader: 'eslint-loader',
        exclude: /node_modules/,
        enforce: 'pre'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: createVueLoaderOptions(isDev)
      },
      {
        test: /\.ejs$/,
        use: ['ejs-loader']
      },
      {
        test: /\.css$/,
        use: [
          isDev ? 'vue-style-loader' : MiniCssExtractPlugin.loader,
          { loader: 'css-loader', options: { importLoaders: 1 } },
          'postcss-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          isDev ? 'vue-style-loader' : MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              sourceMap: true
            }
          },
          'less-loader'
        ]
      },
      {
        test: /\.(jpg|jpeg|png|gif|svg)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              name: '[path][name]-[hash:5].[ext]',
              limit: 1024
            }
          }
        ]
      }
    ]
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
**webpack.dev.js**
```javascript
const merge = require('webpack-merge')
const common = require('./webpack.config.js')

module.exports = merge(common, {
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    port: 8080,
    host: '0.0.0.0',
    overlay: {
      errors: true
    },
    historyApiFallback: {
      index: '/index.html'
    }
  }
})

```
**webpack.build.js**
```javascript
const path = require('path')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const merge = require('webpack-merge')
const common = require('./webpack.config.js')

module.exports = merge(common, {
  mode: 'production',
  optimization: {
    splitChunks: {
      chunks: 'initial',
      automaticNameDelimiter: '.',
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'initial',
          minChunks: 2,
          priority: 3
        },
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: 1
        }
      }
    },
    runtimeChunk: {
      name: entrypoint => `manifest.${entrypoint.name}`
    }
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].css'
    }),
    new CleanWebpackPlugin(
      ['dist'],
      {
        root: path.join(__dirname, '../')
      }
    )
  ]
})

```

修改package.json的命令：
```json
"dev": "cross-env NODE_ENV=development webpack-dev-server --config config/webpack.dev.js",
"build": "cross-env NODE_ENV=production webpack --config config/webpack.build.js --progress --inline --colors"
```
现在分别执行 npm run dev 和 npm run build 就会得到你想要的了。
  
  
