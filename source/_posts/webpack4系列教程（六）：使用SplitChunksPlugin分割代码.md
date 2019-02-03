---
title: webpack4系列教程（六）：使用SplitChunksPlugin分割代码
category:
  - 学习笔记
  - webpack
abbrlink: 2040befe
date: 2019-02-02 21:23:50
tags: webpack
---
# 1. SplitChunksPlugin的概念

起初，chunks(代码块)和导入他们中的模块通过webpack内部的父子关系图连接.在webpack3中，通过CommonsChunkPlugin来避免他们之间的依赖重复。而在webpack4中CommonsChunkPlugin被移除，取而代之的是 optimization.splitChunks 和 optimization.runtimeChunk 配置项，下面展示它们将如何工作。

在默认情况下，SplitChunksPlugin 仅仅影响按需加载的代码块，因为更改初始块会影响HTML文件应包含的脚本标记以运行项目。

webpack将根据以下条件自动拆分代码块：

*   会被共享的代码块或者 node_mudules 文件夹中的代码块
*   体积大于30KB的代码块（在gz压缩前）
*   按需加载代码块时的并行请求数量不超过5个
*   加载初始页面时的并行请求数量不超过3个

### 举例1：

```
// index.js

// 动态加载 a.js
import('./a')
```

![](https://upload-images.jianshu.io/upload_images/2012934-8808e64f84a39ee5.gif?imageMogr2/auto-orient/strip) 

```
// a.js
import 'vue'

// ...
```

![](https://upload-images.jianshu.io/upload_images/2012934-4fb4de61f122b3cf.gif?imageMogr2/auto-orient/strip) 

打包之后的结果会创建一个包含 vue 的独立代码块，当包含 a.js 的原始代码块被调用时，这个独立代码块会并行请求进来。

###  原因：

*   vue 来自 node_modules 文件夹
*   vue 体积超过30KB
*   导入调用时的并行请求数为2
*   不影响页面初始加载

我们这样做的原因是因为，vue代码并不像你的业务代码那样经常变动，把它单独提取出来就可以和你的业务代码分开缓存，极大的提高效率。

### 举例2：

```
// entry.js

import("./a");
import("./b");
```

![](https://upload-images.jianshu.io/upload_images/2012934-23e12a1a84d393ae.gif?imageMogr2/auto-orient/strip) 

```
// a.js
import "./helpers"; // helpers is 40kb in size

// ...
```

![](https://upload-images.jianshu.io/upload_images/2012934-37133ab159ec6a52.gif?imageMogr2/auto-orient/strip) 

```
// b.js
import "./helpers";
import "./more-helpers"; // more-helpers is also 40kb in size

// ...
```

![](https://upload-images.jianshu.io/upload_images/2012934-ca1e61c7a7a3d597.gif?imageMogr2/auto-orient/strip) 

结果：将创建一个单独的块，其中包含`./helpers`它的所有依赖项。在导入调用时，此块与原始块并行加载。

### 原因：

*   条件1：`helpers` 是共享块
*   条件2：`helpers`大于30kb
*   条件3：导入调用的并行请求数为2
*   条件4：不影响初始页面加载时的请求

# 2. SplitChunksPlugin的默认配置

以下是SplitChunksPlugin的默认配置：

```
splitChunks: {
    chunks: "async",
    minSize: 30000, // 模块的最小体积
    minChunks: 1, // 模块的最小被引用次数
    maxAsyncRequests: 5, // 按需加载的最大并行请求数
    maxInitialRequests: 3, // 一个入口最大并行请求数
    automaticNameDelimiter: '~', // 文件名的连接符
    name: true,
    cacheGroups: { // 缓存组
        vendors: {
            test: /[\\/]node_modules[\\/]/,
            priority: -10
        },
        default: {
            minChunks: 2,
            priority: -20,
            reuseExistingChunk: true
        }
    }
}
```

![](https://upload-images.jianshu.io/upload_images/2012934-5a4d9935fd12898e.gif?imageMogr2/auto-orient/strip) 

### 缓存组：

缓存组因该是SplitChunksPlugin中最有趣的功能了。在默认设置中，会将 node_mudules 文件夹中的模块打包进一个叫 vendors的bundle中，所有引用超过两次的模块分配到  default bundle 中。更可以通过 priority 来设置优先级。

## chunks：

chunks属性用来选择分割哪些代码块，可选值有：'all'（所有代码块），'async'（按需加载的代码块），'initial'（初始化代码块）。

# 3\. 在项目中添加SplitChunksPlugin

为了方便演示，我们先安装两个类库： lodash 和 axios，

```
npm i lodash axios -S
```

![](https://upload-images.jianshu.io/upload_images/2012934-2a5a32389d0e0e2b.gif?imageMogr2/auto-orient/strip) 

修改 main.js，引入 lodash 和axios 并调用相应方法：

```
import Modal from './components/modal/modal'
import './assets/style/common.less'
import _ from 'lodash'
import axios from 'axios'
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
console.log(_.camelCase('Foo Bar'))
axios.get('aaa')
```

![](https://upload-images.jianshu.io/upload_images/2012934-acfb0a0b0988cd5f.gif?imageMogr2/auto-orient/strip) 

使用SplitChunksPlugin不需要安装任何依赖，只需在 webpack.config.js 中的 config对象添加 optimization 属性：

```
optimization: {
    splitChunks: {
      chunks: 'initial',
      automaticNameDelimiter: '.',
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: 1
        }
      }
    },
    runtimeChunk: {
      name: entrypoint => `manifest.${entrypoint.name}`
    }
  }
```

![](https://upload-images.jianshu.io/upload_images/2012934-52df2c73e0191da3.gif?imageMogr2/auto-orient/strip) 

配置 runtimeChunk 会给每个入口添加一个只包含runtime的额外的代码块，name 的值也可以是字符串，不过这样就会给每个入口添加相同的 runtime，配置为函数时，返回当前的entry对象，即可分入口设置不同的runtime。

我们再安装一个 webpack-bundle-analyzer，这个插件会清晰的展示出打包后的各个bundle所依赖的模块：

```
npm i webpack-bundle-analyzer -D
```

![](https://upload-images.jianshu.io/upload_images/2012934-a4ce0e6e0d19ac55.gif?imageMogr2/auto-orient/strip) 

引入：

```
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
```

![](https://upload-images.jianshu.io/upload_images/2012934-bb4464bc01afbff2.gif?imageMogr2/auto-orient/strip) 

使用，在plugins数组中添加即可：

```
new BundleAnalyzerPlugin()
```

![](https://upload-images.jianshu.io/upload_images/2012934-07afb9dd3b7c032b.gif?imageMogr2/auto-orient/strip) 

打包之后：

![](http://upload-images.jianshu.io/upload_images/2012934-29eae4f2907bde24?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-0b434d1bf40b2599.gif?imageMogr2/auto-orient/strip) ​

![](http://upload-images.jianshu.io/upload_images/2012934-2b0b82dd7a88552b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-a80e97e65e4cc9c8.gif?imageMogr2/auto-orient/strip) ​

各个模块依赖清晰可见，打开 dist/index.html可见我们的代码顺利运行：

![](http://upload-images.jianshu.io/upload_images/2012934-f1c9374d31f4449c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/2012934-f4a2b3ea23e6c435.gif?imageMogr2/auto-orient/strip) ​

以上就是SplitChunksPlugin的基本用法，更多高级的配置大家可以继续钻研（比如多入口应用）。
