---
title: webpack4系列教程（七）：使用 babel-loader
date: 2019-02-02 21:24:32
tags:
category:
    - 技术文章
    - webpack
---
# 1. 什么是Babel
如今 ES6 语法在开发中已经非常普及，甚至也有许多开发人员用上了 ES7 或 ES8 语法。然而，浏览器对这些高级语法的支持性并不是非常好。因此为了让我们的新语法能在浏览器中都能顺利运行，Babel 应运而生。
Babel是一个JavaScript编译器，能够让我们放心的使用新一代JS语法。比如我们的箭头函数：
```javascript
() => console.log('hello babel')
```
经过Babel编译之后：
```javascript
(function(){
	return console.log('hello babel');
});
```
会编译成浏览器可识别的ES5语法。


# 2. 在webpack中使用babel-loader
安装：
```bash
npm install -D babel-loader @babel/core @babel/preset-env webpack
```

修改 webpack.config.js，加入新的loader：
```javascript
{
	test: /\.js$/,
    loader: 'babel-loader',
    exclude: /node_modules/
}
```
遇到JS文件就先用babel-loader处理，exclude表示排除 node_modules 文件夹中的文件。loader的配置就OK了，可是这样还不能发挥Babel的作用。在项目根目录下创建一个 .babelrc 文件，添加代码：
```json
{
  "presets": [
    "@babel/preset-env"
  ]
}
```
我们还希望能够在项目对一些组件进行懒加载，所以还需要一个Babel插件：
```bash
npm i babel-plugin-syntax-dynamic-import -D
```
在 .babelrc 文件中加入plugins配置：
```json
{
  "presets": [
    "@babel/preset-env"
  ],
  "plugins": [
    "syntax-dynamic-import"
  ]
}
```

在src 目录下创建 helper.js：
```javascript
console.log('this is helper')
```
再来修改我们的 main.js ：
```javascript
import 'babel-polyfill'
import Modal from './components/modal/modal'
import './assets/style/common.less'
import _ from 'lodash'
const App = function () {
  let div = document.createElement('div')
  div.setAttribute('id', 'app')
  document.body.appendChild(div)
  let dom = document.getElementById('app')
  let modal = new Modal()
  dom.innerHTML = modal.template({
    title: '标题',
    content: '内容',
    footer: '底部'
  })
  let button = document.createElement('button')
  button.innerText = 'click me'
  button.onclick = () => {
    const help = () => import('./helper')
    help()
  }
  document.body.appendChild(button)
}
const app = new App()
console.log(_.camelCase('Foo Bar'))
```
当button点击时，加载 helper 然后调用。打包之后可见：
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-f737bd10225bc684?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
多了一个 3.bundle.js，在浏览器打开 dist/index.html ，打开 network查看，3.bundle.js并未加载：
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-27a396aa4f6c7597.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当点击button之后，发现浏览器请求了3.bundle.js，控制台也打印出了数据。
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-729e9ff02dba24c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-aa9a359aa1e18ee1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>由于 Babel 只转换语法(如箭头函数)， 你可以使用 babel-polyfill 支持新的全局变量，例如 Promise 、新的原生方法如 String.padStart (left-pad) 等。

安装：
```bash
npm install --save-dev babel-polyfill
```
在入口文件引入就可以了：
```javascript
import 'babel-polyfill'
```


