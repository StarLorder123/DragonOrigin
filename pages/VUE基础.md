# Vue是什么？
- 谈及Vue，就不得不谈一下React。这两个东西出现的频率几乎是一样的。提及了Vue，大部分情况下就会提及React。因此在正式开始学习Vue的相关的东西之前，我先了解一下这两个东西。
- **React**
	- 除非你最近一直不关注前端的发展，不然你肯定听说过：Facebook创建的JavaScript UI框架——React。它支撑着包括Instagram在内的大多数Facebook网站。React与当时流行的jQuery,Backbone.js和Angular 1等框架不同，它的诞生改变了JavaScript的世界。其中最大的变化是React推广了Virtual DOM（我们稍后探究）并创造了新的语法——JSX，JSX允许开发者在JavaScript中书写HTML。
- **Vue**
	- Vue致力解决的问题与React一致，但却提供了另外一套解决方案。Vue使用模板系统而不是JSX，使其对现有应用的升级更加容易。这是因为模板用的就是普通的HTML，通过Vue来整合现有的系统是比较容易的，不需要整体重构。
- 两者都是JavaScript的UI框架，专注于创造前端的富应用。React和VUE只有框架的骨架，其他的功能：例如路由、状态管理等部分都是框架分离的组件。
- ## 第一个Vue应用程序
- 示例代码
  
  ```
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>VUE学习</title>
                <!-- 开发环境版本，包含了有帮助的命令行警告 -->
                <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        <div id="app">
            {{ message }}
            <br>
            <span>
                {{ student.name }}
            </span>
            <br>
            <span>
                {{ list[0] }}
            </span>
        </div>
        <script>
            var app = new Vue({
                el:'#app',
                data:{
                    message:'Hello!',
                    student:{
                        name:"ljh",
                        age:18
                    },
                    list:["计算机学院","理学院","材料学院"]
                }
            })
        </script>
    </body>
  </html>
  ```
- 以上就是第一个示例的代码。这里我并没有下载Vue.js。直接通过script的标签进行了导入工作。
- el属性内放置的就是关于查找DOM中标签的格式。可以是#id（通过ID查找），.class（通过类名查找）,tag（直接通过标签查找）
- data则是放置了大部分的数据来源。数据来源也包含了普通的、对象以及数组。
- ## 本地应用——v-text
- 示例：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>VUE学习</title>
        <!-- 开发环境版本，包含了有帮助的命令行警告 -->
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        <div id="test">
            <h2 v-text="message">test</h2>
            <h2 v-text="message+'!'">test</h2>
            <h2>{{ message +'!'}}test</h2>
        </div>
        <script>
            var app=new Vue({
                el:"#test",
                data:{
                    message:"ljh",
                    info:"vue学习"
                }
            })
        </script>
        
    </body>
  </html>
  ```
- 效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608368384407-73ba460d-d67b-4e04-9996-79e71d9d09d6.png)
- ## 本地应用——v-html
- 示例：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>VUE学习</title>
        <!-- 开发环境版本，包含了有帮助的命令行警告 -->
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        <div id="test">
            <h2 v-html="content"></h2>
            <h2 v-text="content"></h2>
        </div>
        <script>
            var app=new Vue({
                el:"#test",
                data:{
                    message:"ljh",
                    info:"vue学习",
                    content:"<a href='https://www.baidu.com'>百度</a>"
                }
            })
        </script>
    </body>
  </html>
  ```
