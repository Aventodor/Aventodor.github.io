---
title: webpack4系列教程（四）：处理项目中的资源文件（一）
date: 2019-02-02 21:22:17
tags:
category:
    - 技术文章
    - webpack
---

#  1\. Loader的使用

[之前的博文](https://blog.csdn.net/qq_38286992/article/details/85338683)已经介绍了Loader的概念以及用法，webpack 可以使用 loader 来预处理文件，这允许你打包除 JavaScript 之外的任何静态资源， 甚至允许你直接在 JavaScript 模块中 import CSS文件。

在 src 目录下新建 components 文件夹，新建 modal 组件：

![](http://upload-images.jianshu.io/upload_images/2012934-519e7c14df6655c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-5b27f3099c13faa6.gif?imageMogr2/auto-orient/strip) ​

编写代码：

```
<!--modal.ejs-->

<div class="modal-parent">
    <div class="modal-header">
        <h3 class="modal-title"><%= title %></h3>
    </div>
    <div class="modal-body">
        <%= content %>
    </div>
    <div class="modal-footer">
        <%= footer %>
    </div>
</div>
```

![](https://upload-images.jianshu.io/upload_images/2012934-21882e8c39fcfc4d.gif?imageMogr2/auto-orient/strip) 

```
// modal.js
import template from './modal.ejs'

export default function modal () {
  return {
    name: 'modal',
    template: template
  }
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-7deee7ae59236b4b.gif?imageMogr2/auto-orient/strip) 

修改 main.js：

```
import Modal from './components/modal/modal'

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
}

const app = new App()
```

![](https://upload-images.jianshu.io/upload_images/2012934-c765f0ad8ffd4ec2.gif?imageMogr2/auto-orient/strip) 

 此时执行 npm run build 会报错 ：

![](http://upload-images.jianshu.io/upload_images/2012934-59e57465f2c46875?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-f677b69e94d53924.gif?imageMogr2/auto-orient/strip) ​

webpack 无法解析 .ejs 文件，因此我们需要安装对应的 loader：

```
npm i ejs-loader -D
```

![](https://upload-images.jianshu.io/upload_images/2012934-6dbbd4c3aea8867a.gif?imageMogr2/auto-orient/strip) 

 并修改 webpack.config.js 添加 module 属性：

```
module: {
    rules: [
      {
        test: /.ejs$/,
        use: ['ejs-loader']
      }
    ]
  }
```

![](https://upload-images.jianshu.io/upload_images/2012934-aa746c996285e14c.gif?imageMogr2/auto-orient/strip) 

再次执行 npm run build 就不会报错了，打开 dist/index.html ：

![](http://upload-images.jianshu.io/upload_images/2012934-567926279c5baea4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-45a3491ae667b44c.gif?imageMogr2/auto-orient/strip) ​![image](http://upload-images.jianshu.io/upload_images/2012934-827ba9621c46439f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-472481b12e4a4d5a.gif?imageMogr2/auto-orient/strip) ​

可以看到我们的 modal 组件已经成功渲染出来了。 

# 2\. 处理项目中的CSS文件

在 modal.css 中加入样式代码：

```
.modal-parent{
    width: 500px;
    height: auto;
    border: 1px solid #ddd;
    border-radius: 10px;
}
.modal-title{
    font-size: 20px;
    text-align: center;
    padding: 10px;
    margin: 0;
}
.modal-body{
    border: 1px solid #ddd;
    border-left: 0;
    border-right: 0;
    padding: 10px;
}
.modal-footer{
    padding: 10px;
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-5c7fa85be717e91e.gif?imageMogr2/auto-orient/strip) 

安装 css-loader 和 style-loader：

```
npm i css-loader style-loader -D
```

![](https://upload-images.jianshu.io/upload_images/2012934-e69ae0adafcad013.gif?imageMogr2/auto-orient/strip) 

 修改webpack.config.js 中的 module.rules ，添加css-loader 和 style-loader：

```
module: {
    rules: [
      {
        test: /.ejs$/,
        use: ['ejs-loader']
      },
      {
        test: /.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      }
    ]
  },
```

![](https://upload-images.jianshu.io/upload_images/2012934-fcc1037cd471ec32.gif?imageMogr2/auto-orient/strip) 

在 modal.js 中引入 modal.css：

```
import './modal.css'
```

![](https://upload-images.jianshu.io/upload_images/2012934-2b409dc4be0e787d.gif?imageMogr2/auto-orient/strip) 

再次执行 npm run build ，打开 dist/index.html：

![](http://upload-images.jianshu.io/upload_images/2012934-b76254f3310ea349?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-6c04f88bc3146574.gif?imageMogr2/auto-orient/strip) ​

CSS样式已经通过 style 标签添加到页面上了；

![](http://upload-images.jianshu.io/upload_images/2012934-789f0b03d296061d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-bf866058d3ee356b.gif?imageMogr2/auto-orient/strip) ​

## 3\. 处理项目中的图片 

在src目录下创建 assets/img ，放入两张图片

 ![](http://upload-images.jianshu.io/upload_images/2012934-138ce04ee955bb05.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-5f91b28d679beb14.gif?imageMogr2/auto-orient/strip) ​

给 modal 添加一个背景图的样式：

```
.modal-body{
    border: 1px solid #ddd;
    border-left: 0;
    border-right: 0;
    padding: 10px;
    background: url("../../assets/img/bg.jpg");
    color: #fff;
    height: 500px;
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-4712455d71166e5e.gif?imageMogr2/auto-orient/strip) 

由于webpack无法处理图片资源，所以也要安装对应的 loader

```
npm install --save-dev url-loader file-loader
```

![](https://upload-images.jianshu.io/upload_images/2012934-33f68e843f1db39c.gif?imageMogr2/auto-orient/strip) 

在 webpack.config.js 中添加 loader 

```
rules: [
      {
        test: /.ejs$/,
        use: ['ejs-loader']
      },
      {
        test: /.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },
      {
        test: /.(jpg|jpeg|png|gif|svg)$/,
        use: 'url-loader'
      }
    ]
```

![](https://upload-images.jianshu.io/upload_images/2012934-33abaf8a87b28916.gif?imageMogr2/auto-orient/strip) 

打包代码之后，在浏览器打开 dist/index.html ，可见图片已经显示出来了：

![](http://upload-images.jianshu.io/upload_images/2012934-15bd44cc1d30757a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-88c8c9782c1b9c3a.gif?imageMogr2/auto-orient/strip) ​

仔细查看这张图片可以发现，它是通过 DataURL 加载出来的：![image](http://upload-images.jianshu.io/upload_images/2012934-ea792a00898e338c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-17c9cacd8903e199.gif?imageMogr2/auto-orient/strip) ​

下面更改 url-loader 的配置，limit表示在文件大小低于指定值时，返回一个DataURL

```
{
        test: /.(jpg|jpeg|png|gif|svg)$/,
        use:  [
          {
            loader: 'url-loader',
            options: {
              name: '[name]-[hash:5].[ext]',
              limit: 1024
            }
          }
        ]
      }
```

![](https://upload-images.jianshu.io/upload_images/2012934-d29b17c175109b22.gif?imageMogr2/auto-orient/strip) 

再次打包后，图片会以文件形式展示出来：

![](http://upload-images.jianshu.io/upload_images/2012934-d11259cc4a15361f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-71c6bc549e8903ce.gif?imageMogr2/auto-orient/strip) ​

