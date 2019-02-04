---
title: vue入门（一）：项目搭建
tags: vue
category:
  - 学习笔记
  - vue
abbrlink: c9a8d1ea
date: 2019-02-03 21:34:05
---
### 前言
我的JS水平比较一般，而且还是跨专业半路出家，因此学习是唯一出路。vue并不是我接触的第一个前端框架，之前学习过angular1.x，觉得不太容易，结果没多久2版本就推出了，一看文档：`totally rewrite`。WTF？？？1还没学利索呢，2就重写了？从此抛弃angular。
直到后来，公司需要做个管理后台系统，经过一番比较最终选择了vue，原因：
>* angular已拉黑
>* react里的jsx语法一时不容易掌握
>* vue学习成本较低，简单易上手，性能也很优秀

二话不说立马上手，我之前的项目都是通过vue-cli创建的，而其中的webpack配置并不特别贴合项目中的要求，由于我之前已经写了webpack系列的博文，所以在这里就从0-1搭建一个vue项目。
# 1. 开始
## 1.1 安装
```bash
npm install vue vue-router -S
```
在项目中我们使用 **.vue** 文件进行开发，所以还要安装一些工具：
```bash
npm install vue-loader vue-style-loader vue-template-compiler -D
```
## 1.2 整理目录
打开我们之前的webpack项目，删除 **src** 目录下的所有文件，并在其下创建 **asset** 文件夹，用来放置资源文件；**pages** 文件夹，来放置我们的页面文件；**router** 文件夹，路由配置；**http** 文件夹，ajax请求配置；**app.js** 入口文件，还有一个 **app.vue** 文件，如图所示：
![目录](https://upload-images.jianshu.io/upload_images/2012934-f633969284748524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后就可以写代码啦.........

# 2. 下面正式写代码
## 2.1 认识 **.vue** 文件
这个 **.vue** 文件是啥呢，官方文档大概是这么说的：
> 在很多 Vue 项目中，我们使用 `Vue.component` 来定义全局组件，紧接着用 `new Vue({ el: '#container '})` 在每个页面内指定一个容器元素。
> 这种方式在很多中小规模的项目中运作的很好，在这些项目里 JavaScript 只被用来加强特定的视图。但当在更复杂的项目中，或者你的前端完全由 JavaScript 驱动的时候，下面这些缺点将变得非常明显：
>- 全局定义 (Global definitions) 强制要求每个 component 中的命名不得重复
字符串模板 (String templates) 缺乏语法高亮
>- 不支持 CSS (No CSS support) 意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏
>- 没有构建步骤 (No build step) 限制只能使用 HTML 和 ES5 JavaScript, 而不能使用预处理器，如 Babel

文件扩展名为 .vue 的 single-file components(单文件组件) 为以上所有问题提供了解决方法，并且还可以使用 webpack等构建工具。

这是一个文件名为 Hello.vue 的简单实例：
![vue-component.png](https://upload-images.jianshu.io/upload_images/2012934-d465c96f488d3961.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

编写 **app.vue** 文件：
```html
<template>
  <div>
    <h1 class="red">{{text }} this is main</h1>
  </div>
</template>
<script>
export default {
  data () {
    return {
      text: 'hello world'
    }
  },
  mounted () {
    console.log('app is mounted')
  }
}
</script>
<style>
  .red {
    color: red;
  }
</style>

```
编写入口文件 **app.js**：
```javascript
import Vue from 'vue'
import App from './app.vue'

new Vue({
  el: '#app',
  render: function (h) {
    return h(App)
  }
})
```
## 2.2 配置webpack
在 config 目录下创建 **vue-loader.config.js** ：
```javascript
// vue-loader.config.js 
module.exports = function (isDev) {
  return {
    preserveWhiteSpace: true,
    extractCss: !isDev,
    cssModules: {
      localIdentName: isDev ? '[path]-[name]-[hash:base64:5]' : '[hash:base64:5]',
      camelCase: true
    },
    hotReload: isDev //根据环境变量生成
  }
}
```
修改 **webpack.config.js** ：
```javascript
// 引入vue-loader.config
const createVueLoaderOptions = require('./vue-loader.config')
// 引入VueLoaderPlugin
const VueLoaderPlugin = require('vue-loader/lib/plugin')
```
```javascript
// 修改入口
entry: {
    app: path.join(__dirname, '../src/app.js')
  }
```
```javascript
// 修改loaders配置
{
        test: /\.vue$/,
        loader: 'vue-loader',
        options: createVueLoaderOptions(isDev)
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
      }
```
```javascript
// 添加VueLoaderPlugin
new VueLoaderPlugin(),
// 修改HtmlWebpackPlugin
new HtmlWebpackPlugin({
      template: path.join(__dirname, '../app.html'),
      inject: true,
      minify: {
        removeComments: true
      }
    })
```
至此，所有配置完毕，执行
```bash
npm run dev
```
如果配置没错，项目就成功跑起来了
![Image 3.png](https://upload-images.jianshu.io/upload_images/2012934-f557243502fc27d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
