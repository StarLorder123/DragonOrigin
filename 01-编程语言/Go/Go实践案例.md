# 第一个实践
- 在大型的项目当中，把所有的内容全部放入main.go内，显然是不可能的。这时候就需要将结构拆分，按照dao-service-controller这样划分。
- 示例
  
  ```
  package controller
  
  import (
  "github.com/gin-gonic/gin"
  "net/http"
  "entity"
  )
  
  func Index(context *gin.Context) {
  context.HTML(http.StatusOK,"index.html",nil)
  }
  
  func Create(context *gin.Context) {
  //前端页面发送请求，添加待办。
  // 1. 获取请求中的数据
  var todo entity.Todo
  context.BindJSON(&todo)
  // 2，存储到数据库中
  // 3. 返回内容
  if err:=entity.CreateATodo(&todo);err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  }else{
  context.JSON(http.StatusOK,todo)
  }
  }
  
  func FindAll(context *gin.Context) {
  if todoList,err:=entity.FindAll();err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  }else{
  context.JSON(http.StatusOK,todoList)
  }
  }
  
  func Update(context *gin.Context) {
  id,ok:=context.Params.Get("id")
  if !ok {
  context.JSON(http.StatusOK,gin.H{"err":"未接收到id参数"})
  return
  }
  todo,err:=entity.FinaATodo(id)
  if err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  return
  }
  
  context.BindJSON(&todo)
  if err=entity.UpdateATodo(todo);err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  }else{
  context.JSON(http.StatusOK,todo)
  }
  }
  
  func Delete(context *gin.Context) {
  id,ok:=context.Params.Get("id")
  if !ok {
  context.JSON(http.StatusOK,gin.H{"err":"未接收到id参数"})
  }
  if err:=entity.DeleteATodo(id);err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  }else{
  context.JSON(http.StatusOK,gin.H{"err":"已删除"})
  }
  }
  
  func FindATodo(context *gin.Context)  {
  id,ok:=context.Params.Get("id")
  if !ok {
  context.JSON(http.StatusOK,gin.H{"err":"未接收到id参数"})
  return
  }
  if todo,err:=entity.FinaATodo(id);err!=nil{
  context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  }else{
  context.JSON(http.StatusOK,todo)
  }
  }
  ```
  
  ```
  package dao
  
  import "github.com/jinzhu/gorm"
  
  var (
  DB *gorm.DB
  )
  
  func MySQLinit ()(err error){
  DB, err := gorm.Open("mysql", "root:Just123456@(localhost:3306)/youth?charset=utf8mb4&parseTime=True&loc=Local")
  if err != nil{
  return
  }
  return DB.DB().Ping()
  }
  
  func Close(){
  DB.Close()
  }
  ```
  
  ```
  package entity
  
  import (
  "dao"
  )
  
  type Todo struct {
  ID int `json:"id"`
  Title string `json:"title"`
  Status string `json:"status"`
  }
  
  func CreateATodo(todo *Todo)(err error){
  if err:=dao.DB.Create(&todo).Error;err!=nil{
  return err
  }
  return
  }
  
  func FindAll()(todoList []*Todo,err error){
  if err:=dao.DB.Find(&todoList).Error;err!=nil{
  return nil,err
  }
  return
  }
  
  func FinaATodo(id string) (todo *Todo, err error) {
  todo = new(Todo)
  if err:=dao.DB.Where("id=?",id).First(todo).Error;err!=nil{
  return nil,err
  }
  return
  }
  
  func UpdateATodo(todo *Todo)(err error){
  if err:=dao.DB.Save(&todo).Error;err!=nil{
  return err
  }
  return
  }
  
  func DeleteATodo(id string)(err error){
  err =dao.DB.Where("id=?",id).Delete(&Todo{}).Error
  return err
  }
  ```
  
  ```
  package main
  
  import (
  "controller"
  "dao"
  "entity"
  "github.com/gin-gonic/gin"
  )
  
  func main(){
  // 连接数据库
  err:=dao.MySQLinit()
  if err!=nil{
  panic(err)
  }
  // 延迟关闭
  defer dao.Close()
  
  // 绑定模型
  dao.DB.AutoMigrate(&entity.Todo{})
  
  r := gin.Default()
  
  // 静态文件去哪里拿
  r.Static("/static","static")
  // 加载模板文件
  r.LoadHTMLGlob("src/GoLearn/src/template/*")
  
  r.GET("/", controller.Index)
  
  // v1路由的相关内容
  v1Group:=r.Group("v1")
  {
  // 待办事项
  // 添加
  v1Group.POST("/todo", controller.Create)
  // 查询
  // 查询单个
  v1Group.GET("/todo", controller.FindAll)
  // 查询全部
  v1Group.GET("/todo/:id", controller.FindATodo)
  // 修改
  v1Group.PUT("/todo/:id", controller.Update)
  // 删除
  v1Group.DELETE("/todo/:id", controller.Delete)
  }
  r.Run(":8080")
  }
  ```
