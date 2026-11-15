# 1.  如何构建一个Go项目工程
- 想要构建一个go语言的工程，首先需要构造一个go项目的结构
- 创建项目：
  
  ```
  go mod init <module-name>
  ```
- 检测依赖
  
  ```
  go mod tidy
  ```
	- 检测完成之后，这个项目文件夹下的go.mod会产生变化。会将项目中需要的依赖添加进去。
- 下载依赖
	- 我们需要将依赖下载至本地，而不是使用 go get
	  
	  ```
	  go mod download
	  ```
- 导入依赖
  
  ```
  go mod vendor
  ```
	- 此命令会将刚才下载至 GOPATH 下的依赖转移至该项目根目录下的 vendor(自动新建) 文件夹下, 此时我们就可以使用这些依赖了
- 依赖更新：这里的更新不是指版本的更新，而是指引入新依赖，依赖更新请从检测依赖部分一直执行即可： 
  
  ```
  go mod tidy 
  go mod download 
  go mod vendor
  ```
- # 2.  GO 框架
- ## 2.1.  Gorm
- ### 2.1.1.  初识Gorm
- Object-Relationl Mapping，即对象关系映射，这里的Relationl指的是关系型数据库。
- 它的作用是在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去和复杂的SQL语句打交道，只要像平时操作对象一样操作它就可以了 。
- Gorm：
	- 1.Golang写的，GitHub上活跃度很高的orm库
	- 2.特点：
		- 全功能ORM；关联(包含一个，包含多个，属于，多对多，多种包含)；Callbacks(创建/保存/更新/删除/查找之前/之后)；预加载；事务；复合主键；SQL Builder；自动迁移；日志；可扩展，编写基于GORM回调的插件；每个功能都有测试；开发人员友好
	- 导入相应的包
	  
	  ```
	  go get github.com/jinzhu/gorm
	  ```
- 模型名和表名的对应关系
	- 规则
		- 第一个大写字母变为小写;
		- 遇到其他大写字母变为小写并且在前面加下划线;
		- 连着的几个大写字母，只有第一个遵循上面的两条规则，其他的大写字母转为小写，不加下划线，遇到小写，前面的第一个大写字母变小写并加下划线;
	- 复数形式;
		- 举例
			- User => users 首字母小写，复数
			- UserInfo => user_infos
			- DBUserInfo => db_user_infos
			- DBXXXXUserInfo => dbxxxx_user_infos
- 示例
  
  ```
  package main
  
  import (
  "fmt"
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mysql"
  )
  
  type UserInfo struct {
  ID uint
  Name string
  Gender string
  Hobby string
  }
  
  func main() {
  db, err := gorm.Open("mysql", "root:Just123456@(localhost:3306)/youth?charset=utf8mb4&parseTime=True&loc=Local")
  if err != nil{
  fmt.Println(err)
  }
  defer db.Close()
  db.AutoMigrate(&UserInfo{})
  
  // 创建数据行
  u1 := UserInfo{ID: 1,Name: "李四",Gender: "男",Hobby: "吹喇叭"}
  db.Create(&u1)
  
  //查询
  var u2 UserInfo
  db.First(&u2)
  fmt.Println("u2:%#v\n",u2)
  // 更新
  db.Model(&u2).Update("hobby","双色球")
  
  db.First(&u2)
  fmt.Println("u2:%#v\n",u2)
  
  // 删除
  db.Delete(&u2)
  }
  ```