- 效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608368534154-9242f053-3264-42cb-8ed4-fd62209d179d.png)
- ## 本地应用——v-on
- v-on这个标签是用于给元素添加触发动作的。
- 示例：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>VUE学习</title>
        <!-- 开发环境版本，包含了有帮助的命令行警告 -->
        <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    </head>
    <body>
        <div id="test">
            <input type="button" value="v-on指令" v-on:click="doIt">
            <input type="button" value="v-on简写" @click="doIt">
            <input type="button" value="双击事件" @dblclick="doIt">
            <h2 @click="change">
                {{ info }}
            </h2>
            <h2 @click="change">
                {{ message }}
            </h2>
        </div>
        <script>
            var app=new Vue({
                el:"#test",
                data:{
                    message:"ljh",
                    info:"vue学习",
                    content:"<a href='https://www.baidu.com'>百度</a>"
                },
                methods:{
                    doIt:function(){
                        alert("DoIt")
                    },
                    change:function(){
                        this.info += this.info
                    }
                }
            })
        </script>
        
    </body>
  </html>
  ```
- 效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608368667766-7778de3e-6b43-48b5-8c0e-fc6ef6bb41c9.png)
- 上述代码中的this用法就是对于这个页面中的VUE对象中data数据的数据进行处理。可以看到，第二个h2标签触发事件是更改info，显示数据是message，点击这个标签的话，最后的数据更改的其实还是info。
- # VUE实践
- ## 实践1
- 示例代码：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
      <!-- 计数器功能区 -->
      <div class="container">
        <div class="row justify-content-md-center">
          <div class="col col-lg-2">
            <button @click="sub">-</button>
          </div>
          <div class="col-md-auto">
            {{num}}
          </div>
          <div class="col col-lg-2">
            <button @click="add">+</button>
          </div>
        </div>
      </div>
    </div>
    <script>
      var app = new Vue({
        el: "#app",
        data: {
          num: 0
        },
        methods: {
          sub: function () {
            if (this.num > 0) {
              this.num--;
            } else {
              alert("已经最小啦！")
            }
          },
          add: function () {
            if (this.num < 10) {
              this.num++;
            } else {
              alert("已经最大啦！")
            }
          }
        }
      })
    </script>
  </body>
  
  </html>
  ```
- 为了避免自己撰写相关的css脚本，我直接使用script标签将bootstrap4引入到相应的代码中来。
- 在这个示例代码中，el是挂载点，data是内部存放的数据。v-on指令的作用是绑定事件，简写为@。
- 方法中通过this,关键字获取data中的数据。
- v-text指令的作用是：设置元素的文本值，简写为{{}}。
- 示例代码：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
        integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
        integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
        crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
        integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
        crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
        integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
        crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
        <input type="button" value="显示切换" @click="toggleIsShow"></button>
        <p v-if="isShow">Vue学习 v-if修饰</p>
        <p v-show="isShow">Vue学习 v-show修饰</p>
    </div>
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                isShow:false
            },
            methods: {
                toggleIsShow:function(){
                    this.isShow = !this.isShow
                }
            }
        })
    </script>
  </body>
  
  </html>
  ```
- **v-show**
	- v-show指令的作用是根据真假切换元素的显示状态
	- 原理是修改元素的display，实现显示隐藏
	- 指令后面的内容，最终都会解析为布尔值
	- 值为true元素显示，值为false元素隐藏
- **v-if**
	- v-if指令的作用是根据表达式的真假切换元素的显示状态
	- 本质是通过操纵dom元素来切换显示状态
	- 表达式的值为true，元素存在于dom树中，为false，从dom树中移除。
	- 频繁切换v-show，反之使用if，前者的切换消耗更小。
- 具体的区别效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608441440799-9be014e6-44b6-4b70-b2bc-105878b3e9eb.png)
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608441445199-c6ab2856-0806-4bdc-8010-48535d085c7c.png)
- ## v-bind的使用
- v-bind指令的作用是：为元素绑定属性
- 完整写法是v-bind:属性名
- 简写的话可以直接省略v-bind，只保留：属性名。
- ### 代码实践（图片切换）
- 示例代码
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
        integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
        integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
        crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
        integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
        crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
        integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
        crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
        <div id="carouselExampleControls" class="carousel slide" data-ride="carousel">
            <div class="carousel-inner">
              <img v-bind:src="imgArr[index]" :alt="alt" class="img-thumbnail"/>
            </div>
            <a class="carousel-control-prev" href="#carouselExampleControls" role="button" data-slide="prev" @click="prev" v-show="index!=0">
              <span class="carousel-control-prev-icon" aria-hidden="true"></span>
              <span class="sr-only">Previous</span>
            </a>
            <a class="carousel-control-next" href="#carouselExampleControls" role="button" data-slide="next" @click="next" v-show="index<imgArr.length-1">
              <span class="carousel-control-next-icon" aria-hidden="true"></span>
              <span class="sr-only">Next</span>
            </a>
          </div>
    </div>
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                imgArr:[
                    "../img/00 (1).JPG",
                    "../img/00 (2).JPG",
                    "../img/00 (3).JPG",
                    "../img/00 (4).JPG",
                    "../img/00 (5).JPG",
                    "../img/00 (6).JPG",
                    "../img/00 (7).JPG",
                    "../img/00 (8).JPG",
                    "../img/00 (9).JPG",
                    "../img/00 (10).JPG",
                    "../img/00 (11).JPG"
                ],
                index:0,
                alt:"这是图片"
            },
            methods: {
                prev:function(){
                    this.index--;
                },
                next:function(){
                    this.index++;
                }
            }
        })
    </script>
  </body>
  </html>
  ```