- # 第二个实践
- 添加了数据库的连接以及对于数据库的增删改查等相关的内容
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  "github.com/jinzhu/gorm"
  "net/http"
  )
  
  var (
  DB *gorm.DB
  )
  type TODO struct {
  ID int `json:"id"`
  Title string `json:"title"`
  Status string `json:"status"`
  }
  func MySQLinit ()(err error){
  DB, err := gorm.Open("mysql", "root:Just123456@(localhost:3306)/youth?charset=utf8mb4&parseTime=True&loc=Local")
  if err != nil{
  return
  }
  return DB.DB().Ping()
  }
  func main(){
  // 连接数据库
  err:=MySQLinit()
  if err!=nil{
  panic(err)
  }
  // 延迟关闭
  defer DB.Close()
  
  // 绑定模型
  DB.AutoMigrate(&TODO{})
  
  r := gin.Default()
  
  // 静态文件去哪里拿
  r.Static("/static","static")
  // 加载模板文件
  r.LoadHTMLGlob("src/GoLearn/src/template/*")
  
  r.GET("/", func(context *gin.Context) {
  context.HTML(http.StatusOK,"index.html",nil)
  })
  
  // v1路由的相关内容
  v1Group:=r.Group("v1")
  {
  // 待办事项
  // 添加
  v1Group.POST("/todo", func(context *gin.Context) {
  	//前端页面发送请求，添加待办。
  	// 1. 获取请求中的数据
  	var todo TODO
  	context.BindJSON(&todo)
  	// 2，存储到数据库中
  	// 3. 返回内容
  	if err:=DB.Create(&todo).Error;err!=nil{
  		context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  	}else{
  		context.JSON(http.StatusOK,todo)
  	}
  })
  // 查询
  // 查询单个
  v1Group.GET("/todo/:id", func(context *gin.Context) {
  	var todoList []TODO
  	if err:=DB.Find(&todoList).Error;err!=nil{
  		context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  	}else{
  		context.JSON(http.StatusOK,todoList)
  	}
  })
  // 查询全部
  v1Group.GET("/todo", func(context *gin.Context) {
  
  })
  // 修改
  v1Group.PUT("/todo/:id", func(context *gin.Context) {
  	id,ok:=context.Params.Get("id")
  	if !ok {
  		context.JSON(http.StatusOK,gin.H{"err":"未接收到id参数"})
  	}
  
  	var todo TODO
  	if err:=DB.Where("id=?",id).First(&todo).Error;err!=nil{
  		context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  		return
  	}
  	context.BindJSON(&todo)
  	if err=DB.Save(&todo).Error;err!=nil{
  		context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  	}else{
  		context.JSON(http.StatusOK,todo)
  	}
  
  })
  // 删除
  v1Group.DELETE("/todo/:id", func(context *gin.Context) {
  	id,ok:=context.Params.Get("id")
  	if !ok {
  		context.JSON(http.StatusOK,gin.H{"err":"未接收到id参数"})
  	}
  	if err:=DB.Where("id=?",id).Delete(&TODO{}).Error;err!=nil{
  		context.JSON(http.StatusOK,gin.H{"err":err.Error()})
  	}else{
  		context.JSON(http.StatusOK,gin.H{"err":"已删除"})
  	}
  })
  }
  r.Run(":8080")
  }
  ```
- # 第三个实践
- 做的是跟VUE当时练习的小项目一样的内容，做一个小的待办内容。
- 这次搭建了一个简单的框架
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  "net/http"
  )
  
  type TODO struct {
  ID int `json:"id"`
  Title string `json:"title"`
  Status string `json:"status"`
  }
  func main(){
  r := gin.Default()
  // 查询数据库
  
  // 静态文件去哪里拿
  r.Static("/static","static")
  // 加载模板文件
  r.LoadHTMLGlob("src/GoLearn/src/template/*")
  
  r.GET("/", func(context *gin.Context) {
  context.HTML(http.StatusOK,"index.html",nil)
  })
  
  // v1路由的相关内容
  v1Group:=r.Group("v1")
  {
  // 待办事项
  // 添加
  v1Group.POST("/todo", func(context *gin.Context) {
  
  })
  // 查询
  // 查询单个
  v1Group.GET("/todo/:id", func(context *gin.Context) {
  
  })
  // 查询全部
  v1Group.GET("/todo", func(context *gin.Context) {
  
  })
  // 修改
  v1Group.PUT("/todo", func(context *gin.Context) {
  
  })
  // 删除
  v1Group.DELETE("/todo", func(context *gin.Context) {
  
  })
  }
  r.Run(":8080")
  }
  ```
-