- ### 2.1.2.  ORM Model定义
- 在使用ORM工具时，通常我们需要在代码中定义模型（Models）与数据库中的数据表进行映射，在GORM中模型（Models）通常是正常定义的结构体、基本的go类型或它们的指针。 同时也支持sql.Scanner及driver.Valuer接口（interfaces）。
- gorm.Model
	- 为了方便模型定义，GORM内置了一个gorm.Model结构体。gorm.Model是一个包含了ID, CreatedAt, UpdatedAt, DeletedAt四个字段的Golang结构体。
	  
	  ```
	  // gorm.Model 定义
	  type Model struct {
	  ID        uint `gorm:"primary_key"`
	  CreatedAt time.Time
	  UpdatedAt time.Time
	  DeletedAt *time.Time
	  }
	  ```
	- 可以把Model嵌入到自己的自定义中来
	  
	  ```
	  // 将 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`字段注入到`User`模型中
	  type User struct {
	  gorm.Model
	  Name string
	  }
	  ```
	- 也可以完全定义自己的模型
	  
	  ```
	  // 不使用gorm.Model，自行定义模型
	  type User struct {
	  ID   int
	  Name string
	  }
	  ```
	- 自定义模型示例
	  
	  ```
	  package main
	  
	  import (
	  "database/sql"
	  "fmt"
	  "github.com/jinzhu/gorm"
	  _ "github.com/jinzhu/gorm/dialects/mysql"
	  "time"
	  )
	  
	  type User struct {
	  gorm.Model
	  Name         string
	  Age          sql.NullInt64
	  Birthday     *time.Time
	  Email        string  `gorm:"type:varchar(100);unique_index"`
	  Role         string  `gorm:"size:255"` // 设置字段大小为255
	  MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
	  Num          int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型
	  Address      string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
	  IgnoreMe     int     `gorm:"-"` // 忽略本字段
	  }
	  
	  func main() {
	  db, err := gorm.Open("mysql", "root:Just123456@(localhost:3306)/youth?charset=utf8mb4&parseTime=True&loc=Local")
	  if err != nil{
	  fmt.Println(err)
	  }
	  defer db.Close()
	  db.AutoMigrate(&User{})
	  
	  }
	  ```
- 使用结构体声明模型时，标记（tags）是可选项。gorm支持以下标记:
- 支持的结构体标记（Struct tags）
  
  | 结构体标记（Tag） | 描述 |
  | Column | 指定列名 |
  | Type | 指定列数据类型 |
  | Size | 指定列大小, 默认值255 |
  | PRIMARY_KEY | 将列指定为主键 |
  | UNIQUE | 将列指定为唯一 |
  | DEFAULT | 指定列默认值 |
  | PRECISION | 指定列精度 |
  | NOT NULL | 将列指定为非 NULL |
  | AUTO_INCREMENT | 指定列是否为自增类型 |
  | INDEX | 创建具有或不带名称的索引, 如果多个索引同名则创建复合索引 |
  | UNIQUE_INDEX | 和 INDEX 类似，只不过创建的是唯一索引 |
  | EMBEDDED | 将结构设置为嵌入 |
  | EMBEDDED_PREFIX | 设置嵌入结构的前缀 |
  | - | 忽略此字段 |
- 关联相关标记（tags）
  
  | 结构体标记（Tag） | 描述 |
  | MANY2MANY | 指定连接表 |
  | FOREIGNKEY | 设置外键 |
  | ASSOCIATION_FOREIGNKEY | 设置关联外键 |
  | POLYMORPHIC | 指定多态类型 |
  | POLYMORPHIC_VALUE | 指定多态值 |
  | JOINTABLE_FOREIGNKEY | 指定连接表的外键 |
  | ASSOCIATION_JOINTABLE_FOREIGNKEY | 指定连接表的关联外键 |
  | SAVE_ASSOCIATIONS | 是否自动完成 save 的相关操作 |
  | ASSOCIATION_AUTOUPDATE | 是否自动完成 update 的相关操作 |
  | ASSOCIATION_AUTOCREATE | 是否自动完成 create 的相关操作 |
  | ASSOCIATION_SAVE_REFERENCE | 是否自动完成引用的 save 的相关操作 |
  | PRELOAD | 是否自动完成预加载的相关操作 |
- 表名的使用
	- 表名默认就是结构体名称的复数，例如：
	  
	  ```
	  type User struct {} // 默认表名是 `users`
	  
	  // 将 User 的表名设置为 `profiles`
	  func (User) TableName() string {
	  return "profiles"
	  }
	  
	  func (u User) TableName() string {
	  if u.Role == "admin" {
	   return "admin_users"
	  } else {
	   return "users"
	  }
	  }
	  
	  // 禁用默认表名的复数形式，如果置为 true，则 `User` 的默认表名是 `user`
	  db.SingularTable(true)
	  ```
	- 也可以通过Table()指定表名：
	  
	  ```
	  // 使用User结构体创建名为`deleted_users`的表
	  db.Table("deleted_users").CreateTable(&User{})
	  
	  var deleted_users []User
	  db.Table("deleted_users").Find(&deleted_users)
	  //// SELECT * FROM deleted_users;
	  
	  db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
	  //// DELETE FROM deleted_users WHERE name = 'jinzhu';
	  ```
	- GORM还支持更改默认表名称规则：
	  
	  ```
	  gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
	  return "prefix_" + defaultTableName;
	  }
	  ```
	- 时间戳跟踪
		- createdAt：如果模型有 CreatedAt字段，该字段的值将会是初次创建记录的时间。
		  
		  ```
		  db.Create(&user) // `CreatedAt`将会是当前时间
		  
		  // 可以使用`Update`方法来改变`CreateAt`的值
		  db.Model(&user).Update("CreatedAt", time.Now())
		  ```
		- UpdateAt：如果模型有UpdatedAt字段，该字段的值将会是每次更新记录的时间。
		  
		  ```
		  db.Save(&user) // `UpdatedAt`将会是当前时间
		  db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间
		  ```
		- DeletedAt：
			- 如果模型有DeletedAt字段，调用Delete删除该记录时，将会设置DeletedAt字段为当前时间，而不是直接将记录从数据库中删除。
- ### 2.1.3.  CRUD——创建
- 首先定义模型，然后使用NewRecord（）查询主键是否存在，主键为空使用Create（）进行创建
  
  ```
  package main
  
  import (
  "fmt"
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mysql"
  )
  
  type User struct {
  ID   int64
  Name string `gorm:"default:'小王子'"`
  Age  int64
  }
  
  func main() {
  // 1. 连接数据库
  db, err := gorm.Open("mysql", "root:Just123456@(localhost:3306)/youth?charset=utf8mb4&parseTime=True&loc=Local")
  if err != nil{
  fmt.Println(err)
  }
  defer db.Close()
  // 2. 把模型和数据库中的表对应起来
  db.AutoMigrate(&User{})
  
  // 3. 创建数据
  u:=User{Name:"李四",Age: 19}
  fmt.Println(db.NewRecord(&u))
  db.Debug().Create(&u)
  fmt.Println(db.NewRecord(&u))
  }
  ```
- 创建默认值
	- 可以通过 tag 定义字段的默认值，比如：
	  
	  ```
	  type User struct {
	  ID   int64
	  Name string `gorm:"default:'小王子'"`
	  Age  int64
	  }
	  ```
	- **注意：**通过tag定义字段的默认值，在创建记录时候生成的 SQL 语句会排除没有值或值为 零值 的字段。 在将记录插入到数据库后，Gorm会从数据库加载那些字段的默认值。
	  
	  ```
	  var user = User{Name: "", Age: 99}
	  db.Create(&user)
	  ```
	- 上面代码实际执行的SQL语句是INSERT INTO users("age") values('99');，排除了零值字段Name，而在数据库中这一条数据会使用设置的默认值小王子作为Name字段的值。
	- **注意：**所有字段的零值, 比如0, "",false或者其它零值，都不会保存到数据库内，但会使用他们的默认值。 如果你想避免这种情况，可以考虑使用指针或实现 Scanner/Valuer接口。
- 使用指针方式实现零值存入数据库
  
  ```
  // 使用指针
  type User struct {
  ID   int64
  Name *string `gorm:"default:'小王子'"`
  Age  int64
  }
  user := User{Name: new(string), Age: 18))}
  db.Create(&user)  // 此时数据库中该条记录name字段的值就是''
  ```
- 使用Scanner/Valuer接口方式实现零值存入数据库
  
  ```
  // 使用 Scanner/Valuer
  type User struct {
  ID int64
  Name sql.NullString `gorm:"default:'小王子'"` // sql.NullString 实现了Scanner/Valuer接口
  Age  int64
  }
  user := User{Name: sql.NullString{"", true}, Age:18}
  db.Create(&user)  // 此时数据库中该条记录name字段的值就是''
  ```
- 扩展创建选项
	- 例如PostgreSQL数据库中可以使用下面的方式实现合并插入, 有则更新, 无则插入。
	  
	  ```
	  // 为Instert语句添加扩展SQL选项
	  db.Set("gorm:insert_option", "ON CONFLICT").Create(&product)
	  // INSERT INTO products (name, code) VALUES ("name", "code") ON CONFLICT;
	  ```
- ### 2.1.4.  CRUD——删除
- 删除记录
	- 删除记录时，请确保主键字段有值，GORM 会通过主键去删除记录，如果主键为空，GORM 会删除该 model 的所有记录。
	  
	  ```
	  // 删除现有记录
	  db.Delete(&email)
	  //// DELETE from emails where id=10;
	  
	  // 为删除 SQL 添加额外的 SQL 操作
	  db.Set("gorm:delete_option", "OPTION (OPTIMIZE FOR UNKNOWN)").Delete(&email)
	  //// DELETE from emails where id=10 OPTION (OPTIMIZE FOR UNKNOWN);
	  ```
- 批量删除
	- 删除全部匹配的记录
	  
	  ```
	  db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
	  //// DELETE from emails where email LIKE "%jinzhu%";
	  
	  db.Delete(Email{}, "email LIKE ?", "%jinzhu%")
	  //// DELETE from emails where email LIKE "%jinzhu%";
	  ```
- 软删除
	- 如果一个 model 有 DeletedAt 字段，他将自动获得软删除的功能！ 当调用 Delete 方法时， 记录不会真正的从数据库中被删除， 只会将DeletedAt 字段的值会被设置为当前时间
	  
	  ```
	  db.Delete(&user)
	  //// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;
	  
	  // 批量删除
	  db.Where("age = ?", 20).Delete(&User{})
	  //// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;
	  
	  // 查询记录时会忽略被软删除的记录
	  db.Where("age = 20").Find(&user)
	  //// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
	  
	  // Unscoped 方法可以查询被软删除的记录
	  db.Unscoped().Where("age = 20").Find(&users)
	  //// SELECT * FROM users WHERE age = 20;
	  ```
- 物理删除（真删除）
  
  ```
  // Unscoped 方法可以物理删除记录
  db.Unscoped().Delete(&order)
  //// DELETE FROM orders WHERE id=10;
  ```
- ### 2.1.5.  CRUD——更新
- 更新所有字段
	- Save()默认会更新该对象的所有字段，即使你没有赋值。
	  
	  ```
	  db.First(&user)
	  
	  user.Name = "七米"
	  user.Age = 99
	  db.Save(&user)
	  
	  ////  UPDATE `users` SET `created_at` = '2020-02-16 12:52:20', `updated_at` = '2020-02-16 12:54:55', `deleted_at` = NULL, `name` = '七米', `age` = 99, `active` = true  WHERE `users`.`deleted_at` IS NULL AND `users`.`id` = 1
	  ```
- 更新修改字段
	- 如果你只希望更新指定字段，可以使用Update或者Updates
	  
	  ```
	  // 更新单个属性，如果它有变化
	  db.Model(&user).Update("name", "hello")
	  //// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;
	  
	  // 根据给定的条件更新单个属性
	  db.Model(&user).Where("active = ?", true).Update("name", "hello")
	  //// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
	  
	  // 使用 map 更新多个属性，只会更新其中有变化的属性
	  db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
	  //// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
	  
	  // 使用 struct 更新多个属性，只会更新其中有变化且为非零值的字段
	  db.Model(&user).Updates(User{Name: "hello", Age: 18})
	  //// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;
	  
	  // 警告：当使用 struct 更新时，GORM只会更新那些非零值的字段
	  // 对于下面的操作，不会发生任何更新，"", 0, false 都是其类型的零值
	  db.Model(&user).Updates(User{Name: "", Age: 0, Active: false})
	  ```
- 更新选定字段
	- 如果你想更新或忽略某些字段，你可以使用 Select，Omit
	  
	  ```
	  db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
	  //// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;
	  
	  db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
	  //// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
	  ```
- 无Hooks更新
	- 上面的更新操作会自动运行 model 的 BeforeUpdate, AfterUpdate 方法，更新 UpdatedAt 时间戳, 在更新时保存其 Associations, 如果你不想调用这些方法，你可以使用 UpdateColumn， UpdateColumns
	  
	  ```
	  // 更新单个属性，类似于 `Update`
	  db.Model(&user).UpdateColumn("name", "hello")
	  //// UPDATE users SET name='hello' WHERE id = 111;
	  
	  // 更新多个属性，类似于 `Updates`
	  db.Model(&user).UpdateColumns(User{Name: "hello", Age: 18})
	  //// UPDATE users SET name='hello', age=18 WHERE id = 111;
	  ```
- 批量更新
	- 批量更新时Hooks（钩子函数）不会运行。
	  
	  ```
	  db.Table("users").Where("id IN (?)", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
	  //// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
	  
	  // 使用 struct 更新时，只会更新非零值字段，若想更新所有字段，请使用map[string]interface{}
	  db.Model(User{}).Updates(User{Name: "hello", Age: 18})
	  //// UPDATE users SET name='hello', age=18;
	  
	  // 使用 `RowsAffected` 获取更新记录总数
	  db.Model(User{}).Updates(User{Name: "hello", Age: 18}).RowsAffected
	  ```
- 使用SQL表达式
	- 先查询表中的第一条数据保存至user变量。
	  
	  ```
	  var user User
	  db.First(&user)
	  db.Model(&user).Update("age", gorm.Expr("age * ? + ?", 2, 100))
	  //// UPDATE `users` SET `age` = age * 2 + 100, `updated_at` = '2020-02-16 13:10:20'  WHERE `users`.`id` = 1;
	  
	  db.Model(&user).Updates(map[string]interface{}{"age": gorm.Expr("age * ? + ?", 2, 100)})
	  //// UPDATE "users" SET "age" = age * '2' + '100', "updated_at" = '2020-02-16 13:05:51' WHERE `users`.`id` = 1;
	  
	  db.Model(&user).UpdateColumn("age", gorm.Expr("age - ?", 1))
	  //// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1';
	  
	  db.Model(&user).Where("age > 10").UpdateColumn("age", gorm.Expr("age - ?", 1))
	  //// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1' AND quantity > 10;
	  ```
- 修改Hooks中的值
	- 如果你想修改 BeforeUpdate, BeforeSave 等 Hooks 中更新的值，你可以使用 scope.SetColumn, 例如：
	  
	  ```
	  func (user *User) BeforeSave(scope *gorm.Scope) (err error) {
	  if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
	   scope.SetColumn("EncryptedPassword", pw)
	  }
	  }
	  ```
- 其他更新选项
  
  ```
  // 为 update SQL 添加其它的 SQL
  db.Model(&user).Set("gorm:update_option", "OPTION (OPTIMIZE FOR UNKNOWN)").Update("name", "hello")
  //// UPDATE users SET name='hello', updated_at = '2013-11-17 21:34:10' WHERE id=111 OPTION (OPTIMIZE FOR UNKNOWN);
  ```
- ### 2.1.6.  CRUD——查询
- 一般查询
  
  ```
  // 根据主键查询第一条记录
  db.First(&user)
  //// SELECT * FROM users ORDER BY id LIMIT 1;
  
  // 随机获取一条记录
  db.Take(&user)
  //// SELECT * FROM users LIMIT 1;
  
  // 根据主键查询最后一条记录
  db.Last(&user)
  //// SELECT * FROM users ORDER BY id DESC LIMIT 1;
  
  // 查询所有的记录
  db.Find(&users)
  //// SELECT * FROM users;
  
  // 查询指定的某条记录(仅当主键为整型时可用)
  db.First(&user, 10)
  //// SELECT * FROM users WHERE id = 10;
  ```
- Where查询条件
	- 普通SQL查询
	  
	  ```
	  // Get first matched record
	  db.Where("name = ?", "jinzhu").First(&user)
	  //// SELECT * FROM users WHERE name = 'jinzhu' limit 1;
	  
	  // Get all matched records
	  db.Where("name = ?", "jinzhu").Find(&users)
	  //// SELECT * FROM users WHERE name = 'jinzhu';
	  
	  // <>
	  db.Where("name <> ?", "jinzhu").Find(&users)
	  //// SELECT * FROM users WHERE name <> 'jinzhu';
	  
	  // IN
	  db.Where("name IN (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
	  //// SELECT * FROM users WHERE name in ('jinzhu','jinzhu 2');
	  
	  // LIKE
	  db.Where("name LIKE ?", "%jin%").Find(&users)
	  //// SELECT * FROM users WHERE name LIKE '%jin%';
	  
	  // AND
	  db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
	  //// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;
	  
	  // Time
	  db.Where("updated_at > ?", lastWeek).Find(&users)
	  //// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';
	  
	  // BETWEEN
	  db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
	  //// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
	  ```
- Struct&Map查询
	- 当通过结构体进行查询时，GORM将会只通过非零值字段查询，这意味着如果你的字段值为0，''，false或者其他零值时，将不会被用于构建查询条件。
	- 就可以使用指针或者实现 Scanner/Valuer 接口来避免这个问题
	  
	  ```
	  // 使用指针
	  type User struct {
	  gorm.Model
	  Name string
	  Age  *int
	  }
	  
	  // 使用 Scanner/Valuer
	  type User struct {
	  gorm.Model
	  Name string
	  Age  sql.NullInt64  // sql.NullInt64 实现了 Scanner/Valuer 接口
	  }
	  ```
	  
	  ```
	  // Struct
	  db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
	  //// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;
	  
	  // Map
	  db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
	  //// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;
	  
	  // 主键的切片
	  db.Where([]int64{20, 21, 22}).Find(&users)
	  //// SELECT * FROM users WHERE id IN (20, 21, 22);
	  ```
- Not条件
	- 作用与 Where 类似的情形如下：
	  
	  ```
	  db.Not("name", "jinzhu").First(&user)
	  //// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;
	  
	  // Not In
	  db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
	  //// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");
	  
	  // Not In slice of primary keys
	  db.Not([]int64{1,2,3}).First(&user)
	  //// SELECT * FROM users WHERE id NOT IN (1,2,3);
	  
	  db.Not([]int64{}).First(&user)
	  //// SELECT * FROM users;
	  
	  // Plain SQL
	  db.Not("name = ?", "jinzhu").First(&user)
	  //// SELECT * FROM users WHERE NOT(name = "jinzhu");
	  
	  // Struct
	  db.Not(User{Name: "jinzhu"}).First(&user)
	  //// SELECT * FROM users WHERE name <> "jinzhu";
	  ```
- Or条件
  
  ```
  db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
  //// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';
  
  // Struct
  db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2"}).Find(&users)
  //// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';
  
  // Map
  db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2"}).Find(&users)
  //// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';
  ```
- 内联条件
	- 作用与Where查询类似，当内联条件与多个立即执行方法一起使用时, 内联条件不会传递给后面的立即执行方法。
	  
	  ```
	  // 根据主键获取记录 (只适用于整形主键)
	  db.First(&user, 23)
	  //// SELECT * FROM users WHERE id = 23 LIMIT 1;
	  // 根据主键获取记录, 如果它是一个非整形主键
	  db.First(&user, "id = ?", "string_primary_key")
	  //// SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;
	  
	  // Plain SQL
	  db.Find(&user, "name = ?", "jinzhu")
	  //// SELECT * FROM users WHERE name = "jinzhu";
	  
	  db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
	  //// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;
	  
	  // Struct
	  db.Find(&users, User{Age: 20})
	  //// SELECT * FROM users WHERE age = 20;
	  
	  // Map
	  db.Find(&users, map[string]interface{}{"age": 20})
	  //// SELECT * FROM users WHERE age = 20;
	  ```
- 额外的查询项
  
  ```
  // 为查询 SQL 添加额外的 SQL 操作
  db.Set("gorm:query_option", "FOR UPDATE").First(&user, 10)
  //// SELECT * FROM users WHERE id = 10 FOR UPDATE;
  ```
- FirstOrLint
	- 获取匹配的第一条记录，否则根据给定的条件初始化一个新的对象 (仅支持 struct 和 map 条件)
	  
	  ```
	  // 未找到
	  db.FirstOrInit(&user, User{Name: "non_existing"})
	  //// user -> User{Name: "non_existing"}
	  
	  // 找到
	  db.Where(User{Name: "Jinzhu"}).FirstOrInit(&user)
	  //// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
	  db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})
	  //// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
	  ```
	- Attrs
	  
	  ```
	  // 未找到
	  db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrInit(&user)
	  //// SELECT * FROM USERS WHERE name = 'non_existing';
	  //// user -> User{Name: "non_existing", Age: 20}
	  
	  db.Where(User{Name: "non_existing"}).Attrs("age", 20).FirstOrInit(&user)
	  //// SELECT * FROM USERS WHERE name = 'non_existing';
	  //// user -> User{Name: "non_existing", Age: 20}
	  
	  // 找到
	  db.Where(User{Name: "Jinzhu"}).Attrs(User{Age: 30}).FirstOrInit(&user)
	  //// SELECT * FROM USERS WHERE name = jinzhu';
	  //// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
	  ```
	- Assign
		- 不管记录是否找到，都将参数赋值给 struct.
		  
		  ```
		  // 未找到
		  db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrInit(&user)
		  //// user -> User{Name: "non_existing", Age: 20}
		  
		  // 找到
		  db.Where(User{Name: "Jinzhu"}).Assign(User{Age: 30}).FirstOrInit(&user)
		  //// SELECT * FROM USERS WHERE name = jinzhu';
		  //// user -> User{Id: 111, Name: "Jinzhu", Age: 30}
		  ```
- FirstOrCreate
	- 获取匹配的第一条记录, 否则根据给定的条件创建一个新的记录 (仅支持 struct 和 map 条件)
	  
	  ```
	  // 未找到
	  db.FirstOrCreate(&user, User{Name: "non_existing"})
	  //// INSERT INTO "users" (name) VALUES ("non_existing");
	  //// user -> User{Id: 112, Name: "non_existing"}
	  
	  // 找到
	  db.Where(User{Name: "Jinzhu"}).FirstOrCreate(&user)
	  //// user -> User{Id: 111, Name: "Jinzhu"}
	  ```
	- Attrs
		- 如果记录未找到，将使用参数创建 struct 和记录.
		  
		  ```
		  // 未找到
		  db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrCreate(&user)
		  //// SELECT * FROM users WHERE name = 'non_existing';
		  //// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
		  //// user -> User{Id: 112, Name: "non_existing", Age: 20}
		  
		  // 找到
		  db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 30}).FirstOrCreate(&user)
		  //// SELECT * FROM users WHERE name = 'jinzhu';
		  //// user -> User{Id: 111, Name: "jinzhu", Age: 20}
		  ```
	- Assign
		- 不管记录是否找到，都将参数赋值给 struct 并保存至数据库.
		  
		  ```
		  // 未找到
		  db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrCreate(&user)
		  //// SELECT * FROM users WHERE name = 'non_existing';
		  //// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
		  //// user -> User{Id: 112, Name: "non_existing", Age: 20}
		  
		  // 找到
		  db.Where(User{Name: "jinzhu"}).Assign(User{Age: 30}).FirstOrCreate(&user)
		  //// SELECT * FROM users WHERE name = 'jinzhu';
		  //// UPDATE users SET age=30 WHERE id = 111;
		  //// user -> User{Id: 111, Name: "jinzhu", Age: 30}
		  ```
- 高级查询
	- 子查询
	- 基于 *gorm.expr 的子查询
	  
	  ```
	  db.Where("amount > ?", db.Table("orders").Select("AVG(amount)").Where("state = ?", "paid").SubQuery()).Find(&orders)
	  // SELECT * FROM "orders"  WHERE "orders"."deleted_at" IS NULL AND (amount > (SELECT AVG(amount) FROM "orders"  WHERE (state = 'paid')));
	  ```
	- 选择字段
	- Select，指定你想从数据库中检索出的字段，默认会选择全部字段。
	  
	  ```
	  db.Select("name, age").Find(&users)
	  //// SELECT name, age FROM users;
	  
	  db.Select([]string{"name", "age"}).Find(&users)
	  //// SELECT name, age FROM users;
	  
	  db.Table("users").Select("COALESCE(age,?)", 42).Rows()
	  //// SELECT COALESCE(age,'42') FROM users;
	  ```
	- 排序
	- Order，指定从数据库中检索出记录的顺序。设置第二个参数 reorder 为 true ，可以覆盖前面定义的排序条件。
	  
	  ```
	  db.Order("age desc, name").Find(&users)
	  //// SELECT * FROM users ORDER BY age desc, name;
	  
	  // 多字段排序
	  db.Order("age desc").Order("name").Find(&users)
	  //// SELECT * FROM users ORDER BY age desc, name;
	  
	  // 覆盖排序
	  db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
	  //// SELECT * FROM users ORDER BY age desc; (users1)
	  //// SELECT * FROM users ORDER BY age; (users2)
	  ```
	- 数量
	- Limit，指定从数据库检索出的最大记录数。
	  
	  ```
	  db.Limit(3).Find(&users)
	  //// SELECT * FROM users LIMIT 3;
	  
	  // -1 取消 Limit 条件
	  db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
	  //// SELECT * FROM users LIMIT 10; (users1)
	  //// SELECT * FROM users; (users2)
	  ```
	- 偏移
	- Offset，指定开始返回记录前要跳过的记录数。
	  
	  ```
	  db.Offset(3).Find(&users)
	  //// SELECT * FROM users OFFSET 3;
	  
	  // -1 取消 Offset 条件
	  db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
	  //// SELECT * FROM users OFFSET 10; (users1)
	  //// SELECT * FROM users; (users2)
	  ```
	- 总量
	- Count，该 model 能获取的记录总数。
	- 注意 Count 必须是链式查询的最后一个操作 ，因为它会覆盖前面的 SELECT，但如果里面使用了 count 时不会覆盖。
	  
	  ```
	  db.Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Find(&users).Count(&count)
	  //// SELECT * from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (users)
	  //// SELECT count(*) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (count)
	  
	  db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
	  //// SELECT count(*) FROM users WHERE name = 'jinzhu'; (count)
	  
	  db.Table("deleted_users").Count(&count)
	  //// SELECT count(*) FROM deleted_users;
	  
	  db.Table("deleted_users").Select("count(distinct(name))").Count(&count)
	  //// SELECT count( distinct(name) ) FROM deleted_users; (count)
	  ```
	- Group&Having
	  
	  ```
	  rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
	  for rows.Next() {
	  ...
	  }
	  
	  // 使用Scan将多条结果扫描进事先准备好的结构体切片中
	  type Result struct {
	  Date time.Time
	  Total int
	  }
	  var rets []Result
	  db.Table("users").Select("date(created_at) as date, sum(age) as total").Group("date(created_at)").Scan(&rets)
	  
	  rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
	  for rows.Next() {
	  ...
	  }
	  
	  type Result struct {
	  Date  time.Time
	  Total int64
	  }
	  db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
	  ```
	- 连接
	- Joins，指定连接条件
	  
	  ```
	  rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
	  for rows.Next() {
	  ...
	  }
	  
	  db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)
	  
	  // 多连接及参数
	  db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "411111111111").Find(&user)
	  ```
	- Pluck
	- Pluck，查询 model 中的一个列作为切片，如果您想要查询多个列，您应该使用 [Scan](https://www.liwenzhou.com/posts/Go/gorm_crud/#Scan)
	  
	  ```
	  var ages []int64
	  db.Find(&users).Pluck("age", &ages)
	  
	  var names []string
	  db.Model(&User{}).Pluck("name", &names)
	  
	  db.Table("deleted_users").Pluck("name", &names)
	  
	  // 想查询多个字段？ 这样做：
	  db.Select("name, age").Find(&users)
	  ```
	- 扫描
	- Scan，扫描结果至一个 struct.
	  
	  ```
	  type Result struct {
	  Name string
	  Age  int
	  }
	  
	  var result Result
	  db.Table("users").Select("name, age").Where("name = ?", "Antonio").Scan(&result)
	  
	  var results []Result
	  db.Table("users").Select("name, age").Where("id > ?", 0).Scan(&results)
	  
	  // 原生 SQL
	  db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
	  ```
- ## 2.2.  Gin
- ### 2.2.1.  初识Gin
- Gin是一个简单易用的Web框架
- 安装Gin
  
  ```
  go get -u github.com/gin-gonic/gin
  ```
- 第一个示例
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  _ "github.com/jinzhu/gorm/dialects/mysql"
  "net/http"
  )
  
  func main() {
  r:=gin.Default()
  
  r.GET("/hello", func(context *gin.Context) {
  //
  context.JSON(http.StatusOK,gin.H{
  	"message":"Hello World",
  })
  })
  r.Run()
  }
  ```