- 上述代码使用了v-show,v-if,v-bind这三种标签的使用方式。
- 这里面的标签主要使用的是bootStrap中的一些固定或者特定组件实现的。
- ## 实践2
- ### v-for的使用
- 示例代码：
  
  ```
  <li class="list-group-item" v-for="(item,index) in message">
  <div class="row">
    <div class="col">
      {{item.uname}}
    </div>
    <div class="col">
      {{item.time}}
    </div>
    <div class="col">
      <input type="button" value="del" @click="remove">
    </div>
  </div>
  </li>
  ```
- v-for指令的作用是：根据数据生成列表结构
- 数组经常和v-for结合使用
- 语法是（item,index) in 数据。数据的部分一般为数组，可以是对象数组，也可以是常用对象数组。
- ### v-on的使用拓展
- 示例代码：
  
  ```
  <div class="input-group" style="width: 30%;">
  <div class="input-group-prepend">
    <div class="input-group-text">
      <input type="radio" aria-label="Radio button for following text input">
    </div>
  </div>
  <input type="text" class="form-control" aria-label="Text input with radio button" v-model="username" @keyup.enter="add">
  </div>
  ```
  
  ```
  <ul class="list-group" v-if="message.length!=0">
  <li class="list-group-item" v-for="(item,index) in message">
    <div class="row">
      <div class="col">
        {{item.uname}}
      </div>
      <div class="col">
        {{item.time}}
      </div>
      <div class="col">
        <input type="button" value="del" @click="remove(index)">
      </div>
    </div>
  </li>
  </ul>
  ```
- 事件绑定的方法携程函数调用的形式，可以传入自定义参数。
- 定义方法的时候，需要定义形参来接收传入的实参。
- 事件的后面跟上 .修饰符可以对事件进行限制和修饰。例如，不想让某些内容一输入就出现触发效果，可以通过.enter，即按回车之后才会出现事件。
- ### v-model的使用
- 示例代码：
  
  ```
  <div class="input-group" style="width: 30%;">
  <div class="input-group-prepend">
    <div class="input-group-text">
      <input type="radio" aria-label="Radio button for following text input">
    </div>
  </div>
  <input type="text" class="form-control" aria-label="Text input with radio button" v-model="username" @keyup.enter="add">
  </div>
  <div class="input-group mb-3" style="width: 30%;">
  <div class="input-group-prepend">
    <div class="input-group-text">
      <input type="checkbox" aria-label="Checkbox for following text input">
    </div>
  </div>
  <input type="text" class="form-control" aria-label="Text input with checkbox" v-model="time" >
  </div>
  <script>
  var app = new Vue({
    el: "#app",
    data: {
      message: [
        { uname: "吃饭", time: "12.30" },
        { uname: "午觉", time: "13.30" },
        { uname: "打豆豆", time: "14.30" }
      ],
      username: "",
      time: ""
    },
    methods: {
      add: function () {
        var temp = {
          uname: this.username,
          time: this.time
        }
        this.message.push(temp)
      },
      remove:function(index){
        //这里表示从index下标开始，删除一个
        this.message.splice(index,1);
      },
      clear:function(){
        this.message=[];
      }
    }
  })
  </script>
  ```
