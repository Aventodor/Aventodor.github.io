---
title: webpack4系列教程（八）：使用Eslint审查代码
category:
  - 技术文章
  - webpack
abbrlink: 46ef4800
date: 2019-02-02 21:25:18
tags:
---
### 前言：
本章内容，我们在项目中加入eslint配置，来审查校验代码，这样能够避免一些比较低级的错误。并且在团队协作的时候，保持同一种风格和规范能提高代码的可读性，进而提高我们的工作效率。

# 安装：
eslint-config-standard 是一种较为成熟通用的代码审查规则，这样就不用我们自己去定义规则了，使用起来非常方便，记住还需要安装一些依赖插件：
```bash
npm install --save-dev eslint eslint-config-standard eslint-plugin-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node
```

# 配置：
在项目根目录下创建 **.eslintrc** 文件：
```javascript
{
  "extends": "standard",
  "rules": {
    "no-new": "off"
  }
}
```
在vue项目中，**.vue**文件中的 **script**标签内的代码，eslint 是无法识别的，这时就需要使用插件： eslint-plugin-html
```bash
npm i eslint-plugin-html -D
```
然后在 **.eslintrc** 中配置该插件：
```javascript
{
  "extends": "standard",
  "plugins": [
    "html"
  ],
  "rules": {
    "no-new": "off"
  }
}
```
这样就能解析 **.vue**文件中的JS代码了，官方也是如此推荐。
# 使用：
配置完成，如何使用呢？
在 package.json 文件中添加一条 script：
```json
"scripts": {
    "build": "cross-env NODE_ENV=production webpack --config config/webpack.config.js --progress --inline --colors",
    "lint": "eslint --ext .js --ext .vue src/"
  }
  ```
  **- -ext** 代表需要解析的文件格式，最后接上文件路径，由于我们的主要代码都在src 目录下，这里就配置 src 文件夹。
  ```bash
  npm run lint
  ```
  可见控制台给出了很多错误：
  ![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-05a52d6709117b78?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  在项目前期没有加入eslint的情况下，后期加入必然会审查出许多错误。出现这么多错误之后，如果我们逐条手动去解决会非常耗时，此时可以借助eslint自动修复，方法也很简单。
  只需要添加一条命令即可：
  ```json
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config config/webpack.config.js --progress --inline --colors",
    "lint": "eslint --ext .js --ext .vue src/",
    "lint-fix": "eslint --fix --ext .js --ext .jsx --ext .vue src/"
  }
  ```
  然后执行
  ```bash
  npm run lint-fix
  ```

我们希望在开发过程中能够实时进行eslint代码审查，需要安装两个依赖：
```bash
npm i eslint-loader babel-eslint -D
```
修改 **.eslintrc**：
```json
{
  "extends": "standard",
  "plugins": [
    "html"
  ],
  "rules": {
    "no-new": "off"
  },
  "parserOptions":{
    "parser": "babel-eslint"
  }
}
```
由于我们的项目使用了webpack并且代码都是经过Babel编译的，但是Babel处理过的代码有些语法可能对于eslint支持性不好，所以需要指定一个 parser。
下一步，在webpack.config.js中添加loader：
```javascript
{
        test: /\.(vue|js)$/,
        loader: 'eslint-loader',
        exclude: /node_modules/,
        enforce: 'pre'
 }
 ```
 enforce: 'pre' 表示预处理，因为我们只是希望eslint来审查我们的代码，并不是去改变它，在真正的loader(比如：vue-loader)发挥作用前用eslint去检查代码。
  
    记得在你的IDE中安装并开启eslint插件功能，这样就会有错误提示了。
  比如：
  ![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/2012934-93cc5dd4ad5828cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  图中的错误是未使用的变量。
    
   # editorconfig:
   editorconfig是用来规范我们的IDE配置的，在根目录创建 **.editorconfig**文件：
```
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```
这样就能在各种IDE使用相同的配置了。
  
    同样需要在IDE中安装editorconfig插件
以上就是eslint的配置方法了。
     
     