- 将上面的代码保存并编译执行，然后使用浏览器打开127.0.0.1:8080/hello就能看到一串JSON字符串。
- ### 2.2.2.  Restful风格API
- REST与技术无关，代表的是一种软件架构风格，REST是Representational State Transfer的简称，中文翻译为“表征状态转移”或“表现层状态转化”。
- 简单来说，REST的含义就是客户端与Web服务器之间进行交互的时候，使用HTTP协议中的4个请求方法代表不同的动作。
	- GET用来获取资源
	- POST用来新建资源
	- PUT用来更新资源
	- DELETE用来删除资源。
- 只要API程序遵循了REST风格，那就可以称其为RESTful API。目前在前后端分离的架构中，前后端基本都是通过RESTful API来进行交互。
- 示例代码
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  )
  
  func main() {
  r:=gin.Default()
  
  r.GET("/book", func(c *gin.Context) {
  c.JSON(200, gin.H{
  	"message": "GET",
  })
  })
  
  r.POST("/book", func(c *gin.Context) {
  c.JSON(200, gin.H{
  	"message": "POST",
  })
  })
  
  r.PUT("/book", func(c *gin.Context) {
  c.JSON(200, gin.H{
  	"message": "PUT",
  })
  })
  
  r.DELETE("/book", func(c *gin.Context) {
  c.JSON(200, gin.H{
  	"message": "DELETE",
  })
  })
  r.Run()
  }
  ```
- ### 2.2.3.  template嵌套和自定义函数
- Go的模板支持自定义函数。
- 示例代码
  
  ```
  package main
  
  import (
  "fmt"
  "html/template"
  "io/ioutil"
  "net/http"
  )
  
  type UserInfo struct {
  Name string
  Age int
  Gender string
  }
  
  func sayHello(w http.ResponseWriter,r *http.Request){
  // 2. 解析模板
  t,err:=ioutil.ReadFile("src/GoLearn/src/index.tmpl")
  if err != nil{
  fmt.Printf("template parse failed , err:%v",err)
  return
  }
  // 自定义函数
  kua := func(name string)(string,error) {
  return name+"你真帅",nil
  }
  // 3. 渲染模板
  tmp,err:=template.New("index").Funcs(template.FuncMap{"kua":kua}).Parse(string(t))
  if err != nil{
  fmt.Printf("template render failed , err:%v",err)
  return
  }
  
  userinfo:=UserInfo{
  Name:"李四",
  Age:20,
  Gender:"男",
  }
  
  tmp.Execute(w,userinfo)
  
  }
  
  func main() {
  http.HandleFunc("/",sayHello)
  err:=http.ListenAndServe(":9000",nil)
  if err!=nil{
  fmt.Printf("server start failed,err:%v",err)
  return
  }
  }
  ```
- 这里面可以通过读取文件的方式来实现渲染，也可以通过直接parseFile的方式来实现。
- 可以在template中嵌套其他的template。这个template可以是单独的文件，也可以是通过define定义的template。
- 举个例子： t.tmpl文件内容如下：
  
  ```
  <html>
  <head>
    <title>SayHello</title>
  </head>
  <body>
  {{ kua .Name }}
    <h1>测试嵌套template语法</h1>
    <hr>
  {{template "ul.tmpl"}}
    <hr>
  {{template "ol.tmpl"}}
  </body>
  </html>
  
  {{ define "ol.tmpl"}}
    <ol>
        <li>吃饭</li>
        <li>睡觉</li>
        <li>打豆豆</li>
    </ol>
  {{end}}
  ```
  
  ```
  <ul>
    <li>注释</li>
    <li>日志</li>
    <li>测试</li>
  </ul>
  ```
- 注意：在解析模板时，被嵌套的模板一定要在后面解析，例如上面的示例中t.tmpl模板中嵌套了ul.tmpl，所以ul.tmpl要在t.tmpl后进行解析。
- ### 2.2.4.  template语法
- {{ . }}
	- 模板语法都包含在{{和}}中间，其中{{.}}中的点表示当前对象。当我们传入一个结构体对象时，我们可以根据.来访问结构体的对应字段。
	- 同理，当我们传入的变量是map时，也可以在模板文件中通过.根据key来取值。
- 注释
  
  ```
  {{/* a comment */}}
  注释，执行时会忽略。可以多行。注释不能嵌套，并且必须紧贴分界符始止。
  ```
- pipeline
	- pipeline是指产生数据的操作。比如{{.}}、{{.Name}}等。Go的模板语法中支持使用管道符号|链接多个命令，用法和unix下的管道类似：|前面的命令会将运算结果(或返回值)传递给后一个命令的最后一个位置。
	- **注意：**并不是只有使用了|才是pipeline。Go的模板语法中，pipeline的概念是传递数据，只要能产生数据的，都是pipeline。
- 变量
	- 我们还可以在模板中声明变量，用来保存传入模板的数据或其他语句生成的结果。具体语法如下：
	- $obj := {{.}}
	- 其中$obj是变量的名字，在后续的代码中就可以使用该变量了。
- 移除空格
	- 有时候我们在使用模板语法的时候会不可避免的引入一下空格或者换行符，这样模板最终渲染出来的内容可能就和我们想的不一样，这个时候可以使用{{-语法去除模板内容左侧的所有空白符号， 使用-}}去除模板内容右侧的所有空白符号。
- 条件判断
	- Go模板语法中的条件判断有以下几种:
	  
	  ```
	  {{if pipeline}} T1 {{end}}
	  
	  {{if pipeline}} T1 {{else}} T0 {{end}}
	  
	  {{if pipeline}} T1 {{else if pipeline}} T0 {{end}}
	  ```
- range
	- Go的模板语法中使用range关键字进行遍历，有以下两种写法，其中pipeline的值必须是数组、切片、字典或者通道。
	  
	  ```
	  {{range pipeline}} T1 {{end}}
	  如果pipeline的值其长度为0，不会有任何输出
	  
	  {{range pipeline}} T1 {{else}} T0 {{end}}
	  如果pipeline的值其长度为0，则会执行T0。
	  ```
- with
  
  ```
  {{with pipeline}} T1 {{end}}
  如果pipeline为empty不产生输出，否则将dot设为pipeline的值并执行T1。不修改外面的dot。
  
  {{with pipeline}} T1 {{else}} T0 {{end}}
  如果pipeline为empty，不改变dot并执行T0，否则dot设为pipeline的值并执行T1。
  ```
- 预定义函数
	- 执行模板时，函数从两个函数字典中查找：首先是模板函数字典，然后是全局函数字典。一般不在模板内定义函数，而是使用Funcs方法添加函数到模板里。
	- 预定义的全局函数如下：
	  
	  ```
	  and
	   函数返回它的第一个empty参数或者最后一个参数；
	   就是说"and x y"等价于"if x then y else x"；所有参数都会执行；
	  or
	   返回第一个非empty参数或者最后一个参数；
	   亦即"or x y"等价于"if x then x else y"；所有参数都会执行；
	  not
	   返回它的单个参数的布尔值的否定
	  len
	   返回它的参数的整数类型长度
	  index
	   执行结果为第一个参数以剩下的参数为索引/键指向的值；
	   如"index x 1 2 3"返回x[1][2][3]的值；每个被索引的主体必须是数组、切片或者字典。
	  print
	   即fmt.Sprint
	  printf
	   即fmt.Sprintf
	  println
	   即fmt.Sprintln
	  html
	   返回与其参数的文本表示形式等效的转义HTML。
	   这个函数在html/template中不可用。
	  urlquery
	   以适合嵌入到网址查询中的形式返回其参数的文本表示的转义值。
	   这个函数在html/template中不可用。
	  js
	   返回与其参数的文本表示形式等效的转义JavaScript。
	  call
	   执行结果是调用第一个参数的返回值，该参数必须是函数类型，其余参数作为调用该函数的参数；
	   如"call .X.Y 1 2"等价于go语言里的dot.X.Y(1, 2)；
	   其中Y是函数类型的字段或者字典的值，或者其他类似情况；
	   call的第一个参数的执行结果必须是函数类型的值（和预定义函数如print明显不同）；
	   该函数类型值必须有1到2个返回值，如果有2个则后一个必须是error接口类型；
	   如果有2个返回值的方法返回的error非nil，模板执行会中断并返回给调用模板执行者该错误；
	  ```
- 比较函数
	- 布尔函数会将任何类型的零值视为假，其余视为真。
	- 下面是定义为函数的二元比较运算的集合：
	  
	  ```
	  eq      如果arg1 == arg2则返回真
	  ne      如果arg1 != arg2则返回真
	  lt      如果arg1 < arg2则返回真
	  le      如果arg1 <= arg2则返回真
	  gt      如果arg1 > arg2则返回真
	  ge      如果arg1 >= arg2则返回真
	  ```
- 示例
  
  ```
  <html>
  <head>
    <title>SayHello</title>
  </head>
  <body>
  <p>u1的信息</p>
  <p>Hello, {{ .u1.Name }}</p>
  <p>年龄为 {{ .u1.Age }}</p>
  <p>sex is {{ .u1.Gender }}</p>
  <hr>
  <p>m1的信息</p>
  <p>Hello, {{ .m1.name }}</p>
  <p>年龄为 {{ .m1.age }}</p>
  <p>sex is {{ .m1.gender }}</p>
  <hr>
  {{ $age:= .u1.Age  }}
  {{ if $age}}
  {{ .u1.Age }}
  {{end}}
  </body>
  </html>
  ```
  
  ```
  package main
  
  import (
  "fmt"
  "html/template"
  "net/http"
  )
  
  type UserInfo struct {
  Name string
  Age int
  Gender string
  }
  
  func sayHello(w http.ResponseWriter,r *http.Request){
  // 2. 解析模板
  t,err:=template.ParseFiles("src/GoLearn/src/index.tmpl")
  if err != nil{
  fmt.Printf("template parse failed , err:%v",err)
  return
  }
  // 3. 渲染模板
  u1 := UserInfo{
  Name:"李四",
  Age:19,
  Gender: "man",
  }
  
  m1:=map[string]interface{}{
  "name":"张三",
  "age":20,
  "gender":"female",
  }
  
  err=t.Execute(w,map[string]interface{}{
  "u1":u1,
  "m1":m1,
  })
  
  if err != nil{
  fmt.Printf("template render failed , err:%v",err)
  return
  }
  }
  
  func main() {
  http.HandleFunc("/",sayHello)
  err:=http.ListenAndServe(":9000",nil)
  if err!=nil{
  fmt.Printf("server start failed,err:%v",err)
  return
  }
  }
  ```
- #### 2.2.4.1.  Template渲染
- 在一些前后端不分离的Web架构中，我们通常需要在后端将一些数据渲染到HTML文档中，从而实现动态的网页（网页的布局和样式大致一样，但展示的内容并不一样）效果。
- 我们这里说的模板可以理解为事先定义好的HTML文档文件，模板渲染的作用机制可以简单理解为文本替换操作–使用相应的数据去替换HTML文档中事先准备好的标记。
- 很多编程语言的Web框架中都使用各种模板引擎，比如Python语言中Flask框架中使用的jinja2模板引擎。
- Go语言渲染引擎
	- Go语言内置了文本模板引擎text/template和用于HTML文档的html/template。它们的作用机制可以简单归纳如下：
		- 模板文件通常定义为.tmpl和.tpl为后缀（也可以使用其他的后缀），必须使用UTF8编码。
		- 模板文件中使用{{和}}包裹和标识需要传入的数据。
		- 传给模板这样的数据就可以通过点号（.）来访问，如果数据是复杂类型的数据，可以通过{ { .FieldName }}来访问它的字段。
		- 除{{和}}包裹的内容外，其他内容均不做修改原样输出。
- 模板渲染三部曲：
	- 定义模板
		- 定义模板文件时需要我们按照相关语法规则去编写
	- 解析模板
		- 当然，你也可以使用func New(name string) *Template函数创建一个名为name的模板，然后对其调用上面的方法去解析模板字符串或模板文件。
		- 上面定义好了模板文件之后，可以使用下面的常用方法去解析模板文件，得到模板对象：
		  
		  ```
		  func (t *Template) Parse(src string) (*Template, error)
		  func ParseFiles(filenames ...string) (*Template, error)
		  func ParseGlob(pattern string) (*Template, error)
		  ```
	- 模板渲染
		- 渲染模板简单来说就是使用数据去填充模板，当然实际上可能会复杂很多。
		  
		  ```
		  func (t *Template) Execute(wr io.Writer, data interface{}) error
		  func (t *Template) ExecuteTemplate(wr io.Writer, name string, data interface{}) error
		  ```
- #### 2.2.4.2.  block
- 运行规则
  
  ```
  {{block "name" pipeline}} T1 {{end}}
  ```
- block是定义模板{{define "name"}} T1 {{end}}和执行{{template "name" pipeline}}缩写，典型的用法是定义一组根模板，然后通过在其中重新定义块模板进行自定义。
- 定义一个根模板templates/base.tmpl，内容如下：
  
  ```
  <!DOCTYPE html>
  <html lang="zh-CN">
  <head>
    <title>Go Templates继承</title>
  </head>
  <body>
  <div class="container-fluid">
    {{block "content" . }}{{end}}
  </div>
  </body>
  </html>
  ```
- 然后定义一个templates/index.tmpl，”继承”base.tmpl：
  
  ```
  {{template "base.tmpl"}}
  
  {{define "content"}}
    <div>Hello world!</div>
  {{end}}
  ```
- 然后使用template.ParseGlob按照正则匹配规则解析模板文件，然后通过ExecuteTemplate渲染指定的模板：
  
  ```
  package main
  
  import (
  "fmt"
  "html/template"
  "net/http"
  )
  
  func sayHello(w http.ResponseWriter,r *http.Request){
  // 解析模板
  tmpl, err := template.ParseGlob("src/GoLearn/src/template/*.tmpl")
  if err != nil {
  fmt.Println("create template failed, err:", err)
  return
  }
  msg := "李四"
  
  // 渲染模板
  err = tmpl.ExecuteTemplate(w, "index.tmpl", msg)
  if err != nil {
  fmt.Println("render template failed, err:", err)
  return
  }
  
  }
  
  func main() {
  http.HandleFunc("/",sayHello)
  err:=http.ListenAndServe(":9000",nil)
  if err!=nil{
  fmt.Printf("server start failed,err:%v",err)
  return
  }
  }
  ```
- 如果我们的模板名称冲突了，例如不同业务线下都定义了一个index.tmpl模板，我们可以通过下面两种方法来解决：
	- 在模板文件开头使用{{define 模板名}}语句显式的为模板命名。
	- 可以把模板文件存放在templates文件夹下面的不同目录中，然后使用template.ParseGlob("templates/**/*.tmpl")解析模板。
- ### 2.2.5.  渲染HTML
- 我们首先定义一个存放模板文件的templates文件夹，然后在其内部按照业务分别定义一个posts文件夹和一个users文件夹。 posts/index.html文件的内容如下：
  
  ```
  {{define "posts/index.html"}}
    <!DOCTYPE html>
    <html lang="en">
  
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>posts/index</title>
    </head>
    <body>
    {{.title}}
    </body>
    </html>
  {{end}}
  ```
- users/index.html文件的内容如下：
  
  ```
  {{define "users/index.html"}}
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>users/index</title>
  </head>
  <body>
    {{.title}}
  </body>
  </html>
  {{end}}
  ```
- Gin框架中使用LoadHTMLGlob()或者LoadHTMLFiles()方法进行HTML模板渲染。
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  "net/http"
  )
  
  func main(){
  r := gin.Default()
  r.LoadHTMLGlob("src/GoLearn/src/template/**/*")
  //r.LoadHTMLFiles("templates/posts/index.html", "templates/users/index.html")
  r.GET("/posts/index", func(c *gin.Context) {
  c.HTML(http.StatusOK, "posts/index.html", gin.H{
  	"title": "posts/index",
  })
  })
  
  r.GET("users/index", func(c *gin.Context) {
  c.HTML(http.StatusOK, "users/index.html", gin.H{
  	"title": "users/index",
  })
  })
  
  r.Run(":8080")
  }
  ```
