# 1.  前置知识
- ## 1.1.  虚拟DOM
- DOM的本质是什么：用JS表示的UI元素
- DOM和虚拟DOM的区别：
	- DOM是由浏览器中的JS提供功能，所以我们只能人为的使用 浏览器提供的固定的API来操作DOM对象；
	- 虚拟DOM：并不是由浏览器提供的，而是我们程序员手动模拟实现的，类似于浏览器中的DOM，但是有着本质的区别；
- **DOM树的概念，**一个网页呈现的过程：
	- 1.浏览器请求服务器获取页面HTML代码
	- 2.浏览器要现在内存中解析DOM结构，并在浏览器内存中，渲染出一颗DOM树；
	- 3.浏览器把DOM树，呈现到页面上。
- ## 1.2.  Diff算法
- tree diff：
	- 新旧DOM树，逐层对比的方式，就叫做 tree diff,每当我们从前到后，把所有层的节点对比完后，必然能够找到那些 需要被更新的元素；
- component diff：
	- 在对比每一层的时候，组件之间的对比，叫做 component diff;当对比组件的时候，如果两个组件的类型相同，则暂时认为这个组件不需要被更新，如果组件的类型不同，则立即将旧组件移除，新建一个组件，替换到被移除的位置；
- element diff：
	- 在组件中，每个元素之间也要进行对比，那么，元素级别的对比，叫做 element diff；
- key：
	- key这个属性，可以把 页面上的 DOM节点 和 虚拟DOM中的对象，做一层关联关系；
	  
	  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1615357120803-42744b4f-2427-4ff4-b54f-0c08c9238946.png)
- Diff算法存在验证
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/17.0.1/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/17.0.1/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/17.0.1/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/17.0.1/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      class Time extends React.Component{
  
        state={date:new Date()}
  
        componentDidMount(){
          setInterval(()=>{
            this.setState({
              date:new Date()
            })
          },1000)
        }
  
        render(){
          return(
            <div>
              <h1>hello</h1>
              <input type="text"/>
              <span>
                现在是：{this.state.date.toTimeString()}
                <input type="text"/>
              </span>    
            </div>
          )
        }
      }
    </script>
  </body>
  </html>
  ```
- Diff算法
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/17.0.1/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/17.0.1/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/17.0.1/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/17.0.1/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      /*
            经典面试题：
                1). react/vue中的key有什么作用？（key的内部原理是什么？）
                2).为什么遍历列表时，key最好不要用index？
  
                1.虚拟DOM中key的作用：
                    1).简单的说：key是虚拟DOM对象的标识，在更新显示时key起着极其重要的作用。
                    2).详细的说：当状态中的数据发生变化时，react会根据【新数据】生成【新的虚拟DOM】，
                                随后React进行【新虚拟DOM】与【旧虚拟DOM】的diff比较，比较规则如下：
                            
                            A.旧虚拟DOM中找到了与新虚拟DOM相同的key：
                                （1）.若虚拟DOM中内容没变，直接使用之前的真实DOM
                                （2）.若虚拟DOM中内容变了，则生成新的真实DOM，随后替换掉页面中之前的真实DOM
                            B.旧虚拟DOM中未找到与新虚拟DOM相同的key
                                根据数据创建新的真实DOM，然后渲染到页面
                
                2.用index作为key可能会引发的问题
                    1.若对数据进行：逆序添加、逆序删除等破坏顺序操作会产生没有必要的真实DOM更新
                    2.如果结构中还包含输入类的DOM会产生错误DOM更新
                    3.注意如果不存在对数据的逆序添加、逆序删除等操作，仅用于渲染队列用于展示，使用index作为key是没有问题的。
                3. 开发中如何选择key？
                    1.最好使用每条数据的唯一标识作为key，比如id、手机号
                    2.如果确定只是简单的展示数据，用index也是可以的。
        */
  
      /*
            慢动作回放——使用index索引值作为key
                初始数据：
                    {id:1,name:'小张',age:18}
                    {id:2,name:'小李',age:19}
                初始的虚拟DOM：
                    <li key=0>小张----18</li>
                    <li key=1>小李----19</li>
                
                更新后的数据：
                    {id:3,name:'小王',age:20}
                    {id:1,name:'小张',age:18}
                    {id:2,name:'小李',age:19}
                更新后的虚拟DOM：
                    <li key=0>小王----20</li>
                    <li key=1>小张----18</li>
                    <li key=2>小李----19</li>
  
        */
      class Person extends React.Component {
        state = {
          persons: [
            { id: 1, name: '小张', age: 18 },
            { id: 2, name: '小李', age: 19 }
          ]
        }
  
        add = () => {
          const { } = this.state
          const p={id:persons.length+1,name:'小王',age:20}
          this.setState({persons:[...persons,p]})
        }
        render() {
          return (
            <div>
              <h2>展示人员信息</h2>
              <button onClick={this.add}>添加一个小王</button>
                        <h3>使用index（索引值）作为key</h3>
                        <ul>
                            {
                                this.state.persons.map((personObj, index) => {
                                    return <li key={index}>{personObj.name}----{personObj.age}</li>
                                })
                            }
                        </ul>
                        <hr/>
                        <hr/>
                        <h3>使用id（数据的唯一标识）作为key</h3>
                        <ul>
                            {
                                this.state.persons.map((personObj) => {
                                    return <li key={personObj.id}>{personObj.name}----{personObj.age}</li>
                                })
                            }
                        </ul>
                    </div>
  
                )
            }
        }
    </script>
  </body>
  
  </html>
  ```
- # 2.  环境搭建
- ## 2.1.  脚手架
- React是Facebook研发的一款前端框架(MVC框架：侧重于view层操作)，目前在行业内广泛使用。为了让框架的体积变得更轻量级，设计者们把其定义为“渐进式”框架,也就是:
	- 主体核心的部分都在react/react-dom两个框架中
	- 如果想使用路由，我们需要安装react-router
	- 如果想使用redux进行状态统一管理，我们再安装redux/react-redux
	- 如果想实现其它的功能，我们还可以继续安装 中间件/axious等 ...
	- 根据用户所需，再安装不同的配套组件来完成，这就是"渐进式"设计思想(vue也是这样设计的)
- 我们非常期待有一款自动化部署工具，可以帮助我们自动完成这些编译的操作，webpack就是目前项目中非常常用的自动化部署平台，我们在搭建项目之前，还需要自己配置webpack。 ... 类似于这样的操作还有很多，所以复杂繁琐的项目配置，模块安装等成为开发者们一大痛苦，有人调侃，是不是以后会出现"前端配置工程师"; 为了解决这些问题，市面上有很多react项目开发的脚手架:
	- create-react-app
- 配置脚手架
- **第一步**，全局安装：npm i -g create-react-app
- **第二步**，切换到想创项目的目录，使用命令：create-react-app hello-react
- **第三步**，进入项目文件夹：cd hello-react
- **第四步**，启动项目：npm start
	- 第一步和第二步，都是基于我们自己的电脑配置和网络环境
	- 第二步中的地址和名字都可以更换。
	- 第四步，因为我安装了npm，因此使用的是npm启动
	  
	  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1617241618405-27c9c62b-4e1a-46b0-8a65-b9a423ca76a1.png)
- 启动之后，就是这样的一个界面。
- ## 2.2.  项目结构说明
- ### 2.2.1.  文件结构
  
  public ---- 静态资源文件夹
  
                        favicon.icon ------ 网站页签图标
  
                        **index.html -------- ****主页面**
  
                        logo192.png ------- logo图
  
                        logo512.png ------- logo图
  
                        manifest.json ----- 应用加壳的配置文件
  
                        robots.txt -------- 爬虫协议文件
  
  src ---- 源码文件夹
  
                        App.css -------- App组件的样式
  
                        **App.js --------- App****组件**
  
                        App.test.js ---- 用于给App做测试
  
                        index.css ------ 样式
  
                        **index.js ------- ****入口文件**
  
                        logo.svg ------- logo图
  
                        reportWebVitals.js
  
                                       --- 页面性能分析文件(需要web-vitals库的支持)
  
                        setupTests.js
  
                                       ---- 组件单元测试的文件(需要jest-dom库的支持)
- ### 2.2.2.  项目结构
- 一个新的项目结构，只需要一个index.js、index.html、App.js。
- 为了区分组件和普通js文件，使用jsx的后缀作为组件文件的后缀。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1617416757933-6c82332f-80a1-47b6-b7c0-3d9ce87ba71f.png)
- 接下来就是在项目中是实现一个简单的React组件
	- public/index.html
	  
	  ```
	  <!DOCTYPE html>
	  <html lang="en">
	  <head>
	   <meta charset="UTF-8">
	   <meta http-equiv="X-UA-Compatible" content="IE=edge">
	   <meta name="viewport" content="width=device-width, initial-scale=1.0">
	   <title>脚手架</title>
	  </head>
	  <body>
	   <div id="root"></div>
	  </body>
	  </html>
	  ```
	- src/index.js
	  
	  ```
	  // 引入react核心库
	  import React from 'react'
	  // 引入ReactDOM
	  import ReactDOM from 'react-dom'
	  // 引入App组件
	  import App from './App'
	  
	  // 渲染App到页面
	  ReactDOM.render(<App/>,document.getElementById('root'))
	  ```
	- src/App.jsx
	  
	  ```
	  // 创建外壳组件App
	  import React, { Component } from 'react'
	  import Hello from './Component/Hello/Hello'
	  
	  // 定义组件且暴露App组件
	  export default class App extends React.Component {
	   render() {
	       return (
	           <div>
	               <Hello/>
	           </div>
	       )
	   }
	  }
	  ```
	- src/Component/Hello
	  
	  ```
	  import React,{Component} from 'react'
	  import './Hello.css'
	  
	  export default class Hello extends Component{
	  render(){
	   return <h2 className="Hello">Hello,React!</h2>
	  }
	  }
	  ```
	  
	  ```
	  .Hello{
	   background: grey;
	  }
	  ```
- 暴露组件可以通过export的标签实现，也可以通过不同的方式分别暴露组件。
- 要注意一个组件和组件中属性的区别。
- ## 2.3.  模块与模块化
- 模块
	- 1.   理解：向外提供特定功能的js程序, 一般就是一个js文件
	- 2.  为什么要拆成模块：随着业务逻辑增加，代码越来越多且复杂。
	- 3.  作用：复用js, 简化js的编写, 提高js运行效率
- 模块化
	- 当应用的js都以模块来编写的, 这个应用就是一个模块化的应用
- 组件
	- 1.  理解：用来实现局部功能效果的代码和资源的集合(html/css/js/image等等)
	- 2. 为什么要用组件： 一个界面的功能更复杂
	- 3. 作用：复用编码, 简化项目编码, 提高运行效率
- 组件化
	- 当应用是以多组件的方式实现, 这个应用就是一个组件化的应用
- 函数式
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      // 1.创建函数式组件
      function MyComponent(){
        //此处的this是undefined，因为babel编译后开启了严格模式
        //console.log(this)
        return <h2>我是用函数定义的组件，适用于简单组件</h2>
      }
      // 2.渲染虚拟DOM到页面
      ReactDOM.render(<MyComponent/>, document.getElementById('test'))
      /*
            执行了ReactDOM.render(<MyComponent/>, document.getElementById('test')),发生了什么?
                1.React解析组件标签，找到了MyComponent组件。
                2.发现组件是使用函数定义的，随后调用该函数。
  
        */
    </script>
  </body>
  
  </html>
  ```
- 类式组件
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 1.创建类式组件
        class MyComponent extends React.Component{
            render(){
                console.log('render中的this:',this)
                return <h2>我是用类定义的组件，适用于复杂组件的定义</h2>
            }
        }
        // 2.渲染组件到页面
        ReactDOM.render(<MyComponent/>,document.getElementById('test'))
        /*
            执行了ReactDOM.render(<MyComponent/>,document.getElementById('test'))之后，发生了什么？
                1.React解析组件标签，找到了MyComponent组件
                2.发现组件是使用类定义的，随后new出来该实例，并通过实例调用到原型上的render方法
                3.将render返回的虚拟DOM转化为真实DOM，随后呈现在页面中。
        */
    </script>
  </body>
  </html>
  ```
- # 3.  React基础
- ## 3.1.  虚拟DOM用法
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      /*
            关于虚拟DOM：
                1.本质是Object类型的对象（一般对象）
                2.虚拟DOM比较轻量级，真实DOM比较重，因为虚拟DOM是React内部在用，无需真实DOM上那么多属性。
                3.虚拟DOM最终会被React转化为真实DOM，呈现在页面上。
        */
      // 1.创建虚拟DOM
      /* 此处一定不要写引号，因为不是字符串 */
      const VDOM = <h1>Hello React</h1>
      // 2.渲染虚拟DOM到页面
      ReactDOM.render(VDOM, document.getElementById('test'))
    </script>
  </body>
  
  </html>
  ```
- 导入的时候，包的先后顺序是有区别的。需要按照以上的顺序将包导入。
- 关于虚拟DOM：
	- 1.本质是Object类型的对象（一般对象）
	- 2.虚拟DOM比较轻量级，真实DOM比较重，因为虚拟DOM是React内部在用，无需真实DOM上那么多属性。
	- 3.虚拟DOM最终会被React转化为真实DOM，呈现在页面上。
- 创建虚拟DOM的两种方式
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/javascript">
      // 1.创建虚拟DOM
      /* 此处一定不要写引号，因为不是字符串 */
      const VDOM = React.createElement('h1',{id:'title'},'Hello React')
      // 2.渲染虚拟DOM到页面
      ReactDOM.render(VDOM, document.getElementById('test'))
    </script>
  </body>
  
  </html>
  ```
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/javascript">
        // 1.创建虚拟DOM
        /* 此处一定不要写引号，因为不是字符串 */
        const VDOM = (
            <h1 id="title">
                <span>Hello,React</span>
            </h1>
        )
        // 2.渲染虚拟DOM到页面
        ReactDOM.render(VDOM, document.getElementById('test'))
    </script>
  </body>
  
  </html>
  ```
- ## 3.2.  JXS语法
	- 6. 标签必须闭合
	- 7. 标签首字母
		- (1)  若小写字母开头，则将改标签转为html中同名元素，若html中无该标签对应的同名元素，则报错。
		- (2)  若大写字母开头，react就去渲染对应的组件，若组件没有定义，则报错。
		  
		  ```
		  <!DOCTYPE html>
		  <html lang="en">
		  
		  <head>
		  <meta charset="UTF-8">
		  <meta http-equiv="X-UA-Compatible" content="IE=edge">
		  <meta name="viewport" content="width=device-width, initial-scale=1.0">
		  <title>Hello React</title>
		  <!-- 引入react核心库 -->
		  <script type="text/javascript" src="../js/react.development.js"></script>
		  <!-- 引入react-dom，用于支持react操作DOM -->
		  <script type="text/javascript" src="../js/react-dom.development.js"></script>
		  <!-- 引入babel，用于将jsx转为js -->
		  <script type="text/javascript" src="../js/babel.min.js"></script>
		  </head>
		  
		  <body>
		  <!-- 准备好一个DOM容器-->
		  <div id="test"></div>
		  
		  <script type="text/babel">
		  /*
		      jsx语法规则：
		          1.定义虚拟DOM时，不要写引号
		          2.标签中混入JS表达式时使用{}
		          3.样式的类名指定不使用class，使用className
		          4.内联样式，要用style={{key：value}}的形式去写
		          5.只有一个根标签
		          6.标签必须闭合
		          7.标签首字母
		              (1) 若小写字母开头，则将改标签转为html中同名元素，若html中无该标签对应的同名元素，则报错。
		              (2) 若大写字母开头，react就去渲染对应的组件，若组件没有定义，则报错。
		  */
		      const myId = 'atguigu'
		      const myData = 'Hello react'
		      // 1.创建虚拟DOM
		      const VDOM = (
		          <div>
		              <h2 className="title" id={myId.toLocaleLowerCase()}>
		                  <span style={{ color: 'whith', fontsize: '29px' }}>
		                      {myData.toLocaleUpperCase()}
		                  </span>
		              </h2>
		              <h2 className="title" id={myId.toLocaleUpperCase()}>
		                  <span style={{ color: 'whith', fontsize: '29px' }}>
		                      {myData.toLocaleUpperCase()}
		                  </span>
		              </h2>
		              <input type="text" />
		          </div>
		      )
		      // 2.渲染虚拟DOM到页面
		      ReactDOM.render(VDOM, document.getElementById('test'))
		  </script>
		  </body>
		  </html>
		  ```
- 一定注意区分：【js语句（代码）】与【js表达式】
- 1.表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方
- 下面这些都是表达式：
	- （1） a
	- （2） a+b
	- （3） demo（1）
	- （4） arr.map（）
	- （5） function test（）{}
- 2.语句（代码）：
- 下面这些都是语句（代码）：
	- （1） if(){}
	- （2） for(){}
	- （3） switch(){case:xxxx}
- ## 3.3.  类式组件
- ### 3.3.1.  State属性
- state是组件对象最重要的属性, 值是对象(可以包含多个key-value的组合)
- 组件被称为"状态机", 通过更新组件的state来更新对应的页面显示(重新渲染组件)
- 初始化State
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      // 1.创建类式组件
      class Weather extends React.Component{
        constructor(props){
          super(props)
          // 初始化状态
          this.state={isHot:true}
        }
        render(){
          // 读取状态
          const {isHot} = this.state
          return <h1 onClick={demo}>今天天气很{isHot?'炎热':'凉爽'}</h1>
        }
      }
      // 2.渲染组件到页面
      ReactDOm.render(<Weather/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 这里我可以认为State属性是一个对象，里面可以通过类似python中的字典形式来进行定义对象。
- React中绑定事件
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 1.创建类式组件
        class Weather extends React.Component{
            constructor(props){
                super(props)
                // 初始化状态
                this.state={isHot:true}
            }
            render(){
                // 读取状态
                const {isHot} = this.state
                return <h1 onClick={demo}>今天天气很{isHot?'炎热':'凉爽'}</h1>
            }
        }
        // 2.渲染组件到页面
        ReactDOM.render(<Weather/>,document.getElementById('test'))
  
        function demo(){
            console.log('标题被点击了');
        }
    </script>
  </body>
  </html>
  ```
- React中推荐使用上述中的第三种方法。但是在React的babel语言的严格形式下，原生的onclick方法需要改成onClick的形式。在花括号中的demo函数也需要去掉括号，否则会导致直接的回调调用。
- this对象的使用
- 在面向对象中this简单明确，就是指向类的对象。
- 在Javascript中则没这么简单。this在不同的书写方式，不同的创建方式里，指向的对象则不同，细说起来，没有上万字是不够的。但大致说来主要分为几类，以及以下几类的各类组合了。
	- 1.单独function中的this以及在闭包function中的this.
	- 2.使用strict时的this.
	- 3.使用对象中function的this
	- 4.使用=>箭头函数中的this
	- 5.使用ES6中的this
	- 6.Dom事件中的this
	- 7.使用bind函数中的this
- this在js中的简单理解
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>
  <body>
    <script type="text/javascript">
      class Person{
        constructor(name,age){
          this.name=name
          this.age=age
        }
        study(){
          // study方法放在了哪里？——类的原型对象上，供实例使用
          // 通过Person实例调用study时，study中的this就是Person实例
          console.log(this);
        }
      }
  
      const p1=new Person('tom',18)
      p1.study()
      const x=p1.study
      x()
    </script>
  </body>
  </html>
  ```
- render方法中使用bind方法
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 1.创建类式组件
        class Weather extends React.Component{
            constructor(props){
                super(props)
                // 初始化状态
                this.state={isHot:true}
                this.changeWeather=this.changeWeather.bind(this)
            }
            render(){
                // 读取状态
                const {isHot} = this.state
                return <h1 onClick={this.changeWeather}>今天天气很{isHot?'炎热':'凉爽'}</h1>
            }
            changeWeather(){
                // changeWeather方法放在了哪里？——类的原型对象上，供实例使用
                // 由于changeWeather是作为onClick的回调，所以不是通过实例调用的，是直接调用
                // 类中的方法默认开启了局部的严格模式，所以changeWeather中的this为undefined
                console.log(this);
            }
        }
        // 2.渲染组件到页面
        ReactDOM.render(<Weather/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 修改State参数
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 1.创建类式组件
        class Weather extends React.Component{
            // 构造器调用几次？——1次
            constructor(props){
                super(props)
                // 初始化状态
                this.state={isHot:true}
                // 解决changeWeather中this指向的问题
                this.changeWeather=this.changeWeather.bind(this)
            }
            
            // render调用几次？—— 1+n次 1是初始化的那次，n是状态更新的次数
            render(){
                // 读取状态
                const {isHot} = this.state
                return <h1 onClick={this.changeWeather}>今天天气很{isHot?'炎热':'凉爽'}</h1>
            }
            // changeWeather调用几次？——触发几次调几次
            changeWeather(){
                // changeWeather方法放在了哪里？——类的原型对象上，供实例使用
                // 由于changeWeather是作为onClick的回调，所以不是通过实例调用的，是直接调用
                // 类中的方法默认开启了局部的严格模式，所以changeWeather中的this为undefined
  
                console.log('changeWeather');
                // 获取原来的isHot值
                const isHot=this.state.isHot
                // 特别注意：状态必须通过setState进行更新，且更新是一种合并，并不是替换
                // 特别注意：状态（state）不可直接更改
                // 禁止：this.state.isHot=!isHot
                this.setState({isHot:!isHot})
            }
        }
        // 2.渲染组件到页面
        ReactDOM.render(<Weather/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 这种是通过setState的方式来改变State的。
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 1.创建类式组件
        class Weather extends React.Component{
            // 初始化状态
            state = {isHot:false,wind:'微风'}
  
            render(){
                const {isHot,wind}=this.state
                return <h1 onClick={this.changeWeather}>今天天气很{isHot?'炎热':'凉爽'}</h1>
            }
  
            // 自定义方法——要用赋值语句的形式+箭头函数
            changeWeather=()=>{
                const isHot=this.state.isHot
                this.setState({isHot:!isHot})
            }
        }
        // 2.渲染组件到页面
        ReactDOM.render(<Weather/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- （）=>{} 函数的形式中是没有的this对象的。这里面的this对象是直接使用上一级单位的this对象。
- State总结
- state是组件对象最重要的属性, 值是对象(可以包含多个key-value的组合)
- 组件被称为"状态机", 通过更新组件的state来更新对应的页面显示(重新渲染组件)
- 组件中render方法中的this为组件实例对象
- 组件自定义的方法中this为undefined，如何解决？
	- 强制绑定this: 通过函数对象的bind()
	- 箭头函数
- 状态数据，不能直接修改或更新
- ### 3.3.2.  Props属性
- 每个组件对象都会有props(properties的简写)属性
- 组件标签的所有属性都保存在props中
- props的基本使用
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
    <div id="test1"></div>
    <div id="test2"></div>
  
    <script type="text/babel">
        // 创建组件
        class Person extends React.Component{
            render(){
                console.log(this)
                const {name,age,sex}=this.props
                return (
                    <ul>
                        <li>姓名：{name}</li>
                        <li>性别：{sex}</li>
                        <li>年龄：{age}</li>
                    </ul>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Person name="tom" age="19" sex="男"/>,document.getElementById('test'))
        ReactDOM.render(<Person name="jerry" age="19" sex="男"/>,document.getElementById('test1'))
        ReactDOM.render(<Person name="jack" age="19" sex="男"/>,document.getElementById('test2'))
    </script>
  </body>
  </html>
  ```