- v-model指令的作用就是便捷地设置和获取表单元素的值
- 绑定的数据会和表单元素值相关联，也就是会成为双向绑定。
- 绑定的数据和表单元素之间会产生数据的联动。
- #### 实践项目
- 代码
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
      <div class="input-group" style="width: 30%;">
        <div class="input-group-prepend">
          <div class="input-group-text">
            <input type="radio" aria-label="Radio button for following text input">
          </div>
        </div>
        <input type="text" class="form-control" aria-label="Text input with radio button" v-model="username" @keyup.enter="add">
      </div>
      <div class="input-group mb-3" style="width: 30%;">
        <div class="input-group-prepend">
          <div class="input-group-text">
            <input type="checkbox" aria-label="Checkbox for following text input">
          </div>
        </div>
        <input type="text" class="form-control" aria-label="Text input with checkbox" v-model="time" >
      </div>
      <div>
        <input type="button" value="添加" @click="add">
      </div>
      <div style="width: 30%;">
        <ul class="list-group" v-if="message.length!=0">
          <li class="list-group-item" v-for="(item,index) in message">
            <div class="row">
              <div class="col">
                {{item.uname}}
              </div>
              <div class="col">
                {{item.time}}
              </div>
              <div class="col">
                <input type="button" value="del" @click="remove(index)">
              </div>
            </div>
          </li>
        </ul>
      </div>
      <div style="width: 30%;">
        <div class="row">
          <div class="col">
            {{message.length}} items left
          </div>
          <div class="col" v-show="message.length!=0">
                    <button @click="clear">clear</button>
                </div>
            </div>
        </div>
    </div>
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: [
                    { uname: "吃饭", time: "12.30" },
                    { uname: "午觉", time: "13.30" },
                    { uname: "打豆豆", time: "14.30" }
                ],
                username: "",
                time: ""
            },
            methods: {
                add: function () {
                    var temp = {
                        uname: this.username,
                        time: this.time
                    }
                    this.message.push(temp)
                },
                remove:function(index){
                    //这里表示从index下标开始，删除一个
                    this.message.splice(index,1);
                },
                clear:function(){
                    this.message=[];
                }
            }
        })
    </script>
  </body>
  </html>
  ```
- 效果图：
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608521198642-3390b99a-e985-4a65-ae6f-d986628932da.png)
- 功能说明：
	- 增加：在第一两个框内输入相应的字符串和值，就能添加到下面的列表中。
	- 删除：点击下面的一行内的del键，就会删除相应那一行的代码和内容
	- 清除：点击clear键，就会清除掉所有的内容。
	- 隐藏：当列表内的数组内容=0的时候，清除按钮就会消失。整个列表都会在h5中消失。
	- 计数：计算列表内的数据数量。
- 小结：
	- 以上这个实践项目使用了v-on、v-show等诸多功能，综合在一起使用。
- ### LessCss动态样式语言
- LESSCSS是一种动态样式语言，属于CSS预处理语言的一种，它使用类似CSS的语法，为CSS的赋予了动态语言的特性，如变量、继承、运算、函数等，更方便CSS的编写和维护。
- LESSCSS可以在多种语言、环境中使用，包括浏览器端、桌面客户端、服务端。
- LESSCSS的使用是很容易的，首先，使用你最常使用的代码编辑器，按LESSCSS的语法规则写好.less文件，接下来，使用编译工具它编译成.css，最后再引入页面即可。
- LESSCSS也可以不经编译，直接在浏览器端使用。
- # Axios
- ## axios介绍
- axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端。简单的理解就是ajax的封装
- 具有以下特征
	- 从浏览器中创建 XMLHttpRequest
	- b.从 node.js 发出 http 请求
	- c.支持 Promise API
	- e.拦截请求和响应
	- f.转换请求和响应数据
	- g.取消请求
	- h.自动转换JSON数据
	- i.客户端支持防止 CSRF/XSRF
- 什么是Promise？
	- Promise 是异步编程的一种解决方案，其实是一个构造函数，自己身上有all、reject、resolve这几个方法，原型上有then、catch等方法。
	- 对象有以下两个特点：
		- 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
		- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。
- ## 关于axios的使用
- get请求
	- 示例代码：
	  
	  ```
	  // 为给定 ID 的 user 创建请求
	  axios.get('/user?ID=12345')
	  .then(function (response) {
	   console.log(response);
	  })
	  .catch(function (error) {
	   console.log(error);
	  });
	  
	  // 上面的请求也可以这样做
	  axios.get('/user', {
	  params: {
	   ID: 12345
	  }
	  })
	  .then(function (response) {
	   console.log(response);
	  })
	  .catch(function (error) {
	   console.log(error);
	  });
	  ```
	- 可以看到，get的请求数据可以直接附带在url中传出，也可以另外放置一个params的数据，让程序帮助完成附带工作。
- post请求
	- 示例代码
	  
	  ```
	  axios.post('/user', {
	  firstName: 'Fred',
	  lastName: 'Flintstone'
	  })
	  .then(function (response) {
	   console.log(response);
	  })
	  .catch(function (error) {
	   console.log(error);
	  });
	  ```
- 执行多个并发请求
	- 示例代码
	  
	  ```
	  function getUserAccount() {
	  return axios.get('/user/12345');
	  }
	  
	  function getUserPermissions() {
	  return axios.get('/user/12345/permissions');
	  }
	  
	  axios.all([getUserAccount(), getUserPermissions()])
	  .then(axios.spread(function (acct, perms) {
	   // 两个请求现在都执行完成
	  }));
	  ```
- ## Vue+Axios实例——天气查询
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  </head>
  
  <body>
    <div style="margin-top: 50px;" id="app">
      <div class="container">
        <div class="row">
          <div class="col">
            <div class="form-group mx-sm-3 mb-2">
              <label for="inputPassword2" class="sr-only">Password</label>
              <input type="text" class="form-control" id="inputPassword2" v-model="city" @keyup.enter="searchchWeather">
            </div>
          </div>
          <div class="col">
            <button type="submit" class="btn btn-primary mb-2">搜索</button>
          </div>
        </div>
        <div>
          <a @click="changCity('北京')">北京</a>
          <a @click="changCity('上海')">上海</a>
        </div>
      </div>
      <div class="container">
        <ul class="list-group list-group-horizontal">
          <li class="list-group-item" v-for="item in weatherList">
            <div class="info_type">
              <span>{{item.type}}</span>
            </div>
            <div class="info_temp">
              <b>{{item.low}}</b>
              ~
              <b>{{item.high}}</b>
            </div>
            <div>
              <span>{{item.date}}</span>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <script>
      var app = new Vue({
        el: "#app",
        data: {
          city: '',
          weatherList:[]
        },
        methods: {
          searchchWeather:function(){
            //console.log(this.city)
            console.log("回调之前："+this)
            var that=this;
              axios.get('http://wthrcdn.etouch.cn/weather_mini?city='+this.city)
                    .then(function(response){
                        console.log("回调之后："+this)
                        //console.log(response.data.data.forecast);
                        that.weatherList = response.data.data.forecast
                    }).catch(function(err){
                        console.log(err)
                    })
                },
                changCity:function(city){
                    this.city=city;
                    this.searchchWeather();
                }
            }
        })
    </script>
  </body>
  
  </html>
  ```