- 自定义模板函数
- 定义一个不转义相应内容的safe模板函数如下：
  
  ```
  func main() {
  router := gin.Default()
  router.SetFuncMap(template.FuncMap{
  "safe": func(str string) template.HTML{
  	return template.HTML(str)
  },
  })
  router.LoadHTMLFiles("./index.tmpl")
  
  router.GET("/index", func(c *gin.Context) {
  c.HTML(http.StatusOK, "index.tmpl", "<a href='https://liwenzhou.com'>李文周的博客</a>")
  })
  
  router.Run(":8080")
  }
  ```
- 在index.tmpl中使用定义好的safe模板函数：
  
  ```
  <!DOCTYPE html>
  <html lang="zh-CN">
  <head>
    <title>修改模板引擎的标识符</title>
  </head>
  <body>
  <div>{{ . | safe }}</div>
  </body>
  </html>
  ```
- 静态文件处理
- 当我们渲染的HTML文件中引用了静态文件时，我们只需要按照以下方式在渲染页面前调用gin.Static方法即可。
  
  ```
  func main() {
  r := gin.Default()
  r.Static("/static", "./static")
  r.LoadHTMLGlob("templates/**/*")
   // ...
  r.Run(":8080")
  }
  ```
- ### 2.2.6.  渲染JSON
- Go gin框架渲染各种数据
  
  ```
  package main
  
  import (
  "github.com/gin-gonic/gin"
  "github.com/gin-gonic/gin/testdata/protoexample"
  "net/http"
  )
  
  func main(){
  r := gin.Default()
  // 1. JSON渲染
  r.GET("/someJSON", func(c *gin.Context) {
  // 方式一：自己拼接JSON
  c.JSON(http.StatusOK, gin.H{"message": "Hello world!"})
  })
  
  // 2. JSON结构体渲染
  r.GET("/moreJSON", func(c *gin.Context) {
  // 方法二：使用结构体
  var msg struct {
  	// 可以通过结构体的注释来实现JSON的小写
  	Name    string `json:"user"`
  	Message string
  	Age     int
  }
  msg.Name = "小王子"
  msg.Message = "Hello world!"
  msg.Age = 18
  c.JSON(http.StatusOK, msg)
  })
  
  // 3. XML渲染
  r.GET("/someXML", func(c *gin.Context) {
  c.XML(http.StatusOK, gin.H{"message": "Hello world!"})
  })
  
  // 4. XML结构体渲染
  r.GET("/moreXML", func(c *gin.Context) {
  // 方法二：使用结构体
  type MessageRecord struct {
  	Name    string
  	Message string
  	Age     int
  }
  var msg MessageRecord
  msg.Name = "小王子"
  msg.Message = "Hello world!"
  msg.Age = 18
  c.XML(http.StatusOK, msg)
  })
  
  // 5.YAML渲染
  r.GET("/someYAML", func(c *gin.Context) {
  c.YAML(http.StatusOK, gin.H{"message": "ok", "status": http.StatusOK})
  })
  
  // 6.protobuf渲染
  r.GET("/someProtoBuf", func(c *gin.Context) {
  reps := []int64{int64(1), int64(2)}
  label := "test"
  // protobuf 的具体定义写在 testdata/protoexample 文件中。
  data := &protoexample.Test{
  	Label: &label,
  	Reps:  reps,
  }
  // 请注意，数据在响应中变为二进制数据
  // 将输出被 protoexample.Test protobuf 序列化了的数据
  c.ProtoBuf(http.StatusOK, data)
  })
  
  r.Run(":8080")
  }
  ```
- 继承模板
- Gin框架默认都是使用单模板，如果需要使用block template功能，可以通过"github.com/gin-contrib/multitemplate"库实现，具体示例如下：
- 首先，假设我们项目目录下的templates文件夹下有以下模板文件，其中home.tmpl和index.tmpl继承了base.tmpl：
  
  ```
  templates
  ├── includes
  │   ├── home.tmpl
  │   └── index.tmpl
  ├── layouts
  │   └── base.tmpl
  └── scripts.tmpl
  ```
- 然后我们定义一个loadTemplates函数如下：
  
  ```
  func loadTemplates(templatesDir string) multitemplate.Renderer {
  r := multitemplate.NewRenderer()
  layouts, err := filepath.Glob(templatesDir + "/layouts/*.tmpl")
  if err != nil {
  panic(err.Error())
  }
  includes, err := filepath.Glob(templatesDir + "/includes/*.tmpl")
  if err != nil {
  panic(err.Error())
  }
  // 为layouts/和includes/目录生成 templates map
  for _, include := range includes {
  layoutCopy := make([]string, len(layouts))
  copy(layoutCopy, layouts)
  files := append(layoutCopy, include)
  r.AddFromFiles(filepath.Base(include), files...)
  }
  return r
  }
  ```
- 我们在main函数中
  
  ```
  func indexFunc(c *gin.Context){
  c.HTML(http.StatusOK, "index.tmpl", nil)
  }
  
  func homeFunc(c *gin.Context){
  c.HTML(http.StatusOK, "home.tmpl", nil)
  }
  
  func main(){
  r := gin.Default()
  r.HTMLRender = loadTemplates("./templates")
  r.GET("/index", indexFunc)
  r.GET("/home", homeFunc)
  r.Run()
  }
  ```
- ### 2.2.7.  获取参数
- 获取StringQuery参数
- querystring指的是URL中?后面携带的参数，例如：/user/search?username=小王子&address=沙河。 获取请求的querystring参数的方法如下：
  
  ```
  r.GET("/user/info", func(context *gin.Context) {
  username:=context.Query("username")
  age:=context.DefaultQuery("age","20")
  sex,ok :=context.GetQuery("sex")
  if ok!=false{
  	sex=context.DefaultQuery("sex","男")
  }
  
  //输出json结果给调用方
  context.JSON(http.StatusOK, gin.H{
  	"message":  "ok",
  	"username": username,
  	"age":  age,
  	"sex":  sex,
  })
  })
  ```
- 获取form表单参数
- 当前端请求的数据通过form表单提交时，例如向/user/search发送一个POST请求，获取请求数据的方式如下：
  
  ```
  r.POST("/user/search", func(c *gin.Context) {
  // DefaultPostForm取不到值时会返回指定的默认值
  //username := c.DefaultPostForm("username", "小王子")
  username := c.PostForm("username")
  address := c.PostForm("address")
  //输出json结果给调用方
  c.JSON(http.StatusOK, gin.H{
  	"message":  "ok",
  	"username": username,
  	"address":  address,
  })
  })
  ```
- 获取json参数
- 当前端请求的数据通过JSON提交时，例如向/json发送一个POST请求，则获取请求参数的方式如下：
  
  ```
  r.POST("/json", func(c *gin.Context) {
  // 注意：下面为了举例子方便，暂时忽略了错误处理
  b, _ := c.GetRawData()  // 从c.Request.Body读取请求数据
  // 定义map或结构体
  var m map[string]interface{}
  // 反序列化
  _ = json.Unmarshal(b, &m)
  
  c.JSON(http.StatusOK, m)
  })
  ```
- 获取path参数
- 请求的参数通过URL路径传递，例如：/user/search/小王子/沙河。 获取请求URL路径中的参数的方式如下。
  
  ```
  func main() {
  //Default返回一个默认的路由引擎
  r := gin.Default()
  r.GET("/user/search/:username/:address", func(c *gin.Context) {
  username := c.Param("username")
  address := c.Param("address")
  //输出json结果给调用方
  c.JSON(http.StatusOK, gin.H{
  	"message":  "ok",
  	"username": username,
  	"address":  address,
  })
  })
  
  r.Run(":8080")
  }
  ```
- 参数绑定
- 为了能够更方便的获取请求相关参数，提高开发效率，我们可以基于请求的Content-Type识别请求数据类型并利用反射机制自动提取请求中QueryString、form表单、JSON、XML等参数到结构体中。 下面的示例代码演示了.ShouldBind()强大的功能，它能够基于请求自动提取JSON、form表单和QueryString类型的数据，并把值绑定到指定的结构体对象。
- ShouldBind会按照下面的顺序解析请求中的数据完成绑定：
	- 如果是 GET 请求，只使用 Form 绑定引擎（query）。
	- 如果是 POST 请求，首先检查 content-type 是否为 JSON 或 XML，然后再使用 Form（form-data）。
	  
	  ```
	  // Binding from JSON
	  type Login struct {
	  User     string `form:"user" json:"user" binding:"required"`
	  Password string `form:"password" json:"password" binding:"required"`
	  }
	  
	  func main() {
	  router := gin.Default()
	  
	  // 绑定JSON的示例 ({"user": "q1mi", "password": "123456"})
	  router.POST("/loginJSON", func(c *gin.Context) {
	  var login Login
	  
	  if err := c.ShouldBind(&login); err == nil {
	  fmt.Printf("login info:%#v\n", login)
	  c.JSON(http.StatusOK, gin.H{
	  	"user":     login.User,
	  	"password": login.Password,
	  })
	  } else {
	  c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	  }
	  })
	  
	  // 绑定form表单示例 (user=q1mi&password=123456)
	  router.POST("/loginForm", func(c *gin.Context) {
	  var login Login
	  // ShouldBind()会根据请求的Content-Type自行选择绑定器
	  if err := c.ShouldBind(&login); err == nil {
	  c.JSON(http.StatusOK, gin.H{
	  	"user":     login.User,
	  	"password": login.Password,
	  })
	  } else {
	  c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	  }
	  })
	  
	  // 绑定QueryString示例 (/loginQuery?user=q1mi&password=123456)
	  router.GET("/loginForm", func(c *gin.Context) {
	  var login Login
	  // ShouldBind()会根据请求的Content-Type自行选择绑定器
	  if err := c.ShouldBind(&login); err == nil {
	  c.JSON(http.StatusOK, gin.H{
	  	"user":     login.User,
	  	"password": login.Password,
	  })
	  } else {
	  c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
	  }
	  })
	  
	  // Listen and serve on 0.0.0.0:8080
	  router.Run(":8080")
	  }
	  ```
- ### 2.2.8.  上传文件示例
- HTML
  
  ```
  <html>
  <head>
    <title>SayHello</title>
  </head>
  <body>
  
  <form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file">
    <input type="submit" value="上传">
  </form>
  </body>
  </html>
  ```
- 代码部分
  
  ```
  package main
  
  import (
  "fmt"
  "github.com/gin-gonic/gin"
  "net/http"
  )
  
  func main(){
  r := gin.Default()
  
  r.LoadHTMLFiles("src/GoLearn/src/index.tmpl")
  // 	普通页面
  r.GET("/index", func(context *gin.Context) {
  context.HTML(http.StatusOK,"index.html",nil)
  })
  // 处理上传文件的请求
  r.POST("/upload", func(context *gin.Context) {
  // 从请求中读出文件
  f,err:=context.FormFile("file")
  // 保存到本地
  if err!=nil{
  	context.JSON(http.StatusOK,gin.H{
  		"error":err.Error(),
  	})
  }else{
  	dst:=fmt.Sprintf("./%s",f.Filename)
  	context.SaveUploadedFile(f,dst)
  }
  })
  
  r.Run(":8080")
  }
  ```
- 单个文件上传
  
  ```
  func main() {
  router := gin.Default()
  // 处理multipart forms提交文件时默认的内存限制是32 MiB
  // 可以通过下面的方式修改
  // router.MaxMultipartMemory = 8 << 20  // 8 MiB
  router.POST("/upload", func(c *gin.Context) {
  // 单个文件
  file, err := c.FormFile("f1")
  if err != nil {
  	c.JSON(http.StatusInternalServerError, gin.H{
  		"message": err.Error(),
  	})
  	return
  }
  
  log.Println(file.Filename)
  dst := fmt.Sprintf("C:/tmp/%s", file.Filename)
  // 上传文件到指定的目录
  c.SaveUploadedFile(file, dst)
  c.JSON(http.StatusOK, gin.H{
  	"message": fmt.Sprintf("'%s' uploaded!", file.Filename),
  })
  })
  router.Run()
  }
  ```
- 多个文件上传
  
  ```
  func main() {
  router := gin.Default()
  // 处理multipart forms提交文件时默认的内存限制是32 MiB
  // 可以通过下面的方式修改
  // router.MaxMultipartMemory = 8 << 20  // 8 MiB
  router.POST("/upload", func(c *gin.Context) {
  // Multipart form
  form, _ := c.MultipartForm()
  files := form.File["file"]
  
  for index, file := range files {
  	log.Println(file.Filename)
  	dst := fmt.Sprintf("C:/tmp/%s_%d", file.Filename, index)
  	// 上传文件到指定的目录
  	c.SaveUploadedFile(file, dst)
  }
  c.JSON(http.StatusOK, gin.H{
  	"message": fmt.Sprintf("%d files uploaded!", len(files)),
  })
  })
  router.Run()
  }
  ```
- ### 2.2.9.  路由及重定向
- HTTP重定向
  
  ```
  r.GET("/test", func(c *gin.Context) {
  c.Redirect(http.StatusMovedPermanently, "http://www.sogo.com/")
  })
  ```
- 路由重定向，使用HandleContext：
  
  ```
  r.GET("/test", func(c *gin.Context) {
    // 指定重定向的URL
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
  })
  r.GET("/test2", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"hello": "world"})
  })
  ```
- 普通路由
  
  ```
  r.GET("/index", func(c *gin.Context) {...})
  r.GET("/login", func(c *gin.Context) {...})
  r.POST("/login", func(c *gin.Context) {...})
  ```
	- 此外，还有一个可以匹配所有的路由
	  
	  ```
	  r.Any("/test", func(c *gin.Context) {...})
	  ```
	- 为没有配置处理函数二等路由添加程序，默认之下，会返回404代码下面的代码为没有匹配到路由的请求都返回views/404.html页面。
	  
	  ```
	  r.NoRoute(func(c *gin.Context) {
	  c.HTML(http.StatusNotFound, "views/404.html", nil)
	  })
	  ```