- props属性的应用直接就是在组件内添加属性，然后就可以将相应的属性添加到props这个属性当中。
- js中的...语法
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
  </head>
  <body>
    <script type="text/javascript">
        let arr1=[1,3,5,7,9]
        let arr2=[2,4,6,8,10]
        // 展开一个数组
        console.log(...arr1)
        // 连接数组
        let arr3=[...arr1,...arr2]
  
        // 在函数中使用
        function sum(...numbers){
            return numbers.reduce(
                (preValue,currentValue)=>{
                    return preValue+currentValue
                }
            )
        }
        console.log(sum(1,2,3,4))
  
        // 构造字面量对象时使用展开语法
        let person={name:'tom',age:18}
        let person2={...person}
  
        person.name='jerry'
        console.log(person2)
        console.log(person)
    </script>
  </body>
  </html>
  ```
- 批量注入props的方法
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
    <div id="test1"></div>
    <div id="test2"></div>
  
    <script type="text/babel">
        // 创建组件
        class Person extends React.Component{
            render(){
                console.log(this)
                const {name,age,sex}=this.props
                return (
                    <ul>
                        <li>姓名：{name}</li>
                        <li>性别：{sex}</li>
                        <li>年龄：{age}</li>
                    </ul>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Person name="tom" age="19" sex="男"/>,document.getElementById('test'))
        ReactDOM.render(<Person name="jerry" age="19" sex="男"/>,document.getElementById('test1'))
  
        const p={name:'老刘',age:18,sex:'女'}
        ReactDOM.render(<Person {...p}/>,document.getElementById('test2'))
    </script>
  </body>
  </html>
  ```
- 在label语言和react.development.js包的作用下，允许标签内使用{...}语法作为合理语法来进行赋值。
- 对props进行限制，这里需要引入prop-type.js
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
    <div id="test1"></div>
    <div id="test2"></div>
  
    <script type="text/babel">
        // 创建组件
        class Person extends React.Component{
            render(){
                console.log(this)
                const {name,age,sex}=this.props
                return (
                    <ul>
                        <li>姓名：{name}</li>
                        <li>性别：{sex}</li>
                        <li>年龄：{age}</li>
                    </ul>
                )
            }
        }
        // 对标签属性进行类型、必要性的限制
        Person.propTypes={
            // 限制name为必传，且为字符串
            name:PropTypes.string.isRequired,
            sex:PropTypes.string,
            age:PropTypes.number,
            // 如果传入的是函数，就采用这个形式
            speak:PropTypes.func,
        }
        // 指定默认标签属性值
        Person.defaultProps={
            sex:'不男不女',
            age:18
        }
        // 渲染组件到页面
        ReactDOM.render(<Person name="tom" age="19" sex="男"/>,document.getElementById('test'))
        ReactDOM.render(<Person name="jerry" age="19" sex="男"/>,document.getElementById('test1'))
  
        const p={name:'老刘',age:18,sex:'女'}
        ReactDOM.render(<Person {...p}/>,document.getElementById('test2'))
    </script>
  </body>
  </html>
  ```
- 将属性封装在对象内
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
    <div id="test1"></div>
    <div id="test2"></div>
  
    <script type="text/babel">
        // 创建组件
        class Person extends React.Component {
            // 对标签属性进行类型、必要性的限制
            static propTypes = {
                // 限制name为必传，且为字符串
                name: PropTypes.string.isRequired,
                sex: PropTypes.string,
                age: PropTypes.number,
                // 如果传入的是函数，就采用这个形式
                speak: PropTypes.func,
            }
            // 指定默认标签属性值
            static defaultProps = {
                sex: '不男不女',
                age: 18
            }
            render() {
                console.log(this)
                const { name, age, sex } = this.props
                //props是只读的
                //this.props.name='jack'
                //此行代码会报错，因为props是只读的
                return (
                    <ul>
                        <li>姓名：{name}</li>
                        <li>性别：{sex}</li>
                        <li>年龄：{age + 1}</li>
                    </ul>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Person name="tom" age="19" sex="男" />, document.getElementById('test'))
        ReactDOM.render(<Person name="jerry" age={19} sex="男" />, document.getElementById('test1'))
  
        const p = { name: '老刘', age: 18, sex: '女' }
        ReactDOM.render(<Person {...p} />, document.getElementById('test2'))
    </script>
  </body>
  </html>
  ```
- 类式组件中的构造器与props
  
  ```
  // 创建组件
        class Person extends React.Component{
            constructor(props){
                super(props)
                console.log(props)
            }
            render(){
                console.log(this)
                const {name,age,sex}=this.props
                return (
                    <ul>
                        <li>姓名：{name}</li>
                        <li>性别：{sex}</li>
                        <li>年龄：{age}</li>
                    </ul>
                )
            }
        }
  ```
- 如果不初始化state或不进行方法绑定，则不需要为React组件实现构造函数
	- 在React组件挂载之前，会调用他的构造函数。在为React.Component子类实现构造函数时，应在其他语句之前调用super(props)。否则，this.props在构造函数中可能会出现未定义的bug。
- 通常，在React中，构造函数仅用于以下两种情况：
	- 通过给this.state赋值对象来初始化内部state。
	- 为事件处理函数绑定实例