- 效果图
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608606364433-1f4f6096-2260-4b01-8111-ddc62c860eda.png)
- 应用的逻辑代码可以使用代码和页面分离的形式，可以使用单独的js文件编写。
- axios回调的过程当中this的指针变了，需要提前存一份。
- 至于回调的过程当中，通过代码的验证。原先的this是一个Object对象，在回调之后的this对象指向的就是Window对象了。
- 自定义参数可以让代码的复用性更高。
- methods中定义的方法内部，可以通过this关键字使用其他的方法。
- # 生命周期与钩子函数
  
  ![](https://cdn.nlark.com/yuque/0/2020/png/2713067/1608691416691-165eea78-381c-4fd2-afd6-940f98d06dff.png)
	- 对应的钩子函数是beforeMount，在这一阶段，我们虽然依然得不到具体的DOM元素，但vue挂载的根节点已经创建，下面vue对DOM的操作将围绕这个根元素继续进行；beforeMount这个阶段是过渡性的，一般一个项目只能用到一两次。
	- mounted(载入后)
	- 对应的钩子函数是mounted。mounted是平时我们使用最多的函数了，一般我们的异步请求都写在这里。在这个阶段，数据和DOM都已被渲染出来。
	- eforeUpdate(更新前)
	- 对应的钩子函数是beforeUpdate。在这一阶段，vue遵循数据驱动DOM的原则。beforeUpdate函数在数据更新后虽然没立即更新数据，但是DOM中的数据会改变，这是Vue双向数据绑定的作用。
	- updated（更新后)
	- 对应的钩子函数是updated。在这一阶段DOM会和更改过的内容同步。
	- beforeDestroy(销毁前)
	- 对应的钩子函数是beforeDestroy。在上一阶段Vue已经成功的通过数据驱动DOM更新，当我们不再需要vue操纵DOM时，就要销毁Vue,也就是清除vue实例与DOM的关联，调用destroy方法可以销毁当前组件。在销毁前会触发beforeDestroy钩子函数。
	- destroyed（销毁后)
	- 对应的钩子函数是destroyed。在销毁后，会触发destroyed钩子函数。
	- 还有在组件之间切换的时候都会请求一些请求过的数据，每次请求都会导致重复渲染影响性能。这些数据可以存到缓存。此时使用keep-alive将组建包裹起来。但这样以上八种生命周期钩子将失效。取而代之的时activate和deactivated。
	- activate：是在被包裹组建被激活的状态下使用的生命周期钩子
	- deactivated：在被包裹组件停止使用时调用