- 路由组
	- 我们可以将拥有共同URL前缀的路由划分为一个路由组。习惯性一对{}包裹同组的路由，这只是为了看着清晰，你用不用{}包裹功能上没什么区别。
	  
	  ```
	  func main() {
	  r := gin.Default()
	  userGroup := r.Group("/user")
	  {
	  userGroup.GET("/index", func(c *gin.Context) {...})
	  userGroup.GET("/login", func(c *gin.Context) {...})
	  userGroup.POST("/login", func(c *gin.Context) {...})
	  
	  }
	  shopGroup := r.Group("/shop")
	  {
	  shopGroup.GET("/index", func(c *gin.Context) {...})
	  shopGroup.GET("/cart", func(c *gin.Context) {...})
	  shopGroup.POST("/checkout", func(c *gin.Context) {...})
	  }
	  r.Run()
	  }
	  ```
	- 路由组也是支持嵌套的，例如：
	  
	  ```
	  shopGroup := r.Group("/shop")
	  {
	  shopGroup.GET("/index", func(c *gin.Context) {...})
	  shopGroup.GET("/cart", func(c *gin.Context) {...})
	  shopGroup.POST("/checkout", func(c *gin.Context) {...})
	  // 嵌套路由组
	  xx := shopGroup.Group("xx")
	  xx.GET("/oo", func(c *gin.Context) {...})
	  }
	  ```
	- 通常我们将路由分组用在划分业务逻辑或划分API版本时。
- 路由原理
	- Gin框架中的路由使用的是httprouter这个库。其基本原理就是构造一个路由地址的前缀树。
- ### 2.2.10.  中间件
- Gin框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如登录认证、权限校验、数据分页、记录日志、耗时统计等。
- 定义中间件
	- Gin中的中间件必须是一个gin.HandlerFunc类型。例如我们像下面的代码一样定义一个统计请求耗时的中间件。
	  
	  ```
	  // StatCost 是一个统计耗时请求耗时的中间件
	  func StatCost() gin.HandlerFunc {
	  return func(c *gin.Context) {
	  start := time.Now()
	  c.Set("name", "小王子") // 可以通过c.Set在请求上下文中设置值，后续的处理函数能够取到该值
	  // 调用该请求的剩余处理程序
	  c.Next()
	  // 不调用该请求的剩余处理程序
	  // c.Abort()
	  // 计算耗时
	  cost := time.Since(start)
	  log.Println(cost)
	  }
	  }
	  ```
- 注册中间件
	- 在gin框架中，我们可以为每个路由添加任意数量的中间件。
	- 为全局路由注册
	  
	  ```
	  func main() {
	  // 新建一个没有任何默认中间件的路由
	  r := gin.New()
	  // 注册一个全局中间件
	  r.Use(StatCost())
	  
	  r.GET("/test", func(c *gin.Context) {
	  name := c.MustGet("name").(string) // 从上下文取值
	  log.Println(name)
	  c.JSON(http.StatusOK, gin.H{
	  "message": "Hello world!",
	  })
	  })
	  r.Run()
	  }
	  ```
	- 为某个路由单个注册
	  
	  ```
	  // 给/test2路由单独注册中间件（可注册多个）
	  r.GET("/test2", StatCost(), func(c *gin.Context) {
	  name := c.MustGet("name").(string) // 从上下文取值
	  log.Println(name)
	  c.JSON(http.StatusOK, gin.H{
	  "message": "Hello world!",
	  })
	  })
	  ```
	- 为路由组注册
		- 方法1：
		  
		  ```
		  shopGroup := r.Group("/shop", StatCost())
		  {
		  shopGroup.GET("/index", func(c *gin.Context) {...})
		  ...
		  }
		  ```
		- 方法2：
		  
		  ```
		  shopGroup := r.Group("/shop")
		  shopGroup.Use(StatCost())
		  {
		  shopGroup.GET("/index", func(c *gin.Context) {...})
		  ...
		  }
		  ```
- 中间件的注意事项
	- gin默认中间件
	- gin.Default()默认使用了Logger和Recovery中间件，其中：
		- Logger中间件将日志写入gin.DefaultWriter，即使配置了GIN_MODE=release。
		- Recovery中间件会recover任何panic。如果有panic的话，会写入500响应码。
	- 如果不想使用上面两个默认的中间件，可以使用gin.New()新建一个没有任何默认中间件的路由。
	- gin中间件中使用goroutine
	- 当在中间件或handler中启动新的goroutine时，不能使用原始的上下文（c *gin.Context），必须使用其只读副本（c.Copy()）。
- 示例
  
  ```
  package main
  
  import (
  "log"
  "net/http"
  "time"
  
  "github.com/gin-gonic/gin"
  "golang.org/x/sync/errgroup"
  )
  
  var (
  g errgroup.Group
  )
  
  func router01() http.Handler {
  e := gin.New()
  e.Use(gin.Recovery())
  e.GET("/", func(c *gin.Context) {
  c.JSON(
  	http.StatusOK,
  	gin.H{
  		"code":  http.StatusOK,
  		"error": "Welcome server 01",
  	},
  )
  })
  
  return e
  }
  
  func router02() http.Handler {
  e := gin.New()
  e.Use(gin.Recovery())
  e.GET("/", func(c *gin.Context) {
  c.JSON(
  	http.StatusOK,
  	gin.H{
  		"code":  http.StatusOK,
  		"error": "Welcome server 02",
  	},
  )
  })
  
  return e
  }
  
  func main() {
  server01 := &http.Server{
  Addr:         ":8080",
  Handler:      router01(),
  ReadTimeout:  5 * time.Second,
  WriteTimeout: 10 * time.Second,
  }
  
  server02 := &http.Server{
  Addr:         ":8081",
  Handler:      router02(),
  ReadTimeout:  5 * time.Second,
  WriteTimeout: 10 * time.Second,
  }
   // 借助errgroup.Group或者自行开启两个goroutine分别启动两个服务
  g.Go(func() error {
  return server01.ListenAndServe()
  })
  
  g.Go(func() error {
  return server02.ListenAndServe()
  })
  
  if err := g.Wait(); err != nil {
  log.Fatal(err)
  }
  }
  ```
- ## 2.3.  Go Redis
- Redis支持诸如字符串（strings）、哈希（hashes）、列表（lists）、集合（sets）、带范围查询的排序集合（sorted sets）、位图（bitmaps）、hyperloglogs、带半径查询和流的地理空间索引等数据结构（geospatial indexes）。
- Redis的应用场景
	- 缓存系统，减轻主数据库（MySQL）的压力。
	- 计数场景，比如微博、抖音中的关注数和粉丝数。
	- 热门排行榜，需要排序的场景特别适合使用ZSET。
	- 利用LIST可以实现队列的功能。
- ### 2.3.1.  初识Go Redis
- 安装
	- go get -u github.com/go-redis/redis
- 普通连接方式
  
  ```
  // 声明一个全局的rdb变量
  var rdb *redis.Client
  
  // 初始化连接
  func initClient() (err error) {
  rdb = redis.NewClient(&redis.Options{
  Addr:     "localhost:6379",
  Password: "", // no password set
  DB:       0,  // use default DB
  })
  
  _, err = rdb.Ping().Result()
  if err != nil {
  return err
  }
  return nil
  }
  ```
- V8新版本的连接方式
  
  ```
  package main
  
  import (
  "context"
  "fmt"
  "time"
  
  "github.com/go-redis/redis/v8" // 注意导入的是新版本
  )
  
  var (
  rdb *redis.Client
  )
  
  // 初始化连接
  func initClient() (err error) {
  rdb = redis.NewClient(&redis.Options{
  Addr:     "localhost:16379",
  Password: "",  // no password set
  DB:       0,   // use default DB
  PoolSize: 100, // 连接池大小
  })
  
  ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
  defer cancel()
  
  _, err = rdb.Ping(ctx).Result()
  return err
  }
  
  func V8Example() {
  ctx := context.Background()
  if err := initClient(); err != nil {
  return
  }
  
  err := rdb.Set(ctx, "key", "value", 0).Err()
  if err != nil {
  panic(err)
  }
  
  val, err := rdb.Get(ctx, "key").Result()
  if err != nil {
  panic(err)
  }
  fmt.Println("key", val)
  
  val2, err := rdb.Get(ctx, "key2").Result()
  if err == redis.Nil {
  fmt.Println("key2 does not exist")
  } else if err != nil {
  panic(err)
  } else {
  fmt.Println("key2", val2)
  }
  // Output: key value
  // key2 does not exist
  }
  ```
- 连接Redis哨兵模式
  
  ```
  func initClient()(err error){
  rdb := redis.NewFailoverClient(&redis.FailoverOptions{
  MasterName:    "master",
  SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
  })
  _, err = rdb.Ping().Result()
  if err != nil {
  return err
  }
  return nil
  }
  ```
- 连接Redis集群
  
  ```
  func initClient()(err error){
  rdb := redis.NewClusterClient(&redis.ClusterOptions{
  Addrs: []string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
  })
  _, err = rdb.Ping().Result()
  if err != nil {
  return err
  }
  return nil
  }
  ```
- ### 2.3.2.  基本使用方式
- set/get使用
  
  ```
  func redisExample() {
  err := rdb.Set("score", 100, 0).Err()
  if err != nil {
  fmt.Printf("set score failed, err:%v\n", err)
  return
  }
  
  val, err := rdb.Get("score").Result()
  if err != nil {
  fmt.Printf("get score failed, err:%v\n", err)
  return
  }
  fmt.Println("score", val)
  
  val2, err := rdb.Get("name").Result()
  if err == redis.Nil {
  fmt.Println("name does not exist")
  } else if err != nil {
  fmt.Printf("get name failed, err:%v\n", err)
  return
  } else {
  fmt.Println("name", val2)
  }
  }
  ```
- zset示例
  
  ```
  func redisExample2() {
  zsetKey := "language_rank"
  languages := []redis.Z{
  redis.Z{Score: 90.0, Member: "Golang"},
  redis.Z{Score: 98.0, Member: "Java"},
  redis.Z{Score: 95.0, Member: "Python"},
  redis.Z{Score: 97.0, Member: "JavaScript"},
  redis.Z{Score: 99.0, Member: "C/C++"},
  }
  // ZADD
  num, err := rdb.ZAdd(zsetKey, languages...).Result()
  if err != nil {
  fmt.Printf("zadd failed, err:%v\n", err)
  return
  }
  fmt.Printf("zadd %d succ.\n", num)
  
  // 把Golang的分数加10
  newScore, err := rdb.ZIncrBy(zsetKey, 10.0, "Golang").Result()
  if err != nil {
  fmt.Printf("zincrby failed, err:%v\n", err)
  return
  }
  fmt.Printf("Golang's score is %f now.\n", newScore)
  
  // 取分数最高的3个
  ret, err := rdb.ZRevRangeWithScores(zsetKey, 0, 2).Result()
  if err != nil {
  fmt.Printf("zrevrange failed, err:%v\n", err)
  return
  }
  for _, z := range ret {
  fmt.Println(z.Member, z.Score)
  }
  
  // 取95~100分的
  op := redis.ZRangeBy{
  Min: "95",
  Max: "100",
  }
  ret, err = rdb.ZRangeByScoreWithScores(zsetKey, op).Result()
  if err != nil {
  fmt.Printf("zrangebyscore failed, err:%v\n", err)
  return
  }
  for _, z := range ret {
  fmt.Println(z.Member, z.Score)
  }
  }
  ```
- 根据前缀获取key
  
  ```
  vals, err := rdb.Keys(ctx, "prefix*").Result()
  ```
- 执行自定义命令
  
  ```
  res, err := rdb.Do(ctx, "set", "key", "value").Result()
  ```
- 按通配符删除key
	- 当通配符匹配的key的数量不多时，可以使用Keys()得到所有的key在使用Del命令删除。 如果key的数量非常多的时候，我们可以搭配使用Scan命令和Del命令完成删除。
	  
	  ```
	  ctx := context.Background()
	  iter := rdb.Scan(ctx, 0, "prefix*", 0).Iterator()
	  for iter.Next(ctx) {
	  err := rdb.Del(ctx, iter.Val()).Err()
	  if err != nil {
	  panic(err)
	  }
	  }
	  if err := iter.Err(); err != nil {
	  panic(err)
	  }
	  ```
- pipeline
	- Pipeline 主要是一种网络优化。它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省了每个命令的网络往返时间（RTT）。
	- Pipeline 基本示例如下：
	  
	  ```
	  pipe := rdb.Pipeline()
	  
	  incr := pipe.Incr("pipeline_counter")
	  pipe.Expire("pipeline_counter", time.Hour)
	  
	  _, err := pipe.Exec()
	  fmt.Println(incr.Val(), err)
	  ```
	- 也可以使用Pipelined：
	  
	  ```
	  var incr *redis.IntCmd
	  _, err := rdb.Pipelined(func(pipe redis.Pipeliner) error {
	  incr = pipe.Incr("pipelined_counter")
	  pipe.Expire("pipelined_counter", time.Hour)
	  return nil
	  })
	  fmt.Println(incr.Val(), err)
	  ```
	- 在某些场景下，当我们有多条命令要执行时，就可以考虑使用pipeline来优化。
- 事务
	- Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，Multi/exec能够确保在multi/exec两个语句之间的命令之间没有其他客户端正在执行命令。
	- 在这种场景我们需要使用TxPipeline。TxPipeline总体上类似于上面的Pipeline，但是它内部会使用MULTI/EXEC包裹排队的命令。例如：
	  
	  ```
	  pipe := rdb.TxPipeline()
	  
	  incr := pipe.Incr("tx_pipeline_counter")
	  pipe.Expire("tx_pipeline_counter", time.Hour)
	  
	  _, err := pipe.Exec()
	  fmt.Println(incr.Val(), err)
	  ```
	- 还有一个与上文类似的TxPipelined方法，使用方法如下：
	  
	  ```
	  var incr *redis.IntCmd
	  _, err := rdb.TxPipelined(func(pipe redis.Pipeliner) error {
	  incr = pipe.Incr("tx_pipelined_counter")
	  pipe.Expire("tx_pipelined_counter", time.Hour)
	  return nil
	  })
	  fmt.Println(incr.Val(), err)
	  ```
- Watch
	- 在某些场景下，我们除了要使用MULTI/EXEC命令外，还需要配合使用WATCH命令。在用户使用WATCH命令监视某个键之后，直到该用户执行EXEC命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行EXEC的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。
	  
	  ```
	  Watch(fn func(*Tx) error, keys ...string) error
	  ```
	- Watch方法接收一个函数和一个或多个key作为参数。基本使用示例如下：
	  
	  ```
	  // 监视watch_count的值，并在值不变的前提下将其值+1
	  key := "watch_count"
	  err = client.Watch(func(tx *redis.Tx) error {
	  n, err := tx.Get(key).Int()
	  if err != nil && err != redis.Nil {
	  return err
	  }
	  _, err = tx.Pipelined(func(pipe redis.Pipeliner) error {
	  pipe.Set(key, n+1, 0)
	  return nil
	  })
	  return err
	  }, key)
	  ```
- ## 2.4.  Go MongDB
- ### 2.4.1.  初识Go MongDB
- mongoDB是目前比较流行的一个基于分布式文件存储的数据库，它是一个介于关系数据库和非关系数据库(NoSQL)之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
- mongoDB中将一条数据存储为一个文档（document），数据结构由键值（key-value）对组成。 其中文档类似于我们平常编程中用到的JSON对象。 文档中的字段值可以包含其他文档，数组及文档数组。
- mongoDB中相关概念与我们熟悉的SQL概念对比如下：
  
  | MongoDB术语/概念 | 说明 | 对比SQL术语/概念 |
  | database | 数据库 | database |
  | collection | 集合 | table |
  | document | 文档 | row |
  | field | 字段 | column |
  | index | index | 索引 |
  | primary key | 主键 MongoDB自动将_id字段设置为主键 | primary key |
