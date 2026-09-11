- webpack本身是, node的一个第三方模块包, 用于打包代码
- 现代 javascript 应用程序的 **静态模块打包器 (module bundler)**
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1703671686397-b1bb9908-cd1b-4f7d-8f86-f9eb7615a8e5.png)
- ## 1.  安装Webpack
- 安装依赖包
  
  ```
  yarn add webpack webpack-cli -D
  ```
- 整个项目根目录下的package.json中的scripts是可以定义脚本的内容的。在那里可以定义跟webpack相关的脚本文件。
- ## 2.  webpack的配置
- ### 2.1.  webpack-出入口
- 默认入口: ./src/index.js
- 默认出口: ./dist/main.js
- webpack配置 - webpack.config.js(默认)
- 新建src并列处, webpack.config.js
- 填入配置项
  
  ```
  const path = require("path")
  
  module.exports = {
    entry: "./src/main.js", // 入口
    output: { 
        path: path.join(__dirname, "dist"), // 出口路径
        filename: "bundle.js" // 出口文件名
    }
  }
  ```
- 修改package.json, 自定义打包命令 - 让webpack使用配置文件
  
  ```
  "scripts": {
    "build": "webpack"
  }
  ```
- ### 2.2.  打包流程
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1703671938483-a1c21966-8d2d-41c1-9922-9b353d599c3b.png)
- 重点: 所有要被打包的资源都要跟入口产生直接/间接的引用关系
- 这里不一定需要使用yarn命令，也可以使用npm run命令
- ### 2.3.  Mode模式
- mode模式分为开发阶段和发布阶段
- development 开发阶段，简易打包，打包速度快
- production 发布阶段，打包精细，打包速度慢（但是没关系不会经常production）
  
  ```
  mode: 'development || production'
  ```
- ### 2.4.  自动生成html文件
- 网址：[https://www.webpackjs.com/plugins/html-webpack-plugin/](https://www.webpackjs.com/plugins/html-webpack-plugin/)
- 目标: html-webpack-plugin插件, 让webpack打包后生成html文件并自动引入打包后的js
- 1. 下载插件
  
  ```
  yarn add html-webpack-plugin  -D
  ```
- 2. webpack.config.js配置
  
  ```
  // 引入自动生成 html 的插件
  const HtmlWebpackPlugin = require('html-webpack-plugin')
  
  module.exports = {
    // ...省略其他代码
    plugins: [
        new HtmlWebpackPlugin()
    ]
  }
  ```
- 3. 重新打包后观察dist下是否多出html并运行看效果
	- 打包后的index.html自动引入打包后的js文件
- 4. 自定义打包的html模版，和输出文件名字
  
  ```
  plugins: [
  new HtmlWebpackPlugin({
    template: './public/index.html',
    filename: 'index.html'
  })
  ]
  ```
- ### 2.5.  webpack-dev-server
- 问题背景：每次修改代码, 都需要重新 yarn build 打包, 才能看到最新的效果, 实际工作中, 打包 yarn build 非常费时 (30s - 60s) 之间
- 费时的原因：
- 构建依赖
- 磁盘读取对应的文件到内存, 才能加载
- 将处理完的内容, 输出到磁盘指定目录
- 包的意义：启动本地服务, 可实时更新修改的代码, 打包变化代码到内存中, 然后直接提供端口和网页访问
- 1. 下载包
  
  ```
  yarn add webpack-dev-server -D
  ```
- 2. 配置自定义命令
  
  ```
  scripts: {
  "build": "webpack",
  "serve": "webpack serve"
  }
  ```
- 3. 运行命令-启动webpack开发服务器
  
  ```
  yarn serve
  #或者 npm run serve
  ```
- 总结: 以后改了src下的资源代码, 就会直接更新到内存打包, 然后反馈到浏览器上了
- 在webpack.config.js中添加服务器配置
  
  ```
  module.exports = {
    // ...其他配置
    devServer: {
      port: 3000, // 端口号
      open: true
    }
  }
  ```
- 配置参考：[https://webpack.docschina.org/configuration/dev-server/#devserverafter](https://webpack.docschina.org/configuration/dev-server/#devserverafter)
- ### 2.6.  处理css文件
- 目标: 自己准备css文件, 引入到webpack入口, 测试webpack是否能打包css文件
- 1.新建 - src/styles/index.css
- 2.编写样式
  
  ```
  .banner {
  width: 100px;
  height: 100px;
  background-color: hotpink;
  }
  ```
- 3.(重要) 一定要引入到入口才会被webpack打包
- 4.执行打包命令观察效果
- 配置css和style的loader
	- 1. 安装依赖
	  
	  ```
	  yarn add style-loader css-loader -D
	  ```
	- 2. webpack.config.js 配置
	  
	  ```
	  const HtmlWebpackPlugin = require('html-webpack-plugin')
	  
	  module.exports = {
	   // ...其他代码
	   module: { 
	       rules: [ // loader的规则
	         {
	           test: /\.css$/, // 匹配所有的css文件
	           // use数组里从右向左运行
	           // 先用 css-loader 让webpack能够识别 css 文件的内容并打包
	           // 再用 style-loader 将样式, 把css插入到dom中
	           use: [ "style-loader", "css-loader"]
	         }
	       ]
	   }
	  }
	  ```
- ### 2.7.  处理less文件
- 目标: less-loader让webpack处理less文件, less模块翻译less代码
- 网址：[https://webpack.docschina.org/loaders/less-loader/](https://webpack.docschina.org/loaders/less-loader/)
- 1. 安装less-loader
  
  ```
  yarn add less less-loader -D
  ```
- 2. webpack.config.js 配置
  
  ```
  module: {
  rules: [ // loader的规则
    // ...省略其他
    {
    	test: /\.less$/,
    	// 使用less-loader, 让webpack处理less文件, 内置还会用less翻译less代码成css内容
        use: [ "style-loader", "css-loader", 'less-loader']
    }
  ]
  }
  ```
- ### 2.8.  处理图片
- 目标: 用asset module方式(webpack5版本新增)
- 参考网址：[https://webpack.docschina.org/guides/asset-modules](https://webpack.docschina.org/guides/asset-modules/)/
- 如果使用的是webpack5版本的, 直接配置在webpack.config.js - 的 rules里即可
  
  ```
  {
    test: /\.(png|jpg|gif|jpeg)$/i,
    type: 'asset'
  }
  ```
-