- 图中的红色部分就是在Vue生命周期中的八大钩子函数，这个函数就是用于更好地控制vue生命周期中的逻辑过程。
- 各周期适合的场景：
	- beforecreate : 可以在这加个loading事件，在加载实例时触发
	- created : 初始化完成时的事件写在这里，如在这结束loading事件，异步请求也适宜在这里调用
	- mounted : 挂载元素，获取到DOM节点
	- updated : 如果对数据统一处理，在这里写上相应函数
	- beforeDestroy : 可以做一个确认停止事件的确认框
- created和mounted的区别
	- created:在模板渲染成html前调用，即通常初始化某些属性值，然后再渲染成视图。
	- mounted:在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作。
- 示例代码：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
      {{msg}}
    </div>
    <script>
      var app= new Vue({
        el:"#app",
        data:{
          msg:"H"
        },
        created(){
          this.msg="Hello!";
          console.log(this)
        }
      })
    </script>
  </body>
  </html>
  ```
- # computed与watch
- ## 计算属性的使用（Computed）
- 示例代码
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
      <h2>
        你的生日是：
        {{ birth }}
      </h2>
    </div>
    <script type="text/javascript">
      var app = new Vue({
        el:"#app",
        data:{
          birthday:1429032123201
        },
        computed:{
          birth:function(){
            const date = new Date(this.birthday);
            return date.getFullYear()+"-"+(date.getMonth()+1)+"-"+date.getDay();
          }
        }
  
      })
    </script>
  </body>
  
  </html>
  ```