- 安装MongDB驱动
	- go get github.com/mongodb/mongo-go-driver
- go代码的示例代码
  
  ```
  package main
  
  import (
  "context"
  "fmt"
  "log"
  
  "go.mongodb.org/mongo-driver/mongo"
  "go.mongodb.org/mongo-driver/mongo/options"
  )
  
  // Split 把字符串s按照给定的分隔符sep进行分割返回字符串切片
  func main() {
  // 设置客户端连接配置
  clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
  
  // 连接到MongoDB
  client, err := mongo.Connect(context.TODO(), clientOptions)
  if err != nil {
  log.Fatal(err)
  }
  
  // 检查连接
  err = client.Ping(context.TODO(), nil)
  if err != nil {
  log.Fatal(err)
  }
  fmt.Println("Connected to MongoDB!")
  
  // 指定获取要操作的数据集
  collection := client.Database("test").Collection("col")
  fmt.Println(collection)
  
  // 断开连接
  err = client.Disconnect(context.TODO())
  if err != nil {
  log.Fatal(err)
  }
  fmt.Println("Connection to MongoDB closed.")
  }
  ```
- 连接池模式
  
  ```
  import (
  "context"
  "time"
  
  "go.mongodb.org/mongo-driver/mongo"
  "go.mongodb.org/mongo-driver/mongo/options"
  )
  
  func ConnectToDB(uri, name string, timeout time.Duration, num uint64) (*mongo.Database, error) {
  ctx, cancel := context.WithTimeout(context.Background(), timeout)
  defer cancel()
  o := options.Client().ApplyURI(uri)
  o.SetMaxPoolSize(num)
  client, err := mongo.Connect(ctx, o)
  if err != nil {
  return nil, err
  }
  
  return client.Database(name), nil
  }
  ```
- ### 2.4.2.  BSON
- MongoDB中的JSON文档存储在名为BSON(二进制编码的JSON)的二进制表示中。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示，使其包含额外的类型，如int、long、date、浮点数和decimal128。这使得应用程序更容易可靠地处理、排序和比较数据。
- 连接MongoDB的Go驱动程序中有两大类型表示BSON数据：D和Raw。
- 类型D家族被用来简洁地构建使用本地Go类型的BSON对象。这对于构造传递给MongoDB的命令特别有用。D家族包括四类:
	- D：一个BSON文档。这种类型应该在顺序重要的情况下使用，比如MongoDB命令。
	- M：一张无序的map。它和D是一样的，只是它不保持顺序。
	- A：一个BSON数组。
	- E：D里面的一个元素。
- 要使用BSON，需要先导入下面的包：
	- import "go.mongodb.org/mongo-driver/bson"
- Raw类型家族用于验证字节切片。你还可以使用Lookup()从原始类型检索单个元素。如果你不想要将BSON反序列化成另一种类型的开销，那么这是非常有用的。
- ### 2.4.3.  CRUD
- 前期准备工作
	- 首先定义一个Student，然后并为之创建一些对象
	  
	  ```
	  type Student struct {
	  Name string
	  Age int
	  }
	  
	  s1 := Student{"小红", 12}
	  s2 := Student{"小兰", 10}
	  s3 := Student{"小黄", 11}
	  ```
- 插入文档
	- 使用collection.InsertOne()方法插入一条文档记录：
	  
	  ```
	  insertResult, err := collection.InsertOne(context.TODO(), s1)
	  if err != nil {
	  log.Fatal(err)
	  }
	  
	  fmt.Println("Inserted a single document: ", insertResult.InsertedID)
	  ```
	- 使用collection.InsertMany()方法插入多条文档记录：
	  
	  ```
	  students := []interface{}{s2, s3}
	  insertManyResult, err := collection.InsertMany(context.TODO(), students)
	  if err != nil {
	  log.Fatal(err)
	  }
	  fmt.Println("Inserted multiple documents: ", insertManyResult.InsertedIDs)
	  ```
- 更新文档
	- updateone()方法允许你更新单个文档。它需要一个筛选器文档来匹配数据库中的文档，并需要一个更新文档来描述更新操作。你可以使用bson.D类型来构建筛选文档和更新文档:
	  
	  ```
	  filter := bson.D{{"name", "小兰"}}
	  
	  update := bson.D{
	  {"$inc", bson.D{
	  {"age", 1},
	  }},
	  }
	  updateResult, err := collection.UpdateOne(context.TODO(), filter, update)
	  if err != nil {
	  log.Fatal(err)
	  }
	  fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
	  ```
- 查询文档
	- 要找到一个文档，你需要一个filter文档，以及一个指向可以将结果解码为其值的指针。要查找单个文档，使用collection.FindOne()。这个方法返回一个可以解码为值的结果。
	- 我们使用上面定义过的那个filter来查找姓名为’小兰’的文档。
	  
	  ```
	  // 创建一个Student变量用来接收查询的结果
	  var result Student
	  err = collection.FindOne(context.TODO(), filter).Decode(&result)
	  if err != nil {
	  log.Fatal(err)
	  }
	  fmt.Printf("Found a single document: %+v\n", result)
	  ```
	- 要查找多个文档，请使用collection.Find()。此方法返回一个游标。游标提供了一个文档流，你可以通过它一次迭代和解码一个文档。当游标用完之后，应该关闭游标。下面的示例将使用options包设置一个限制以便只返回两个文档。
	  
	  ```
	  // 查询多个
	  // 将选项传递给Find()
	  findOptions := options.Find()
	  findOptions.SetLimit(2)
	  
	  // 定义一个切片用来存储查询结果
	  var results []*Student
	  
	  // 把bson.D{{}}作为一个filter来匹配所有文档
	  cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
	  if err != nil {
	  log.Fatal(err)
	  }
	  
	  // 查找多个文档返回一个光标
	  // 遍历游标允许我们一次解码一个文档
	  for cur.Next(context.TODO()) {
	  // 创建一个值，将单个文档解码为该值
	  var elem Student
	  err := cur.Decode(&elem)
	  if err != nil {
	  log.Fatal(err)
	  }
	  results = append(results, &elem)
	  }
	  
	  if err := cur.Err(); err != nil {
	  log.Fatal(err)
	  }
	  
	  // 完成后关闭游标
	  cur.Close(context.TODO())
	  fmt.Printf("Found multiple documents (array of pointers): %#v\n", results)
	  ```
- 删除文档
	- 最后，可以使用collection.DeleteOne()或collection.DeleteMany()删除文档。如果你传递bson.D{{}}作为过滤器参数，它将匹配数据集中的所有文档。还可以使用collection. drop()删除整个数据集。
	  
	  ```
	  // 删除名字是小黄的那个
	  deleteResult1, err := collection.DeleteOne(context.TODO(), bson.D{{"name","小黄"}})
	  if err != nil {
	  log.Fatal(err)
	  }
	  fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult1.DeletedCount)
	  // 删除所有
	  deleteResult2, err := collection.DeleteMany(context.TODO(), bson.D{{}})
	  if err != nil {
	  log.Fatal(err)
	  }
	  fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult2.DeletedCount)
	  ```
- ## 2.5.  Go 单元测试
- ### 2.5.1.  Go Test
- Go语言中的测试依赖go test命令。编写测试代码和编写普通的Go代码过程是类似的，并不需要学习新的语法、规则或工具。
- go test命令是一个按照一定约定和组织的测试代码的驱动程序。在包目录内，所有以_test.go为后缀名的源代码文件都是go test测试的一部分，不会被go build编译到最终的可执行文件中。
- 在*_test.go文件中有三种类型的函数，单元测试函数、基准测试函数和示例函数。
  
  | 类型 | 格式 | 作用 |
  | 测试函数 | 函数名前缀为Test | 测试程序的一些逻辑行为是否正确 |
  | 基准函数 | 函数名前缀为Benchmark | 测试函数的性能 |
  | 示例函数 | 函数名前缀为Example | 为文档提供示例文档 |
- go test命令会遍历所有的*_test.go文件中符合上述命名规则的函数，然后生成一个临时的main包用于调用相应的测试函数，然后构建并运行、报告测试结果，最后清理测试中生成的临时文件。
- 每个测试函数必须导入testing包，测试函数的基本格式（签名）如下：
  
  ```
  func TestName(t *testing.T){
    // ...
  }
  ```
- 测试函数的名字必须以Test开头，可选的后缀名必须以大写字母开头，举几个例子：
  
  ```
  func TestAdd(t *testing.T){ ... }
  func TestSum(t *testing.T){ ... }
  func TestLog(t *testing.T){ ... }
  ```
- 其中参数t用于报告测试失败和附加的日志信息。 testing.T的拥有的方法如下：
  
  ```
  func (c *T) Cleanup(func())
  func (c *T) Error(args ...interface{})
  func (c *T) Errorf(format string, args ...interface{})
  func (c *T) Fail()
  func (c *T) FailNow()
  func (c *T) Failed() bool
  func (c *T) Fatal(args ...interface{})
  func (c *T) Fatalf(format string, args ...interface{})
  func (c *T) Helper()
  func (c *T) Log(args ...interface{})
  func (c *T) Logf(format string, args ...interface{})
  func (c *T) Name() string
  func (c *T) Skip(args ...interface{})
  func (c *T) SkipNow()
  func (c *T) Skipf(format string, args ...interface{})
  func (c *T) Skipped() bool
  func (c *T) TempDir() string
  ```
- Go test的例子
- 就像细胞是构成我们身体的基本单位，一个软件程序也是由很多单元组件构成的。单元组件可以是函数、结构体、方法和最终用户可能依赖的任意东西。总之我们需要确保这些组件是能够正常运行的。单元测试是一些利用各种方法测试单元组件的程序，它会将结果与预期输出进行比较。
- 接下来，我们在base_demo包中定义了一个Split函数，具体实现如下：
  
  ```
  package day01
  
  import "strings"
  
  // Split 把字符串s按照给定的分隔符sep进行分割返回字符串切片
  func Split(s, sep string) (result []string) {
  i := strings.Index(s, sep)
  
  for i > -1 {
  result = append(result, s[:i])
  s = s[i+1:]
  i = strings.Index(s, sep)
  }
  result = append(result, s)
  return
  }
  ```
- 在当前目录下，我们创建一个split_test.go的测试文件，并定义一个测试函数如下：
  
  ```
  package day01
  
  import (
  "reflect"
  "testing"
  )
  
  func TestSplit(t *testing.T) { // 测试函数名必须以Test开头，必须接收一个*testing.T类型参数
  got := Split("a:b:c", ":")         // 程序输出的结果
  want := []string{"a", "b", "c"}    // 期望的结果
  if !reflect.DeepEqual(want, got) { // 因为slice不能比较直接，借助反射包中的方法比较
  t.Errorf("expected:%v, got:%v", want, got) // 测试失败输出错误提示
  }
  }
  ```
- ### 2.5.2.  GoConvey
- GoConvey是一个非常非常好用的Go测试框架，它直接与go test集成，提供了很多丰富的断言函数，能够在终端输出可读的彩色测试结果，并且还支持全自动的Web UI。
- 安装
	- go get github.com/smartystreets/goconvey
- 使用示例
	- 我们使用goconvey来为最开始的基础示例中的Split函数编写单元测试。Split函数如下：
	  
	  ```
	  // split.go
	  
	  func Split(s, sep string) (result []string) {
	  result = make([]string, 0, strings.Count(s, sep)+1)
	  i := strings.Index(s, sep)
	  for i > -1 {
	  result = append(result, s[:i])
	  s = s[i+len(sep):]
	  i = strings.Index(s, sep)
	  }
	  result = append(result, s)
	  return
	  }
	  ```
	- 单元测试文件内容：
	  
	  ```
	  // split_test.go
	  
	  import (
	  "testing"
	  
	  c "github.com/smartystreets/goconvey/convey"  // 别名导入
	  )
	  
	  func TestSplit(t *testing.T) {
	  c.Convey("基础用例", t, func() {
	  var (
	  s      = "a:b:c"
	  sep    = ":"
	  expect = []string{"a", "b", "c"}
	  )
	  got := Split(s, sep)
	  c.So(got, c.ShouldResemble, expect) // 断言
	  })
	  
	  c.Convey("不包含分隔符用例", t, func() {
	  var (
	  s      = "a:b:c"
	  sep    = "|"
	  expect = []string{"a:b:c"}
	  )
	  got := Split(s, sep)
	  c.So(got, c.ShouldResemble, expect) // 断言
	  })
	  }
	  ```
	- 命令行执行单元测试，会在终端输出可读性非常好的彩色结果。
	  
	  ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1636328514983-f1b1f177-3d7c-4ba8-8231-ead81385112a.png)
	- goconvey还支持在单元测试中根据需要嵌套调用，比如：
	  
	  ```
	  func TestSplit(t *testing.T) {
	  // ...
	  
	  // 只需要在顶层的Convey调用时传入t
	  c.Convey("分隔符在开头或结尾用例", t, func() {
	  tt := []struct {
	  name   string
	  s      string
	  sep    string
	  expect []string
	  }{
	  {"分隔符在开头", "*1*2*3", "*", []string{"", "1", "2", "3"}},
	  {"分隔符在结尾", "1+2+3+", "+", []string{"1", "2", "3", ""}},
	  }
	  for _, tc := range tt {
	  c.Convey(tc.name, func() { // 嵌套调用Convey
	  	got := Split(tc.s, tc.sep)
	  	c.So(got, c.ShouldResemble, tc.expect)
	  })
	  }
	  })
	  }
	  ```