- 在constructor（）函数中不要调用setState（）方法，如果你的组件需要使用内部state，请直接在构造函数中为this.state赋值初始化state。
- 函数式方法使用props
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
    <div id="test1"></div>
    <div id="test2"></div>
  
    <script type="text/babel">
        // 创建组件
        function Person(props){
            const {name,age,sex}=props
            return (
                <ul>
                    <li>姓名：{name}</li>
                    <li>年龄：{age}</li>
                    <li>性别：{sex}</li>
                </ul>
            )
        }
        // 渲染组件到页面
        ReactDOM.render(<Person name="tom" age="19" sex="男" />, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 在函数式组件中，只能使用props属性，不能使用state和之后要学习的ref属性。
- 总结
- 理解
  
  1.      每个组件对象都会有props(properties的简写)属性
  
  2.     组件标签的所有属性都保存在props中
- 作用
  
  1.      通过标签属性从组件外向组件内传递变化的数据
  
  2.     注意: 组件内部不要修改props数据
- ### 3.3.3.  Ref属性
- 组件内的标签可以定义ref属性来标识自己
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      // 创建类式组件
      class Demo extends React.Component{
        // 展示左侧输入框的数据内容
        showData=()=>{
          const{input1}=this.refs
          alert(input1.value)
        }
        showData2=()=>{
          const{input1}=this.refs
          alert(input1.value)
        }
        render(){
          return(
            <div>
              <input ref="input1" type="text" placeholder="点击按钮提示数据"/>
              <button onClick={this.showData}>点我提示</button>
              <input ref="input2" type="text" onBlur={this.showData2} placeholder="失去焦点提示数据"/>
            </div>
          )
        }
      }
      // 渲染组件到页面
      ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 使用ref之后就可以避免使用document.getElementById方法来对组件进行查找。
- 这里面使用的是字符串形式的。但是自从16.8之后，不再使用对应的内容了。
- ref的简写形式应用
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Demo extends React.Component{
            // 展示左侧输入框的数据内容
            showData=()=>{
                const{input1}=this
                alert(input1.value)
            }
            showData2=()=>{
                const{input2}=this
                alert(input2.value)
            }
            render(){
                return(
                    <div>
                        <input ref={c=>this.input1=c} type="text" placeholder="点击按钮提示数据"/>
                        <button onClick={this.showData}>点我提示</button>
                        <input ref={(a)=>this.input2=a} type="text" onBlur={this.showData2} placeholder="失去焦点提示数据"/>
                    </div>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 在React16版本之后，就不再使用字符串的形式进行ref注释了。原因主要是因为一旦使用过多之后，就会导致性能下降。
- 这里面使用了两种方式，在使用了这样的方式之后，就需要用this来进行对象绑定。
- Ref的回调问题
- 如果ref回调函数是以内联函数的方式定义的，在更新过程中他会被执行两次，第一次传入参数null，然后第二次会传入参数DOM元素。这是应为在每次渲染的时候会创建一个新的函数实例，所以React清空旧的ref并设置新的ref。通过将ref的回调函数定义成class的绑定函数的方式可以避免上述的问题，但是大多数情况下，这是无关精要的。
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Demo extends React.Component{
            state={isHot:false}
  
            showInfo=()=>{
                const {input1}=this
                alert(input1.value)
            }
  
            changeWeather=()=>{
                // 获取原来的状态
                const {isHot}=this.state
                // 更新状态
                this.setState({isHot:!isHot})
            }
  
            saveInput=(c)=>{
                this.input1=c
                console.log('@',c)
            }
  
            render(){
                const {isHot}=this.state
                return(
                    <div>
                        <h2>今天天气很{isHot?'炎热':'凉爽'}</h2>
                        {/*<input ref={(currentNode)=>{this.input1=currentNode}} type="text"/>*/}
                        <input ref={this.saveInput} type="text"/>
                        <br/><br/>
                        <button onClick={this.shoInfo}>点我提示输入的数据</button>
                        <button onClick={this.changeWeather}>点我切换天气</button> 
                    </div>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- createRef的使用
- 因为新版的React对String类型的refs实现了过期，因此可以使用回调函数或createRef API的方式代替。
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Demo extends React.Component{
            /*
                React.creatRef调用后可以返回一个容器，该容器可以存储被ref所标识的节点，该容器是专人专用的。
            */
            myRef=React.createRef()
            myRef2=React.createRef()
  
            showData=()=>{
                alert(this.myRef.current.value)
            }
  
            showData2=()=>{
                alert(this.myRef2.current.value)
            }
  
            render(){
                const {isHot}=this.state
                return(
                    <div>
                        {/*<input ref={(currentNode)=>{this.input1=currentNode}} type="text"/>*/}
                        <input ref={this.myRef} type="text"/>
                        <input ref={this.myRef2} type="text"/>
                        <br/><br/>
                        <button onClick={this.showData}>点我提示输入的数据</button>
                    </div>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- Ref事件处理
- 通过onXxx属性指定事件处理函数(注意大小写)
	- React使用的是自定义(合成)事件, 而不是使用的原生DOM事件
	- React中的事件是通过事件委托方式处理的(委托给组件最外层的元素)
- 通过event.target得到发生事件的DOM元素对象
- ## 3.4.  组件的生命周期
- 组件从创建到死亡它会经历一些特定的阶段。
- React组件中包含一系列勾子函数(生命周期回调函数), 会在特定的时刻调用。
- 我们在定义组件时，会在特定的生命周期回调函数中，做特定的工作。
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建组件
        // 生命周期回调函数 <=> 生命周期钩子函数
        class Life extends React.Component{
            state={
                opacity:1
            }
  
            death=()=>{
                // 卸载组件
                ReactDOM.unmountComponentAtNode(document.getElementById('test'))
            }
  
            // 组件挂载完毕
            componentDidMount(){
                setInterval(()=>{
                    // 获取原状态
                    let {opacity}=this.state
                    // 减小0.1
                    opacity-=0.1
                    if(opacity<=0) opacity=1
                    // 设置新的透明度
                    this.setState({opacity})
                },200)
            }
  
            // 组件将要卸载
            componentWillUnmount(){
                // 清楚定时器
                clearInterval(this.timer)
            }
  
            render(){
                return(
                    <div>
                        <h2 style={{opacity:this.state.opacity}}>React学习</h2>
                        <button onClick={this.death}>不活了</button>
                    </div>
                )
            }
        }
        // 渲染组件
        ReactDOM.render(<Life/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1616810898358-03b5baa8-ded8-4b43-baa0-54417530477f.png)
- forceUpdate函数
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建组件
        // 生命周期回调函数 <=> 生命周期钩子函数
        class Life extends React.Component{
            
            // 构造器
            constructor(props){
                console.log('count-constructor');
                super(props)
                // 初始化状态
                this.state={count:0}
            }
  
            // 加1按钮的回调
            add=()=>{
                // 获取原状态
                const {count}=this.state
                // 更新状态
                this.setSate({count:count+1})
            }
  
            // 卸载组件按钮的回调
            death=()=>{
                ReactDOM.unmountComponentAtNode(document.getElementById('test'))
            }
  
            // 强制更新按钮的回调
            force=()=>{
                // 函数体
                this.forceUpdate()
            }
  
            // 组件将要挂载的钩子
            componentWillMount(){
                console.log('Count---componentWillMount')
            }
  
            // 组件挂载完毕的钩子
            componentDidMount(){
                console.log('Count---componentDidMount')
            }
  
            // 组件将要卸载的钩子
            componentWillUnmount(){
                console.log('Count---componentWillUnmount')
            }
  
            // 控制组件更新的开关
            shouldComponentUpdate(){
                console.log('Count---shouldComponentUpdate')
                return true
            }
  
            // 组件将要更新的钩子
            componentWillUpdate(){
                console.log('Count---componentWillUpdate')
            }
  
            // 组件更新完毕的钩子
            componentDidUpdate(){
                console.log('Count---componentDidUpdate')
            }
            
            render(){
                const {count}=this.state
                return(
                    <div>
                        <h2>当前求和为：{count}</h2>
                        <button onClick={this.add}>点我+1</button>
                        <button onClick={this.death}>卸载组件</button>
                        <button onClick={this.forceUpdate}>不更改任何状态数据，强制更新一下</button>
                    </div>
                )
            }
        }
        // 渲染组件
        ReactDOM.render(<Life/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建组件
        // 生命周期回调函数 <=> 生命周期钩子函数
        class Life extends React.Component{
            state={
                opacity:1
            }
  
            death=()=>{
                // 卸载组件
                ReactDOM.unmountComponentAtNode(document.getElementById('test'))
            }
  
            // 组件挂载完毕
            componentDidMount(){
                setInterval(()=>{
                    // 获取原状态
                    let {opacity}=this.state
                    // 减小0.1
                    opacity-=0.1
                    if(opacity<=0) opacity=1
                    // 设置新的透明度
                    this.setState({opacity})
                },200)
            }
  
            // 组件将要卸载
            componentWillUnmount(){
                // 清楚定时器
                clearInterval(this.timer)
            }
  
            render(){
                return(
                    <div>
                        <h2 style={{opacity:this.state.opacity}}>React学习</h2>
                        <button onClick={this.death}>不活了</button>
                    </div>
                )
            }
        }
  
        class A extends React.Component{
            // 初始化状态
            state={carName:'奔驰'}
  
            changeCar=()=>{
                this.setState({carName:'奥拓'})
            }
  
            render(){
                return(
                    <div>
                        <div>A</div>
                        <button onClick={this.changeCar}>换车</button>
                        <B carName={this.state.carName}/>
                    </div>
                )
            }
        }
  
        class B extends React.Component{
  
            // 组件即将更新的时候
            componentWillReceiveProps(){
                console.log('B---componentWillReceiveProps')
            }
  
            // 控组组件更新的阀门
            shouldComponentUpdate(){
                console.log('B---shouldComponentUpdate')
                return true
            }
  
            // 组件将要更新的钩子
            componentWillUpdate(){
                console.log('B---componentWillUpdate')
            }
  
            // 组件更新完毕的钩子
            componentDidUpdate(){
                console.log('B---componentDidUpdate')
            }
  
            render(){
                console.log('B---render')
                return(
                    <div>我是B组件，接收到的车是:{this.props.carName}</div>
                )
            }
        }
        // 渲染组件
        ReactDOM.render(<Life/>,document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 初次渲染
  
  **1. ****初始化阶段****:** 由ReactDOM.render()触发---初次渲染
- constructor()
- componentWillMount()
- render()
- componentDidMount()
  
  **2.更新阶段****: **由组件内部this.setSate()或父组件重新render触发
- shouldComponentUpdate()
- componentWillUpdate()
- render()
- componentDidUpdate()
- 钩子函数的区别
	- 在16之后的版本内，对于三个Will的钩子函数，componentWillReceiveProps、componentWillMount、componentWillUpdate在17版本之后将会进入一个添加前缀的阶段UNSAFE_。
	- 此生命周期之前名为 componentWillMount。该名称将继续使用至 React 17。可以使用 rename-unsafe-lifecycles codemod 自动更新你的组件。
	  
	  ```
	  <!DOCTYPE html>
	  <html lang="en">
	  
	  <head>
	   <meta charset="UTF-8">
	   <meta http-equiv="X-UA-Compatible" content="IE=edge">
	   <meta name="viewport" content="width=device-width, initial-scale=1.0">
	   <title>Hello React</title>
	   <!-- 引入react核心库 -->
	   <script type="text/javascript" src="../js/17.0.1/react.development.js"></script>
	   <!-- 引入react-dom，用于支持react操作DOM -->
	   <script type="text/javascript" src="../js/17.0.1/react-dom.development.js"></script>
	   <!-- 引入babel，用于将jsx转为js -->
	   <script type="text/javascript" src="../js/17.0.1/babel.min.js"></script>
	   <!-- 引入prop-types,用于对组件标签属性进行限制-->
	   <script type="text.javascript" src="../js/17.0.1/prop-types.js"></script>
	  </head>
	  <body>
	   <!-- 准备好一个DOM容器-->
	   <div id="test"></div>
	  
	   <script type="text/babel">
	       // 创建组件
	       // 生命周期回调函数 <=> 生命周期钩子函数
	       class Life extends React.Component{
	           state={
	               opacity:1
	           }
	  
	           death=()=>{
	               // 卸载组件
	               ReactDOM.unmountComponentAtNode(document.getElementById('test'))
	           }
	  
	           // 组件挂载完毕
	           componentDidMount(){
	               setInterval(()=>{
	                   // 获取原状态
	                   let {opacity}=this.state
	                   // 减小0.1
	                   opacity-=0.1
	                   if(opacity<=0) opacity=1
	                   // 设置新的透明度
	                   this.setState({opacity})
	               },200)
	           }
	  
	           // 组件将要卸载
	           UNSAFE_componentWillUnmount(){
	               // 清楚定时器
	               clearInterval(this.timer)
	           }
	  
	           render(){
	               return(
	                   <div>
	                       <h2 style={{opacity:this.state.opacity}}>React学习</h2>
	                       <button onClick={this.death}>不活了</button>
	                   </div>
	               )
	           }
	       }
	  
	       class A extends React.Component{
	           // 初始化状态
	           state={carName:'奔驰'}
	  
	           changeCar=()=>{
	               this.setState({carName:'奥拓'})
	           }
	  
	           render(){
	               return(
	                   <div>
	                       <div>A</div>
	                       <button onClick={this.changeCar}>换车</button>
	                       <B carName={this.state.carName}/>
	                   </div>
	               )
	           }
	       }
	  
	       class B extends React.Component{
	  
	           // 组件即将更新的时候
	           UNSAFE_componentWillReceiveProps(){
	               console.log('B---componentWillReceiveProps')
	           }
	  
	           // 控组组件更新的阀门
	           shouldComponentUpdate(){
	               console.log('B---shouldComponentUpdate')
	               return true
	           }
	  
	           // 组件将要更新的钩子
	           UNSAFE_componentWillUpdate(){
	               console.log('B---componentWillUpdate')
	           }
	  
	           // 组件更新完毕的钩子
	           componentDidUpdate(){
	               console.log('B---componentDidUpdate')
	           }
	  
	           render(){
	               console.log('B---render')
	               return(
	                   <div>我是B组件，接收到的车是:{this.props.carName}</div>
	               )
	           }
	       }
	       // 渲染组件
	       ReactDOM.render(<Life/>,document.getElementById('test'))
	   </script>
	  </body>
	  </html>
	  ```
- 新旧生命周期对比
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1616897995023-714dfc8e-45af-4c03-9a0d-61aa034be469.png)
- 可以发现，新的周期里面除了舍弃了三个钩子函数之外，还添加了两个函数。
- getDerivedStateFromProps函数
	- getDerivedStateFromProps 会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。
	- 此方法适用于罕见的用例，即 state 的值在任何时候都取决于 props。例如，实现 <Transition> 组件可能很方便，该组件会比较当前组件与下一组件，以决定针对哪些组件进行转场动画。
	- 派生状态会导致代码冗余，并使组件难以维护。 确保你已熟悉这些简单的替代方案：
	- 如果你需要执行副作用（例如，数据提取或动画）以响应 props 中的更改，请改用 componentDidUpdate。
		- 如果只想在 prop 更改时重新计算某些数据，请使用 memoization helper 代替。
		- 如果你想在 prop 更改时“重置”某些 state，请考虑使组件完全受控或使用 key 使组件完全不受控 代替。
		- 此方法无权访问组件实例。如果你需要，可以通过提取组件 props 的纯函数及 class 之外的状态，在getDerivedStateFromProps()和其他 class 方法之间重用代码。
	- 请注意，不管原因是什么，都会在每次渲染前触发此方法。这与 UNSAFE_componentWillReceiveProps 形成对比，后者仅在父组件重新渲染时触发，而不是在内部调用 setState 时。
- getSnapshotBeforeUpdate钩子函数
	- getSnapshotBeforeUpdate() 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 componentDidUpdate()。
	- 此用法并不常见，但它可能出现在 UI 处理中，如需要以特殊方式处理滚动位置的聊天线程等。
	- 应返回 snapshot 的值（或 null）。
	- 在上述示例中，重点是从 getSnapshotBeforeUpdate 读取 scrollHeight 属性，因为 “render” 阶段生命周期（如 render）和 “commit” 阶段生命周期（如 getSnapshotBeforeUpdate 和 componentDidUpdate）之间可能存在延迟。
	  
	  ```
	  <!DOCTYPE html>
	  <html lang="en">
	  
	  <head>
	   <meta charset="UTF-8">
	   <meta http-equiv="X-UA-Compatible" content="IE=edge">
	   <meta name="viewport" content="width=device-width, initial-scale=1.0">
	   <title>Hello React</title>
	   <!-- 引入react核心库 -->
	   <script type="text/javascript" src="../js/17.0.1/react.development.js"></script>
	   <!-- 引入react-dom，用于支持react操作DOM -->
	   <script type="text/javascript" src="../js/17.0.1/react-dom.development.js"></script>
	   <!-- 引入babel，用于将jsx转为js -->
	   <script type="text/javascript" src="../js/17.0.1/babel.min.js"></script>
	   <!-- 引入prop-types,用于对组件标签属性进行限制-->
	   <script type="text.javascript" src="../js/17.0.1/prop-types.js"></script>
	  </head>
	  <body>
	   <!-- 准备好一个DOM容器-->
	   <div id="test"></div>
	  
	   <script type="text/babel">
	       // 创建组件
	       // 生命周期回调函数 <=> 生命周期钩子函数
	       class Life extends React.Component{
	           state={
	               opacity:1
	           }
	  
	           death=()=>{
	               // 卸载组件
	               ReactDOM.unmountComponentAtNode(document.getElementById('test'))
	           }
	  
	           // 组件挂载完毕
	           componentDidMount(){
	               setInterval(()=>{
	                   // 获取原状态
	                   let {opacity}=this.state
	                   // 减小0.1
	                   opacity-=0.1
	                   if(opacity<=0) opacity=1
	                   // 设置新的透明度
	                   this.setState({opacity})
	               },200)
	           }
	  
	           // 组件将要卸载
	           UNSAFE_componentWillUnmount(){
	               // 清楚定时器
	               clearInterval(this.timer)
	           }
	  
	           render(){
	               return(
	                   <div>
	                       <h2 style={{opacity:this.state.opacity}}>React学习</h2>
	                       <button onClick={this.death}>不活了</button>
	                   </div>
	               )
	           }
	       }
	  
	       class A extends React.Component{
	           // 初始化状态
	           state={carName:'奔驰'}
	  
	           changeCar=()=>{
	               this.setState({carName:'奥拓'})
	           }
	  
	           render(){
	               return(
	                   <div>
	                       <div>A</div>
	                       <button onClick={this.changeCar}>换车</button>
	                       <B carName={this.state.carName}/>
	                   </div>
	               )
	           }
	       }
	  
	       class B extends React.Component{
	  
	           // 组件即将更新的时候
	           UNSAFE_componentWillReceiveProps(){
	               console.log('B---componentWillReceiveProps')
	           }
	  
	           // 控组组件更新的阀门
	           shouldComponentUpdate(){
	               console.log('B---shouldComponentUpdate')
	               return true
	           }
	  
	           // 组件将要更新的钩子
	           UNSAFE_componentWillUpdate(){
	               console.log('B---componentWillUpdate')
	           }
	  
	           // 若state的值在任何时候都取决于props，那么可以使用getDerivedStateFromProps
	           static getDerivedStateFromProps(props,state){
	               console.log('getDerivedStateFromProps',props,state)
	               return null
	           }
	  
	           getSnapshotBeforeUpdate(){
	               console.log('getSnapshotBeforeUpdate')
	               return 'atguigu'
	           }
	  
	           // 组件更新完毕的钩子
	           componentDidUpdate(preProps,proState,snapshotValue){
	               console.log('B---componentDidUpdate',preProps,proState,snapshotValue)
	           }
	  
	           render(){
	               console.log('B---render')
	               return(
	                   <div>我是B组件，接收到的车是:{this.props.carName}</div>
	               )
	           }
	       }
	       // 渲染组件
	       ReactDOM.render(<Life/>,document.getElementById('test'))
	   </script>
	  </body>
	  </html>
	  ```
- 示例
	- 这次通过完成一个案例来总结新的生命周期。
	- 主要实现的功能就是：
		- 每隔一段时间，就会出现一个新的新闻，然后固定在相应的地方。
		  
		  ```
		  <!DOCTYPE html>
		  <html lang="en">
		  
		  <head>
		  <meta charset="UTF-8">
		  <meta http-equiv="X-UA-Compatible" content="IE=edge">
		  <meta name="viewport" content="width=device-width, initial-scale=1.0">
		  <title>Hello React</title>
		  <!-- 引入react核心库 -->
		  <script type="text/javascript" src="../js/17.0.1/react.development.js"></script>
		  <!-- 引入react-dom，用于支持react操作DOM -->
		  <script type="text/javascript" src="../js/17.0.1/react-dom.development.js"></script>
		  <!-- 引入babel，用于将jsx转为js -->
		  <script type="text/javascript" src="../js/17.0.1/babel.min.js"></script>
		  <!-- 引入prop-types,用于对组件标签属性进行限制-->
		  <script type="text.javascript" src="../js/17.0.1/prop-types.js"></script>
		  </head>
		  <body>
		  <!-- 准备好一个DOM容器-->
		  <div id="test"></div>
		  
		  <script type="text/babel">
		      /*
		      	1. 初始化阶段: 由ReactDOM.render()触发---初次渲染
		                  1.	constructor()
		                  2.	componentWillMount()
		                  3.	render()
		                  4.	componentDidMount()
		       2. 更新阶段: 由组件内部this.setSate()或父组件重新render触发
		                  1.	shouldComponentUpdate()
		                  2.	componentWillUpdate()
		                  3.	render()
		                  4.	componentDidUpdate()
		       3. 卸载组件: 由ReactDOM.unmountComponentAtNode()触发
		                  1.	componentWillUnmount()
		      */
		      class NewsList extends React.Component{
		          state={newsArr:[]}
		  
		          componentDidMount(){
		              setInterval(()=>{
		                  // 获取原状态
		                  const {newsArr}=this.state
		                  // 模拟一条新闻
		                  const news='新闻'+(newsArr.length+1)
		                  // 更新状态
		                  this.setState({newsArr:[news,...newsArr]})
		              },1000)
		          }
		  
		          getSnapshotBeforeUpdate(){
		              return this.refs.list.list.scrollHeight
		          }
		  
		          componentDidUpdate(preProps,proState,height){
		              this.refs.list.scrollTop+=this.refs.list.scrollHeight-height
		          }
		  
		          render(){
		              return(
		                  <div className="list" ref="list">
		                      {
		                          this.state.newsArr.map((n,index)=>{
		                              return <div key={index} className="news">{n}</div>
		                          })
		                      }
		                  </div>
		              )
		          }
		      }
		  </script>
		  </body>
		  </html>
		  ```
- ## 3.5.  事件处理与是否可控组件
- 事件处理
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Demo extends React.Component{
            /*
                (1).通过onXxx属性指定时间处理函数（注意大小写）
                    a.React使用的是自定义时间，而不是使用的原生DOM时间——为了更好的兼容性
                    b.React中的事件是通过事件委托方式处理的——为了高效
                (2).通过event.target得到发生时间的DOM元素对象
                React.creatRef调用后可以返回一个容器，该容器可以存储被ref所标识的节点，该容器是专人专用的。
                注意：请不要过度使用refs
            */
            myRef=React.createRef()
            myRef2=React.createRef()
  
            showData=()=>{
                alert(this.myRef.current.value)
            }
  
            showData2=()=>{
                alert(this.myRef2.current.value)
            }
  
            render(){
                const {isHot}=this.state
                return(
                    <div>
                        {/*<input ref={(currentNode)=>{this.input1=currentNode}} type="text"/>*/}
                        <input ref={this.myRef} type="text"/>
                        <input ref={this.myRef2} type="text"/>
                        <br/><br/>
                        <button onClick={this.showData}>点我提示输入的数据</button>
                    </div>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 受控组件和非受控组件的区别：受控组件就是现提现用；非受控组件就是直接提取。按照react的提示，要减少使用ref。
- 受控组件示例：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Login extends React.Component{
  
            // 初始化状态
            state={
                username:'',// 用户名
                password:''// 姓名
            }
  
            saveUsername=(event)=>{
                this.setState({username:event.target.value})
            }
  
            savePassword=(event)=>{
                this.setState({password:event.target.value})
            }
  
            handleSubmit = (event)=>{
                event.preventDefault() //阻止表单提交
                const {username,password}=this
                alert(`你输入的用户名是：${username},你输入的密码是：${password}`)
            }
  
            render(){
                return(
                    <form onSubmit={this.handleSubmit}>
                        用户名：<input type="text" onChange={this.saveUsername} name="username"/>
                        密码：<input type="password" name="password" onChange={this.savePassword}/>
                        <button>登录</button> 
                    </form>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- 非受控组件示例：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
        // 创建类式组件
        class Login extends React.Component{
            handleSubmit = (event)=>{
                event.preventDefault() //阻止表单提交
                const {username,password}=this
                alert(`你输入的用户名是：${username.value},你输入的密码是：${password.value}`)
            }
  
            render(){
                return(
                    <form onSubmit={this.handleSubmit}>
                        用户名：<input type="text" ref={c=>this.username=c} name="username"/>
                        密码：<input type="password" name="password" ref={c=>this.password=c}/>
                        <button>登录</button> 
                    </form>
                )
            }
        }
        // 渲染组件到页面
        ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- ## 3.6.  高阶函数和柯里化
- 高阶函数：如果一个函数符合下面2个规范中的任何一个，那该函数就是高阶函数：
	- 1.若A函数，接收的参数是一个函数，那么A就可以称之为高阶函数。
	- 2.若A函数，调用的返回值依然是一个函数，那么A就可以称为是高阶函数
	- 常见的高阶函数有：Promise、setTimeout、arr.map()等等
- 函数的柯里化：通过函数调用继续返回函数的方式，实现多次接收参数最后统一处理的函数编码形式
- 柯里化示例
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello React</title>
    <!-- 引入react核心库 -->
    <script type="text/javascript" src="../js/react.development.js"></script>
    <!-- 引入react-dom，用于支持react操作DOM -->
    <script type="text/javascript" src="../js/react-dom.development.js"></script>
    <!-- 引入babel，用于将jsx转为js -->
    <script type="text/javascript" src="../js/babel.min.js"></script>
    <!-- 引入prop-types,用于对组件标签属性进行限制-->
    <script type="text.javascript" src="../js/prop-types.js"></script>
  </head>
  <body>
    <!-- 准备好一个DOM容器-->
    <div id="test"></div>
  
    <script type="text/babel">
      /*
            高阶函数：如果一个函数符合下面2个规范中的任何一个，那该函数就是高阶函数：
                1.若A函数，接收的参数是一个函数，那么A就可以称之为高阶函数。
                2.若A函数，调用的返回值依然是一个函数，那么A就可以称为是高阶函数
                常见的高阶函数有：Promise、setTimeout、arr.map()等等
            函数的柯里化：通过函数调用继续返回函数的方式，实现多次接收参数最后统一处理的函数编码形式
        */
      // 创建类式组件
      class Login extends React.Component{
  
        // 初始化状态
        state={
          username:'',// 用户名
          password:''// 姓名
        }
  
        // 保存表单数据到状态中
        saveFormData=(dataType)=>{
          return (event)=>{
            this.setState({[dataType]:event.target.value})
          }
        }
  
        handleSubmit = (event)=>{
          event.preventDefault() //阻止表单提交
          const {username,password}=this
          alert(`你输入的用户名是：${username},你输入的密码是：${password}`)
        }
  
        render(){
          return(
            <form onSubmit={this.handleSubmit}>
              用户名：<input type="text" onChange={this.saveFormData} name="username"/>
              密码：<input type="password" name="password" onChange={this.saveFormData}/>
              <button>登录</button> 
            </form>
          )
        }
      }
      // 渲染组件到页面
      ReactDOM.render(<Demo a="1" b="2"/>, document.getElementById('test'))
    </script>
  </body>
  </html>
  ```
- ## 3.7.  Proxy代理
- 1.     React本身只关注于界面, 并不包含发送ajax请求的代码
- 2.     前端应用需要通过ajax请求与后台进行交互(json数据)
- 3.     react应用中需要集成第三方ajax库(或自己封装)
- 常用的ajax请求库
	- 1.     jQuery: 比较重, 如果需要另外引入不建议使用
	- 2.     axios: 轻量级, 建议使用
		- 1)      封装XmlHttpRequest对象的ajax
		- 2)      promise风格
		- 3)     可以用在浏览器端和node服务器端
- 代理示例1
  
  ```
  import React, { Component } from 'react'
  import axios from 'axios'
  
  export default class App extends Component {
  getStudentData=()=>{
    axios.get('http://localhost:3000/students').then(
      response=>{
        console.log('成功了',response.data)
      },
      error=>{
        console.log('失败了',error)
      }
    )
  }
  
  render() {
    return (
      <div>
      <button onClick={this.getStudentData}>点击我获取数据</button>
      </div>
    )
  }
  }
  ```
  
  ```
  // 引入react核心库
  import React from 'react'
  // 引入ReactDOM
  import ReactDOM from 'react-dom'
  // 引入App组件
  import App from './App'
  
  // 渲染App到页面
  ReactDOM.render(<App/>,document.getElementById('root'))
  ```
  
  ```
  const express = require('express')
  const app = express()
  
  app.use((request,response,next)=>{
  console.log('有人请求服务器1了');
  console.log('请求来自于',request.get('Host'));
  console.log('请求的地址',request.url);
  next()
  })
  
  app.get('/students',(request,response)=>{
  const students = [
  {id:'001',name:'tom',age:18},
  {id:'002',name:'jerry',age:19},
  {id:'003',name:'tony',age:120},
  ]
  response.send(students)
  })
  
  app.listen(5000,(err)=>{
  if(!err) console.log('服务器1启动成功了,请求学生信息地址为：http://localhost:5000/students');
  })
  ```
- 首先要开启服务器，然后在package.json中添加一个proxy的属性，这就是代理，代理到相应的网址上去。
- React的proxy代理第二种方式
- 第一步：创建代理配置文件
	- 在src下创建配置文件：src/setupProxy.js
- 编写setupProxy.js配置具体代码
  
  ```
  const proxy = require('http-proxy-middleware')
  
  module.exports = function(app) {
  app.use(
    proxy('/api1', {  //api1是需要转发的请求(所有带有/api1前缀的请求都会转发给5000)
      target: 'http://localhost:5000', //配置转发目标地址(能返回数据的服务器地址)
      changeOrigin: true, //控制服务器接收到的请求头中host字段的值
      /*
      	changeOrigin设置为true时，服务器收到的请求头中的host为：localhost:5000
      	changeOrigin设置为false时，服务器收到的请求头中的host为：localhost:3000
      	changeOrigin默认值为false，但我们一般将changeOrigin值设为true
      */
      pathRewrite: {'^/api1': ''} //去除请求前缀，保证交给后台服务器的是正常请求地址(必须配置)
    }),
    proxy('/api2', { 
      target: 'http://localhost:5001',
      changeOrigin: true,
      pathRewrite: {'^/api2': ''}
    })
  )
  }
  ```
- 说明：
- 优点：可以配置多个代理，可以灵活的控制请求是否走代理。
- 缺点：配置繁琐，前端请求资源时必须加前缀。
- ## 3.8.  SetState
- setState更新状态的2种写法
	- （1）setState（stateChange，[callback]）————对象式的setState
		- stateChange为状态改变对象（该对象可以体现出状态的更改）
		- callback是可选回调函数，他在状态更新完毕、界面也更新后才能被调用
	- （2）setState（updater，[callback]）————函数式的setState
		- updater为返回stateChange对象的函数
		- updater可以接收到state和props
		- callback是可选的回调函数，在状态更新、界面也更新后才会被调用。
- 总结：
	- 对象式的setState是函数式的setState的简写方式（语法糖）
	- 使用原则：
		- 如果新状态不依赖于原状态=>使用对象方式
		- 如果新状态依赖于原状态=>使用函数方式
		- 如果需要在setState()执行后获取最新的状态数据，要在第二个callback函数中读取。
		  
		  ```
		  import React, { Component } from 'react'
		  
		  export default class Demo extends Component {
		  
		  state={count:0}
		  
		  add=()=>{
		      //1.获取原来的count
		      const {count}=this.state
		      //2.更新状态
		      this.setState({count:count+1},()=>{
		          console.log(this.state.count)
		      })
		      console.log('输出为',this.state.count)
		  
		      // 函数式setState
		      // this.setState(state=>({count:state.count+1}))
		  }
		  
		  render() {
		      return (
		          <div>
		              <h1>当前求和为：{this.state.count}</h1>
		              <button onClick={this.add}>点我+1</button>
		          </div>
		      )
		  }
		  }
		  ```
- ## 3.9.  LazyLoad和Hooks
  
  ```
  //1.通过React的lazy函数配合import()函数动态加载路由组件 ===> 路由组件代码会被分开打包
  const Login = lazy(()=>import('@/pages/Login'))
  
  //2.通过<Suspense>指定在加载得到路由打包文件前显示一个自定义loading界面
  <Suspense fallback={<h1>loading.....</h1>}>
        <Switch>
            <Route path="/xxx" component={Xxxx}/>
            <Redirect to="/login"/>
        </Switch>
    </Suspense>
  ```
- 示例
  
  ```
  import React, { Component,lazy, Suspense } from 'react'
  import { NavLink, Route } from 'react-router-dom'
  // import About from './components/About'
  // import Home from './components/Home'
  
  const Home = lazy(()=>import('./Home'))
  const About = lazy(()=>import('./About'))
  
  export default class Demo extends Component {
  
    render() {
        return (
            <div>
                <div className="row">
                    <div className="col-xs-offset-2 col-xs-8">
                        <div className="page-header"><h2>React Router Demo</h2></div>
                    </div>
                </div>
                <div className="row">
                    <div className="col-xs-2 col-xs-offset-2">
                        <div className="list-group">
                            {/* 在React中靠路由链接实现切换组件 */}
                            <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink>
                        </div>
                    </div>
                    <div className="col-xs-6">
                        <div className="panel">
                            <div className="panel-body">
                                <Suspense fallback={<h1>Loading......</h1>}>
                                    {/* 注册路由 */}
                                    <Route path="/about" component={About} />
                                    <Route path="/home" component={Home} />
                                </Suspense>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        )
    }
  }
  ```
- lazyLoad的用法就是为了不同时加载所有的路径而实现的一种路由组件。
- ## 3.10.  EffectHook与RefHook
- **EffectHook**
- (1). Effect Hook 可以让你在函数组件中执行副作用操作(用于模拟类组件中的生命周期钩子)
- (2). React中的副作用操作:
  
        发ajax请求数据获取
  
        设置订阅 / 启动定时器
  
        手动更改真实DOM
- (3). 语法和说明:  
  
        useEffect(() => {  
  
          // 在此可以执行任何带副作用操作
  
          return () => { // 在组件卸载前执行
  
            // 在此做一些收尾工作, 比如清除定时器/取消订阅等
  
          }
  
        }, [stateValue]) // 如果指定的是[], 回调函数只会在第一次render()后执行
- (4). 可以把 useEffect Hook 看做如下三个函数的组合
  
        componentDidMount()
  
        componentDidUpdate()
  
    	componentWillUnmount()
- **RefHook**
- (1). Ref Hook可以在函数组件中存储/查找组件内的标签或任意其它数据
- (2). 语法: const refContainer = useRef()
- (3). 作用:保存标签对象,功能与React.createRef()一样
  
  ```
  import React, { Component } from 'react'
  
  function Demo(){
  
    const[count,setCount] = React.useState(0)
    const[name,setName] = React.useState('tom')
    const myRef=React.useRef()
  
    React.useEffect(()=>{
        let timer = setInterval(()=>{
            setCount(count=>count+1)
        },1000)
        return ()=>{
            clearInterval(timer)
        }
    },[])
  
    // 加的回调
    function add(){
        // setCount(count+1) // 第一种写法
        setCount(count => count+1)
    }
  
    // 提示输入的回调
    function show(){
        alert(myRef.current.value)
    }
    // 修改名字的回调
    function changeName(){
        setName('jack')
    }
    return (
        <div>
            <input type="text" ref={myRef} />
            <h2>当前求和为：{count}</h2>
            <h2>我的名字是：{name}</h2>
            <button onClick={add}>点我+1</button>
            <button onClick={changeName}>点我修改名字</button>
            <button onClick={show}>点我显示数据</button>
        </div>
    )
  }
  
  export default Demo
  ```
- ## 3.11.  Fragment和Context
- **Fragment**
- 作用就是可以不用必须有一个真实的DOM根标签。
- **这个标签不能携带任何的属性**
  
  ```
  import React, { Component, Fragment } from 'react'
  
  export default class Demo extends Component {
    render() {
        return (
            <Fragment>
                <input type="text"/>
                <input type="text"/>
            </Fragment>
        )
    }
  }
  ```
- **Context**
- 是一种组件间通信方式，常用于【祖组件】与【后代组件】间通信
- 1) 创建Context容器对象：
  
  const XxxContext = React.createContext()
- 2) 渲染子组时，外面包裹xxxContext.Provider, 通过value属性给后代组件传递数据：
  
  <xxxContext.Provider value={数据}>
  
              子组件
  
        </xxxContext.Provider>
- 3) 后代组件读取数据：
  
  //第一种方式:仅适用于类组件  
  
   static contextType = xxxContext  // 声明接收context
  
   this.context // 读取context中的value数据
  
   
  
  //第二种方式: 函数组件与类组件都可以
  
   <xxxContext.Consumer>
  
     {
  
       value => ( // value就是context中的value数据
  
         要显示的内容
  
       )
  
     }
  
   </xxxContext.Consumer>
- 在应用开发中一般不用context, 一般都用它的封装react插件
  
  ```
  import React, { Component } from 'react'
  
  // 创建Context对象
  const MyContext=React.createContext()
  const {Provider}=MyContext
  export default class A extends Component {
  
    state={username:'tom',age:18}
  
    render() {
        const {username,age}=this.state
        return (
            <div className="parent">
                <h3>我是A组件</h3>
                <h4>我的用户名是：{username}</h4>
                <Provider value={username}>
                    <B/>
                </Provider>
            </div>
        )
    }
  }
  
  class B extends Component{
    render(){
        return (
            <div className="child">
                <h3>我是B组件</h3>
                <C/>
            </div>
        )
    }
  }
  
  class C extends Component{
    static contextType=MyContext
    render(){
        console.log(this.context)
        return (
            <div className="grant">
                <h3>我是C组件</h3>
                <h4>我是从A组件接收到的用户名：{this.context}</h4>
            </div>
        )
    }
  }
  ```
- ## 3.12.  PureComponent
- Component的2个问题
- 只要执行setState(),即使不改变状态数据, 组件也会重新render() ==> 效率低
- 只当前组件重新render(), 就会自动重新render子组件，纵使子组件没有用到父组件的任何数据 ==> 效率低
- 只有当组件的state或props数据发生改变时才重新render（）
- Component中的shouldComponentUpdate（）总是返回true
- 解决办法
	- 办法1:
		- 重写shouldComponentUpdate()方法
		- 比较新旧state或props数据, 如果有变化才返回true, 如果没有返回false
	- 办法2:
		- 使用PureComponent
		- PureComponent重写了shouldComponentUpdate(), 只有state或props数据有变化才返回true
	- 注意:
		- 只是进行state和props数据的浅比较, 如果只是数据对象内部数据变了, 返回false
		- 不要直接修改state数据, 而是要产生新数据
	- 项目中一般使用PureComponent来优化
- 示例
  
  ```
  import React, { PureComponent } from 'react'
  
  export default class Parent extends PureComponent {
  
    state = {carName:"奔驰c36",stus:['小张','小李','小王']}
  
    addStu=()=>{
        const {stus}=this.state
        this.setState({stus:['小刘',...stus]})
    }
  
    changeCar = ()=>{
        this.setState({carName:"迈巴赫"})
    }
  
    shouldComponentUpdate(nextProps,nextState){
        console.log(this.props,this.state) // 目前的props和state
        console.log(nextProps,nextState)
        return !this.state.carName===nextState.carName
    }
  
    render() {
        return (
            <div className="parent">
                <h3>我是Parent组件</h3>
                <span>我的车名字是：{carName}</span>
                <button onClick={this.changeCar}>点我换车</button>
                <Child carName="奥拓"/>
            </div>
        )
    }
  }
  
  class Child extends PureComponent {
  
    // shouldComponentUpdate(nextProps,nextState){
    //     console.log(this.props,this.state) // 目前的props和state
    //     console.log(nextProps,nextState)
    //     return !this.state.carName===nextState.carName
    // }
  
    render() {
        return (
            <div className="child">
                <h3>我是Child组件</h3>
                <span>我的车名字是：{this.props.carName}</span>
            </div>
        )
    }
  }
  ```
- ## 3.13.  Renderprops
- Vue中:
	- 使用slot技术, 也就是通过组件标签体传入结构  <A><B/></A>
- React中:
	- 使用children props: 通过组件标签体传入结构
	- 使用render props: 通过组件标签属性传入结构,而且可以携带数据，一般用render函数属性
- childprops
  
  <A>
  
  <B>xxxx</B>
  
  </A>
  
  {this.props.children}
  
  问题: 如果B组件需要A组件内的数据, ==> 做不到
- renderprops
  
  <A render={(data) => <C data={data}></C>}></A>
  
  A组件: {this.props.render(内部state数据)}
  
  C组件: 读取A组件传入的数据显示 {this.props.data} 
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class Parent extends Component {
    render() {
        return (
            <div className="parent">
                <h3>我是Parent组件</h3>
                <A render={(name)=><B name={name}/>}/>
            </div>
        )
    }
  }
  
  class A extends Component {
    state={name:'tom'}
    render() {
        console.log(this.props)
        const {name}=this.state
        return (
            <div className="a">
                <h3>我是A组件</h3>
                {this.props.render(name)}
            </div>
        )
    }
  }
  
  class B extends Component {
    render() {
        console.log('B--render')
        return (
            <div className="b">
                <h3>我是B组件</h3>
            </div>
        )
    }
  }
  ```
- ## 3.14.  错误边界
- 错误边界(Error boundary)：用来捕获后代组件错误，渲染出备用页面。
- 只能捕获后代组件生命周期产生的错误，不能捕获自己组件产生的错误和其他组件在合成事件、定时器中产生的错误。
- 错误边界的使用范围必须是在容易出错的组件的父组件上使用才可以。
- 使用方法：getDerivedStateFromError配合componentDidCatch
	- 父组件
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Parent extends Component {
	  
	   state={
	       hasError:'' // 用于标识子组件是否产生错误
	   }
	  
	   // 当Parent的子组件出现报错时候，会触发getDerivedStateFromError调用，并携带错误信息
	   static getDerivedStateFromError(error){
	       console.log('@@@',error)
	       return {hasError:error}
	   }
	  
	   render() {
	       return (
	           <div>
	               <h2>我是Parent组件</h2>
	               {this.state.hasError?<h2>当前网络不稳定，稍后再试</h2>:<Child/>}
	           </div>
	       )
	   }
	  }
	  ```
	- 子组件
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Child extends Component {
	   state={
	       // users:[
	       //     {id:'001',name:'tom',age:18},
	       //     {id:'002',name:'jack',age:19},
	       //     {id:'003',name:'peiqi',age:20}
	       // ]
	       users:'abc'
	   }
	   render() {
	       return (
	           <div>
	               <h2>我是Child组件</h2>
	               {
	                   this.state.users.map((userObj)=>{
	                       return <h4 key={userObj.id}>{userObj.name}-----{userObj.age}</h4>
	                   })
	               }
	           </div>
	       )
	   }
	  }
	  ```
- ## 3.15.  组件间通信
- 通信方式总结
  
  1.props：
  
  (1).children props
  
  (2).render props
  
  2.消息订阅-发布：
  
  pubs-sub、event等等
  
  3.集中式管理：
  
  redux、dva等等
  
  4.conText:
  
  生产者-消费者模式
- 配合方式
  
  父子组件：props
  
  兄弟组件：消息订阅-发布、集中式管理
  
  祖孙组件(跨级组件)：消息订阅-发布、集中式管理、conText(开发用的少，封装插件用的多)
- # 4.  重要组件
- ## 4.1.  路由
- SPA指的是单页的Web应用（Single page web application）
- 整个应用只有一个完整的页面。
- 点击页面中的链接不会刷新页面，只会做页面的局部更新
- 数据都需要通过ajax请求获取，并在前端异步展现。
- ### 4.1.1.  介绍
- 介绍
	- 1.    一个路由就是一个映射关系(key:value)
	- 2.    key为路径, value可能是function或component
- 分类
	- 1.    后端路由：
	- 1)    理解： value是function, 用来处理客户端提交的请求。
	- 2)    注册路由：router.get(path, function(req, res))
	- 3)    工作过程：当node接收到一个请求时, 根据请求路径找到匹配的路由, 调用路由中的函数来处理请求, 返回响应数据
- 2.   前端路由：
	- 1)    浏览器端路由，value是component，用于展示页面内容。
	- 2)    注册路由: <Route path="/test" component={Test}>
	- 3)    工作过程：当浏览器的path变为/test时, 当前路由组件就会变为Test组件
- History
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>前端路由的基石_history</title>
  </head>
  <body>
    <a href="http://www.atguigu.com" onclick="return push('/test1') ">push test1</a><br><br>
    <button onClick="push('/test2')">push test2</button><br><br>
    <button onClick="replace('/test3')">replace test3</button><br><br>
    <button onClick="back()">&lt;= 回退</button>
    <button onClick="forword()">前进 =&gt;</button>
  
    <script type="text/javascript" src="https://cdn.bootcss.com/history/4.7.2/history.js"></script>
    <script type="text/javascript">
      // let history = History.createBrowserHistory() //方法一，直接使用H5推出的history身上的API
      let history = History.createHashHistory() //方法二，hash值（锚点）
  
      function push (path) {
        history.push(path)
        return false
      }
  
      function replace (path) {
        history.replace(path)
      }
  
      function back() {
        history.goBack()
      }
  
      function forword() {
        history.goForward()
      }
  
      history.listen((location) => {
        console.log('请求路由路径变化了', location)
      })
    </script>
  </body>
  </html>
  ```
- 在整个前端的路由中，表现出来的就是history这一个堆栈对于路由的掌控。这就是前端实现路由转换的基石。
- ### 4.1.2.  路由的基本使用
- react-router-dom的理解
	- 1.     react的一个插件库。
	- 2.     专门用来实现一个SPA应用。
	- 3.     基于react的项目基本都会用到此库。
- 示例
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="./css/bootstrap.css">
  </head>
  
  <body>
    <div id="root">
    </div>
  </body>
  </html>
  ```
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import About from './components/About'
  import Home from './components/Home'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <div className="page-header"><h2>React Router Demo</h2></div>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    <Link className="lislist-group-item" to="/about">About</Link>
      <Link className="lislist-group-item" to="/home">Home</Link>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  // 引入react核心库
  import React from 'react'
  // 引入ReactDOM
  import ReactDOM from 'react-dom'
  // 引入BrowserRouter组件
  import {BrowserRouter} from 'react-router-dom'
  // 引入App组件
  import App from './App'
  
  // 渲染App到页面
  ReactDOM.render(
  <BrowserRouter>
  <App/>
  </BrowserRouter>,
  document.getElementById('root'))
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class About extends Component {
  render() {
    return (
      <h3>
      这是About页面。
      </h3>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Home extends Component {
  render() {
    return (
      <h3>
      这是Home页面。
      </h3>
    )
  }
  }
  ```
- 使用BrowserRouter标签的时候，要使用在同一个标签下才能起作用，否则无法起作用。
- ### 4.1.3.  路由组件
- 一般组件如果不传props的话，就是在传输过程中不会出现任何props。但是路由组件即使不传，也会带上这样的三个属性对象。
  
  ```
  history:
      action: "PUSH"
      block: ƒ block(prompt)
      createHref: ƒ createHref(location)
      go: ƒ go(n)
      goBack: ƒ goBack()
      goForward: ƒ goForward()
      length: 3
      listen: ƒ listen(listener)
      location: {pathname: "/about", search: "", hash: "", state: null, key: "sevxl7"}
      push: ƒ push(path, state)
      replace: ƒ replace(path, state)
      __proto__: Object
  location:
      hash: ""
      key: "sevxl7"
      pathname: "/about"
      search: ""
      state: null
      __proto__: Object
  match:
      isExact: true
      params: {}
      path: "/about"
      url: "/about"
      __proto__: Object
      staticContext: undefined
  __proto__:
      constructor: ƒ Object()
      hasOwnProperty: ƒ hasOwnProperty()
      isPrototypeOf: ƒ isPrototypeOf()
      propertyIsEnumerable: ƒ propertyIsEnumerable()
      toLocaleString: ƒ toLocaleString()
      toString: ƒ toString()
      valueOf: ƒ valueOf()
      __defineGetter__: ƒ __defineGetter__()
      __defineSetter__: ƒ __defineSetter__()
      __lookupGetter__: ƒ __lookupGetter__()
      __lookupSetter__: ƒ __lookupSetter__()
      get __proto__: ƒ __proto__()
      set __proto__: ƒ __proto__()
  ```
- ### 4.1.4.  NavLinker组件和封装NavLinker组件
- 示例
  
  ```
  import React, { Component } from 'react'
  import { Route } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Headers extends Component {
    render() {
        return (
            <div>
                <div className="page-header"><h2>React Router Demo</h2></div>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import { NavLink} from 'react-router-dom'
  
  export default class MyNavLinker extends Component {
    render() {
        return (
            <div>
                <NavLink className="lislist-group-item" to="/about" {...this.props}></NavLink>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class About extends Component {
    render() {
        console.log(this.props)
        return (
            <h3>
                这是About页面。
            </h3>
        )
    }
  }
  ```
- {...this.props}属性的作用就是将props下的所有属性都放置进来。
- ### 4.1.5.  switch的使用
- swtich组件的使用就是为了解决路由在对比的时候发现会全部对比的情况发生。
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- swtich的原理就是一旦找到相应路由之后就不会再向下对比。
- 如果使用了超过两级以上的路由之后，就会出现样式表丢失的问题，解决这个问题的方式有三种。
	- 第一种就是直接去掉"."
	- 第二种就是使用%PUBLIC_URL%代替"."
	- 第三种就是使用HashRouter代替原来的路由标签。
- 通常情况下我们使用的是第一种方式，最少使用的是第三种。
- ### 4.1.6.  模糊路由
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route exact={true} path="/about" component={About} />
      <Route exact path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- 一般来说，如果不特殊说明就是直接针对路由进行匹配。模糊匹配就是只需要在一级路由匹配即可进行跳转。严格模式需要添加一个新的标签exact。严格模式下必须要严格按照路径才能进行匹配。
- ### 4.1.7.  重定向的使用
  
  ```
  import React, { Component } from 'react'
  import { Redirect, Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route exact={true} path="/about" component={About} />
      <Route exact path="/home" component={Home} />
      <Redirect to="/about"/>
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- Redirect标签就是如果没有匹配到任何地方的时候就使用这样的一个路由。
- ### 4.1.8.  嵌套路由
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Message extends Component {
    render() {
        return (
            <ul>
                <li>
                    <a href="/message1">message001</a>&nbsp;&nbsp;
                    </li>
                <li>
                    <a href="/message2">message002</a>&nbsp;&nbsp;
                    </li>
                <li>
                    <a href="/message/3">message003</a>&nbsp;&nbsp;
                    </li>
            </ul>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class News extends Component {
    render() {
        return (
            <ul>
                <li>news001</li>
                <li>news002</li>
                <li>news003</li>
            </ul>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router'
  import MyNavLinker from '../../components/MyNavLinker'
  import Message from './Message'
  import News from './News'
  
  export default class Home extends Component {
    render() {
        return (
            <div>
              <h2>Home组件内容</h2>
              <div>
                <ul className="nav nav-tabs">
                  <li>
                    {/* <a className="list-group-item active" href="./home-news.html">News</a> */}
                    <MyNavLinker to="/home/news">News</MyNavLinker>
                  </li>
                  <li>
                    {/* <a className="list-group-item " href="./home-message.html">Message</a> */}
                    <MyNavLinker to="/home/message">Message</MyNavLinker>
                  </li>
                </ul>
                {/* 注册路由 */}
                <Switch>
                    <Route path="/home/news" component={News}/>
                    <Route path="/home/message" component={Message}/>
                </Switch>
              </div>
            </div>
        )
    }
  }
  ```
- ### 4.1.9.  向路由传递参数
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
  state = {
    messageArr: [
      { id: '01', title: 'message1' },
      { id: '02', title: 'message2' },
      { id: '03', title: 'message3' },
    ]
  }
  render() {
    const { messageArr } = this.state
    return (
      <div>
      <ul>
      {
        messageArr.map((msgObj) => {
          return (
            <li key={msgObj.id}>
                       {/* 向路由组件传递params参数 */}
                       <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link>
                       </li>
                      )
  })
  }
  </ul>
  <hr/>
  {/* 声明接收params参数 */}
  <Route path="/home/message/detail/:id/:title" component={Detail}/>
  </div>
  )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  const data=[
  {id:'01',content:'你好，同学'},
  {id:'02',content:'你好，世界'},
  {id:'03',content:'你好，未来的自己'}
  ]
  export default class Detail extends Component {
  render() {
    const {id,title}=this.props.match.params
    const findResult=data.find(
      (detailObj)=>{
        return detailObj.id===id
      }
    )
    return (
      <div>
      <ul>
      <li>ID:{id}</li>
      <li>TITLE:{title}</li>
      <li>CONTENT:{findResult.content}</li>
      </ul>
      </div>
    )
  }
  }
  ```
- Search参数
- 路由链接（携带参数）：
  
  ```
  <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link>
  ```
- 注册路由（声明接收）：
  
  ```
  <Route path="/home/message/detail" component={Detail}/>
  ```
- 接收参数：
  
  ```
  this.props.location
  ```
- 备注：获取到的search是urlencoded编码字符串中，需要借助querystring解析。
- ### 4.1.10.  state参数解析
- 路由链接（携带参数）：
  
  ```
  <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}}>{msgObj.title}</Link>
  ```
- 注册路由（声明接收）：
  
  ```
  <Route path="/home/message/detail" component={Detail}/>
  ```
- 接收参数：
  
  ```
  this.props.location.state||{}
  ```
- 备注：刷新也可以保留住参数
- ### 4.1.11.  push和replace的使用
- push：a-b-c可以返回到上一级
- replace：a-b-c不可以返回到上一级，适用于登录后，不需要重新回到登页面
- 可以把浏览器的浏览地址看为一个地址栈。每访问一个页面，地址就会进入地址栈。push和replace就在这个情况下使用。
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
    state = {
        messageArr: [
            { id: '01', title: 'message1' },
            { id: '02', title: 'message2' },
            { id: '03', title: 'message3' },
        ]
    }
    render() {
        const { messageArr } = this.state
        return (
            <div>
                <ul>
                    {
                        messageArr.map((msgObj) => {
                            return (
                                <li key={msgObj.id}>
                                    {/* 向路由组件传递params参数 */}
                                    {/* <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    {/* <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}} replace>{msgObj.title}</Link>
  
                                </li>
                            )
                        })
                    }
                </ul>
                <hr/>
                {/* 声明接收params参数 */}
                {/* <Route path="/home/message/detail/:id/:title" component={Detail}/> */}
  
                {/* 声明接收search参数
                <Route path="/home/message/detail" component={Detail}/> */}
  
                {/* 声明接收search参数 */}
                <Route path="/home/message/detail" component={Detail}/>
            </div>
        )
    }
  }
  ```
- ### 4.1.12.  编程式路由
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
    state = {
        messageArr: [
            { id: '01', title: 'message1' },
            { id: '02', title: 'message2' },
            { id: '03', title: 'message3' },
        ]
    }
  
    replaceShow=(id,title)=>{
        // replace跳转+携带params参数
        // this.props.history.replace(`/home/message/detail/${id}/${title}`)
  
        // replace跳转+携带search参数
        // this.props.history.replace(`/home/message/detail?id=${id}&title=${title}`)
  
        // replace跳转+携带state参数
        this.props.history.replace(`/home/message/detail`,{id,title})
    }
  
    pushShow=(id,title)=>{
        // push跳转+携带params参数
        // this.props.history.push(`/home/message/detail/${id}/${title}`)
  
        // push跳转+携带search参数
        // this.props.history.push(`/home/message/detail?id=${id}&title=${title}`)
  
        // push跳转+携带state参数
        this.props.history.push(`/home/message/detail`,{id,title})
    }
  
    back=()=>{
        this.props.history.goBack()
    }
  
    forward=()=>{
        this.props.history.goForward()
    }
  
    goto=()=>{
        this.props.history.go(2)
    }
  
    render() {
        const { messageArr } = this.state
        return (
            <div>
                <ul>
                    {
                        messageArr.map((msgObj) => {
                            return (
                                <li key={msgObj.id}>
                                    {/* 向路由组件传递params参数 */}
                                    {/* <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    {/* <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递state参数 */}
                                    <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}}>{msgObj.title}</Link>
                                    &nbsp;<button onClick={()=>{this.replaceShow(msgObj.id,msgObj.title)}}>replace</button>
                                    &nbsp;<button onClick={()=>{this.pushShow(msgObj.id,msgObj.title)}}>push</button>
  
                                </li>
                            )
                        })
                    }
                </ul>
                <hr/>
                {/* 声明接收params参数 */}
                {/* <Route path="/home/message/detail/:id/:title" component={Detail}/> */}
  
                {/* 声明接收search参数
                <Route path="/home/message/detail" component={Detail}/> */}
  
                {/* 声明接收state参数 */}
                <Route path="/home/message/detail" component={Detail}/>
  
                <button onClick={this.back}>后退</button>
                <button onClick={this.forward}>前进</button>
                <button onClick={this.goto}>go</button>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import qs from 'querystring'
  
  const data=[
    {id:'01',content:'你好，同学'},
    {id:'02',content:'你好，世界'},
    {id:'03',content:'你好，未来的自己'}
  ]
  export default class Detail extends Component {
    render() {
  
        // 接收params参数
        // const {id,title}=this.props.match.params
  
        //接收search参数
        // const {search}=this.props.location
        // const {id,title}=qs.parse(search.slice(1))
        // const findResult=data.find(
        //     (detailObj)=>{
        //         return detailObj.id===id
        //     }
        // )
  
        // 接收state参数
        const {id,title}=this.props.location.state||{}
  
        const findResult=data.find(
            (detailObj)=>{
                return detailObj.id===id
            }
        )||{}
        return (
            <div>
                <ul>
                    <li>ID:{id}</li>
                    <li>TITLE:{title}</li>
                    <li>CONTENT:{findResult.content}</li>
                </ul>
            </div>
        )
    }
  }
  ```
- 对应的路由方式需要对应一定的Route否则就不会产生相应的效果。
- BrowserRouter和HashRouter的区别
- 底层原理不一样
	- BrowserRouter使用的是H5的history API，不兼容IE9以下的版本。
	- HashROuter使用的是URL的哈希值。
- path表现形式不一样
	- BrowserRouter的路径中没有#，例如：localhost:3000/demo/test
	- HashRouter的路径包含#，例如：localhost:3000/#/demo/test
- 刷新后对路由state参数的影响
	- BrowserRouter没有任何影响，因为state保存在history对象中，
	- HashRouter刷新后会导致路由state参数的丢失。
- 备注：HashRouter可以用于解决一些路径错误相关的问题。
- withRouter函数
- 这个函数能让一般的组件变成为路由组件。
  
  ```
  import React, { Component } from 'react'
  import { withRouter } from 'react-router'
  
  class News extends Component {
    render() {
        return (
            <ul>
                <li>news001</li>
                <li>news002</li>
                <li>news003</li>
            </ul>
        )
    }
  }
  
  export default withRouter(News)
  // withRouter可以加工一般组件，让一般组件具有路由组件所特有的API
  // withRouter是一个新的组件
  ```
- ## 4.2.  Ant Design UI
- ### 4.2.1.  按需引入
  
  ```
  // 配置具体的修改规则
  const {override, fixBabelImports} =require('customize-cra')
  
  module.exports=override(
  fixBabelImports(
    'import',{
      libraryName:'antd',
      libraryDirectory:'es',
      style:'css',
    }
  )
  )
  ```
	- src/App.jsx
	  
	  ```
	  import React, { Component } from 'react'
	  import {Button,DatePicker} from 'antd'
	  import {WechatOutlined,WeiboOutlined,SearchOutlined} from '@ant-design/icons'
	  
	  export default class App extends Component {
	  
	   render() {
	       return (
	           <div>
	               App......
	               <button>点我</button>
	               <Button type="primary">按钮1</Button>
	               <Button>按钮2</Button>
	               <Button type="link">按钮3</Button>
	               <Button type="primary" icon={<SearchOutlined/>}>Search</Button>
	               <WechatOutlined/>
	               <WeiboOutlined/>
	               <DatePicker/>
	           </div>
	       )
	   }
	  }
	  ```
- ### 4.2.2.  Antd 自定义样式
- config-orverride.js
  
  ```
  // 配置具体的修改规则
  const {override, fixBabelImports} =require('customize-cra')
  
  module.exports=override(
  fixBabelImports(
    'import',{
      libraryName:'antd',
      libraryDirectory:'es',
      style:'css',
    }
  ),
  addLessLoader({
    javascriptEnabled:true,
    modifyVars:{
      '@primary-color':'#1DA57A'
    }
  })
  )
  ```
- ## 4.3.  Redux
- 首先，redux并不是必须的，它的作用相当于在顶层组件之上又加了一个组件，作用是进行逻辑运算、储存数据和实现组件尤其是顶层组件的通信。如果组件之间的交流不多，逻辑不复杂，只是单纯的进行视图的渲染，这时候用回调，context就行，没必要用redux，用了反而影响开发速度。但是如果组件交流特别频繁，逻辑很复杂，那redux的优势就特别明显了。我第一次做react项目的时候并没有用redux，所有的逻辑都是在组件内部实现，当时为了实现一个逻辑比较复杂的购物车，洋洋洒洒居然写了800多行代码，回头一看我自己都不知道写的是啥，画面太感人。
- 先简单说一下redux和react是怎么配合的。react-redux提供了connect和Provider两个好基友，它们一个将组件与redux关联起来，一个将store传给组件。组件通过dispatch发出action，store根据action的type属性调用对应的reducer并传入state和这个action，reducer对state进行处理并返回一个新的state放入store，connect监听到store发生变化，调用setState更新组件，此时组件的props也就跟着变化。
- ### 4.3.1.  Redux工作原理
- 随着应用程序单页面需求的越来越复杂，应用状态的管理也变得越来越混乱，而Redux的就是为解决这一问题而出现的。在一个大型的应用程序中，应用的状态不仅包括从服务器获取的数据，还包括本地创建的数据，以及反应本地UI状态的数据，而Redux正是为解决这一复杂问题而存在的。
- redux作为一种单向数据流的实现，配合react非常好用，尤其是在项目比较大，逻辑比较复杂的时候，单项数据流的思想能使数据的流向、变化都能得到清晰的控制，并且能很好的划分业务逻辑和视图逻辑。下图是redux的基本运作的流程。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1619420470572-c3826de0-6f42-4a0e-bb1f-bd5ce3b16904.png)
- 如上图所示，该图展示了Redux框架数据的基本工作流程。简单来说，首先由view dispatch拦截action，然后执行对应reducer并更新到store中，最终views会根据store数据的改变执行界面的刷新渲染操作。
- 同时，作为一款应用状态管理框架，为了让应用的状态管理不再错综复杂，使用Redux时应遵循三大基本原则，否则应用程序很容易出现难以察觉的问题。这三大原则包括：
	- 单一数据源
		- 整个应用的State被存储在一个状态树中，且只存在于唯一的Store中。
	- State是只读的
		- 对于Redux来说，任何时候都不能直接修改state，唯一改变state的方法就是通过触发action来间接的修改。而这一看似繁琐的状态修改方式实际上反映了Rudux状态管理流程的核心思想，并因此保证了大型应用中状态的有效管理。
	- 应用状态的改变通过纯函数来完成
		- Redux使用纯函数方式来执行状态的修改，Action表明了修改状态值的意图，而真正执行状态修改的则是Reducer。且Reducer必须是一个纯函数，当Reducer接收到Action时，Action并不能直接修改State的值，而是通过创建一个新的状态对象来返回修改的状态。
- ### 4.3.2.  React中的实现
  
  ```
  import React, { Component } from 'react'
  
  export default class Count extends Component {
    state={count:0}
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        this.setState({count:count+value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        this.setState({count:count-value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        if(count % 2!==0){
            this.setState({count:count+value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        setTimeout(()=>{
            this.setState({count:count+value*1})
        },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：0</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  // 引入store，用于获取redux中保存的状态
  import store from '../../reduxs/store'
  
  export default class Count extends Component {
  
    componentDidMount(){
        // 检测redux中状态的变化，只要变化，就可以调用render
        store.subscribe(()=>{
            this.setState({})
        })
    }
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'increment',data:value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'decrement',data:value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const count=store.getState()
        if(count % 2!==0){
            store.dispatch({type:'increment',data:value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        setTimeout(()=>{
            store.dispatch({type:'increment',data:value*1})
        },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：{store.getState()}</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  /*
    1.该文件是用于创建一个为Count组件服务的reducer，reducer的本质就是一个函数
    2.reducer函数会接到两个参数，分别为：之前的状态（preState）,动作对象（action）
  */
  const initState=0
  export default function countReducer(preState=initState,action){
    if(preState===undefined) preState=0
    // 从action对象中获取：type、data
    const {type,data}=action
    // 根据type决定如何加工数据
    switch(type){
        case 'increment':
            return preState+data
        case 'decrement':
            return preState-data
        default:
            return 0;
    }
  }
  ```
  
  ```
  /*
    该文件专门用于暴露一个store对象，整个应用只有一个store对象
  */
  // 引入createStore，专门用于创建redux中最为核心的store对象
  import {createStore} from 'redux'
  // 引入为Count组件服务的reducer
  import countReducer from './count_reducer'
  // 暴露store
  export default createStore(countReducer)
  ```
- 以上就是一个简易版的redux组件实现的案例。
- ### 4.3.3.  Redux完整实现
- 新增文件：
	- count_action.js 专门用于创建action对象
	- constant.js 防止编码疏忽写错action中的内容
	  
	  ```
	  /*
	   1.该文件是用于创建一个为Count组件服务的reducer，reducer的本质就是一个函数
	   2.reducer函数会接到两个参数，分别为：之前的状态（preState）,动作对象（action）
	  */
	  import {INCREMENT,DECREMENT} from './constant'
	  
	  const initState=0
	  export default function countReducer(preState=initState,action){
	   if(preState===undefined) preState=0
	   // 从action对象中获取：type、data
	   const {type,data}=action
	   // 根据type决定如何加工数据
	   switch(type){
	       case INCREMENT:
	           return preState+data
	       case DECREMENT:
	           return preState-data
	       default:
	           return 0;
	   }
	  }
	  ```
	  
	  ```
	  /*
	   该文件专门为Count组件生成action对象
	  */
	  import {INCREMENT,DECREMENT} from './constant'
	  
	  export const createIncrementAction=data=>({type:INCREMENT,data})
	  export const createDecrementAction=data=>({type:DECREMENT,data})
	  ```
	  
	  ```
	  /*
	   该模块是用于定义action对象中type类型的常量值
	   目的就是便于管理的同时防止程序员单词写错
	  */
	  export const INCREMENT = 'increment'
	  export const DECREMENT = 'decrement'
	  ```
- ### 4.3.4.  Redux异步action
- 明确：延迟的动作不想提交给组件本身，想要交给action
- 何时需要异步action：想要对状态进行操作，但是具体的数据靠异步任务返回（非必须）。
- 具体编码：
	- npm add redux-thunk，并配置在store中
	- 创建action的函数不再返回一般对象，而是一个函数，该函数中写异步任务。
	- 异步任务有结果只会，分发一个同步的action去真正操作数据。
- 备注：异步action不是必须要写的，完全可以自己等待异步任务的结果了再去分发同步action。
  
  ```
  /*
    该文件专门为Count组件生成action对象
  */
  import {INCREMENT,DECREMENT} from './constant'
  import store from './store'
  
  // 同步action，就是指action的值为Object类型的一般对象
  export const createIncrementAction=data=>({type:INCREMENT,data})
  export const createDecrementAction=data=>({type:DECREMENT,data})
  
  // 异步action，就是指action的值为函数,异步action中一般都会调用同步action
  // 异步action不是必须要用的
  export const createIncrementAsyncAction=(data,time)=>{
    return ()=>{
        setTimeout(()=>{
            dispatch(createIncrementAction(data))
        },time)
    }
  }
  ```
  
  ```
  /*
    该文件专门用于暴露一个store对象，整个应用只有一个store对象
  */
  // 引入createStore，专门用于创建redux中最为核心的store对象
  import {createStore,applyMiddleware} from 'redux'
  // 引入为Count组件服务的reducer
  import countReducer from './count_reducer'
  // 引入redux-thunk，用于支持异步action
  import thunk from 'redux-thunk'
  // 暴露store
  export default createStore(countReducer, applyMiddleware(thunk))
  ```
  
  ```
  import React, { Component } from 'react'
  // 引入store，用于获取redux中保存的状态
  import store from '../../reduxs/store'
  
  export default class Count extends Component {
  
    componentDidMount(){
        // 检测redux中状态的变化，只要变化，就可以调用render
        store.subscribe(()=>{
            this.setState({})
        })
    }
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'increment',data:value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'decrement',data:value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const count=store.getState()
        if(count % 2!==0){
            store.dispatch({type:'increment',data:value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        // setTimeout(()=>{
            store.dispatch({type:'increment',data:value*1})
        // },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：{store.getState()}</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
- ## 4.4.  React-Redux
- ### 4.4.1.  简介
- UI组件中不能使用任何redux的api
- 容器组件会传给UI组件：
	- redux中所保存的状态
	- 用于操作状态的方法。
- 备注：容器给UI传递：状态、操作状态的方法，均通过props传递。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1619766742634-7a450dec-65f2-4801-918b-340e4f926c9a.png)
  
  ```
  import React, { Component } from 'react'
  
  export default class Count extends Component {
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：？？？？</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  // 引入Count的UI组件
  import CountUI from '../../components/Count'
  // 引入connect用于连续UI组件与redux
  import {connect} from 'react-redux'
  
  // 使用connect()()创建并暴露一个Count的容器组件
  export default connect()(CountUI)
  ```
  
  ```
  import React, { Component } from 'react'
  import Count from './containers/Count'
  import store from './reduxs/store'
  
  export default class App extends Component {
    render() {
        return (
            <div>
                <Count store={store}/>
            </div>
        )
    }
  }
  ```
- ### 4.4.2.  基本使用
	- 备注：容器组件中的store是靠props传进去的，而不是在容器组件中直接引入。
	  
	  ```
	  // 引入Count的UI组件
	  import CountUI from '../../components/Count'
	  // 引入action
	  import {
	   createIncrementAction,
	   createDecrementAction,
	   createIncrementAsyncAction,
	  } from '../../reduxs/count_action'
	  // 引入connect用于连接UI组件与redux
	  import {connect} from 'react-redux'
	  
	  /*
	   1.mapStateToProps函数返回的是一个对象
	   2.返回的对象中的key该作为传递给UI组件props的key，value就作为传递给UI组件props的value
	   3.mapStateToProps用于传递状态
	  */
	  function mapStateToProps(state){
	   return {count:state}
	  }
	  
	  /*
	   1.mapDispatchToProps函数返回的是一个对象；
	   2.返回的对象中的key就作为传递给UI组件props的key，value就作为传递给UI组件props的value
	   3.mapDispatchToProps用于传递操作状态的方法
	  */
	  function mapDispatchToProps(dispatch){
	   return {
	       jia:number=>dispatch(createIncrementAction(number)),
	       jian:number=>dispatch(createDecrementAction(number)),
	       jiaAsync:(number,time)=>dispatch(createIncrementAsyncAction(number,time))
	   }
	  }
	  
	  // 使用connect()()创建并暴露一个Count的容器组件
	  export default connect(mapStateToProps,mapDispatchToProps)(CountUI)
	  ```
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Count extends Component {
	  
	   // 加法
	   increment=()=>{
	       const {value} = this.selectNumber
	       this.props.jia(value*1)
	   }
	  
	   // 减法
	   decrement=()=>{
	       const {value} = this.selectNumber
	       this.props.jian(value*1)
	   }
	  
	   // 奇数再加
	   incrementIfOdd=()=>{
	       const {value} = this.selectNumber
	       if(this.props.count%2!==0){
	           this.props.jia(value*1)
	       }
	   }
	  
	   // 异步加
	   incrementAsync=()=>{
	       const {value} = this.selectNumber
	       this.props.jianAsync(value*1,500)
	   }
	  
	   render() {
	       return (
	           <div>
	               <h1>当前求和为：{this.props.count}</h1>
	               <select ref={c=>this.selectNumber=c}>
	                   <option value="1">1</option>
	                   <option value="2">2</option>
	                   <option value="3">3</option>
	               </select>&nbsp;
	               <button onClick={this.increment}>+</button>&nbsp;
	               <button onClick={this.decrement}>-</button>&nbsp;
	               <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
	               <button onClick={this.incrementAsync}>异步加</button>&nbsp;
	           </div>
	       )
	   }
	  }
	  ```
- 对于这部分优化就是对于在容器组件中的一个组件进行优化
  
  ```
  // 引入Count的UI组件
  import CountUI from '../../components/Count'
  // 引入action
  import {
    createIncrementAction,
    createDecrementAction,
    createIncrementAsyncAction,
  } from '../../reduxs/count_action'
  // 引入connect用于连接UI组件与redux
  import {connect} from 'react-redux'
  
  // 使用connect()()创建并暴露一个Count的容器组件
  export default connect(
    // mapDispatchToProps的一般写法
    /*
    dispatch=>({
        jia:number=>dispatch(createIncrementAction(number)),
        jian:number=>dispatch(createDecrementAction(number)),
        jiaAsync:(number,time)=>dispatch(createIncrementAsyncAction(number,time))
    })
    */
    state=>({count:state}),
    // mapDispatchToProps的简写
    {
        jia:createIncrementAction,
        jian:createDecrementAction,
        jiaAsync:createIncrementAsyncAction
    }
  )(CountUI)
  ```
- ### 4.4.3.  Provider的使用
- 为了减少在每一次调用的时候需要使用到store={store}这句话。可以使用Provider这样一个组件直接使用。
  
  ```
  import React from 'react'
  import ReactDOM from 'react-dom'
  import App from './App'
  import store from './reduxs/store'
  import {Provider} from 'react-redux'
  
  ReactDOM.render(
    <Provider store={store}>
        <App/>
    </Provider>,
    document.getElementById('root')
  )
  
  // 检测redux中状态的改变，如redux的状态发生了改变，那么重新渲染App组件
  // store.subscribe(()=>{
  //     ReactDOM.render(<App/>,document.getElementById('root'))
  // })
  ```
- ### 4.4.4.  编写Person案例
  
  ```
  import React, { Component } from 'react'
  import {nanoid} from 'nanoid'
  
  export default class Person extends Component {
  
    addPerson=()=>{
        const name=this.nameNode.value
        const age=this.ageNode.value
        const personObj={id:nanoid(),name,age}
        console.log(personObj)
    }
  
    render() {
        return (
            <div>
                <h2>Person...</h2>
                <input ref={c=>this.nameNode=c} type="text" placeholder="输入名字"/>
                <input ref={c=>this.ageNode=c} type="text" placeholder="输入年龄"/>
                <button onClick={this.addPerson}>添加</button>
                <ul>
                    <li>姓名1---年龄1</li>
                    <li>姓名2---年龄2</li>
                    <li>姓名3---年龄3</li>
                </ul>
            </div>
        )
    }
  }
  ```
	- 上述代码就是将一个完整的静态页面组件编写完毕。
	  
	  ```
	  import {ADD_PERSON} from '../constant'
	  
	  // 创建一个增加人的action动作对象
	  export const createAddPersonAction= personObj =>({
	   type:ADD_PERSON,
	   data:personObj
	  })
	  ```
	  
	  ```
	  import {ADD_PERSON} from '../constant'
	  
	  // 初始化列表
	  const initState=[{id:'001',name:'tom',age:10}]
	  export default function personReducer(perState=initState,action){
	   const {type,data}=action
	   switch (type){
	       case ADD_PERSON:
	           return [data,...perState]
	       default:
	           return perState
	   }
	  }
	  ```
	- 以上就是对于redux中一个组件的准备工作的编写。首先是完成静态组件的编写，然后再将相应的action和reducer编写好。
- ### 4.4.5.  Redux数据共享完整版
- 定义一个Person组件，和Count组件通过Redux进行数据共享。
- 为Person组件编写：reducer、action，配置constant常量。
- 重点：Person的reducer和Count的Reducer要使用combineReducers进行合并，合并后的总状态是一个对象。
- 交给store的是总reducer，最后注意在组件中取出状态的时候，记得要取到位。
  
  ```
  import React, { Component } from 'react'
  import {nanoid} from 'nanoid'
  import {connect} from 'react-redux'
  import {createAddPersonAction} from '../../reduxs/action/person'
  
  class Person extends Component {
  
    addPerson=()=>{
        const name=this.nameNode.value
        const age=this.ageNode.value
        const personObj={id:nanoid(),name,age}
        this.props.AddOne(personObj)
    }
  
    render() {
        return (
            <div>
                <h2>这是Person组件。上面的组件的和Count为:{this.props.num}</h2>
                <input ref={c=>this.nameNode=c} type="text" placeholder="输入名字"/>
                <input ref={c=>this.ageNode=c} type="text" placeholder="输入年龄"/>
                <button onClick={this.addPerson}>添加</button>
                <ul>
                    {
                        this.props.persons.map((p)=>{
                            return <li>{p.name}--{p.age}</li>
                        })
                    }
                </ul>
            </div>
        )
    }
  }
  
  export default connect(
    // 映射状态
    state=>({persons:state.rens,num:state.he}),
    // 映射操作状态的方法
    {AddOne:createAddPersonAction}
  )(Person)
  ```
	- 这里面的Person的静态组件和容器组件合在了一起。
	  
	  ```
	  /*
	   该文件专门用于暴露一个store对象，整个应用只有一个store对象
	  */
	  // 引入createStore，专门用于创建redux中最为核心的store对象
	  import {createStore,applyMiddleware,combineReducers} from 'redux'
	  // 引入为Count组件服务的reducer
	  import countReducer from './reducer/count'
	  // 引入为Person组件服务的reducer
	  import personReducer from './reducer/person'
	  // 引入redux-thunk，用于支持异步action
	  import thunk from 'redux-thunk'
	  
	  // 汇总所有的reducer变为一个总的reducer
	  const allReducer=combineReducers({
	   he:countReducer,
	   rens:personReducer
	  })
	  
	  // 暴露store
	  export default createStore(allReducer, applyMiddleware(thunk))
	  ```
- ### 4.4.6.  纯函数
- 一类特别的函数：只要是同样的输入（实参），必定得到同样的输出（返回）
- 必须遵守以下的一些约束：
	- 不得改写参数数据；
	- 不会产生任何副作用，例如网络请求，输入和输出设备；
	- 不能调用Data.now()或者Math.random()等不纯的方法；
- redux的reducer函数必须是一个纯函数。
  
  ```
  import {ADD_PERSON} from '../constant'
  
  // 初始化列表
  const initState=[{id:'001',name:'tom',age:10}]
  export default function personReducer(perState=initState,action){
    const {type,data}=action
    switch (type){
        case ADD_PERSON:
            // preState.unshift(data)
            // 此处不可以这样写，这样会导致preState被该写了，reducer就不是纯函数了。
            return [data,...perState]
        default:
            return perState
    }
  }
  ```
- ## 4.5.  Pubsubjs
- ### 4.5.1.  介绍
- pubsubjs就是可以在js中进行订阅和发布等工作。
- 通过订阅机制，可以取消掉一定要通过prop属性进行数据传递的方式。可以订阅机制，可以进行任意组件间的通信。
- ### 4.5.2.  使用
  
  ```
  import React, { Component } from 'react'
  import List from './components/List'
  import Search from './components/Search'
  
  export default class App extends Component {
  
  render() {
    return (
      <div className="container">
      <Search updateAppState={this.updateAppState}/>
  <List {...this.state}/>
  </div>
  )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import PubSub from 'pubsub-js'
  import axios from 'axios'
  
  export default class Search extends Component {
    search=()=>{
        // 获取用户的输入（连续解构赋值+重命名）
        const {keyWordElement:{value:keyWord}}=this
        console.log(keyWord)
        // 发送请求前通知App更新状态
        // this.props.updateAppState({isFirst:false,isLoading:true,})
        PubSub.publish('information',{isFirst:false,isLoading:true})
        // 发送网络请求
        axios.get(`/search/users?q=${keyWord}`).then(
            response=>{
                // this.props.updateAppState({isLoading:false,users:response.data.items})
                PubSub.publish('information',{isLoading:false,users:response.data.items})
            },
            error=>{
                // 请求失败后通知App更新状态
                // this.props.updateAppState({isLoading:false,err:error.message})
                PubSub.publish('information',{isLoading:false,err:error.message})
            }
        )
    }
    render() {
        return (
            <div>
                <section className="jumbotron">
                    <h3 className="jumbotron-heading">Search Github Users</h3>
                    <div>
                        <input ref={c=>this.keyWordElement=c} type="text" placeholder="enter the name you search" />&nbsp;
                        <button onClick={this.search}>Search</button>
                    </div>
                </section>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class List extends Component {
  
    state={ //初始化状态
        users:[], // users初始值的数组
        isFirst:true, //是否为第一次打开页面
        isLoading:false, //标识是否处于加载中
        err:'',// 存储请求相关的错误信息
    }
  
    componentDidMount(){
        this.token=PubSub.subscribe('information',(_,stateObj)=>{
            this.setState(stateObj)
        })
    }
  
    // 在组件即将卸载的时候，取消订阅。
    componentWillUnmount(){
        PubSub.unsubscribe()
    }
  
    render() {
        const {users,isFirst,isLoading,err}=this.props
        return (
            <div className="row">
                {
                    isFirst?<h2>欢迎使用，输入关键字，然后点击搜索</h2>:
                    isLoading?<h2>Load......</h2>:
                    err?<h2 style={{color:'red'}}>{err}</h2>:
                    users.map((userObj) => {
                        return (
                            <div key={userObj.id} className="card">
                                <a rel="noreferrer" href={userObj.html_url} target="_blank">
                                    <img alt="Head_portrait" src={userObj.avatar_url} style={{ width: '100px' }} />
                                </a>
                                <p className="card-text">{userObj.login}</p>
                            </div>
                        )
                    })
                }
            </div>
        )
    }
  }
  ```
- ## 4.6.  fetach
- ### 4.6.1.  介绍
- fetch是用来取代传统的XMLHttpRequest的。 它的优点很多，包括链式调用的语法、返回promise等。
- fetch api是基于promise的设计，它是为了取代传统xhr的不合理的写法而生的。
- fetch() 返回的是一个Promise对象。fetch使用的promise对象可以使得我们使用同步的方式写异步函数。
- fetch api是可以结合 async 和 await 来使用的。 fetch是基于promise实现的，但是使用promise的写法，我们还是可以看到callback的影子，如果结合 async和await来使用，还是非常不错的。
- Fetch 提供了对 Request 和 Response （以及其他与网络请求有关的）对象的通用定义。所以在一个Fetch请求中，完全可以只使用Request 和 Response两个对象，通过Request 设置参数，通过Response 对返回值进行处理。
- ### 4.6.2.  使用
  
  ```
  import React, { Component } from 'react'
  import PubSub from 'pubsub-js'
  import axios from 'axios'
  
  export default class Search extends Component {
    search=()=>{
        //#region 
        // 获取用户的输入（连续解构赋值+重命名）
        const {keyWordElement:{value:keyWord}}=this
        console.log(keyWord)
        // 发送请求前通知App更新状态
        // this.props.updateAppState({isFirst:false,isLoading:true,})
        PubSub.publish('information',{isFirst:false,isLoading:true})
        // 发送网络请求
        axios.get(`/search/users?q=${keyWord}`).then(
            response=>{
                // this.props.updateAppState({isLoading:false,users:response.data.items})
                PubSub.publish('information',{isLoading:false,users:response.data.items})
            },
            error=>{
                // 请求失败后通知App更新状态
                // this.props.updateAppState({isLoading:false,err:error.message})
                PubSub.publish('information',{isLoading:false,err:error.message})
            }
        )
        //#endregion
  
        //#region 
        // fetch(`/search/users?q=${keyWord}`).then(
        //     response=>{
        //         console.log('联系服务器成功了')
        //         return response.json()
        //     },
        //     error=>{
        //         console.log('联系服务器失败了',error)
        //         return new Promise(()=>{})
        //     }
        // ).then(
        //     response=>{console.log('联系服务器成功了',response)},
        //     error=>{console.log('联系服务器失败了',error)}
        // )
        //#endregion
        // 发送网络请求——使用fetch（优化）
        try{
            const response=await fetch(`/search/users?q=${keyWord}`)
            const data=await response.json()
            console.log(data)
            PubSub.publish('information',{isLoading:false,users:response.data.items})
        } catch(error){
            console.log('请求出错',error)
            PubSub.publish('information',{isLoading:false,err:error.message})
        }
    }
    render() {
        return (
            <div>
                <section className="jumbotron">
                    <h3 className="jumbotron-heading">Search Github Users</h3>
                    <div>
                        <input ref={c=>this.keyWordElement=c} type="text" placeholder="enter the name you search" />&nbsp;
                        <button onClick={this.search}>Search</button>
                    </div>
                </section>
            </div>
        )
    }
  }
  ```
- ## 4.7.  路由
- ### 4.7.1.  SPA介绍
- SPA指的是单页的Web应用（Single page web application）
- 整个应用只有一个完整的页面。
- 点击页面中的链接不会刷新页面，只会做页面的局部更新
- 数据都需要通过ajax请求获取，并在前端异步展现。
- ### 4.7.2.  路由介绍
- 介绍
	- 1.    一个路由就是一个映射关系(key:value)
	- 2.    key为路径, value可能是function或component
- 分类
	- 1.    后端路由：
	- 1)    理解： value是function, 用来处理客户端提交的请求。
	- 2)    注册路由：router.get(path, function(req, res))
	- 3)    工作过程：当node接收到一个请求时, 根据请求路径找到匹配的路由, 调用路由中的函数来处理请求, 返回响应数据
- 2.   前端路由：
	- 1)    浏览器端路由，value是component，用于展示页面内容。
	- 2)    注册路由: <Route path="/test" component={Test}>
	- 3)    工作过程：当浏览器的path变为/test时, 当前路由组件就会变为Test组件
- ### 4.7.3.  History的使用和理解
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>前端路由的基石_history</title>
  </head>
  <body>
    <a href="http://www.atguigu.com" onclick="return push('/test1') ">push test1</a><br><br>
    <button onClick="push('/test2')">push test2</button><br><br>
    <button onClick="replace('/test3')">replace test3</button><br><br>
    <button onClick="back()">&lt;= 回退</button>
    <button onClick="forword()">前进 =&gt;</button>
  
    <script type="text/javascript" src="https://cdn.bootcss.com/history/4.7.2/history.js"></script>
    <script type="text/javascript">
      // let history = History.createBrowserHistory() //方法一，直接使用H5推出的history身上的API
      let history = History.createHashHistory() //方法二，hash值（锚点）
  
      function push (path) {
        history.push(path)
        return false
      }
  
      function replace (path) {
        history.replace(path)
      }
  
      function back() {
        history.goBack()
      }
  
      function forword() {
        history.goForward()
      }
  
      history.listen((location) => {
        console.log('请求路由路径变化了', location)
      })
    </script>
  </body>
  </html>
  ```
- 在整个前端的路由中，表现出来的就是history这一个堆栈对于路由的掌控。这就是前端实现路由转换的基石。
- ### 4.7.4.  路由的基本使用
- react-router-dom的理解
	- 1.     react的一个插件库。
	- 2.     专门用来实现一个SPA应用。
	- 3.     基于react的项目基本都会用到此库。
- 示例
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" href="./css/bootstrap.css">
  </head>
  
  <body>
    <div id="root">
    </div>
  </body>
  </html>
  ```
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import About from './components/About'
  import Home from './components/Home'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <div className="page-header"><h2>React Router Demo</h2></div>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    <Link className="lislist-group-item" to="/about">About</Link>
      <Link className="lislist-group-item" to="/home">Home</Link>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  // 引入react核心库
  import React from 'react'
  // 引入ReactDOM
  import ReactDOM from 'react-dom'
  // 引入BrowserRouter组件
  import {BrowserRouter} from 'react-router-dom'
  // 引入App组件
  import App from './App'
  
  // 渲染App到页面
  ReactDOM.render(
  <BrowserRouter>
  <App/>
  </BrowserRouter>,
  document.getElementById('root'))
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class About extends Component {
  render() {
    return (
      <h3>
      这是About页面。
      </h3>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Home extends Component {
  render() {
    return (
      <h3>
      这是Home页面。
      </h3>
    )
  }
  }
  ```
- 使用BrowserRouter标签的时候，要使用在同一个标签下才能起作用，否则无法起作用。
- ### 4.7.5.  路由组件
- 一般组件如果不传props的话，就是在传输过程中不会出现任何props。但是路由组件即使不传，也会带上这样的三个属性对象。
  
  ```
  history:
      action: "PUSH"
      block: ƒ block(prompt)
      createHref: ƒ createHref(location)
      go: ƒ go(n)
      goBack: ƒ goBack()
      goForward: ƒ goForward()
      length: 3
      listen: ƒ listen(listener)
      location: {pathname: "/about", search: "", hash: "", state: null, key: "sevxl7"}
      push: ƒ push(path, state)
      replace: ƒ replace(path, state)
      __proto__: Object
  location:
      hash: ""
      key: "sevxl7"
      pathname: "/about"
      search: ""
      state: null
      __proto__: Object
  match:
      isExact: true
      params: {}
      path: "/about"
      url: "/about"
      __proto__: Object
      staticContext: undefined
  __proto__:
      constructor: ƒ Object()
      hasOwnProperty: ƒ hasOwnProperty()
      isPrototypeOf: ƒ isPrototypeOf()
      propertyIsEnumerable: ƒ propertyIsEnumerable()
      toLocaleString: ƒ toLocaleString()
      toString: ƒ toString()
      valueOf: ƒ valueOf()
      __defineGetter__: ƒ __defineGetter__()
      __defineSetter__: ƒ __defineSetter__()
      __lookupGetter__: ƒ __lookupGetter__()
      __lookupSetter__: ƒ __lookupSetter__()
      get __proto__: ƒ __proto__()
      set __proto__: ƒ __proto__()
  ```
- ### 4.7.6.  NavLinker组件和封装NavLinker组件
- NavLinker组件就是根据某些时候需要激活状态而催生的。
- 示例
  
  ```
  import React, { Component } from 'react'
  import { Route } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Headers extends Component {
    render() {
        return (
            <div>
                <div className="page-header"><h2>React Router Demo</h2></div>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import { NavLink} from 'react-router-dom'
  
  export default class MyNavLinker extends Component {
    render() {
        return (
            <div>
                <NavLink className="lislist-group-item" to="/about" {...this.props}></NavLink>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class About extends Component {
    render() {
        console.log(this.props)
        return (
            <h3>
                这是About页面。
            </h3>
        )
    }
  }
  ```
- {...this.props}属性的作用就是将props下的所有属性都放置进来。
- ### 4.7.7.  Switch的使用
- swtich组件的使用就是为了解决路由在对比的时候发现会全部对比的情况发生。
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- swtich的原理就是一旦找到相应路由之后就不会再向下对比。
- 如果使用了超过两级以上的路由之后，就会出现样式表丢失的问题，解决这个问题的方式有三种。
	- 第一种就是直接去掉"."
	- 第二种就是使用%PUBLIC_URL%代替"."
	- 第三种就是使用HashRouter代替原来的路由标签。
- 通常情况下我们使用的是第一种方式，最少使用的是第三种。
- ### 4.7.8.  模糊路由
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route exact={true} path="/about" component={About} />
      <Route exact path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- 一般来说，如果不特殊说明就是直接针对路由进行匹配。模糊匹配就是只需要在一级路由匹配即可进行跳转。严格模式需要添加一个新的标签exact。严格模式下必须要严格按照路径才能进行匹配。
- ### 4.7.9.  Redirect的使用
  
  ```
  import React, { Component } from 'react'
  import { Redirect, Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route exact={true} path="/about" component={About} />
      <Route exact path="/home" component={Home} />
      <Redirect to="/about"/>
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
- Redirect标签就是如果没有匹配到任何地方的时候就使用这样的一个路由。
- ### 4.7.10.  嵌套路由
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router-dom'
  import About from './pages/About'
  import Home from './pages/Home'
  import Headers from './components/Headers'
  import MyNavLinker from './components/MyNavLinker'
  
  export default class App extends Component {
  
  render() {
    return (
      <div>
      <div className="row">
      <div className="col-xs-offset-2 col-xs-8">
      <Headers/>
      </div>
      </div>
      <div className="row">
      <div className="col-xs-2 col-xs-offset-2">
      <div className="list-group">
      {/* 原生的html中，靠<a>跳转不同的页面
                            <a className="list-group-item active" href="./about.html">About</a>
                            <a className="list-group-item" href="./home.html">Home</a>
                            */}
    {/* 在React中靠路由链接实现切换组件 */}
    {/* <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink> */}
    <MyNavLinker to="/about">About</MyNavLinker>
      <MyNavLinker to="/home">Home</MyNavLinker>
      </div>
      </div>
      <div className="col-xs-6">
      <div className="panel">
      <div className="panel-body">
      {/* 注册路由 */}
      <Switch>
      <Route path="/about" component={About} />
      <Route path="/home" component={Home} />
      </Switch>
      </div>
      </div>
      </div>
      </div>
      </div>
    )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class Message extends Component {
    render() {
        return (
            <ul>
                <li>
                    <a href="/message1">message001</a>&nbsp;&nbsp;
                    </li>
                <li>
                    <a href="/message2">message002</a>&nbsp;&nbsp;
                    </li>
                <li>
                    <a href="/message/3">message003</a>&nbsp;&nbsp;
                    </li>
            </ul>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  export default class News extends Component {
    render() {
        return (
            <ul>
                <li>news001</li>
                <li>news002</li>
                <li>news003</li>
            </ul>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import { Route, Switch } from 'react-router'
  import MyNavLinker from '../../components/MyNavLinker'
  import Message from './Message'
  import News from './News'
  
  export default class Home extends Component {
    render() {
        return (
            <div>
              <h2>Home组件内容</h2>
              <div>
                <ul className="nav nav-tabs">
                  <li>
                    {/* <a className="list-group-item active" href="./home-news.html">News</a> */}
                    <MyNavLinker to="/home/news">News</MyNavLinker>
                  </li>
                  <li>
                    {/* <a className="list-group-item " href="./home-message.html">Message</a> */}
                    <MyNavLinker to="/home/message">Message</MyNavLinker>
                  </li>
                </ul>
                {/* 注册路由 */}
                <Switch>
                    <Route path="/home/news" component={News}/>
                    <Route path="/home/message" component={Message}/>
                </Switch>
              </div>
            </div>
        )
    }
  }
  ```
- ### 4.7.11.  向路由组件传递参数
- params参数
	- 路由链接（携带参数）：
	  
	  ```
	  <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link>
	  ```
	- 注册路由（声明接收）：
	  
	  ```
	  <Route path="/home/message/detail/:id/:title" component={Detail}/>
	  ```
	- 接收参数：
	  
	  ```
	  const {id,title}=this.props.match.params
	  ```
- 示例：
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
  state = {
    messageArr: [
      { id: '01', title: 'message1' },
      { id: '02', title: 'message2' },
      { id: '03', title: 'message3' },
    ]
  }
  render() {
    const { messageArr } = this.state
    return (
      <div>
      <ul>
      {
        messageArr.map((msgObj) => {
          return (
            <li key={msgObj.id}>
                       {/* 向路由组件传递params参数 */}
                       <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link>
                       </li>
                      )
  })
  }
  </ul>
  <hr/>
  {/* 声明接收params参数 */}
  <Route path="/home/message/detail/:id/:title" component={Detail}/>
  </div>
  )
  }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  
  const data=[
  {id:'01',content:'你好，同学'},
  {id:'02',content:'你好，世界'},
  {id:'03',content:'你好，未来的自己'}
  ]
  export default class Detail extends Component {
  render() {
    const {id,title}=this.props.match.params
    const findResult=data.find(
      (detailObj)=>{
        return detailObj.id===id
      }
    )
    return (
      <div>
      <ul>
      <li>ID:{id}</li>
      <li>TITLE:{title}</li>
      <li>CONTENT:{findResult.content}</li>
      </ul>
      </div>
    )
  }
  }
  ```
- ### 4.7.12.  Search参数
- 路由链接（携带参数）：
  
  ```
  <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link>
  ```
- 注册路由（声明接收）：
  
  ```
  <Route path="/home/message/detail" component={Detail}/>
  ```
- 接收参数：
  
  ```
  this.props.location
  ```
- 备注：获取到的search是urlencoded编码字符串中，需要借助querystring解析。
- ### 4.7.13.  state参数解析
- 路由链接（携带参数）：
  
  ```
  <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}}>{msgObj.title}</Link>
  ```
- 注册路由（声明接收）：
  
  ```
  <Route path="/home/message/detail" component={Detail}/>
  ```
- 接收参数：
  
  ```
  this.props.location.state||{}
  ```
- 备注：刷新也可以保留住参数
- ### 4.7.14.  push和replace的使用
- push：a-b-c可以返回到上一级
- replace：a-b-c不可以返回到上一级，适用于登录后，不需要重新回到登页面
- 可以把浏览器的浏览地址看为一个地址栈。每访问一个页面，地址就会进入地址栈。push和replace就在这个情况下使用。
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
    state = {
        messageArr: [
            { id: '01', title: 'message1' },
            { id: '02', title: 'message2' },
            { id: '03', title: 'message3' },
        ]
    }
    render() {
        const { messageArr } = this.state
        return (
            <div>
                <ul>
                    {
                        messageArr.map((msgObj) => {
                            return (
                                <li key={msgObj.id}>
                                    {/* 向路由组件传递params参数 */}
                                    {/* <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    {/* <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}} replace>{msgObj.title}</Link>
  
                                </li>
                            )
                        })
                    }
                </ul>
                <hr/>
                {/* 声明接收params参数 */}
                {/* <Route path="/home/message/detail/:id/:title" component={Detail}/> */}
  
                {/* 声明接收search参数
                <Route path="/home/message/detail" component={Detail}/> */}
  
                {/* 声明接收search参数 */}
                <Route path="/home/message/detail" component={Detail}/>
            </div>
        )
    }
  }
  ```
- ### 4.7.15.  编程式路由
  
  ```
  import React, { Component } from 'react'
  import { Link, Route } from 'react-router-dom'
  import Detail from './Detail'
  
  export default class Message extends Component {
    state = {
        messageArr: [
            { id: '01', title: 'message1' },
            { id: '02', title: 'message2' },
            { id: '03', title: 'message3' },
        ]
    }
  
    replaceShow=(id,title)=>{
        // replace跳转+携带params参数
        // this.props.history.replace(`/home/message/detail/${id}/${title}`)
  
        // replace跳转+携带search参数
        // this.props.history.replace(`/home/message/detail?id=${id}&title=${title}`)
  
        // replace跳转+携带state参数
        this.props.history.replace(`/home/message/detail`,{id,title})
    }
  
    pushShow=(id,title)=>{
        // push跳转+携带params参数
        // this.props.history.push(`/home/message/detail/${id}/${title}`)
  
        // push跳转+携带search参数
        // this.props.history.push(`/home/message/detail?id=${id}&title=${title}`)
  
        // push跳转+携带state参数
        this.props.history.push(`/home/message/detail`,{id,title})
    }
  
    back=()=>{
        this.props.history.goBack()
    }
  
    forward=()=>{
        this.props.history.goForward()
    }
  
    goto=()=>{
        this.props.history.go(2)
    }
  
    render() {
        const { messageArr } = this.state
        return (
            <div>
                <ul>
                    {
                        messageArr.map((msgObj) => {
                            return (
                                <li key={msgObj.id}>
                                    {/* 向路由组件传递params参数 */}
                                    {/* <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递search参数 */}
                                    {/* <Link to={`/home/message/detail?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link> */}
  
                                    {/* 向路由组件传递state参数 */}
                                    <Link to={{pathname:'/home/message/detail',state:{id:msgObj.id,title:msgObj.title}}}>{msgObj.title}</Link>
                                    &nbsp;<button onClick={()=>{this.replaceShow(msgObj.id,msgObj.title)}}>replace</button>
                                    &nbsp;<button onClick={()=>{this.pushShow(msgObj.id,msgObj.title)}}>push</button>
  
                                </li>
                            )
                        })
                    }
                </ul>
                <hr/>
                {/* 声明接收params参数 */}
                {/* <Route path="/home/message/detail/:id/:title" component={Detail}/> */}
  
                {/* 声明接收search参数
                <Route path="/home/message/detail" component={Detail}/> */}
  
                {/* 声明接收state参数 */}
                <Route path="/home/message/detail" component={Detail}/>
  
                <button onClick={this.back}>后退</button>
                <button onClick={this.forward}>前进</button>
                <button onClick={this.goto}>go</button>
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  import qs from 'querystring'
  
  const data=[
    {id:'01',content:'你好，同学'},
    {id:'02',content:'你好，世界'},
    {id:'03',content:'你好，未来的自己'}
  ]
  export default class Detail extends Component {
    render() {
  
        // 接收params参数
        // const {id,title}=this.props.match.params
  
        //接收search参数
        // const {search}=this.props.location
        // const {id,title}=qs.parse(search.slice(1))
        // const findResult=data.find(
        //     (detailObj)=>{
        //         return detailObj.id===id
        //     }
        // )
  
        // 接收state参数
        const {id,title}=this.props.location.state||{}
  
        const findResult=data.find(
            (detailObj)=>{
                return detailObj.id===id
            }
        )||{}
        return (
            <div>
                <ul>
                    <li>ID:{id}</li>
                    <li>TITLE:{title}</li>
                    <li>CONTENT:{findResult.content}</li>
                </ul>
            </div>
        )
    }
  }
  ```
- 对应的路由方式需要对应一定的Route否则就不会产生相应的效果。
- ### 4.7.16.  BrowserRouter和HashRouter的区别
- 底层原理不一样
	- BrowserRouter使用的是H5的history API，不兼容IE9以下的版本。
	- HashROuter使用的是URL的哈希值。
- path表现形式不一样
	- BrowserRouter的路径中没有#，例如：localhost:3000/demo/test
	- HashRouter的路径包含#，例如：localhost:3000/#/demo/test
- 刷新后对路由state参数的影响
	- BrowserRouter没有任何影响，因为state保存在history对象中，
	- HashRouter刷新后会导致路由state参数的丢失。
- 备注：HashRouter可以用于解决一些路径错误相关的问题。
- ### 4.7.17.  withRouter函数
- 这个函数能让一般的组件变成为路由组件。
  
  ```
  import React, { Component } from 'react'
  import { withRouter } from 'react-router'
  
  class News extends Component {
    render() {
        return (
            <ul>
                <li>news001</li>
                <li>news002</li>
                <li>news003</li>
            </ul>
        )
    }
  }
  
  export default withRouter(News)
  // withRouter可以加工一般组件，让一般组件具有路由组件所特有的API
  // withRouter是一个新的组件
  ```
- ## 4.8.  Ant Design UI 框架
- 蚂蚁集团的企业级产品是一个庞大且复杂的系统，数量多且功能复杂，而且变动和并发频繁，常常需要设计者与开发者能快速做出响应。同时这类产品中有存在很多类似的页面以及组件，可以通过抽象得到一些稳定且高复用性的内容。
- 随着商业化的趋势，越来越多的企业级产品对更好的用户体验有了进一步的要求。带着这样的一个终极目标，我们（蚂蚁集团体验技术部）经过大量项目实践和总结，逐步打磨出一个服务于企业级产品的设计体系 —— Ant Design。基于「自然」、「确定性」、「意义感」、「生长性」四大设计价值观，通过模块化解决方案，降低冗余的生产成本，让设计者专注于更好的用户体验。
  
  ```
  import React, { Component } from 'react'
  import {Button,DatePicker} from 'antd'
  import {WechatOutlined,WeiboOutlined,SearchOutlined} from '@ant-design/icons'
  import 'antd/dist/antd.css'
  
  export default class App extends Component {
  
    render() {
        return (
            <div>
                App......
                <button>点我</button>
                <Button type="primary">按钮1</Button>
                <Button>按钮2</Button>
                <Button type="link">按钮3</Button>
                <Button type="primary" icon={<SearchOutlined/>}>Search</Button>
                <WechatOutlined/>
                <WeiboOutlined/>
                <DatePicker/>
            </div>
        )
    }
  }
  ```
- ### 4.8.1.  antd按需引入
- 为了减少代码的传输量，可以按需引入相应的css文件
- 安装antd         npm install antd --save
- 安装（react-app-rewired）一个对 create-react-app 进行自定义配置的社区解决方案
- 示例
	- package.json
	  
	  ```
	  {
	  "name": "react_staging",
	  "version": "0.1.0",
	  "private": true,
	  "homepage": ".",
	  "dependencies": {
	   "@ant-design/icons": "^4.6.2",
	   "@ant-design/icons-react": "^2.0.1",
	   "@testing-library/jest-dom": "^5.11.10",
	   "@testing-library/react": "^11.2.6",
	   "@testing-library/user-event": "^12.8.3",
	   "antd": "^4.15.2",
	   "axios": "^0.21.1",
	   "nanoid": "^3.1.22",
	   "prop-types": "^15.7.2",
	   "pubsub-js": "^1.9.3",
	   "react": "^17.0.2",
	   "react-dom": "^17.0.2",
	   "react-router-dom": "^5.2.0",
	   "react-scripts": "4.0.3",
	   "web-vitals": "^1.1.1"
	  },
	  "scripts": {
	   "start": "react-app-rewired start",
	   "build": "react-app-rewired build",
	   "test": "react-app-rewired test",
	   "eject": "react-scripts eject"
	  },
	  "eslintConfig": {
	   "extends": [
	     "react-app",
	     "react-app/jest"
	   ]
	  },
	  "browserslist": {
	   "production": [
	     ">0.2%",
	     "not dead",
	     "not op_mini all"
	   ],
	   "development": [
	     "last 1 chrome version",
	     "last 1 firefox version",
	     "last 1 safari version"
	   ]
	  }
	  }
	  ```
	- config-override.js
	  
	  ```
	  // 配置具体的修改规则
	  const {override, fixBabelImports} =require('customize-cra')
	  
	  module.exports=override(
	  fixBabelImports(
	   'import',{
	     libraryName:'antd',
	     libraryDirectory:'es',
	     style:'css',
	   }
	  )
	  )
	  ```
	- src/App.jsx
	  
	  ```
	  import React, { Component } from 'react'
	  import {Button,DatePicker} from 'antd'
	  import {WechatOutlined,WeiboOutlined,SearchOutlined} from '@ant-design/icons'
	  
	  export default class App extends Component {
	  
	   render() {
	       return (
	           <div>
	               App......
	               <button>点我</button>
	               <Button type="primary">按钮1</Button>
	               <Button>按钮2</Button>
	               <Button type="link">按钮3</Button>
	               <Button type="primary" icon={<SearchOutlined/>}>Search</Button>
	               <WechatOutlined/>
	               <WeiboOutlined/>
	               <DatePicker/>
	           </div>
	       )
	   }
	  }
	  ```
- ### 4.8.2.  Antd 自定义样式
- config-orverride.js
  
  ```
  // 配置具体的修改规则
  const {override, fixBabelImports} =require('customize-cra')
  
  module.exports=override(
  fixBabelImports(
    'import',{
      libraryName:'antd',
      libraryDirectory:'es',
      style:'css',
    }
  ),
  addLessLoader({
    javascriptEnabled:true,
    modifyVars:{
      '@primary-color':'#1DA57A'
    }
  })
  )
  ```
- ## 4.9.  Redux
- 首先，redux并不是必须的，它的作用相当于在顶层组件之上又加了一个组件，作用是进行逻辑运算、储存数据和实现组件尤其是顶层组件的通信。如果组件之间的交流不多，逻辑不复杂，只是单纯的进行视图的渲染，这时候用回调，context就行，没必要用redux，用了反而影响开发速度。但是如果组件交流特别频繁，逻辑很复杂，那redux的优势就特别明显了。我第一次做react项目的时候并没有用redux，所有的逻辑都是在组件内部实现，当时为了实现一个逻辑比较复杂的购物车，洋洋洒洒居然写了800多行代码，回头一看我自己都不知道写的是啥，画面太感人。
- 先简单说一下redux和react是怎么配合的。react-redux提供了connect和Provider两个好基友，它们一个将组件与redux关联起来，一个将store传给组件。组件通过dispatch发出action，store根据action的type属性调用对应的reducer并传入state和这个action，reducer对state进行处理并返回一个新的state放入store，connect监听到store发生变化，调用setState更新组件，此时组件的props也就跟着变化。
- ### 4.9.1.  Redux工作原理
- 随着应用程序单页面需求的越来越复杂，应用状态的管理也变得越来越混乱，而Redux的就是为解决这一问题而出现的。在一个大型的应用程序中，应用的状态不仅包括从服务器获取的数据，还包括本地创建的数据，以及反应本地UI状态的数据，而Redux正是为解决这一复杂问题而存在的。
- redux作为一种单向数据流的实现，配合react非常好用，尤其是在项目比较大，逻辑比较复杂的时候，单项数据流的思想能使数据的流向、变化都能得到清晰的控制，并且能很好的划分业务逻辑和视图逻辑。下图是redux的基本运作的流程。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1619420470572-c3826de0-6f42-4a0e-bb1f-bd5ce3b16904.png)
- 如上图所示，该图展示了Redux框架数据的基本工作流程。简单来说，首先由view dispatch拦截action，然后执行对应reducer并更新到store中，最终views会根据store数据的改变执行界面的刷新渲染操作。
- 同时，作为一款应用状态管理框架，为了让应用的状态管理不再错综复杂，使用Redux时应遵循三大基本原则，否则应用程序很容易出现难以察觉的问题。这三大原则包括：
	- 单一数据源
		- 整个应用的State被存储在一个状态树中，且只存在于唯一的Store中。
	- State是只读的
		- 对于Redux来说，任何时候都不能直接修改state，唯一改变state的方法就是通过触发action来间接的修改。而这一看似繁琐的状态修改方式实际上反映了Rudux状态管理流程的核心思想，并因此保证了大型应用中状态的有效管理。
	- 应用状态的改变通过纯函数来完成
		- Redux使用纯函数方式来执行状态的修改，Action表明了修改状态值的意图，而真正执行状态修改的则是Reducer。且Reducer必须是一个纯函数，当Reducer接收到Action时，Action并不能直接修改State的值，而是通过创建一个新的状态对象来返回修改的状态。
- ### 4.9.2.  React中的实现
  
  ```
  import React, { Component } from 'react'
  
  export default class Count extends Component {
    state={count:0}
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        this.setState({count:count+value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        this.setState({count:count-value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        if(count % 2!==0){
            this.setState({count:count+value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        const {count} = this.state
        setTimeout(()=>{
            this.setState({count:count+value*1})
        },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：0</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  import React, { Component } from 'react'
  // 引入store，用于获取redux中保存的状态
  import store from '../../reduxs/store'
  
  export default class Count extends Component {
  
    componentDidMount(){
        // 检测redux中状态的变化，只要变化，就可以调用render
        store.subscribe(()=>{
            this.setState({})
        })
    }
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'increment',data:value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'decrement',data:value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const count=store.getState()
        if(count % 2!==0){
            store.dispatch({type:'increment',data:value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        setTimeout(()=>{
            store.dispatch({type:'increment',data:value*1})
        },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：{store.getState()}</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  /*
    1.该文件是用于创建一个为Count组件服务的reducer，reducer的本质就是一个函数
    2.reducer函数会接到两个参数，分别为：之前的状态（preState）,动作对象（action）
  */
  const initState=0
  export default function countReducer(preState=initState,action){
    if(preState===undefined) preState=0
    // 从action对象中获取：type、data
    const {type,data}=action
    // 根据type决定如何加工数据
    switch(type){
        case 'increment':
            return preState+data
        case 'decrement':
            return preState-data
        default:
            return 0;
    }
  }
  ```
  
  ```
  /*
    该文件专门用于暴露一个store对象，整个应用只有一个store对象
  */
  // 引入createStore，专门用于创建redux中最为核心的store对象
  import {createStore} from 'redux'
  // 引入为Count组件服务的reducer
  import countReducer from './count_reducer'
  // 暴露store
  export default createStore(countReducer)
  ```
- 以上就是一个简易版的redux组件实现的案例。
- ### 4.9.3.  Redux完整实现
- 新增文件：
	- count_action.js 专门用于创建action对象
	- constant.js 防止编码疏忽写错action中的内容
	  
	  ```
	  /*
	   1.该文件是用于创建一个为Count组件服务的reducer，reducer的本质就是一个函数
	   2.reducer函数会接到两个参数，分别为：之前的状态（preState）,动作对象（action）
	  */
	  import {INCREMENT,DECREMENT} from './constant'
	  
	  const initState=0
	  export default function countReducer(preState=initState,action){
	   if(preState===undefined) preState=0
	   // 从action对象中获取：type、data
	   const {type,data}=action
	   // 根据type决定如何加工数据
	   switch(type){
	       case INCREMENT:
	           return preState+data
	       case DECREMENT:
	           return preState-data
	       default:
	           return 0;
	   }
	  }
	  ```
	  
	  ```
	  /*
	   该文件专门为Count组件生成action对象
	  */
	  import {INCREMENT,DECREMENT} from './constant'
	  
	  export const createIncrementAction=data=>({type:INCREMENT,data})
	  export const createDecrementAction=data=>({type:DECREMENT,data})
	  ```
	  
	  ```
	  /*
	   该模块是用于定义action对象中type类型的常量值
	   目的就是便于管理的同时防止程序员单词写错
	  */
	  export const INCREMENT = 'increment'
	  export const DECREMENT = 'decrement'
	  ```
- ### 4.9.4.  Redux异步action
- 明确：延迟的动作不想提交给组件本身，想要交给action
- 何时需要异步action：想要对状态进行操作，但是具体的数据靠异步任务返回（非必须）。
- 具体编码：
	- npm add redux-thunk，并配置在store中
	- 创建action的函数不再返回一般对象，而是一个函数，该函数中写异步任务。
	- 异步任务有结果只会，分发一个同步的action去真正操作数据。
- 备注：异步action不是必须要写的，完全可以自己等待异步任务的结果了再去分发同步action。
  
  ```
  /*
    该文件专门为Count组件生成action对象
  */
  import {INCREMENT,DECREMENT} from './constant'
  import store from './store'
  
  // 同步action，就是指action的值为Object类型的一般对象
  export const createIncrementAction=data=>({type:INCREMENT,data})
  export const createDecrementAction=data=>({type:DECREMENT,data})
  
  // 异步action，就是指action的值为函数,异步action中一般都会调用同步action
  // 异步action不是必须要用的
  export const createIncrementAsyncAction=(data,time)=>{
    return ()=>{
        setTimeout(()=>{
            dispatch(createIncrementAction(data))
        },time)
    }
  }
  ```
  
  ```
  /*
    该文件专门用于暴露一个store对象，整个应用只有一个store对象
  */
  // 引入createStore，专门用于创建redux中最为核心的store对象
  import {createStore,applyMiddleware} from 'redux'
  // 引入为Count组件服务的reducer
  import countReducer from './count_reducer'
  // 引入redux-thunk，用于支持异步action
  import thunk from 'redux-thunk'
  // 暴露store
  export default createStore(countReducer, applyMiddleware(thunk))
  ```
  
  ```
  import React, { Component } from 'react'
  // 引入store，用于获取redux中保存的状态
  import store from '../../reduxs/store'
  
  export default class Count extends Component {
  
    componentDidMount(){
        // 检测redux中状态的变化，只要变化，就可以调用render
        store.subscribe(()=>{
            this.setState({})
        })
    }
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'increment',data:value*1})
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
        store.dispatch({type:'decrement',data:value*1})
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
        const count=store.getState()
        if(count % 2!==0){
            store.dispatch({type:'increment',data:value*1})
        }
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
        // setTimeout(()=>{
            store.dispatch({type:'increment',data:value*1})
        // },500)
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：{store.getState()}</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
- ## 4.10.  React-redux
- ### 4.10.1.  简介
- 所有的UI组件都应该包裹一个容器组件，他们是父子关系
- 容器组件是真正和redux打交道的，里面可以随意的使用redux的api
- UI组件中不能使用任何redux的api
- 容器组件会传给UI组件：
	- redux中所保存的状态
	- 用于操作状态的方法。
- 备注：容器给UI传递：状态、操作状态的方法，均通过props传递。
  
  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1619766742634-7a450dec-65f2-4801-918b-340e4f926c9a.png)
  
  ```
  import React, { Component } from 'react'
  
  export default class Count extends Component {
  
    // 加法
    increment=()=>{
        const {value} = this.selectNumber
    }
  
    // 减法
    decrement=()=>{
        const {value} = this.selectNumber
    }
  
    // 奇数再加
    incrementIfOdd=()=>{
        const {value} = this.selectNumber
    }
  
    // 异步加
    incrementAsync=()=>{
        const {value} = this.selectNumber
    }
  
    render() {
        return (
            <div>
                <h1>当前求和为：？？？？</h1>
                <select ref={c=>this.selectNumber=c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
  }
  ```
  
  ```
  // 引入Count的UI组件
  import CountUI from '../../components/Count'
  // 引入connect用于连续UI组件与redux
  import {connect} from 'react-redux'
  
  // 使用connect()()创建并暴露一个Count的容器组件
  export default connect()(CountUI)
  ```
  
  ```
  import React, { Component } from 'react'
  import Count from './containers/Count'
  import store from './reduxs/store'
  
  export default class App extends Component {
    render() {
        return (
            <div>
                <Count store={store}/>
            </div>
        )
    }
  }
  ```
- ### 4.10.2.  基本使用
- 概念
	- UI组件：不能使用任何redux的api，只负责页面的呈现、交互等。
	- 容器组件：负责和redux通信，将结果交给UI组件
- 如何创建一个容器组件——靠react-redux的connect函数
	- connect（mapStateToProps，mapDispatchToProps）（UI组件）
		- mapStateToProps：映射状态，返回值是一个对象
		- mapDispatchToProps：映射操作状态的方法，返回值是一个对象
	- 备注：容器组件中的store是靠props传进去的，而不是在容器组件中直接引入。
	  
	  ```
	  // 引入Count的UI组件
	  import CountUI from '../../components/Count'
	  // 引入action
	  import {
	   createIncrementAction,
	   createDecrementAction,
	   createIncrementAsyncAction,
	  } from '../../reduxs/count_action'
	  // 引入connect用于连接UI组件与redux
	  import {connect} from 'react-redux'
	  
	  /*
	   1.mapStateToProps函数返回的是一个对象
	   2.返回的对象中的key该作为传递给UI组件props的key，value就作为传递给UI组件props的value
	   3.mapStateToProps用于传递状态
	  */
	  function mapStateToProps(state){
	   return {count:state}
	  }
	  
	  /*
	   1.mapDispatchToProps函数返回的是一个对象；
	   2.返回的对象中的key就作为传递给UI组件props的key，value就作为传递给UI组件props的value
	   3.mapDispatchToProps用于传递操作状态的方法
	  */
	  function mapDispatchToProps(dispatch){
	   return {
	       jia:number=>dispatch(createIncrementAction(number)),
	       jian:number=>dispatch(createDecrementAction(number)),
	       jiaAsync:(number,time)=>dispatch(createIncrementAsyncAction(number,time))
	   }
	  }
	  
	  // 使用connect()()创建并暴露一个Count的容器组件
	  export default connect(mapStateToProps,mapDispatchToProps)(CountUI)
	  ```
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Count extends Component {
	  
	   // 加法
	   increment=()=>{
	       const {value} = this.selectNumber
	       this.props.jia(value*1)
	   }
	  
	   // 减法
	   decrement=()=>{
	       const {value} = this.selectNumber
	       this.props.jian(value*1)
	   }
	  
	   // 奇数再加
	   incrementIfOdd=()=>{
	       const {value} = this.selectNumber
	       if(this.props.count%2!==0){
	           this.props.jia(value*1)
	       }
	   }
	  
	   // 异步加
	   incrementAsync=()=>{
	       const {value} = this.selectNumber
	       this.props.jianAsync(value*1,500)
	   }
	  
	   render() {
	       return (
	           <div>
	               <h1>当前求和为：{this.props.count}</h1>
	               <select ref={c=>this.selectNumber=c}>
	                   <option value="1">1</option>
	                   <option value="2">2</option>
	                   <option value="3">3</option>
	               </select>&nbsp;
	               <button onClick={this.increment}>+</button>&nbsp;
	               <button onClick={this.decrement}>-</button>&nbsp;
	               <button onClick={this.incrementIfOdd}>当前求和为奇数再加</button>&nbsp;
	               <button onClick={this.incrementAsync}>异步加</button>&nbsp;
	           </div>
	       )
	   }
	  }
	  ```
- ### 4.10.3.  mapDispatchToProps优化
- 对于这部分优化就是对于在容器组件中的一个组件进行优化
  
  ```
  // 引入Count的UI组件
  import CountUI from '../../components/Count'
  // 引入action
  import {
    createIncrementAction,
    createDecrementAction,
    createIncrementAsyncAction,
  } from '../../reduxs/count_action'
  // 引入connect用于连接UI组件与redux
  import {connect} from 'react-redux'
  
  // 使用connect()()创建并暴露一个Count的容器组件
  export default connect(
    // mapDispatchToProps的一般写法
    /*
    dispatch=>({
        jia:number=>dispatch(createIncrementAction(number)),
        jian:number=>dispatch(createDecrementAction(number)),
        jiaAsync:(number,time)=>dispatch(createIncrementAsyncAction(number,time))
    })
    */
    state=>({count:state}),
    // mapDispatchToProps的简写
    {
        jia:createIncrementAction,
        jian:createDecrementAction,
        jiaAsync:createIncrementAsyncAction
    }
  )(CountUI)
  ```
- ### 4.10.4.  Provider的使用
- 为了减少在每一次调用的时候需要使用到store={store}这句话。可以使用Provider这样一个组件直接使用。
  
  ```
  import React from 'react'
  import ReactDOM from 'react-dom'
  import App from './App'
  import store from './reduxs/store'
  import {Provider} from 'react-redux'
  
  ReactDOM.render(
    <Provider store={store}>
        <App/>
    </Provider>,
    document.getElementById('root')
  )
  
  // 检测redux中状态的改变，如redux的状态发生了改变，那么重新渲染App组件
  // store.subscribe(()=>{
  //     ReactDOM.render(<App/>,document.getElementById('root'))
  // })
  ```
- ### 4.10.5.  编写Person案例
  
  ```
  import React, { Component } from 'react'
  import {nanoid} from 'nanoid'
  
  export default class Person extends Component {
  
    addPerson=()=>{
        const name=this.nameNode.value
        const age=this.ageNode.value
        const personObj={id:nanoid(),name,age}
        console.log(personObj)
    }
  
    render() {
        return (
            <div>
                <h2>Person...</h2>
                <input ref={c=>this.nameNode=c} type="text" placeholder="输入名字"/>
                <input ref={c=>this.ageNode=c} type="text" placeholder="输入年龄"/>
                <button onClick={this.addPerson}>添加</button>
                <ul>
                    <li>姓名1---年龄1</li>
                    <li>姓名2---年龄2</li>
                    <li>姓名3---年龄3</li>
                </ul>
            </div>
        )
    }
  }
  ```
	- 上述代码就是将一个完整的静态页面组件编写完毕。
	  
	  ```
	  import {ADD_PERSON} from '../constant'
	  
	  // 创建一个增加人的action动作对象
	  export const createAddPersonAction= personObj =>({
	   type:ADD_PERSON,
	   data:personObj
	  })
	  ```
	  
	  ```
	  import {ADD_PERSON} from '../constant'
	  
	  // 初始化列表
	  const initState=[{id:'001',name:'tom',age:10}]
	  export default function personReducer(perState=initState,action){
	   const {type,data}=action
	   switch (type){
	       case ADD_PERSON:
	           return [data,...perState]
	       default:
	           return perState
	   }
	  }
	  ```
	- 以上就是对于redux中一个组件的准备工作的编写。首先是完成静态组件的编写，然后再将相应的action和reducer编写好。
- ### 4.10.6.  Redux数据共享完整版
- 定义一个Person组件，和Count组件通过Redux进行数据共享。
- 为Person组件编写：reducer、action，配置constant常量。
- 重点：Person的reducer和Count的Reducer要使用combineReducers进行合并，合并后的总状态是一个对象。
- 交给store的是总reducer，最后注意在组件中取出状态的时候，记得要取到位。
  
  ```
  import React, { Component } from 'react'
  import {nanoid} from 'nanoid'
  import {connect} from 'react-redux'
  import {createAddPersonAction} from '../../reduxs/action/person'
  
  class Person extends Component {
  
    addPerson=()=>{
        const name=this.nameNode.value
        const age=this.ageNode.value
        const personObj={id:nanoid(),name,age}
        this.props.AddOne(personObj)
    }
  
    render() {
        return (
            <div>
                <h2>这是Person组件。上面的组件的和Count为:{this.props.num}</h2>
                <input ref={c=>this.nameNode=c} type="text" placeholder="输入名字"/>
                <input ref={c=>this.ageNode=c} type="text" placeholder="输入年龄"/>
                <button onClick={this.addPerson}>添加</button>
                <ul>
                    {
                        this.props.persons.map((p)=>{
                            return <li>{p.name}--{p.age}</li>
                        })
                    }
                </ul>
            </div>
        )
    }
  }
  
  export default connect(
    // 映射状态
    state=>({persons:state.rens,num:state.he}),
    // 映射操作状态的方法
    {AddOne:createAddPersonAction}
  )(Person)
  ```
	- 这里面的Person的静态组件和容器组件合在了一起。
	  
	  ```
	  /*
	   该文件专门用于暴露一个store对象，整个应用只有一个store对象
	  */
	  // 引入createStore，专门用于创建redux中最为核心的store对象
	  import {createStore,applyMiddleware,combineReducers} from 'redux'
	  // 引入为Count组件服务的reducer
	  import countReducer from './reducer/count'
	  // 引入为Person组件服务的reducer
	  import personReducer from './reducer/person'
	  // 引入redux-thunk，用于支持异步action
	  import thunk from 'redux-thunk'
	  
	  // 汇总所有的reducer变为一个总的reducer
	  const allReducer=combineReducers({
	   he:countReducer,
	   rens:personReducer
	  })
	  
	  // 暴露store
	  export default createStore(allReducer, applyMiddleware(thunk))
	  ```
- ### 4.10.7.  纯函数
- 一类特别的函数：只要是同样的输入（实参），必定得到同样的输出（返回）
- 必须遵守以下的一些约束：
	- 不得改写参数数据；
	- 不会产生任何副作用，例如网络请求，输入和输出设备；
	- 不能调用Data.now()或者Math.random()等不纯的方法；
- redux的reducer函数必须是一个纯函数。
  
  ```
  import {ADD_PERSON} from '../constant'
  
  // 初始化列表
  const initState=[{id:'001',name:'tom',age:10}]
  export default function personReducer(perState=initState,action){
    const {type,data}=action
    switch (type){
        case ADD_PERSON:
            // preState.unshift(data)
            // 此处不可以这样写，这样会导致preState被该写了，reducer就不是纯函数了。
            return [data,...perState]
        default:
            return perState
    }
  }
  ```
- ## 4.11.  补充内容
- ### 4.11.1.  SetState
- setState更新状态的2种写法
	- （1）setState（stateChange，[callback]）————对象式的setState
		- stateChange为状态改变对象（该对象可以体现出状态的更改）
		- callback是可选回调函数，他在状态更新完毕、界面也更新后才能被调用
	- （2）setState（updater，[callback]）————函数式的setState
		- updater为返回stateChange对象的函数
		- updater可以接收到state和props
		- callback是可选的回调函数，在状态更新、界面也更新后才会被调用。
- 总结：
	- 对象式的setState是函数式的setState的简写方式（语法糖）
	- 使用原则：
		- 如果新状态不依赖于原状态=>使用对象方式
		- 如果新状态依赖于原状态=>使用函数方式
		- 如果需要在setState()执行后获取最新的状态数据，要在第二个callback函数中读取。
		  
		  ```
		  import React, { Component } from 'react'
		  
		  export default class Demo extends Component {
		  
		  state={count:0}
		  
		  add=()=>{
		      //1.获取原来的count
		      const {count}=this.state
		      //2.更新状态
		      this.setState({count:count+1},()=>{
		          console.log(this.state.count)
		      })
		      console.log('输出为',this.state.count)
		  
		      // 函数式setState
		      // this.setState(state=>({count:state.count+1}))
		  }
		  
		  render() {
		      return (
		          <div>
		              <h1>当前求和为：{this.state.count}</h1>
		              <button onClick={this.add}>点我+1</button>
		          </div>
		      )
		  }
		  }
		  ```
- ### 4.11.2.  LazyLoad和Hooks
  
  ```
  //1.通过React的lazy函数配合import()函数动态加载路由组件 ===> 路由组件代码会被分开打包
  const Login = lazy(()=>import('@/pages/Login'))
  
  //2.通过<Suspense>指定在加载得到路由打包文件前显示一个自定义loading界面
  <Suspense fallback={<h1>loading.....</h1>}>
        <Switch>
            <Route path="/xxx" component={Xxxx}/>
            <Redirect to="/login"/>
        </Switch>
    </Suspense>
  ```
- 示例
  
  ```
  import React, { Component,lazy, Suspense } from 'react'
  import { NavLink, Route } from 'react-router-dom'
  // import About from './components/About'
  // import Home from './components/Home'
  
  const Home = lazy(()=>import('./Home'))
  const About = lazy(()=>import('./About'))
  
  export default class Demo extends Component {
  
    render() {
        return (
            <div>
                <div className="row">
                    <div className="col-xs-offset-2 col-xs-8">
                        <div className="page-header"><h2>React Router Demo</h2></div>
                    </div>
                </div>
                <div className="row">
                    <div className="col-xs-2 col-xs-offset-2">
                        <div className="list-group">
                            {/* 在React中靠路由链接实现切换组件 */}
                            <NavLink className="lislist-group-item" to="/about">About</NavLink>
                            <NavLink className="lislist-group-item" to="/home">Home</NavLink>
                        </div>
                    </div>
                    <div className="col-xs-6">
                        <div className="panel">
                            <div className="panel-body">
                                <Suspense fallback={<h1>Loading......</h1>}>
                                    {/* 注册路由 */}
                                    <Route path="/about" component={About} />
                                    <Route path="/home" component={Home} />
                                </Suspense>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        )
    }
  }
  ```
- lazyLoad的用法就是为了不同时加载所有的路径而实现的一种路由组件。
- ### 4.11.3.  EffectHook与RefHook
- **EffectHook**
- (1). Effect Hook 可以让你在函数组件中执行副作用操作(用于模拟类组件中的生命周期钩子)
- (2). React中的副作用操作:
  
        发ajax请求数据获取
  
        设置订阅 / 启动定时器
  
        手动更改真实DOM
- (3). 语法和说明:  
  
        useEffect(() => {  
  
          // 在此可以执行任何带副作用操作
  
          return () => { // 在组件卸载前执行
  
            // 在此做一些收尾工作, 比如清除定时器/取消订阅等
  
          }
  
        }, [stateValue]) // 如果指定的是[], 回调函数只会在第一次render()后执行
- (4). 可以把 useEffect Hook 看做如下三个函数的组合
  
        componentDidMount()
  
        componentDidUpdate()
  
    	componentWillUnmount()
- **RefHook**
- (1). Ref Hook可以在函数组件中存储/查找组件内的标签或任意其它数据
- (2). 语法: const refContainer = useRef()
- (3). 作用:保存标签对象,功能与React.createRef()一样
  
  ```
  import React, { Component } from 'react'
  
  function Demo(){
  
    const[count,setCount] = React.useState(0)
    const[name,setName] = React.useState('tom')
    const myRef=React.useRef()
  
    React.useEffect(()=>{
        let timer = setInterval(()=>{
            setCount(count=>count+1)
        },1000)
        return ()=>{
            clearInterval(timer)
        }
    },[])
  
    // 加的回调
    function add(){
        // setCount(count+1) // 第一种写法
        setCount(count => count+1)
    }
  
    // 提示输入的回调
    function show(){
        alert(myRef.current.value)
    }
    // 修改名字的回调
    function changeName(){
        setName('jack')
    }
    return (
        <div>
            <input type="text" ref={myRef} />
            <h2>当前求和为：{count}</h2>
            <h2>我的名字是：{name}</h2>
            <button onClick={add}>点我+1</button>
            <button onClick={changeName}>点我修改名字</button>
            <button onClick={show}>点我显示数据</button>
        </div>
    )
  }
  
  export default Demo
  ```
- ### 4.11.4.  Fragment和Context
- **Fragment**
- 作用就是可以不用必须有一个真实的DOM根标签。
- **这个标签不能携带任何的属性**
  
  ```
  import React, { Component, Fragment } from 'react'
  
  export default class Demo extends Component {
    render() {
        return (
            <Fragment>
                <input type="text"/>
                <input type="text"/>
            </Fragment>
        )
    }
  }
  ```
- **Context**
- 是一种组件间通信方式，常用于【祖组件】与【后代组件】间通信
- 1) 创建Context容器对象：
  
  const XxxContext = React.createContext()
- 2) 渲染子组时，外面包裹xxxContext.Provider, 通过value属性给后代组件传递数据：
  
  <xxxContext.Provider value={数据}>
  
              子组件
  
        </xxxContext.Provider>
- 3) 后代组件读取数据：
  
  //第一种方式:仅适用于类组件  
  
   static contextType = xxxContext  // 声明接收context
  
   this.context // 读取context中的value数据
  
   
  
  //第二种方式: 函数组件与类组件都可以
  
   <xxxContext.Consumer>
  
     {
  
       value => ( // value就是context中的value数据
  
         要显示的内容
  
       )
  
     }
  
   </xxxContext.Consumer>
- 在应用开发中一般不用context, 一般都用它的封装react插件
  
  ```
  import React, { Component } from 'react'
  
  // 创建Context对象
  const MyContext=React.createContext()
  const {Provider}=MyContext
  export default class A extends Component {
  
    state={username:'tom',age:18}
  
    render() {
        const {username,age}=this.state
        return (
            <div className="parent">
                <h3>我是A组件</h3>
                <h4>我的用户名是：{username}</h4>
                <Provider value={username}>
                    <B/>
                </Provider>
            </div>
        )
    }
  }
  
  class B extends Component{
    render(){
        return (
            <div className="child">
                <h3>我是B组件</h3>
                <C/>
            </div>
        )
    }
  }
  
  class C extends Component{
    static contextType=MyContext
    render(){
        console.log(this.context)
        return (
            <div className="grant">
                <h3>我是C组件</h3>
                <h4>我是从A组件接收到的用户名：{this.context}</h4>
            </div>
        )
    }
  }
  ```
- ### 4.11.5.  PureComponent
- Component的2个问题
- 只要执行setState(),即使不改变状态数据, 组件也会重新render() ==> 效率低
- 只当前组件重新render(), 就会自动重新render子组件，纵使子组件没有用到父组件的任何数据 ==> 效率低
- 只有当组件的state或props数据发生改变时才重新render（）
- Component中的shouldComponentUpdate（）总是返回true
- 解决办法
	- 办法1:
		- 重写shouldComponentUpdate()方法
		- 比较新旧state或props数据, 如果有变化才返回true, 如果没有返回false
	- 办法2:
		- 使用PureComponent
		- PureComponent重写了shouldComponentUpdate(), 只有state或props数据有变化才返回true
	- 注意:
		- 只是进行state和props数据的浅比较, 如果只是数据对象内部数据变了, 返回false
		- 不要直接修改state数据, 而是要产生新数据
	- 项目中一般使用PureComponent来优化
- 示例
  
  ```
  import React, { PureComponent } from 'react'
  
  export default class Parent extends PureComponent {
  
    state = {carName:"奔驰c36",stus:['小张','小李','小王']}
  
    addStu=()=>{
        const {stus}=this.state
        this.setState({stus:['小刘',...stus]})
    }
  
    changeCar = ()=>{
        this.setState({carName:"迈巴赫"})
    }
  
    shouldComponentUpdate(nextProps,nextState){
        console.log(this.props,this.state) // 目前的props和state
        console.log(nextProps,nextState)
        return !this.state.carName===nextState.carName
    }
  
    render() {
        return (
            <div className="parent">
                <h3>我是Parent组件</h3>
                <span>我的车名字是：{carName}</span>
                <button onClick={this.changeCar}>点我换车</button>
                <Child carName="奥拓"/>
            </div>
        )
    }
  }
  
  class Child extends PureComponent {
  
    // shouldComponentUpdate(nextProps,nextState){
    //     console.log(this.props,this.state) // 目前的props和state
    //     console.log(nextProps,nextState)
    //     return !this.state.carName===nextState.carName
    // }
  
    render() {
        return (
            <div className="child">
                <h3>我是Child组件</h3>
                <span>我的车名字是：{this.props.carName}</span>
            </div>
        )
    }
  }
  ```
- ### 4.11.6.  Renderprops
- Vue中:
	- 使用slot技术, 也就是通过组件标签体传入结构  <A><B/></A>
- React中:
	- 使用children props: 通过组件标签体传入结构
	- 使用render props: 通过组件标签属性传入结构,而且可以携带数据，一般用render函数属性
- childprops
  
  <A>
  
  <B>xxxx</B>
  
  </A>
  
  {this.props.children}
  
  问题: 如果B组件需要A组件内的数据, ==> 做不到
- renderprops
  
  <A render={(data) => <C data={data}></C>}></A>
  
  A组件: {this.props.render(内部state数据)}
  
  C组件: 读取A组件传入的数据显示 {this.props.data} 
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class Parent extends Component {
    render() {
        return (
            <div className="parent">
                <h3>我是Parent组件</h3>
                <A render={(name)=><B name={name}/>}/>
            </div>
        )
    }
  }
  
  class A extends Component {
    state={name:'tom'}
    render() {
        console.log(this.props)
        const {name}=this.state
        return (
            <div className="a">
                <h3>我是A组件</h3>
                {this.props.render(name)}
            </div>
        )
    }
  }
  
  class B extends Component {
    render() {
        console.log('B--render')
        return (
            <div className="b">
                <h3>我是B组件</h3>
            </div>
        )
    }
  }
  ```
- ### 4.11.7.  错误边界
- 错误边界(Error boundary)：用来捕获后代组件错误，渲染出备用页面。
- 只能捕获后代组件生命周期产生的错误，不能捕获自己组件产生的错误和其他组件在合成事件、定时器中产生的错误。
- 错误边界的使用范围必须是在容易出错的组件的父组件上使用才可以。
- 使用方法：getDerivedStateFromError配合componentDidCatch
	- 父组件
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Parent extends Component {
	  
	   state={
	       hasError:'' // 用于标识子组件是否产生错误
	   }
	  
	   // 当Parent的子组件出现报错时候，会触发getDerivedStateFromError调用，并携带错误信息
	   static getDerivedStateFromError(error){
	       console.log('@@@',error)
	       return {hasError:error}
	   }
	  
	   render() {
	       return (
	           <div>
	               <h2>我是Parent组件</h2>
	               {this.state.hasError?<h2>当前网络不稳定，稍后再试</h2>:<Child/>}
	           </div>
	       )
	   }
	  }
	  ```
	- 子组件
	  
	  ```
	  import React, { Component } from 'react'
	  
	  export default class Child extends Component {
	   state={
	       // users:[
	       //     {id:'001',name:'tom',age:18},
	       //     {id:'002',name:'jack',age:19},
	       //     {id:'003',name:'peiqi',age:20}
	       // ]
	       users:'abc'
	   }
	   render() {
	       return (
	           <div>
	               <h2>我是Child组件</h2>
	               {
	                   this.state.users.map((userObj)=>{
	                       return <h4 key={userObj.id}>{userObj.name}-----{userObj.age}</h4>
	                   })
	               }
	           </div>
	       )
	   }
	  }
	  ```
- ### 4.11.8.  通信方式总结
- 通信方式总结
  
  1.props：
  
  (1).children props
  
  (2).render props
  
  2.消息订阅-发布：
  
  pubs-sub、event等等
  
  3.集中式管理：
  
  redux、dva等等
  
  4.conText:
  
  生产者-消费者模式
- 配合方式
  
  父子组件：props
  
  兄弟组件：消息订阅-发布、集中式管理
  
  祖孙组件(跨级组件)：消息订阅-发布、集中式管理、conText(开发用的少，封装插件用的多)
- # 5.  简单案例
- ## 5.1.  TodoList案例
- ##### ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1617684660920-ea80ee77-42c1-4fd7-b0e4-a2ccca667071.png)
- 鼠标进入高亮状态
- 这一个主要就是针对整个页面当中的最小的一个list的组件进行修改。主要通过控制state中的某个属性就可以达到相应的效果。
- 示例：
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class Item extends Component {
  
    // 鼠标移入、移出状态
    state={mouse:false}
  
    // 鼠标移入移出的问题
    handleMouse=(flag)=>{
        return ()=>{
            this.setState({mouse:flag})
        }
    }
  
    render() {
        const {id,name,done}=this.props
        const {mouse}=this.state
        return (
            <div>
                <li style={{backgroundColor:mouse?'#ddd':'white'}} onMouseEnter={this.handleMouse(true)} onMouseLeave={this.handleMouse(false)}>
                    <label>
                        <input type="checkbox" defaultChecked={done} onChange={this.handleCheck(id)}/>
                        <span>{name}</span>
                    </label>
                    <button className="btn btn-danger" style={{ display: mouse?'block':'none' }}>删除</button>
                </li>
            </div>
        )
    }
  }
  ```
- 修改整体的todo的属性
- 如果取消选择列表中的任何一个待办事项，都需要保持相应属性的一致性，这就需要从子组件将属性修改至子组件，然后再回到父组件的一个流程。
- 示例：
	- src/App.jsx
	  
	  ```
	  import React, { Component } from 'react'
	  import Header from './components/Header'
	  import List from './components/List'
	  import Footer from './components/Footer'
	  import './App.css'
	  
	  export default class App extends Component {
	   // 初始化状态
	   state={
	       todos:[
	           {id:'001',name:'吃饭',done:true},
	           {id:'002',name:'睡觉',done:true},
	           {id:'003',name:'打豆豆',done:false},
	       ]
	   }
	  
	   //addTodo用于添加一个todo,接收的参数是todo对象
	   addTodo=(todoObj)=>{
	       // 获取原todos
	       const {todos}=this.state
	       // 追加一个todo
	       const newTodos=[todoObj,...todos]
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   // updateTodo用于更新一个todo
	   updateTodo=(id,done)=>{
	       // 获取状态中的todos
	       const {todos}=this.state
	       // 匹配处理数据
	       const newTodos=todos.map((todoObj)=>{
	           if(todoObj.id===id) return {...todoObj,done}
	           else return todoObj
	       })
	       this.setState({todos:newTodos})
	   }
	  
	   render() {
	       const {todos}=this.state
	       return (
	           <div>
	               <div className="todo-container">
	                   <div className="todo-wrap">
	                       <Header addTodo={this.addTodo}/>
	                       <List todos={todos} updateTodo={this.updateTodo}/>
	                       <Footer/>
	                   </div>
	               </div>
	           </div>
	       )
	   }
	  }
	  ```
	- src/components/List
	  
	  ```
	  import React, { Component } from 'react'
	  import Item from '../Item'
	  import './index.css'
	  import PropTypes from 'prop-types'
	  
	  export default class List extends Component {
	  
	   //对接收的props进行：类型、必要性的限制
	   static PropTypes = {
	       todos: PropTypes.array.isRequired,
	       updateTodo:PropTypes.array.isRequired
	   }
	  
	   render() {
	       const { todos, updateTodo } = this.props
	       return (
	           <div>
	               <ul className="todo-main">
	                   {
	                       todos.map((todo) => {
	                           return <Item key={todo.id} {...todo} updateTodo={updateTodo} />
	                       })
	                   }
	               </ul>
	           </div>
	       )
	   }
	  }
	  ```
	- src/components/Item
	  
	  ```
	  import React, { Component } from 'react'
	  import './index.css'
	  
	  export default class Item extends Component {
	  
	   // 鼠标移入、移出状态
	   state={mouse:false}
	  
	   // 鼠标移入移出的问题
	   handleMouse=(flag)=>{
	       return ()=>{
	           this.setState({mouse:flag})
	       }
	   }
	  
	   // 勾选、取消勾选某一个todo的回调
	   handleCheck=(id)=>{
	       return (event)=>{
	           this.props.updateTodo(id,event.target.checked)
	       }
	   }
	  
	   render() {
	       const {id,name,done}=this.props
	       const {mouse}=this.state
	       return (
	           <div>
	               <li style={{backgroundColor:mouse?'#ddd':'white'}} onMouseEnter={this.handleMouse(true)} onMouseLeave={this.handleMouse(false)}>
	                   <label>
	                       <input type="checkbox" defaultChecked={done} onChange={this.handleCheck(id)}/>
	                       <span>{name}</span>
	                   </label>
	                   <button className="btn btn-danger" style={{ display: mouse?'block':'none' }}>删除</button>
	               </li>
	           </div>
	       )
	   }
	  }
	  ```
- 对props进行限制
- 在脚手架中并没有涉及到prop-types的相关包和属性，因此，需要首先安装一下,npm add prop-types。
  
  ```
  import React, { Component } from 'react'
  import Item from '../Item'
  import './index.css'
  import PropTypes from 'prop-types'
  
  export default class List extends Component {
  
    //对接收的props进行：类型、必要性的限制
    static PropTypes = {
        todos: PropTypes.array.isRequired,
        updateTodo:PropTypes.array.isRequired
    }
  
    render() {
        const { todos, updateTodo } = this.props
        return (
            <div>
                <ul className="todo-main">
                    {
                        todos.map((todo) => {
                            return <Item key={todo.id} {...todo} updateTodo={updateTodo} />
                        })
                    }
                </ul>
            </div>
        )
    }
  }
  ```
- ##### ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1617764582596-b6ff214f-0e83-4a03-a19c-c864eb86e42d.png)
- 删除一个todo
- 实现一个删除功能，主要针对的就是删除相应的任务列表
	- src/App.jsx
	  
	  ```
	  import React, { Component } from 'react'
	  import Header from './components/Header'
	  import List from './components/List'
	  import Footer from './components/Footer'
	  import './App.css'
	  
	  export default class App extends Component {
	   // 初始化状态
	   state={
	       todos:[
	           {id:'001',name:'吃饭',done:true},
	           {id:'002',name:'睡觉',done:true},
	           {id:'003',name:'打豆豆',done:false},
	       ]
	   }
	  
	   //addTodo用于添加一个todo,接收的参数是todo对象
	   addTodo=(todoObj)=>{
	       // 获取原todos
	       const {todos}=this.state
	       // 追加一个todo
	       const newTodos=[todoObj,...todos]
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   // updateTodo用于更新一个todo
	   updateTodo=(id,done)=>{
	       // 获取状态中的todos
	       const {todos}=this.state
	       // 匹配处理数据
	       const newTodos=todos.map((todoObj)=>{
	           if(todoObj.id===id) return {...todoObj,done}
	           else return todoObj
	       })
	       this.setState({todos:newTodos})
	   }
	  
	   // deleteTodo用于删除一个todo对象
	   deleteTodo=(id)=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 删除指定id的todo对象
	       const newTodos=todos.filter((todoObj)=>{
	           return todoObj.id !== id
	       })
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   // checkAllTodo用于全选
	   checkAllTodo=(done)=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 加工数据
	       const newTodos=todos.map((todoObj)=>{
	           return {...todoObj,done:done}
	       })
	       // 更新数据
	       this.setState({todos:newTodos})
	   }
	  
	   // clearAllDone
	   clearAllDone=()=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 过滤数据
	       const newTodos=todos.filter((todoObj)=>{
	           return !todoObj.done
	       })
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   render() {
	       const {todos}=this.state
	       return (
	           <div>
	               <div className="todo-container">
	                   <div className="todo-wrap">
	                       <Header addTodo={this.addTodo}/>
	                       <List todos={todos} updateTodo={this.updateTodo} deleteTodo={this.deleteTodo}/>
	                       <Footer todos={todos} checkAllTodo={this.checkAllTodo} clearAllDone={this.clearAllDone}/>
	                   </div>
	               </div>
	           </div>
	       )
	   }
	  }
	  ```
	- src/components/List
	  
	  ```
	  import React, { Component } from 'react'
	  import Item from '../Item'
	  import './index.css'
	  import PropTypes from 'prop-types'
	  
	  export default class List extends Component {
	  
	   //对接收的props进行：类型、必要性的限制
	   static propTypes = {
	       todos: PropTypes.array.isRequired,
	       updateTodo:PropTypes.array.isRequired,
	       deleteTodo:PropTypes.func.isRequired
	   }
	  
	   render() {
	       const { todos, updateTodo, deleteTodo } = this.props
	       return (
	           <div>
	               <ul className="todo-main">
	                   {
	                       todos.map((todo) => {
	                           return <Item key={todo.id} {...todo} updateTodo={updateTodo} deleteTodo={deleteTodo} />
	                       })
	                   }
	               </ul>
	           </div>
	       )
	   }
	  }
	  ```
	- src/components/Item
	  
	  ```
	  import React, { Component } from 'react'
	  import './index.css'
	  
	  export default class Item extends Component {
	  
	   // 鼠标移入、移出状态
	   state={mouse:false}
	  
	   // 鼠标移入移出的问题
	   handleMouse=(flag)=>{
	       return ()=>{
	           this.setState({mouse:flag})
	       }
	   }
	  
	   // 勾选、取消勾选某一个todo的回调
	   handleCheck=(id)=>{
	       return (event)=>{
	           this.props.updateTodo(id,event.target.checked)
	       }
	   }
	  
	   // 删除一个todo的回调
	   handleDelete=(id)=>{
	       if(window.confirm('确定删除吗?')){
	           this.props.deleteTodo(id)
	       }
	   }
	  
	   render() {
	       const {id,name,done}=this.props
	       const {mouse}=this.state
	       return (
	           <div>
	               <li style={{backgroundColor:mouse?'#ddd':'white'}} onMouseEnter={this.handleMouse(true)} onMouseLeave={this.handleMouse(false)}>
	                   <label>
	                       <input type="checkbox" checked={done} onChange={this.handleCheck(id)}/>
	                       <span>{name}</span>
	                   </label>
	                   <button onClick={()=>this.handleDelete(id)} className="btn btn-danger" style={{ display: mouse?'block':'none' }}>删除</button>
	               </li>
	           </div>
	       )
	   }
	  }
	  ```
- 实现底部功能
- 主要实现底部栏的几个功能。
	- src/App.jsx
	  
	  ```
	  import React, { Component } from 'react'
	  import Header from './components/Header'
	  import List from './components/List'
	  import Footer from './components/Footer'
	  import './App.css'
	  
	  export default class App extends Component {
	   // 初始化状态
	   state={
	       todos:[
	           {id:'001',name:'吃饭',done:true},
	           {id:'002',name:'睡觉',done:true},
	           {id:'003',name:'打豆豆',done:false},
	       ]
	   }
	  
	   //addTodo用于添加一个todo,接收的参数是todo对象
	   addTodo=(todoObj)=>{
	       // 获取原todos
	       const {todos}=this.state
	       // 追加一个todo
	       const newTodos=[todoObj,...todos]
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   // updateTodo用于更新一个todo
	   updateTodo=(id,done)=>{
	       // 获取状态中的todos
	       const {todos}=this.state
	       // 匹配处理数据
	       const newTodos=todos.map((todoObj)=>{
	           if(todoObj.id===id) return {...todoObj,done}
	           else return todoObj
	       })
	       this.setState({todos:newTodos})
	   }
	  
	   // deleteTodo用于删除一个todo对象
	   deleteTodo=(id)=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 删除指定id的todo对象
	       const newTodos=todos.filter((todoObj)=>{
	           return todoObj.id !== id
	       })
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   // checkAllTodo用于全选
	   checkAllTodo=(done)=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 加工数据
	       const newTodos=todos.map((todoObj)=>{
	           return {...todoObj,done:done}
	       })
	       // 更新数据
	       this.setState({todos:newTodos})
	   }
	  
	   // clearAllDone
	   clearAllDone=()=>{
	       // 获取原来的todos
	       const {todos}=this.state
	       // 过滤数据
	       const newTodos=todos.filter((todoObj)=>{
	           return !todoObj.done
	       })
	       // 更新状态
	       this.setState({todos:newTodos})
	   }
	  
	   render() {
	       const {todos}=this.state
	       return (
	           <div>
	               <div className="todo-container">
	                   <div className="todo-wrap">
	                       <Header addTodo={this.addTodo}/>
	                       <List todos={todos} updateTodo={this.updateTodo} deleteTodo={this.deleteTodo}/>
	                       <Footer todos={todos} checkAllTodo={this.checkAllTodo} clearAllDone={this.clearAllDone}/>
	                   </div>
	               </div>
	           </div>
	       )
	   }
	  }
	  ```
	- src/components/footer
	  
	  ```
	  import React, { Component } from 'react'
	  import './index.css'
	  
	  export default class Footer extends Component {
	  
	   // 全选checkbox的回调
	   handleCheckAll =(event)=>{
	       this.props.checkAllTodo(event.target.checked)
	   }
	  
	   // 清楚已完成任务的回调
	   handleClearAllDone=()=>{
	       this.props.clearAllDone()
	   }
	  
	   render() {
	       const {todos}=this.props
	       // 已完成的个数
	       const doneCount=todos.reduce((pre,todo)=>{
	           return pre+(todo.done?1:0)
	       },0)
	       // 总数
	       const total=todos.length
	       return (
	           <div>
	               <div className="todo-footer">
	                   <label>
	                       <input type="checkbox" onChange={this.handleCheckAll} checked={doneCount===total&&total!==0?true:false} />
	                   </label>
	                   <span>
	                       <span>已完成{doneCount}</span> / 全部{total}
	                           </span>
	                   <button onClick={this.handleClearAllDone} className="btn btn-danger">清除已完成任务</button>
	               </div>
	           </div>
	       )
	   }
	  }
	  ```
- ### 5.1.1.  总结
- todoList案例的相关知识点
	- 1. 拆分组件、实现静态组件，注意：className、style的写法
	- 2. 动态初始化列表，如何确定将数据放在哪个组件的state中？
		- 某个组件使用：放在其自身的state中
		- 某些组件使用：放在他们共同的父组件state中（官方称此为：状态提升）
	- 3. 关于父子之间通信：
		- 【父组件】给【子组件】传递数据：通过props传递
		- 【子组件】给【父组件】传递数据：通过props传递，要求父组件提前给子组件传递一个函数
	- 4. 注意defaultChecked和checked的区别，类似的还有：defaultValue和value。
	- 5. 状态在哪里，操作状态的方法就在那里。
- ## 5.2.  搜索案例
- ##### ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1618022327402-9720154c-69ac-4fd7-9686-8020e75540eb.png)
- 静态组件的设计
- src/App.jsx
  
  ```
  import React, { Component } from 'react'
  import List from './components/List'
  import Search from './components/Search'
  
  export default class App extends Component {
    render() {
        return (
            <div class="container">
                <Search/>
                <List/>
            </div>
        )
    }
  }
  ```
- src/index.js
  
  ```
  // 引入react核心库
  import React from 'react'
  // 引入ReactDOM
  import ReactDOM from 'react-dom'
  // 引入App组件
  import App from './App'
  
  // 渲染App到页面
  ReactDOM.render(<App/>,document.getElementById('root'))
  ```
- src/components/Search
  
  ```
  import React, { Component } from 'react'
  import axios from 'axios'
  
  export default class Search extends Component {
    render() {
        return (
            <div>
                <section class="jumbotron">
                    <h3 class="jumbotron-heading">Search Github Users</h3>
                    <div>
                        <input type="text" placeholder="enter the name you search" />&nbsp;<button>Search</button>
                    </div>
                </section>
            </div>
        )
    }
  }
  ```
- src/components/List
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class List extends Component {
    render() {
        return (
            <div className="row">
                <div className="card">
                    <a rel="noreferrer" href="https://github.com/reactjs" target="_blank">
                        <img alt="Head_portrait" src="https://avatars.githubusercontent.com/u/6412038?v=3" style={{width: '100px'}} />
                    </a>
                    <p className="card-text">reactjs</p>
                </div>
                <div className="card">
                    <a rel="noreferrer" href="https://github.com/reactjs" target="_blank">
                        <img alt="Head_portrait" src="https://avatars.githubusercontent.com/u/6412038?v=3" style={{width: '100px'}} />
                    </a>
                    <p className="card-text">reactjs</p>
                </div>
                <div className="card">
                    <a rel="noreferrer" href="https://github.com/reactjs" target="_blank">
                        <img alt="Head_portrait" src="https://avatars.githubusercontent.com/u/6412038?v=3" style={{width: '100px'}} />
                    </a>
                    <p className="card-text">reactjs</p>
                </div>
                <div className="card">
                    <a rel="noreferrer" href="https://github.com/reactjs" target="_blank">
                        <img alt="Head_portrait" src="https://avatars.githubusercontent.com/u/6412038?v=3" style={{width: '100px'}} />
                    </a>
                    <p className="card-text">reactjs</p>
                </div>
                <div className="card">
                    <a rel="noreferrer" href="https://github.com/reactjs" target="_blank">
                        <img alt="Head_portrait" src="https://avatars.githubusercontent.com/u/6412038?v=3" style={{width: '100px'}} />
                    </a>
                    <p className="card-text">reactjs</p>
                </div>
            </div>
        )
    }
  }
  ```
  
  ```
  .album {
    min-height: 50rem; /* Can be removed; just added for demo purposes */
    padding-top: 3rem;
    padding-bottom: 3rem;
    background-color: #f7f7f7;
  }
  
  .card {
    float: left;
    width: 33.333%;
    padding: .75rem;
    margin-bottom: 2rem;
    border: 1px solid #efefef;
    text-align: center;
  }
  
  .card > img {
    margin-bottom: .75rem;
    border-radius: 100px;
  }
  
  .card-text {
    font-size: 85%;
  }
  ```
- 添加请求模块
  
  ```
  import React, { Component } from 'react'
  import axios from 'axios'
  
  export default class Search extends Component {
    Search=()=>{
        // 获取用户的输入（连续解构赋值+重命名）
        const {keyWordElement:{value:keyWord}}=this
        // 发送网络请求
        axios.get(`http://localhost:5000/search/users?q=${keyWord}`).then(
            response=>{console.log('成功了',response.data)},
            error=>{console.log('失败了',error)}
        )
    }
    render() {
        return (
            <div>
                <section class="jumbotron">
                    <h3 class="jumbotron-heading">Search Github Users</h3>
                    <div>
                        <input type="text" placeholder="enter the name you search" />&nbsp;
                        <button onClick={this.Search}>Search</button>
                    </div>
                </section>
            </div>
        )
    }
  }
  ```
- 完成github搜索案例
- src/App.jsx
  
  ```
  import React, { Component } from 'react'
  import List from './components/List'
  import Search from './components/Search'
  
  export default class App extends Component {
  
    state={ //初始化状态
        users:[], // users初始值的数组
        isFirst:true, //是否为第一次打开页面
        isLoading:false, //标识是否处于加载中
        err:'',// 存储请求相关的错误信息
    }
  
    // 更新App的state
    updateAppState=(stateObj)=>{
        this.setState(stateObj)
    }
  
    render() {
        return (
            <div className="container">
                <Search updateAppState={this.updateAppState}/>
                <List {...this.state}/>
            </div>
        )
    }
  }
  ```
- src/components/List/index.jsx
  
  ```
  import React, { Component } from 'react'
  import './index.css'
  
  export default class List extends Component {
    render() {
        const {users,isFirst,isLoading,err}=this.props
        return (
            <div className="row">
                {
                    isFirst?<h2>欢迎使用，输入关键字，然后点击搜索</h2>:
                    isLoading?<h2>Load......</h2>:
                    err?<h2 style={{color:'red'}}>{err}</h2>:
                    users.map((userObj) => {
                        return (
                            <div key={userObj.id} className="card">
                                <a rel="noreferrer" href={userObj.html_url} target="_blank">
                                    <img alt="Head_portrait" src={userObj.avatar_url} style={{ width: '100px' }} />
                                </a>
                                <p className="card-text">{userObj.login}</p>
                            </div>
                        )
                    })
                }
            </div>
        )
    }
  }
  ```
- src/components/Search/index.jsx
  
  ```
  import React, { Component } from 'react'
  import axios from 'axios'
  
  export default class Search extends Component {
    search=()=>{
        // 获取用户的输入（连续解构赋值+重命名）
        const {keyWordElement:{value:keyWord}}=this
        console.log(keyWord)
        // 发送请求前通知App更新状态
        this.props.updateAppState({
            isFirst:false,
            isLoading:true,
        })
        // 发送网络请求
        axios.get(`/search/users?q=${keyWord}`).then(
            response=>{
                this.props.updateAppState({
                    isLoading:false,
                    users:response.data.items,
                })
            },
            error=>{this.props.updateAppState({
                isLoading:false,
                err:error.message,
            })}
        )
    }
    render() {
        return (
            <div>
                <section className="jumbotron">
                    <h3 className="jumbotron-heading">Search Github Users</h3>
                    <div>
                        <input ref={c=>this.keyWordElement=c} type="text" placeholder="enter the name you search" />&nbsp;
                        <button onClick={this.search}>Search</button>
                    </div>
                </section>
            </div>
        )
    }
  }
  ```
- 这样就算完成了一个github的搜索案例。这里面综合运用了很多的知识。让我也对ref、state、prop这些属性有了更加深刻的了解和认识。
- # 6.  工程实践
- ## 6.1.  Springboot整合React项目
- 首先就是在react项目之下进行打包。使用npm run build命令。
	- 打包完成之后，会生成一个build文件夹
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1637415599210-c8a8b607-8d82-461e-ba11-644d0c0ae2b3.png)
- 然后将index.html放入/src/resources/templates/文件夹下，将build文件内的其余文件，放入/src/resources/static/文件夹下。
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1637415722203-6bf413fd-8b6f-4af6-b27f-eb9bb1a97ee8.png)
- 在项目的pom文件中，需要引入themeleaf这一模块，这个模块是springboot中指定的mvc的模块。
  
  ```
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
  ```
- 然后设置application配置文件
  
  ```
  # 静态资源配置
  spring.web.resources.static-locations=classpath:/static/
  #视图层控制
  spring.mvc.view.prefix=classpath:/templates/
  spring.mvc.view.suffix=.html
  ```
- 最后在Controller中设置相关的访问方式
  
  ```
  package nju.software.cloud.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.RequestMapping;
  
  @Controller
  @RequestMapping(value = "/")
  public class webController {
    @RequestMapping(value = {"/*","/*/*"})
    public String show(){
        return "index";
    }
  }
  ```
	- "/*","/*/*" 表示匹配各种路径。
- # 参考资料
- React文档：[https://react.docschina.org/learn](https://react.docschina.org/learn)
- React Router：[https://reactrouter.com/](https://reactrouter.com/)
- Ant Design：[https://ant.design/components/overview-cn/](https://ant.design/components/overview-cn/)
- Ant Design X：[https://x.ant.design/index-cn/](https://x.ant.design/index-cn/)
- Tailwind CSS：[https://www.tailwindcss.cn/docs/installation](https://www.tailwindcss.cn/docs/installation)
-