- 模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护，所以，对于复杂逻辑，vue 提倡使用计算属性。
- computed是计算属性，依赖其他的属性值，并且computed的属性值有缓存属性，当属性值变化的时候，下一次获取computed属性的时候才会重新计算computed的值。
- 当我们需要依赖于其他数据进行计算的时候，为了避免每次都需要重新取值重新计算，通常都使用computed。
- 事实上，methods完全可以代替computed，但是为了好维护，逻辑性更加的明晰，单纯的出现页面内的复杂计算时，不涉及异步操作或请求操作时，都是用计算属性。
- ## 监听属性的使用（Watch）
- 示例代码
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  
  <head>
    <meta charset="UTF-8">
    <title>VUE学习</title>
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <!-- 导入BootStrap4的CSS模块 -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/css/bootstrap.min.css"
      integrity="sha384-9aIt2nRpC12Uk9gS9baDl411NQApFmC26EwAOH8WgZl5MYYxFfc+NcPb1dKGj7Sk" crossorigin="anonymous">
    <!-- 导入BootStrap4的JavaScript模块 -->
    <script src="https://cdn.jsdelivr.net/npm/jquery@3.5.1/dist/jquery.slim.min.js"
      integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
      integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
      crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.0/dist/js/bootstrap.min.js"
      integrity="sha384-OgVRvuATP1z7JjHLkuOU7Xw704+h835Lr+6QL9UvYjZE3Ipu6Tp75j7Bh/kR0JKI"
      crossorigin="anonymous"></script>
  </head>
  
  <body>
    <div id="app">
      <input type="text" v-model="message">
      <hr><br>
      <input type="text" v-model="person.name">
      <input type="text" v-model="person.age">
      <button @click="person.age++">+</button>
      <br>
      <h2>
        姓名：{{person.name}},年龄：{{person.age}}
      </h2>
    </div>
    <script type="text/javascript">
      var app = new Vue({
        el:"#app",
        data:{
          message:"Vue学习",
          person:{
            name:"ljh",
            age:21
          }
        },
        watch:{
          message(newValue,oldValue){
            console.log("新值："+newValue+";旧值："+oldValue);
          },
          person:{
            //开启深度监控；监控对象中的属性值变化
            deep:true,
            //可以获取到最新的对象属性数据
            handler(obj){
              console.log("name="+obj.name+";age="+obj.age);
            }
          }
        }
      })
    </script>
  </body>
  
  </html>
  ```
- Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：**侦听属性**。更多的是一种观察的作用，用于监听某些数据的回调。每当所监听的数据发生变化时才能执行回调处理后续操作。
- 当需要在数据变化时执行异步或者开销较大的操作时，watch方法是最有用的。
  
  ```
  watch:{
  message(newValue,oldValue){
    console.log("新值："+newValue+";旧值："+oldValue);
  },
  person:{
    //开启深度监控；监控对象中的属性值变化
    deep:true,
      //可以获取到最新的对象属性数据
      handler(obj){
      console.log("name="+obj.name+";age="+obj.age);
    }
  }
  }
  ```
- 对于以上的代码进行细致的分析：
	- 第一个可以发现了，对于message这一个属性进行了监听。从data中可以发现，这个message是一个基本属性的属性值，因此，只需要进行简单的监听即可。
	- 当我们像第二个一样，需要对一个person这样的对象内部的某个属性进行监听的时候，这样浅层次的监听就会显得无能为力了，因此引入了深度监听这样的一个概念。
	- 在deep属性中改为true，就开启了深度监听。此时就会在这个函数内，优于一切函数，首先执行handler这个函数。