- 断言方法
	- GoConvey为我们提供了很多种类断言方法在So()函数中使用。
	- 一般相等类
	  
	  ```
	  So(thing1, ShouldEqual, thing2)
	  So(thing1, ShouldNotEqual, thing2)
	  So(thing1, ShouldResemble, thing2)		// 用于数组、切片、map和结构体相等
	  So(thing1, ShouldNotResemble, thing2)
	  So(thing1, ShouldPointTo, thing2)
	  So(thing1, ShouldNotPointTo, thing2)
	  So(thing1, ShouldBeNil)
	  So(thing1, ShouldNotBeNil)
	  So(thing1, ShouldBeTrue)
	  So(thing1, ShouldBeFalse)
	  So(thing1, ShouldBeZeroValue)
	  ```
	- 数字数量比较类
	  
	  ```
	  So(1, ShouldBeGreaterThan, 0)
	  So(1, ShouldBeGreaterThanOrEqualTo, 0)
	  So(1, ShouldBeLessThan, 2)
	  So(1, ShouldBeLessThanOrEqualTo, 2)
	  So(1.1, ShouldBeBetween, .8, 1.2)
	  So(1.1, ShouldNotBeBetween, 2, 3)
	  So(1.1, ShouldBeBetweenOrEqual, .9, 1.1)
	  So(1.1, ShouldNotBeBetweenOrEqual, 1000, 2000)
	  So(1.0, ShouldAlmostEqual, 0.99999999, .0001)   // tolerance is optional; default 0.0000000001
	  So(1.0, ShouldNotAlmostEqual, 0.9, .0001)
	  ```
	- 包含类
	  
	  ```
	  So([]int{2, 4, 6}, ShouldContain, 4)
	  So([]int{2, 4, 6}, ShouldNotContain, 5)
	  So(4, ShouldBeIn, ...[]int{2, 4, 6})
	  So(4, ShouldNotBeIn, ...[]int{1, 3, 5})
	  So([]int{}, ShouldBeEmpty)
	  So([]int{1}, ShouldNotBeEmpty)
	  So(map[string]string{"a": "b"}, ShouldContainKey, "a")
	  So(map[string]string{"a": "b"}, ShouldNotContainKey, "b")
	  So(map[string]string{"a": "b"}, ShouldNotBeEmpty)
	  So(map[string]string{}, ShouldBeEmpty)
	  So(map[string]string{"a": "b"}, ShouldHaveLength, 1) // supports map, slice, chan, and string
	  ```
	- 字符串类
	  
	  ```
	  So("asdf", ShouldStartWith, "as")
	  So("asdf", ShouldNotStartWith, "df")
	  So("asdf", ShouldEndWith, "df")
	  So("asdf", ShouldNotEndWith, "df")
	  So("asdf", ShouldContainSubstring, "稍等一下")		// optional 'expected occurences' arguments?
	  So("asdf", ShouldNotContainSubstring, "er")
	  So("adsf", ShouldBeBlank)
	  So("asdf", ShouldNotBeBlank)
	  ```
	- panic类
	  
	  ```
	  So(func(), ShouldPanic)
	  So(func(), ShouldNotPanic)
	  So(func(), ShouldPanicWith, "")		// or errors.New("something")
	  So(func(), ShouldNotPanicWith, "")	// or errors.New("something")
	  ```
	- 类型检查类
	  
	  ```
	  So(1, ShouldHaveSameTypeAs, 0)
	  So(1, ShouldNotHaveSameTypeAs, "asdf")
	  ```
	- 时间和时间间隔类
	  
	  ```
	  So(time.Now(), ShouldHappenBefore, time.Now())
	  So(time.Now(), ShouldHappenOnOrBefore, time.Now())
	  So(time.Now(), ShouldHappenAfter, time.Now())
	  So(time.Now(), ShouldHappenOnOrAfter, time.Now())
	  So(time.Now(), ShouldHappenBetween, time.Now(), time.Now())
	  So(time.Now(), ShouldHappenOnOrBetween, time.Now(), time.Now())
	  So(time.Now(), ShouldNotHappenOnOrBetween, time.Now(), time.Now())
	  So(time.Now(), ShouldHappenWithin, duration, time.Now())
	  So(time.Now(), ShouldNotHappenWithin, duration, time.Now())
	  ```
	- 自定义断言方法
	  
	  ```
	  func should<do-something>(actual interface{}, expected ...interface{}) string {
	   if <some-important-condition-is-met(actual, expected)> {
	       return ""   // 返回空字符串表示断言通过
	   }
	   return "<一些描述性消息详细说明断言失败的原因...>"
	  }
	  ```
- ### 2.5.3.  Go monkey
- [monkey](https://github.com/bouk/monkey)是一个Go单元测试中十分常用的打桩工具，它在运行时通过汇编语言重写可执行文件，将目标函数或方法的实现跳转到桩实现，其原理类似于热补丁。
- monkey库很强大，但是使用时需注意以下事项：
	- monkey不支持内联函数，在测试的时候需要通过命令行参数-gcflags=-l关闭Go语言的内联优化。
	- monkey不是线程安全的，所以不要把它用到并发的单元测试中。
- 安装
	- go get bou.ke/monkey
- 使用示例
	- 假设你们公司中台提供了一个用户中心的库varys，使用这个库可以很方便的根据uid获取用户相关信息。但是当你编写代码的时候这个库还没实现，或者这个库要经过内网请求但你现在没这能力，这个时候要为MyFunc编写单元测试，就需要做一些mock工作。
	  
	  ```
	  // func.go
	  
	  func MyFunc(uid int64)string{
	  u, err := varys.GetInfoByUID(uid)
	  if err != nil {
	  return "welcome"
	  }
	  
	  // 这里是一些逻辑代码...
	  
	  return fmt.Sprintf("hello %s\n", u.Name)
	  }
	  ```
	- 我们使用monkey库对varys.GetInfoByUID进行打桩。
	  
	  ```
	  // func_test.go
	  
	  func TestMyFunc(t *testing.T) {
	  // 对 varys.GetInfoByUID 进行打桩
	  // 无论传入的uid是多少，都返回 &varys.UserInfo{Name: "liwenzhou"}, nil
	  monkey.Patch(varys.GetInfoByUID, func(int64)(*varys.UserInfo, error) {
	  return &varys.UserInfo{Name: "liwenzhou"}, nil
	  })
	  
	  ret := MyFunc(123)
	  if !strings.Contains(ret, "liwenzhou"){
	  t.Fatal()
	  }
	  }
	  ```
	- 执行单元测试：
	  
	  ```
	  go test -run=TestMyFunc -v -gcflags=-l
	  ```
- 除了对函数进行mock外monkey也支持对方法进行mock。
  
  ```
  // method.go
  
  type User struct {
  Name string
  Birthday string
  }
  
  // CalcAge 计算用户年龄
  func (u *User) CalcAge() int {
  t, err := time.Parse("2006-01-02", u.Birthday)
  if err != nil {
  return -1
  }
  return int(time.Now().Sub(t).Hours()/24.0)/365
  }
  
  // GetInfo 获取用户相关信息
  func (u *User) GetInfo()string{
  age := u.CalcAge()
  if age <= 0 {
  return fmt.Sprintf("%s很神秘，我们还不了解ta。", u.Name)
  }
  return fmt.Sprintf("%s今年%d岁了，ta是我们的朋友。", u.Name, age)
  }
  ```
- 如果我们为GetInfo编写单元测试的时候CalcAge方法的功能还未完成，这个时候我们可以使用monkey进行打桩。
  
  ```
  // method_test.go
  
  func TestUser_GetInfo(t *testing.T) {
  var u = &User{
  Name:     "q1mi",
  Birthday: "1990-12-20",
  }
  
  // 为对象方法打桩
  monkey.PatchInstanceMethod(reflect.TypeOf(u), "CalcAge", func(*User)int {
  return 18
  })
  
  ret := u.GetInfo()  // 内部调用u.CalcAge方法时会返回18
  if !strings.Contains(ret, "朋友"){
  t.Fatal()
  }
  }
  ```
- monkey基本上能满足我们在单元测试中打桩的任何需求。
- 社区中还有一个参考monkey库实现的gomonkey库，原理和使用过程基本相似，这里就不再啰嗦了。
- ### 2.5.4.  Go mock
- 打桩
- 软件测试中的打桩是指用一些代码（桩stub）代替目标代码，通常用来屏蔽或补齐业务逻辑中的关键代码方便进行单元测试。主要的用途：
	- 屏蔽：不想在单元测试用引入数据库连接等重资源
	- 补齐：依赖的上下游函数或方法还未实现
- 上面代码中就用到了打桩，当传入Get函数的参数为liwenzhou.com时就返回1, nil的返回值。
- gomock支持针对参数、返回值、调用次数、调用顺序等进行打桩操作。
- **参数**
	- 参数相关的用法有： - gomock.Eq(value)：表示一个等价于value值的参数 - gomock.Not(value)：表示一个非value值的参数 - gomock.Any()：表示任意值的参数 - gomock.Nil()：表示空值的参数 - SetArg(n, value)：设置第n（从0开始）个参数的值，通常用于指针参数或切片
	- 具体示例如下：
	  
	  ```
	  m.EXPECT().Get(gomock.Not("q1mi")).Return(10, nil)
	  m.EXPECT().Get(gomock.Any()).Return(20, nil)
	  m.EXPECT().Get(gomock.Nil()).Return(-1, nil)
	  ```
	- 这里单独说一下SetArg的适用场景，假设你有一个需要mock的接口如下：
	  
	  ```
	  type YourInterface {
	  SetValue(arg *int)
	  }
	  ```
	- 此时，打桩的时候就可以使用SetArg来修改参数的值。
	  
	  ```
	  m.EXPECT().SetValue(gomock.Any()).SetArg(0, 7)  // 将SetValue的第一个参数设置为7
	  ```
- 返回值
	- gomock中跟返回值相关的用法有以下几个：
		- Return()：返回指定值
		- Do(func)：执行操作，忽略返回值
		- DoAndReturn(func)：执行并返回指定值
	- 例子
	  
	  ```
	  m.EXPECT().Get(gomock.Any()).Return(20, nil)
	  m.EXPECT().Get(gomock.Any()).Do(func(key string) {
	  t.Logf("input key is %v\n", key)
	  })
	  m.EXPECT().Get(gomock.Any()).DoAndReturn(func(key string)(int, error) {
	  t.Logf("input key is %v\n", key)
	  return 10, nil
	  })
	  ```
	- 调用次数
		- 使用gomock工具mock的方法都会有期望被调用的次数，默认每个mock方法只允许被调用一次。
		- gomock为我们提供了如下方法设置期望被调用的次数。
			- Times() 断言 Mock 方法被调用的次数。
			- MaxTimes() 最大次数。
			- MinTimes() 最小次数。
			- AnyTimes() 任意次数（包括 0 次）。
			  
			  ```
			  m.
			  EXPECT().
			  Get(gomock.Eq("liwenzhou.com")). // 参数
			  Return(1, nil).                  // 返回值
			  Times(1)                         // 设置Get方法期望被调用次数为1
			  
			  // 调用GetFromDB函数时传入上面的mock对象m
			  if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
			  t.Fatal()
			  }
			  // 再次调用上方mock的Get方法时不满足调用次数为1的期望
			  if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
			  t.Fatal()
			  }
			  ```
	- 调用顺序
		- gomock还支持使用InOrder方法指定mock方法的调用顺序：
		  
		  ```
		  // 指定顺序
		  gomock.InOrder(
		  m.EXPECT().Get("1"),
		  m.EXPECT().Get("2"),
		  m.EXPECT().Get("3"),
		  )
		  
		  // 按顺序调用
		  GetFromDB(m, "1")
		  GetFromDB(m, "2")
		  GetFromDB(m, "3")
		  ```
	- 此外知名的Go测试库[testify](https://github.com/stretchr/testify)目前也提供类似的mock工具—testify/mock和mockery。
- gomock是Go官方提供的测试框架，它在内置的testing包或其他环境中都能够很方便的使用。我们使用它对代码中的那些接口类型进行mock，方便编写单元测试。
- 安装gomockgen
	- 首先需要确保你的$GOPATH/bin已经加入到环境变量中。
	- Go版本号<1.16时：
		- GO111MODULE=on go get github.com/golang/mock/mockgen@v1.6.0
	- Go版本>=1.16时：
		- go install github.com/golang/mock/mockgen@v1.6.0
	- 如果是在你的CI流水线中安装，则需要安装与你的CI环境匹配的合适版本。
- 运行mockgen
	- mockgen 有两种操作模式：源码（source）模式和反射（reflect）模式。
	- 源码模式
		- 源码模式根据源文件mock接口。它是通过使用 -source 标志启用。在这个模式下可能有用的其他标志是 -imports 和 -aux_files。
		- mockgen -source=foo.go [other options]
	- 反射模式
		- 反射模式通过构建使用反射来理解接口的程序来mock接口。它是通过传递两个非标志参数来启用的：一个导入路径和一个逗号分隔的符号列表。可以使用 ”.”引用当前路径的包。
		- mockgen database/sql/driver Conn,Driver
		- # Convenient for `go:generate`. mockgen . Conn,Driver
	- flag
		- mockgen 命令用来为给定一个包含要mock的接口的Go源文件，生成mock类源代码。它支持以下标志：
			- -source：包含要mock的接口的文件。
			- -destination：生成的源代码写入的文件。如果不设置此项，代码将打印到标准输出。
			- -package：用于生成的模拟类源代码的包名。如果不设置此项包名默认在原包名前添加mock_前缀。
			- -imports：在生成的源代码中使用的显式导入列表。值为foo=bar/baz形式的逗号分隔的元素列表，其中bar/baz是要导入的包，foo是要在生成的源代码中用于包的标识符。
			- -aux_files：需要参考以解决的附加文件列表，例如在不同文件中定义的嵌入式接口。指定的值应为foo=bar/baz.go形式的以逗号分隔的元素列表，其中bar/baz.go是源文件，foo是-source文件使用的文件的包名。
			- -build_flags：（仅反射模式）一字不差地传递标志给go build
			- -mock_names：生成的模拟的自定义名称列表。这被指定为一个逗号分隔的元素列表，形式为Repository = MockSensorRepository,Endpoint=MockSensorEndpoint，其中Repository是接口名称，mockSensorrepository是所需的mock名称(mock工厂方法和mock记录器将以mock命名)。如果其中一个接口没有指定自定义名称，则将使用默认命名约定。
			- -self_package：生成的代码的完整包导入路径。使用此flag的目的是通过尝试包含自己的包来防止生成代码中的循环导入。如果mock的包被设置为它的一个输入(通常是主输入)，并且输出是stdio，那么mockgen就无法检测到最终的输出包，这种情况就会发生。设置此标志将告诉 mockgen 排除哪个导入
			- -copyright_file：用于将版权标头添加到生成的源代码中的版权文件
			- -debug_parser：仅打印解析器结果
			- -exec_only：（反射模式） 如果设置，则执行此反射程序
			- -prog_only：（反射模式）只生成反射程序；将其写入标准输出并退出。
			- -write_package_comment：如果为true，则写入包文档注释 (godoc)。（默认为true）
- 构建mock
- 这里就以日常开发中经常用到的数据库操作为例，讲解一下如何使用gomock来mock接口的单元测试。
- 假设有查询MySQL数据库的业务代码如下，其中DB是一个自定义的接口类型：
  
  ```
  // db.go
  
  // DB 数据接口
  type DB interface {
  Get(key string)(int, error)
  Add(key string, value int) error
  }
  
  // GetFromDB 根据key从DB查询数据的函数
  func GetFromDB(db DB, key string) int {
  if v, err := db.Get(key);err == nil{
  return v
  }
  return -1
  }
  ```
- 我们现在要为GetFromDB函数编写单元测试代码，可是我们又不能在单元测试过程中连接真实的数据库，这个时候就需要mock DB这个接口来方便进行单元测试。
- 使用上面提到的 mockgen 工具来为生成相应的mock代码。通过执行下面的命令，我们就能在当前项目下生成一个mocks文件夹，里面存放了一个db_mock.go文件。
  
  ```
  mockgen -source=db.go -destination=mocks/db_mock.go -package=mocks
  ```
- db_mock.go文件中的内容就是mock相关接口的代码了。
- 我们通常不需要编辑它，只需要在单元测试中按照规定的方式使用它们就可以了。例如，我们编写TestGetFromDB 函数如下：
  
  ```
  // db_test.go
  
  func TestGetFromDB(t *testing.T) {
  // 创建gomock控制器，用来记录后续的操作信息
  ctrl := gomock.NewController(t)
  // 断言期望的方法都被执行
  // Go1.14+的单测中不再需要手动调用该方法
  defer ctrl.Finish()
  // 调用mockgen生成代码中的NewMockDB方法
  // 这里mocks是我们生成代码时指定的package名称
  m := mocks.NewMockDB(ctrl)
  // 打桩（stub）
  // 当传入Get函数的参数为liwenzhou.com时返回1和nil
  m.
  EXPECT().
  Get(gomock.Eq("liwenzhou.com")). // 参数
  Return(1, nil).                  // 返回值
  Times(1)                         // 调用次数
  
  // 调用GetFromDB函数时传入上面的mock对象m
  if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
  t.Fatal()
  }
  }
  ```
- ### 2.5.5.  mysql/redis单元测试
- sqlmock 是一个实现 sql/driver 的mock库。它不需要建立真正的数据库连接就可以在测试中模拟任何 sql 驱动程序的行为。使用它可以很方便的在编写单元测试的时候mock sql语句的执行结果。
- 安装
	- go get github.com/DATA-DOG/go-sqlmock
- 示例
	- 需要测试的方法
	  
	  ```
	  // recordStats 记录用户浏览产品信息
	  func recordStats(db *sql.DB, userID, productID int64) (err error) {
	  // 开启事务
	  // 操作views和product_viewers两张表
	  tx, err := db.Begin()
	  if err != nil {
	  return
	  }
	  
	  defer func() {
	  switch err {
	  case nil:
	  err = tx.Commit()
	  default:
	  tx.Rollback()
	  }
	  }()
	  
	  // 更新products表
	  if _, err = tx.Exec("UPDATE products SET views = views + 1"); err != nil {
	  return
	  }
	  // product_viewers表中插入一条数据
	  if _, err = tx.Exec(
	  "INSERT INTO product_viewers (user_id, product_id) VALUES (?, ?)",
	  userID, productID); err != nil {
	  return
	  }
	  return
	  }
	  ```
	- go-sqlmock测试
	  
	  ```
	  // TestShouldUpdateStats sql执行成功的测试用例
	  func TestShouldUpdateStats(t *testing.T) {
	  // mock一个*sql.DB对象，不需要连接真实的数据库
	  db, mock, err := sqlmock.New()
	  if err != nil {
	  t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
	  }
	  defer db.Close()
	  
	  // mock执行指定SQL语句时的返回结果
	  mock.ExpectBegin()
	  mock.ExpectExec("UPDATE products").WillReturnResult(sqlmock.NewResult(1, 1))
	  mock.ExpectExec("INSERT INTO product_viewers").WithArgs(2, 3).WillReturnResult(sqlmock.NewResult(1, 1))
	  mock.ExpectCommit()
	  
	  // 将mock的DB对象传入我们的函数中
	  if err = recordStats(db, 2, 3); err != nil {
	  t.Errorf("error was not expected while updating stats: %s", err)
	  }
	  
	  // 确保期望的结果都满足
	  if err := mock.ExpectationsWereMet(); err != nil {
	  t.Errorf("there were unfulfilled expectations: %s", err)
	  }
	  }
	  
	  // TestShouldRollbackStatUpdatesOnFailure sql执行失败回滚的测试用例
	  func TestShouldRollbackStatUpdatesOnFailure(t *testing.T) {
	  db, mock, err := sqlmock.New()
	  if err != nil {
	  t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
	  }
	  defer db.Close()
	  
	  mock.ExpectBegin()
	  mock.ExpectExec("UPDATE products").WillReturnResult(sqlmock.NewResult(1, 1))
	  mock.ExpectExec("INSERT INTO product_viewers").
	  WithArgs(2, 3).
	  WillReturnError(fmt.Errorf("some error"))
	  mock.ExpectRollback()
	  
	  // now we execute our method
	  if err = recordStats(db, 2, 3); err == nil {
	  t.Errorf("was expecting an error, but there was none")
	  }
	  
	  // we make sure that all expectations were met
	  if err := mock.ExpectationsWereMet(); err != nil {
	  t.Errorf("there were unfulfilled expectations: %s", err)
	  }
	  }
	  ```
	- 上面的代码中，定义了一个执行成功的测试用例和一个执行失败回滚的测试用例，确保我们代码中的每个逻辑分支都能被测试到，提高单元测试覆盖率的同时也保证了代码的健壮性。
- 除了经常用到MySQL外，Redis在日常开发中也会经常用到。接下来的这一小节，我们将一起学习如何在单元测试中mock Redis的相关操作。
- [miniredis](https://github.com/alicebob/miniredis)是一个纯go实现的用于单元测试的redis server。它是一个简单易用的、基于内存的redis替代品，它具有真正的TCP接口，你可以把它当成是redis版本的net/http/httptest。
- 当我们为一些包含Redis操作的代码编写单元测试时就可以使用它来mock Redis操作。
- 安装
	- go get github.com/alicebob/miniredis/v2
- 示例
	- 需要测试的方法
	  
	  ```
	  func DoSomethingWithRedis(rdb *redis.Client, key string) bool {
	  // 这里可以是对redis操作的一些逻辑
	  ctx := context.TODO()
	  if !rdb.SIsMember(ctx, KeyValidWebsite, key).Val() {
	  return false
	  }
	  val, err := rdb.Get(ctx, key).Result()
	  if err != nil {
	  return false
	  }
	  if !strings.HasPrefix(val, "https://") {
	  val = "https://" + val
	  }
	  // 设置 blog key 五秒过期
	  if err := rdb.Set(ctx, "blog", val, 5*time.Second).Err(); err != nil {
	  return false
	  }
	  return true
	  }
	  ```
	- 下面的代码是我使用miniredis库为DoSomethingWithRedis函数编写的单元测试代码，其中miniredis不仅支持mock常用的Redis操作，还提供了很多实用的帮助函数，例如检查key的值是否与预期相等的s.CheckGet()和帮助检查key过期时间的s.FastForward()。
	  
	  ```
	  func TestDoSomethingWithRedis(t *testing.T) {
	  // mock一个redis server
	  s, err := miniredis.Run()
	  if err != nil {
	  panic(err)
	  }
	  defer s.Close()
	  
	  // 准备数据
	  s.Set("q1mi", "liwenzhou.com")
	  s.SAdd(KeyValidWebsite, "q1mi")
	  
	  // 连接mock的redis server
	  rdb := redis.NewClient(&redis.Options{
	  Addr: s.Addr(), // mock redis server的地址
	  })
	  
	  // 调用函数
	  ok := DoSomethingWithRedis(rdb, "q1mi")
	  if !ok {
	  t.Fatal()
	  }
	  
	  // 可以手动检查redis中的值是否复合预期
	  if got, err := s.Get("blog"); err != nil || got != "https://liwenzhou.com" {
	  t.Fatalf("'blog' has the wrong value")
	  }
	  // 也可以使用帮助工具检查
	  s.CheckGet(t, "blog", "https://liwenzhou.com")
	  
	  // 过期检查
	  s.FastForward(5 * time.Second) // 快进5秒
	  if s.Exists("blog") {
	  t.Fatal("'blog' should not have existed anymore")
	  }
	  }
	  ```
- miniredis基本上支持绝大多数的Redis命令，大家可以通过查看文档了解更多用法。
- 当然除了使用miniredis搭建本地redis server这种方法外，还可以使用各种打桩工具对具体方法进行打桩。在编写单元测试时具体使用哪种mock方式还是要根据实际情况来决定。
- ### 2.5.6.  HTTP单元测试
- 在Web开发场景下的单元测试，如果涉及到HTTP请求推荐大家使用Go标准库 net/http/httptest 进行测试，能够显著提高测试效率。
- 在这一小节，我们以常见的gin框架为例，演示如何为http server编写单元测试。
- 假设我们的业务逻辑是搭建一个http server端，对外提供HTTP服务。我们编写了一个helloHandler函数，用来处理用户请求。
  
  ```
  // gin.go
  package httptest_demo
  
  import (
  "fmt"
  "net/http"
  
  "github.com/gin-gonic/gin"
  )
  
  // Param 请求参数
  type Param struct {
  Name string `json:"name"`
  }
  
  // helloHandler /hello请求处理函数
  func helloHandler(c *gin.Context) {
  var p Param
  if err := c.ShouldBindJSON(&p); err != nil {
  c.JSON(http.StatusOK, gin.H{
  	"msg": "we need a name",
  })
  return
  }
  c.JSON(http.StatusOK, gin.H{
  "msg": fmt.Sprintf("hello %s", p.Name),
  })
  }
  
  // SetupRouter 路由
  func SetupRouter() *gin.Engine {
  router := gin.Default()
  router.POST("/hello", helloHandler)
  return router
  }
  ```
- 现在我们需要为helloHandler函数编写单元测试，这种情况下我们就可以使用httptest这个工具mock一个HTTP请求和响应记录器，让我们的server端接收并处理我们mock的HTTP请求，同时使用响应记录器来记录server端返回的响应内容。
- 单元测试的示例代码如下：
  
  ```
  // gin_test.go
  package httptest_demo
  
  import (
  "encoding/json"
  "net/http"
  "net/http/httptest"
  "strings"
  "testing"
  
  "github.com/stretchr/testify/assert"
  )
  
  func Test_helloHandler(t *testing.T) {
  // 定义两个测试用例
  tests := []struct {
  name   string
  param  string
  expect string
  }{
  {"base case", `{"name": "liwenzhou"}`, "hello liwenzhou"},
  {"bad case", "", "we need a name"},
  }
  
  r := SetupRouter()
  
  for _, tt := range tests {
  t.Run(tt.name, func(t *testing.T) {
  	// mock一个HTTP请求
  	req := httptest.NewRequest(
  		"POST",                      // 请求方法
  		"/hello",                    // 请求URL
  		strings.NewReader(tt.param), // 请求参数
  	)
  
  	// mock一个响应记录器
  	w := httptest.NewRecorder()
  
  	// 让server端处理mock请求并记录返回的响应内容
  	r.ServeHTTP(w, req)
  
  	// 校验状态码是否符合预期
  	assert.Equal(t, http.StatusOK, w.Code)
  
  	// 解析并检验响应内容是否复合预期
  	var resp map[string]string
  	err := json.Unmarshal([]byte(w.Body.String()), &resp)
  	assert.Nil(t, err)
  	assert.Equal(t, tt.expect, resp["msg"])
  })
  }
  }
  ```
- 执行单元测试，查看测试结果。
- Mock的使用背景是：
	- 在产品经理发布需求后，前后端同学先根据功能需求出一份 API 文档，然后再按照 API 文档并行开发。
	- 不依赖后端提供数据的情况下，如何让前端独立于后端进行开发呢？
	- 使用 Mock,你可以在开发环境代码内置 Mock，拦截请求，模拟真实 API 返回。如果公司使用了接口管理平台，文档发布的时候可以还通过平台生成 Mock API 直接对接。
	- 使用Mock 假数据替代我们想控制但控制困难的部分。例如：
		- 某些 API 依赖其他 API 的返回值，使用 Mock 方便的对返回值进行改变，测试不同场景下 API 的表现。
		- 某个 API 特别慢，可以暂时用 Mock 代替它，快速调通整个场景测试流程。
	- 敏捷开发过程中，调整需求是很常见的。通过 Mock 可以快速建立功能原型，直观的看到业务逻辑，方便产品调整需求，还可以使用假数据对系统进行演示。
- gock的安装
	- go get -u gopkg.in/h2non/gock.v1
- 使用案例
  
  ```
  // api_test.go
  package gock_demo
  
  import (
  "testing"
  
  "github.com/stretchr/testify/assert"
  "gopkg.in/h2non/gock.v1"
  )
  
  func TestGetResultByAPI(t *testing.T) {
  defer gock.Off() // 测试执行后刷新挂起的mock
  
  // mock 请求外部api时传参x=1返回100
  gock.New("http://your-api.com").
  Post("/post").
  MatchType("json").
  JSON(map[string]int{"x": 1}).
  Reply(200).
  JSON(map[string]int{"value": 100})
  
  // 调用我们的业务函数
  res := GetResultByAPI(1, 1)
  // 校验返回结果是否符合预期
  assert.Equal(t, res, 101)
  
  // mock 请求外部api时传参x=2返回200
  gock.New("http://your-api.com").
  Post("/post").
  MatchType("json").
  JSON(map[string]int{"x": 2}).
  Reply(200).
  JSON(map[string]int{"value": 200})
  
  // 调用我们的业务函数
  res = GetResultByAPI(2, 2)
  // 校验返回结果是否符合预期
  assert.Equal(t, res, 202)
  
  assert.True(t, gock.IsDone()) // 断言mock被触发
  }
  ```
- ### 2.5.7.  Go Testify
- 断言是编程术语，**表示为一些布尔表达式**，程序员相信在程序中的某个特定点该表达式值为真，可以在任何时候启用和禁用断言验证，因此可以在测试时启用断言而在部署时禁用断言。同样，程序投入运行后，最终用户在遇到问题时可以重新启用断言。
- 使用断言可以创建更稳定、品质更好且 不易于出错的代码。当需要在一个值为FALSE时中断当前操作的话，可以使用断言。单元测试必须使用断言（Junit/JunitX）。
- 断言只能用于调试，不能作为程序的功能。断言不是用来检查程序的错误的，断言为假只会中断程序的执行，报告程序是错误的，而不会对错误进行相应的处理。断言不能用来判断有可能发生的情况是否会发生，不能对函数的入口参数进行合法性检查，不能用断言代替条件语句，不能用断言判断有可能发生的错误，只能用于检查程序中不能发生的错误确实不会发生。
- 适合使用断言的情况：
	- 检查程序中各种假设的正确性，eg确定一个计算结果确实在合理的范围内；
	- 证明或者测试某种不可能发生的状况确实不会发生，eg：一些理论上不会被执行到的分支，确实不会被执行；
- 使用断言的基本原则：
	- 使用断言可以捕获不应该或者不可能发生的情况，不等同于非法情况
	- 一个assert只能检验一个假设
- testify是一个社区非常流行的Go单元测试工具包，其中使用最多的功能就是它提供的断言工具——testify/assert或testify/require。
- 安装
	- go get github.com/stretchr/testify
- 使用示例
	- 我们在写单元测试的时候，通常需要使用断言来校验测试结果，但是由于Go语言官方没有提供断言，所以我们会写出很多的if...else...语句。而testify/assert为我们提供了很多常用的断言函数，并且能够输出友好、易于阅读的错误描述信息。
	- 使用testify/assert之后就能将上述判断过程简化如下：
	  
	  ```
	  package day01
	  
	  import (
	  "github.com/stretchr/testify/assert"
	  "testing"
	  )
	  
	  func TestSplit(t *testing.T) { // 测试函数名必须以Test开头，必须接收一个*testing.T类型参数
	  // 定义测试表格
	  // 这里使用匿名结构体定义了若干个测试用例
	  // 并且为每个测试用例设置了一个名称
	  tests := []struct {
	  name  string
	  input string
	  sep   string
	  want  []string
	  }{
	  {"base case", "a:b:c", ":", []string{"a", "b", "c"}},
	  {"wrong sep", "a:b:c", ",", []string{"a:b:c"}},
	  {"more sep", "abcd", "bc", []string{"a", "d"}},
	  {"leading sep", "沙河有沙又有河", "沙", []string{"", "河有", "又有河"}},
	  }
	  // 遍历测试用例
	  for _, tt := range tests {
	  t.Run(tt.name, func(t *testing.T) { // 使用t.Run()执行子测试
	  got := Split(tt.input, tt.sep)
	  assert.Equal(t, got, tt.want)  // 使用assert提供的断言函数
	  })
	  }
	  }
	  ```
	- 运行之后就能产生如下效果：
	- ![](https://cdn.nlark.com/yuque/0/2021/png/2713067/1635810525540-fa0c605f-0137-4082-89a7-5c35daca3a78.png)
	- 当我们有多个断言语句时，还可以使用assert := assert.New(t)创建一个assert对象，它拥有前面所有的断言方法，只是不需要再传入Testing.T参数了。
	  
	  ```
	  func TestSomething(t *testing.T) {
	  assert := assert.New(t)
	  
	  // assert equality
	  assert.Equal(123, 123, "they should be equal")
	  
	  // assert inequality
	  assert.NotEqual(123, 456, "they should not be equal")
	  
	  // assert for nil (good for errors)
	  assert.Nil(object)
	  
	  // assert for not nil (good when you expect something)
	  if assert.NotNil(object) {
	  
	   // now we know that object isn't nil, we are safe to make
	   // further assertions without causing any errors
	   assert.Equal("Something", object.Value)
	  }
	  }
	  ```
	- testify/assert提供了非常多的断言函数，这里没办法一一列举出来，大家可以查看[官方文档](https://pkg.go.dev/github.com/stretchr/testify/assert#pkg-functions)了解。
	- testify/require拥有testify/assert所有断言函数，它们的唯一区别就是——testify/require遇到失败的用例会立即终止本次测试。
	-