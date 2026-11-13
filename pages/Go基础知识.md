# 1.  在VSCode中安装GO
- 下载Go的环境：[https://golang.google.cn/dl](https://golang.google.cn/dl)或者[https://www.golangtc.com/download](https://www.golangtc.com/download)
- 下载下来之后，直接安装即可
- 配置windows的环境变量：
	- 在系统环境变量中新增GOROOT环境变量，然后将其变量设置为GO的环境文件夹
	- 然后在path的环境变量中添加 “%GOROOT%\bin”值
- 配置GO Mod的代理
  
  ```
  go env -w GO111MODULE=on
  go env -w GOPROXY=https://goproxy.cn,direct
  ```
- 在VSCode中安装Go插件
- 使用go version看一下go的语言环境是否成功。
- # 2.  GO基本语法
- ## 2.1.  第一个程序
- 执行如下命令创建一个Go项目
  
  ```
  go mod init example.com/a
  go work init
  ```
- 创建一个helloworld.go
  
  ```
  package main
  
  import "fmt"
  
  func main() {
  fmt.Println("Hello World!")
  }
  ```
- 然后使用go run helloworld.go来看效果
- ## 2.2.  变量、作用域、常量、枚举
- ### 2.2.1.  变量声明和命名规则
- 对于纯粹的变量声明，Go 语言引入了关键字 var，并且将类型信息放在变量名之后，此外，变量声明语句不需要使用分号作为结束符（实际上，所有的 Go 语句都不需要分号作为结束符。
- var 关键字的另一种用法是可以将若干个需要声明的变量放置在一起，免得程序员需要重复写 var 关键字。
  
  ```
  var v1 int 
  var (
    v1 int 
    v2 string
  ）
  ```
- Go语言支持多种数据类型。需要注意的是，变量在声明之后，系统会自动将变量值初始化为对应类型的零值，比如上述 v1 的值为 0，v2 的值空字符串，v3 的值为 false……
  
  ```
  var v1 int            // 整型
  var v2 string         // 字符串
  var v3 bool           // 布尔型
  var v4 [10]int        // 数组，数组元素类型为整型
  var v5 struct {       // 结构体，成员变量 f 的类型为64位浮点型
    f float64
  } 
  var v6 *int           // 指针，指向整型
  var v7 map[string]int   // map（字典），key为字符串类型，value为整型
  var v8 func(a int) int  // 函数，参数类型为整型，返回值类型为整型
  ```
- 如果变量名包含多个单词，Go 语言变量命名规则遵循**驼峰命名法**，即首个单词小写，每个新单词的首字母大写，如 userName，但如果你的全局变量希望能够被外部包所使用，则需要将首个单词的首字母也大写。
- 变量初始化。如果声明变量时想要同时对变量值进行初始化，可以通过以下这些方式：
  
  ```
  var v1 int = 10   // 方式一，常规的初始化操作
  var v2 = 10       // 方式二，此时变量类型会被编译器自动推导出来
  v3 := 10          // 方式三，可以省略 var，编译器可以自动推导出v3的类型
  ```
- ### 2.2.2.  变量赋值和多重赋值
	- 在 Go 语言中，变量初始化和变量赋值是两个不同的概念，变量初始化集变量声明和赋值为一条语句，变量赋值则是先声明变量，再对其进行赋值，初始化只能执行一次，赋值则可以执行多次，下面是变量赋值过程：
	  
	  ```
	  var v10 int 
	  v10 = 123
	  ```
	- Go 语言的变量赋值与多数语言一致，但 Go 语言中提供了程序员期盼多年的**多重赋值**功能，比如下面这个交换 i 和 j 变量的语句：
	  
	  ```
	  i, j = j, i
	  ```
- ### 2.2.3.  匿名变量
- 在 Go 语言中，这种情况可以通过结合使用多重赋值和匿名变量来避免这种丑陋的写法，让代码看起来更加优雅，多重赋值上面已经介绍过，匿名变量则通过下划线 _ 来声明，任何赋予它的值都会被丢弃。
- 示例：
	- 假设 GetName() 函数的定义如下，它返回两个值，分别为 userName 和 nickName：
	  
	  ```
	  func GetName() (userName, nickName string) { 
	   return "nonfu", "学院君"
	  }
	  ```
	- 若只想获得 nickName，则函数调用语句可以用如下方式实现：
	  
	  ```
	  _, nickName := GetName()
	  ```
- ### 2.2.4.  变量的作用域
- 每个变量在程序中都有一定的作用范围，称之为**作用域**。如果一个变量在函数体外声明，则被认为是全局变量，可以在整个包甚至外部包（变量名以大写字母开头）使用，不管你声明在哪个源文件里或在哪个源文件里调用该变量。在函数体内声明的变量称之为局部变量，它们的作用域只在函数体内，函数的参数和返回值变量也是局部变量。
- 尽管变量的标识符必须是唯一的，但你可以在某个代码块的内层代码块中使用相同名称的变量，此时外部的同名变量将会暂时隐藏（结束内部代码块的执行后隐藏的外部同名变量又会出现，而内部同名变量则被释放），你任何的操作都只会影响内部代码块的局部变量。
- ### 2.2.5.  常量定义
- 通过 const 关键字定义常量时，可以指定常量类型，也可以省略（底层会自动推导），常见的常量定义方式如下：
  
  ```
  const Pi float64 = 3.14159265358979323846 
  const zero = 0.0 // 无类型浮点常量 
  const (          // 通过一个 const 关键字定义多个常量，和 var 类似
    size int64 = 1024
    eof = -1  // 无类型整型常量 
  ) 
  const u, v float32 = 0, 3  // u = 0.0, v = 3.0，常量的多重赋值 
  const a, b, c = 3, 4, "foo" // a = 3, b = 4, c = "foo", 无类型整型和字符串常量
  ```
- 结合 Go 语言变量定义方式，可以看到 Go 这种变量和常量的声明方式可读性很好，从左往右，第一个标识符 var 或 const 表明声明的是变量还是常量，第二个标识符标识变量或常量的内存存储块别名，以便后续引用，第三个标识符表示变量或常量的数据类型，可以省略，省略的情况下底层会在编译期自动推导对应的变量或常量类型。
- 由于常量的赋值是一个编译期行为，所以右值不能出现任何需要运行期才能得出结果的表达式，比如试图以如下方式定义常量就会导致编译错误：
  
  ```
  func GetNumber() int {
    return 100
  }
  
  const num = GetNumber()
  ```
- 预定义常量
	- Go 语言预定义了这些常量：true、false 和 iota。
	- 前面两个熟悉其他语言的应该都很熟悉，是布尔类型的真假值，iota 比较特殊，可以被认为是一个可被编译器修改的常量，在每一个 const 关键字出现时被重置为 0，然后在下一个 const 出现之前，每出现一次 iota，其所代表的数字会自动增 1。
	- 示例：
	  
	  ```
	  package main
	  
	  const (    // iota 被重置为 0
	   c0 = iota   // c0 = 0
	   c1 = iota   // c1 = 1
	   c2 = iota   // c2 = 2
	  )
	  
	  const (
	   u = iota * 2;  // u = 0
	   v = iota * 2;  // v = 2
	   w = iota * 2;  // w = 4
	  )
	  
	  const x = iota;  // x = 0
	  const y = iota;  // y = 0
	  ```
- 常量的作用域
	- 和函数体外声明的变量一样，以大写字母开头的常量在包外可见（类似于 public 修饰的类属性），比如上面介绍的 Pi、Sunday 等，而以小写字母开头的常量只能在包内访问（类似于通过 protected 修饰的类属性），函数体内声明的常量只能在函数体内生效。
- ### 2.2.6.  枚举
- 常量还可以用于枚举。枚举中包含了一系列相关的常量，比如下面关于一个星期中每天的定义。Go 语言并不支持其他语言用于表示枚举的 enum 关键字，而是通过在 const 后跟一对圆括号定义一组常量的方式来实现枚举。
- 示例
  
  ```
  const (
    Sunday = iota 
    Monday 
    Tuesday 
    Wednesday 
    Thursday 
    Friday 
    Saturday 
    numberOfDays
  )
  ```
- ## 2.3.  基本数据类型
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1698018335170-8240ed60-518c-469b-9760-bc647475e854.png)
- 以上就是go语言基本的数据类型，有了数据类型，我们就可以使用这些类型来定义变量，Go 语言变量名由字母、数字、下划线组成，其中首个字符不能为数字。
- ### 2.3.1.  指针
- 与C相同，Go语言让程序员决定何时使用指针。变量其实是一种使用方便的占位符，用于引用计算机内存地址。Go 语言中的的取地址符是&，放到一个变量前使用就会返回相应变量的内存地址。指针变量其实就是用于存放某一个对象的内存地址。
- 指针的声明和初始化
	- 和基础类型数据相同，在使用指针变量之前我们首先需要申明指针，声明格式如下：var var_name *var-type，其中的var-type 为指针类型，var_name 为指针变量名，* 号用于指定变量是作为一个指针。
	- 代码举例如下：
	  
	  ```
	  var ip *int        /* 指向整型*/
	  var fp *float32    /* 指向浮点型 */
	  ```
	- 指针的初始化就是取出相对应的变量地址对指针进行赋值，具体如下：
	  
	  ```
	  var a int= 20   /* 声明实际变量 */
	  var ip *int        /* 声明指针变量 */
	  ip = &a  /* 指针变量的存储地址 */
	  ```
- 空指针
	- 当一个指针被定义后**没有分配到任何变量**时，它的值为 **nil**，也称为空指针。它概念上和其它语言的null、NULL一样，都指代零值或空值。
- ### 2.3.2.  数组
- 和c语言相同，Go语言也提供了数组类型的数据结构，数组是具有**相同唯一类型**的一组已编号且**长度固定**的数据项序列，这种类型可以是任意的原始类型例如整型、字符串或者自定义类型。
- 声明数组
	- Go 语言数组声明需要指定元素类型及元素个数，语法格式如下：
	- var variable_name [SIZE] variable_type
	- 以上就可以定一个一维数组，我们举例代码如下：
	  
	  ```
	  var balance [10] float32
	  ```
- 初始化数组
	- 数组的初始化方式有不止一种方式，我们列举如下：
		- 直接进行初始化：var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
		- 通过字面量在声明数组的同时快速初始化数组：balance := [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
		- 数组长度不确定，编译器通过元素个数自行推断数组长度，在[ ]中填入...，举例如下：var balance = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}和balance := [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
		- 数组长度确定，指定下标进行部分初始化：balanced := [5]float32(1:2.0, 3:7.0)
		  
		  注意：初始化数组中 {} 中的元素个数不能大于 [] 中的数字。
		  如果忽略 [] 中的数字不设置数组大小，Go 语言会根据元素的个数来设置数组的大小。
	- 在c语言中我们知道数组名在本质上是数组中第一个元素的地址，而在go语言中，数组名仅仅表示整个数组，是一个完整的值，一个数组变量即是表示整个数组。
	- 所以在go中一个数组变量被赋值或者被传递的时候实际上就会复制整个数组。如果数组比较大的话，这种复制往往会占有很大的开销。所以为了避免这种开销，往往需要传递一个指向数组的指针，这个数组指针并不是数组。关于数组指针具体在指针的部分深入的了解。
- 数组指针
	- 通过数组和指针的知识我们就可以定义一个数组指针，代码如下：
	  
	  ```
	  var a = [...]int{1, 2, 3} // a 是一个数组
	  var b = &a                // b 是指向数组的指针
	  ```
	- 数组指针除了可以防止数组作为参数传递的时候浪费空间，还可以利用其和for range来遍历数组，具体代码如下：
	  
	  ```
	  for i, v := range b {     // 通过数组指针迭代数组的元素
	   fmt.Println(i, v)
	  }
	  ```
- ### 2.3.3.  字符串
- 一个字符串是一个**不可改变**的字节序列，字符串通常是用来包含人类可读的文本数据。和数组不同的是，字符串的元素不可修改，是一个只读的**字节数组**。每个字符串的长度虽然也是固定的，但是字符串的长度并不是字符串类型的一部分。
- 字符串的定义和初始化
- Go语言字符串的底层结构在reflect.StringHeader中定义，具体如下：
  
  ```
  type StringHeader struct {
    Data uintptr
    Len  int
  }
  ```
- 也就是说字符串结构由两个信息组成：第一个是字符串指向的底层字节数组，第二个是字符串的字节的长度。
- 字符串其实是一个结构体，因此字符串的赋值操作也就是reflect.StringHeader结构体的复制过程，并不会涉及底层字节数组的复制，所以我们也可以将字符串数组看作一个结构体数组。
- 字符串和数组类似，内置的len函数返回字符串的长度。
- 根据Go语言规范，Go语言的源文件都是采用UTF8编码。因此，Go源文件中出现的字符串面值常量一般也是UTF8编码的（对于转义字符，则没有这个限制）。提到Go字符串时，我们一般都会假设字符串对应的是一个合法的UTF8编码的字符序列。
- Go语言的字符串中可以存放任意的二进制字节序列，而且即使是UTF8字符序列也可能会遇到坏的编码。如果遇到一个错误的UTF8编码输入，将生成一个特别的Unicode字符‘\uFFFD’，这个字符在不同的软件中的显示效果可能不太一样，在印刷中这个符号通常是一个黑色六角形或钻石形状，里面包含一个白色的问号‘�’。
- 下面的字符串中，我们故意损坏了第一字符的第二和第三字节，因此第一字符将会打印为“�”，第二和第三字节则被忽略；后面的“abc”依然可以正常解码打印（错误编码不会向后扩散是UTF8编码的优秀特性之一）。代码如下：
  
  ```
  fmt.Println("\xe4\x00\x00\xe7\x95\x8cabc") // �界abc
  ```
- 不过在for range迭代这个含有损坏的UTF8字符串时，第一字符的第二和第三字节依然会被单独迭代到，不过此时迭代的值是损坏后的0
  
  ```
  // 0 65533  // \uFFFD, 对应 �
  // 1 0      // 空字符
  // 2 0      // 空字符
  // 3 30028  // 界
  // 6 97     // a
  // 7 98     // b
  // 8 99     // c
  ```
- 在上文中我们知道源代码往往会采用UTF8编码，如果不想解码UTF8字符串，想直接遍历原始的字节码：
	- 可以将字符串强制转为[]byte字节序列后再行遍历（这里的转换一般不会产生运行时开销）
	- 采用传统的下标方式遍历字符串的字节数组
- 除此以外，字符串相关的强制类型转换主要涉及到[]byte和[]rune两种类型。每个转换都可能隐含重新分配内存的代价，最坏的情况下它们的运算时间复杂度都是O(n)。
- 不过字符串和[]rune的转换要更为特殊一些，因为一般这种强制类型转换要求两个类型的底层内存结构要尽量一致，显然它们底层对应的[]byte和[]int32类型是完全不同的内部布局，因此这种转换可能隐含重新分配内存的操作。
- ### 2.3.4.  slice（切片）
- 切片就是一种简化版的动态数组。因为动态数组的长度不固定，切片的长度自然也就不能是类型的组成部分了。数组虽然有适用它们的地方，但是数组的类型和操作都不够灵活，而切片则使用得相当广泛。
- 切片高效操作的要点是要降低内存分配的次数，尽量保证append操作（在后续的插入和删除操作中都涉及到这个函数）不会超出cap的容量，降低触发内存分配的次数和每次分配内存大小。
- slice定义
  
  ```
  type SliceHeader struct {
    Data uintptr   // 指向底层的的数组指针
    Len  int	   // 切片长度
    Cap  int	   // 切片最大长度
  }
  ```
	- 和数组一样，内置的len函数返回切片中有效元素的长度，内置的cap函数返回切片容量大小，容量必须大于或等于切片的长度。
	- 切片可以和nil进行比较，只有当切片底层数据指针为空时切片本身为nil，这时候切片的长度和容量信息将是无效的。如果有切片的底层数据指针为空，但是长度和容量不为0的情况，那么说明切片本身已经被损坏了
	- 只要是切片的底层数据指针、长度和容量没有发生变化的话，对切片的遍历、元素的读取和修改都和数组是一样的。在对切片本身赋值或参数传递时，和数组指针的操作方式类似，只是复制切片头信息（reflect.SliceHeader），并不会复制底层的数据。对于类型，和数组的最大不同是，切片的类型和长度信息无关，只要是相同类型元素构成的切片均对应相同的切片类型。
	- 当我们想定义声明一个切片时可以如下：
	- 对切片本身赋值或参数传递时，和数组指针的操作方式类似，只是复制切片头信息·（reflect.SliceHeader），并不会复制底层的数据。对于类型，和数组的最大不同是，切片的类型和长度信息无关，只要是相同类型元素构成的切片均对应相同的切片类型。
- append添加元素
	- 在切片尾部追加元素
	  
	  ```
	  var a []int
	  a = append(a, 1)               // 追加1个元素
	  a = append(a, 1, 2, 3)         // 追加多个元素, 手写解包方式
	  a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包
	  ```
	  
	  注意：尾部添加在容量不足的条件下需要重新分配内存，可能导致巨大的内存分配和复制数据代价。即使容量足够，依然需要用append函数的返回值来更新切片本身，因为新切片的长度已经发生了变化。
	- 在切片头部添加元素
	  
	  ```
	  var a = []int{1,2,3}
	  a = append([]int{0}, a...)        // 在开头位置添加1个元素
	  a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片
	  ```
	  
	  注意：在开头一般都会导致内存的重新分配，而且会导致已有的元素全部复制1次。因此，从切片的开头添加元素的性能一般要比从尾部追加元素的性能差很多。
	- append链式操作
	  
	  ```
	  var a []int
	  a = append(a[:i], append([]int{x}, a[i:]...)...)     // 在第i个位置插入x
	  a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第i个位置插入切片
	  ```
	  
	  每个添加操作中的第二个append调用都会创建一个**临时切片**，并将a[i:]的内容复制到新创建的切片中，然后将临时创建的切片再追加到a[:i]。
	- append和copy组合
	  
	  ```
	  a = append(a, 0)     // 切片扩展1个空间
	  copy(a[i+1:], a[i:]) // a[i:]向后移动1个位置
	  a[i] = x             // 设置新添加的元素
	  ```
	  
	  第三个操作中会创建一个临时对象，我们可以借用copy函数避免这个操作，这种方式操作语句虽然冗长了一点，但是相比前面的方法，可以减少中间创建的临时切片。
- 删除元素
	- 从头部删除元素
	- 直接移动数据指针，代码如下：
	  
	  ```
	  a = []int{1, 2, 3, ...}
	  a = a[1:]                       // 删除开头1个元素
	  a = a[N:]                       // 删除开头N个元素
	  ```
	- append方式
	  
	  ```
	  a = []int{1, 2, 3, ...}
	  a = append(a[:0], a[1:]...) // 删除开头1个元素
	  a = append(a[:0], a[N:]...) // 删除开头N个元素
	  ```
	- copy方式
	  
	  ```
	  a = []int{1, 2, 3}
	  a = a[:copy(a, a[1:])] // 删除开头1个元素
	  a = a[:copy(a, a[N:])] // 删除开头N个元素
	  ```
	- 从中间位置删除
	- 对于删除中间的元素，需要对剩余的元素进行一次整体挪动，同样可以用append或copy原地完成：
	  
	  ```
	  a = []int{1, 2, 3, ...}
	  a = append(a[:i], a[i+1], ...)
	  a = append(a[:i], a[i+N:], ...)
	  ```
	  
	  ```
	  a = []int{1, 2, 3}
	  a = a[:copy(a[:i], a[i+1:])] // 删除中间1个元素
	  a = a[:copy(a[:i], a[i+N:])] // 删除中间N个元素
	  ```
	- 从尾部删除
	- 代码如下所示：
	  
	  ```
	  a = []int{1, 2, 3, ...}
	  
	  a = a[:len(a)-1]   // 删除尾部1个元素
	  a = a[:len(a)-N]   // 删除尾部N个元素
	  ```
- ## 2.4.  结构体
- ### 2.4.1.  声明结构体
- 在声明结构体之前我们首先需要定义一个结构体类型，这需要使用type和struct，type用于设定结构体的名称，struct用于定义一个新的数据类型。具体结构如下：
  
  ```
  type struct_variable_type struct {
   member definition
   member definition
   ...
   member definition
  }
  ```
- 定义好了结构体类型，我们就可以使用该结构体声明这样一个结构体变量，语法如下：
  
  ```
  variable_name := structure_variable_type {value1, value2...valuen}
  
  variable_name := structure_variable_type { key1: value1, key2: value2..., keyn: valuen}
  ```
- ### 2.4.2.  访问结构体成员
- 如果要访问结构体成员，需要使用点号 . 操作符，格式为：结构体变量名.成员名。举例代码如下：
  
  ```
  package main
  
  import "fmt"
  
  type Books struct {
   title string
   author string
  }
  
  func main() {
  var book1 Books
  Book1.title = "Go 语言入门"
  Book1.author = "mars.hao"	
  }
  ```
- ### 2.4.3.  结构体指针
- 关于结构体指针的定义和申明同样可以套用前文中讲到的指针的相关定义，从而使用一个指针变量存放一个结构体变量的地址。
- 定义一个结构体变量的语法：var struct_pointer *Books。
- 这种指针变量的初始化和上文指针部分的初始化方式相同struct_pointer = &Book1，但是和c语言中有所不同，使用结构体指针访问结构体成员仍然使用.操作符。格式如下：struct_pointer.title
- ## 2.5.  流程控制和运算符号
- ### 2.5.1.  运算符号
- 算术运算符号
  
  | **运算符** | **术语** | **示例** | **结果** |
  | + | 加 | 10 + 5 | 15 |
  | - | 减 | 10 - 5 | 5 |
  | * | 乘 | 10 * 5 | 50 |
  | / | 除 | 10 / 5 | 2 |
  | % | 取模(取余) | 10 % 3 | 1 |
  | ++ | 后自增，没有前自增 | a=0; a++ | a=1 |
  | -- | 后自减，没有前自减 | a=2; a-- | a=1 |
- 关系运算符号
  
  | **运算符** | **术语** | **示例** | **结果** |
  | == | 相等于 | 4 == 3 | false |
  | != | 不等于 | 4 != 3 | true |
  | <  | 小于 | 4 < 3 | false |
  | >  | 大于 | 4 > 3 | true |
  | <= | 小于等于 | 4 <= 3 | false |
  | >= | 大于等于 | 4 >= 1 | true |
- 逻辑运算符号
  
  | **运算符** | **术语** | **示例** | **结果** |
  | ! | 非 | !a | 如果a为假，则!a为真；如果a为真，则!a为假。 |
  | && | 与 | a && b | 如果a和b都为真，则结果为真，否则为假。 |
  | || | 或 | a || b | 如果a和b有一个为真，则结果为真，二者都为假时，结果为假。 |
- 位运算符号
  
  | **运算符** | **术语** | **说明** | **示例** |
  | & | 按位与 | 参与运算的两数各对应的二进位相与 | 60 & 13 结果为12 |
  | | | 按位或 | 参与运算的两数各对应的二进位相或 | 60 | 13 结果为61 |
  | ^ | 异或 | 参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1 | 60 ^ 13 结果为240 |
  | <<  | 左移 | 左移n位就是乘以2的n次方。 左边丢弃，右边补0。 | 4 << 2 结果为16 |
  | >>  | 右移 | 右移n位就是除以2的n次方。 右边丢弃，左边补位。 | 4 >> 2 结果为1 |
- 赋值运算符号
  
  | **运算符** | **说明** | **示例** |
  | = | 普通赋值 | c = a + b 将 a + b 表达式结果赋值给 c |
  | += | 相加后再赋值 | c += a 等价于 c = c + a |
  | -= | 相减后再赋值 | c -= a 等价于 c = c - a |
  | *= | 相乘后再赋值 | c *= a 等价于 c = c * a |
  | /= | 相除后再赋值 | c /= a 等价于 c = c / a |
  | %= | 求余后再赋值 | c %= a 等价于 c = c % a |
  | <<= | 左移后赋值 | c <<= 2 等价于 c = c << 2 |
  | >>= | 右移后赋值 | c >>= 2 等价于 c = c >> 2 |
  | &= | 按位与后赋值 | c &= 2 等价于 c = c & 2 |
  | ^= | 按位异或后赋值 | c ^= 2 等价于 c = c ^ 2 |
  | |= | 按位或后赋值 | c |= 2 等价于 c = c | 2 |
- 其他运算符号
  
  | **运算符** | **术语** | **示例** | **说明** |
  | & | 取地址运算符 | &a | 变量a的地址 |
  | * | 取值运算符 | *a | 指针变量a所指向内存的值 |
- 运算符号优先级
  
  | **优先级** | **运算符** |
  | **7** | **^****      ****!****   ** |
  | **6** | *******       ****/****    ****%****    ****<<****    ****>>****    ****&****      ****&^** |
  | **5** | **+****      ****-****     ****|****      ****^** |
  | **4** | **==****   ****!=****   ****<****    ****<=****    ****>=****    ****>** |
  | **3** | **<-** |
  | **2** | **&&** |
  | **1** | **||** |
- ### 2.5.2.  流程控制
- Go语言支持最基本的三种程序运行结构：顺序结构、选择结构、循环结构。
	- 顺序结构：程序按顺序执行，不发生跳转。
	- 选择结构：依据是否满足条件，有选择的执行相应功能。
	- 循环结构：依据条件是否满足，循环多次执行某段代码。
- if语句
  
  ```
  var a int = 3
    if a == 3 { //条件表达式没有括号
        fmt.Println("a==3")
    }
  
    //支持一个初始化表达式, 初始化字句和条件表达式直接需要用分号分隔
    if b := 3; b == 3 {
        fmt.Println("b==3")
    }
  ```
  
  ```
  if a := 3; a == 4 {
        fmt.Println("a==4")
    } else { //左大括号必须和条件语句或else在同一行
        fmt.Println("a!=4")
    }
  ```
  
  ```
  if a := 3; a > 3 {
        fmt.Println("a>3")
    } else if a < 3 {
        fmt.Println("a<3")
    } else if a == 3 {
        fmt.Println("a==3")
    } else {
        fmt.Println("error")
    }
  ```
- switch语句
  
  ```
  var score int = 90
  
    switch score {
    case 90:
        fmt.Println("优秀")
        //fallthrough
    case 80:
        fmt.Println("良好")
        //fallthrough
    case 50, 60, 70:
        fmt.Println("一般")
        //fallthrough
    default:
        fmt.Println("差")
    }
  ```
  
  ```
  //1
    switch s1 := 90; s1 { //初始化语句;条件
    case 90:
        fmt.Println("优秀")
    case 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
  
    //2
    var s2 int = 90
    switch { //这里没有写条件
    case s2 >= 90: //这里写判断语句
        fmt.Println("优秀")
    case s2 >= 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
  
    //3
    switch s3 := 90; { //只有初始化语句，没有条件
    case s3 >= 90: //这里写判断语句
        fmt.Println("优秀")
    case s3 >= 80:
        fmt.Println("良好")
    default:
        fmt.Println("一般")
    }
  ```
- 循环语句
  
  ```
  var i, sum int
    for i = 1; i <= 100; i++ {
        sum += i
    }
    fmt.Println("sum = ", sum)
  ```
  
  ```
  s := "abc"
    for i := range s { //支持 string/array/slice/map。
        fmt.Printf("%c\n", s[i])
    }
  
    for _, c := range s { // 忽略 index
        fmt.Printf("%c\n", c)
    }
  
    for i, c := range s {
        fmt.Printf("%d, %c\n", i, c)
    }
  ```
- 跳转语句
  
  ```
  for i := 0; i < 5; i++ {
        if 2 == i {
            //break    //break操作是跳出当前循环
            continue //continue是跳过本次循环
        }
        fmt.Println(i)
    }
  ```
  
  ```
  func main() {
    for i := 0; i < 5; i++ {
        for {
            fmt.Println(i)
            goto LABEL //跳转到标签LABEL，从标签处，执行代码
        }
    }
  
    fmt.Println("this is test")
  
  LABEL:
    fmt.Println("it is over")
  }
  ```
- ## 2.6.  函数
- ### 2.6.1.  函数分类
- 为完成某一功能的程序指令(语句)的集合，称为函数。在Go语言中，函数是第一类对象，我们可以将函数保持到变量中。函数主要有**具名**和**匿名**之分，包级函数一般都是具名函数，具名函数是匿名函数的一种特例，当匿名函数引用了外部作用域中的变量时就成了**闭包函数**，闭包函数是函数式编程语言的核心。
- 具名函数：就和c语言中的普通函数意义相同，具有函数名、返回值以及函数参数的函数。
  
  ```
  func Add(a, b int) int {
    return a+b
  }
  ```
- 匿名函数：指不需要定义函数名的一种函数实现方式，它由一个不带函数名的函数声明和函数体组成。
  
  ```
  var Add = func(a, b int) int {
    return a+b
  }
  ```
  
  解释几个名词如下：
  
  闭包函数：返回为函数对象，不仅仅是一个函数对象，在该函数外还包裹了一层作用域，这使得，该函数无论在何处调用，优先使用自己外层包裹的作用域。
  
  一级对象：支持闭包的多数语言都将函数作为第一级对象，就是说函数可以存储到变量中作为参数传递给其他函数，最重要的是能够被函数动态创建和返回。
  
  包：go的每一个文件都是属于一个包的，也就是说go是以包的形式来管理文件和项目目录结构的。
- ### 2.6.2.  函数声明和定义
- Go 语言函数定义格式如下：
  
  ```
  func fuction_name([parameter list])[return types]{
  函数体
  }
  ```
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1698279378173-76a734bf-4a07-4a53-b351-46a139602e76.png)
- ### 2.6.3.  函数传参
- Go语言中的函数可以有多个参数和多个返回值，参数和返回值都是以传值的方式和被调用者交换数据。在语法上，函数还支持可变数量的参数，可变数量的参数必须是最后出现的参数，可变数量的参数其实是一个切片类型的参数。
- 当可变参数是一个**空接口类型**时，调用者是否解包可变参数会导致不同的结果，我们解释一下解包的含义，代码如下：
  
  ```
  func main(){
  var a = []int{1, 2, 3}
  Print(a...)   // 解包
  Print(a)	  // 未解包
  }
  
  func Print(a ...int{}) {
  fmt.Println(a...)
  }
  ```
- 以上当传入参数为a...时即是对切片a进行了解包，此时其实相当于直接调用Print(1,2,3)。当传入参数直接为 a时等价于直接调用Print([]int{}{1,2,3})
- ### 2.6.4.  函数返回值
- 不仅函数的参数可以有名字，也可以给函数的返回值命名。
- 如果返回值命名了，可以通过名字来修改返回值，也可以通过defer语句在return语句之后修改返回值
  
  ```
  func mian() {
  for i := 0 ; i<3; i++ {
  defer func() { println(i) }
  }
  }
  
  // 该函数最终的输出为：
  // 3
  // 3
  // 3
  ```
- 以上代码中如果没有defer其实返回值就是0,1,2，但defer语句会在函数return之后才会执行，也就是或只有以上函数在执行结束return之后才会执行defer语句，而该函数return时的i值将会达到3，所以最终的defer语句执行printlin的输出都是3。
- defer语句延迟执行的其实是一个匿名函数，因为这个匿名函数捕获了外部函数的局部变量v，这种函数我们一般叫闭包。闭包对捕获的外部变量并不是传值方式访问，而是以引用的方式访问。
- 这种方式往往会带来一些问题，修复方法为在每一轮迭代中都为defer函数提供一个独有的变量，修改代码如下：
  
  ```
  func main() {
    for i := 0; i < 3; i++ {
        i := i // 定义一个循环体内局部变量i
        defer func(){ println(i) } ()
    }
  }
  
  func main() {
    for i := 0; i < 3; i++ {
        // 通过函数传入i
        // defer 语句会马上对调用参数求值
        // 不再捕获，而是直接传值
        defer func(i int){ println(i) } (i)
    }
  }
  ```
- ### 2.6.5.  递归调用
- Go语言中，函数还可以直接或间接地调用自己，也就是支持递归调用。Go语言函数的递归调用**深度逻辑上没有限制**，函数调用的栈是**不会出现溢出错误**的，因为Go语言运行时会根据需要动态地调整函数栈的大小。
  
  ```
  func recursion() {
   recursion() /* 函数调用自身 */
  }
  
  func main() {
   recursion()
  }
  ```
- ## 2.7.  方法和接口
- ### 2.7.1.  方法
- 方法一般是面向对象编程(OOP)的一个特性，在C++语言中方法对应一个类对象的成员函数，是关联到具体对象上的虚表中的。但是Go语言的方法却是关联到类型的，这样可以在编译阶段完成方法的静态绑定。一个面向对象的程序会用方法来表达其属性对应的操作，这样使用这个对象的用户就不需要直接去操作对象，而是借助方法来做这些事情。
- 示例：
	- 实现C语言中的一组函数如下：
	  
	  ```
	  // 文件对象
	  type File struct {
	   fd int
	  }
	  
	  // 打开文件
	  func OpenFile(name string) (f *File, err error) {
	   // ...
	  }
	  
	  // 关闭文件
	  func CloseFile(f *File) error {
	   // ...
	  }
	  
	  // 读文件数据
	  func ReadFile(f *File, offset int64, data []byte) int {
	   // ...
	  }
	  ```
	- 以上的三个函数都是普通的函数，需要占用包级空间中的名字资源。不过CloseFile和ReadFile函数只是针对File类型对象的操作，这时候我们更希望这类函数和操作对象的类型紧密绑定在一起。
	- 所以在go语言中我们修改如下：
	  
	  ```
	  // 关闭文件
	  func (f *File) CloseFile() error {
	   // ...
	  }
	  
	  // 读文件数据
	  func (f *File) ReadFile(offset int64, data []byte) int {
	   // ...
	  }
	  ```
- 将CloseFile和ReadFile函数的第一个参数移动到函数名的开头，这两个函数就成了File类型独有的方法了（而不是File对象方法）
- 从代码角度看虽然只是一个小的改动，但是从编程哲学角度来看，Go语言已经是进入面向对象语言的行列了。我们可以给任何自定义类型添加一个或多个方法。每种类型对应的方法必须和类型的定义在同一个包中，因此是无法给int这类内置类型添加方法的（因为方法的定义和类型的定义不在一个包中）。对于给定的类型，每个方法的名字必须是唯一的，同时方法和函数一样也不支持重载。
- ### 2.7.2.  接口
- Go 语言提供了另外一种数据类型即接口，它把所有的具有共性的方法定义在一起，任何其他类型只要实现了这些方法就是实现了这个接口。
- Go的接口类型是对其它类型行为的抽象和概括；因为接口类型不会和特定的实现细节绑定在一起，通过这种抽象的方式我们可以让对象更加灵活和更具有适应能力。很多面向对象的语言都有相似的接口概念，但Go语言中接口类型的独特之处在于它是满足隐式实现的鸭子类型。
- 所谓鸭子类型说的是：只要走起路来像鸭子、叫起来也像鸭子，那么就可以把它当作鸭子。
- interface实际上就是一个结构体，包含两个成员。其中一个成员是指向具体数据的指针，另一个成员中包含了类型信息。
- 将一个具体类型数据赋值给interface这样的抽象类型，需要进行类型转换。这个转换过程中涉及哪些操作呢？
- 如果转换为空接口，返回一个Eface，将Eface中的data指针指向原型数据，type指针会指向数据的Type结构体。
- 如果将其转化为带方法的interface，需要进行一次检测，该类型必须实现interface中声明的所有方法才可以进行转换，这个检测将会在编译过程中进行。检测过程具体实现式通过比较具体类型的方法表和接口类型的方法表来进行的。
- 具体类型方法表：Type的UncommonType中有一个方法表，某个具体类型实现的所有方法都会被收集到这张表中。
- 接口类型方法表：Iface的Itab的InterfaceType中也有一张方法表，这张方法表中是接口所声明的方法。Iface中的Itab的func域也是一张方法表，这张表中的每一项就是一个函数指针，也就是只有实现没有声明。
- 这两处方法表都是排序过的，只需要一遍顺序扫描进行比较，应该可以知道Type中否实现了接口中声明的所有方法。最后还会将Type方法表中的函数指针，拷贝到Itab的fun字段中。Iface中的Itab的func域也是一张方法表，这张表中的每一项就是一个函数指针，也就是只有实现没有声明。
- ## 2.8.  异常处理
- ### 2.8.1.  error接口
- Go语言引入了一个关于错误处理的标准模式，即error接口，它是Go语言内建的接口类型，该接口的定义如下：
  
  ```
  type error interface {
    Error() string
  }
  ```
- Go语言的标准库代码包errors为用户提供如下方法：
  
  ```
  package errors
  
  type errorString struct { 
    text string 
  }
  
  func New(text string) error { 
    return &errorString{text} 
  }
  
  func (e *errorString) Error() string { 
    return e.text 
  }
  ```
- 另一个可以生成error类型值的方法是调用fmt包中的Errorf函数：
  
  ```
  package fmt
  import "errors"
  
  func Errorf(format string, args ...interface{}) error {
    return errors.New(Sprintf(format, args...))
  }
  ```
- 示例代码：
  
  ```
  import (
    "errors"
    "fmt"
  )
  
  func main() {
    var err1 error = errors.New("a normal err1")
    fmt.Println(err1) //a normal err1
  
    var err2 error = fmt.Errorf("%s", "a normal err2")
    fmt.Println(err2) //a normal err2
  }
  ```
- 函数通常在最后的返回值中返回错误信息：
  
  ```
  import (
    "errors"
    "fmt"
  )
  
  func Divide(a, b float64) (result float64, err error) {
    if b == 0 {
        result = 0.0
        err = errors.New("runtime error: divide by zero")
        return
    }
  
    result = a / b
    err = nil
    return
  }
  
  func main() {
    r, err := Divide(10.0, 0)
    if err != nil {
        fmt.Println(err) //错误处理 runtime error: divide by zero
    } else {
        fmt.Println(r) // 使用返回值
    }
  }
  ```
- ### 2.8.2.  panic恐慌
- 在通常情况下，向程序使用方报告错误状态的方式可以是返回一个额外的error类型值。
- 但是，当遇到不可恢复的错误状态的时候，如数组访问越界、空指针引用等，这些运行时错误会引起painc异常。这时，上述错误处理方式显然就不适合了。反过来讲，在一般情况下，我们不应通过调用panic函数来报告普通的错误，而应该只把它作为报告致命错误的一种方式。当某些不应该发生的场景发生时，我们就应该调用panic。
- 一般而言，当panic异常发生时，程序会中断运行，并立即执行在该goroutine（可以先理解成线程，在中被延迟的函数（defer 机制）。随后，程序崩溃并输出日志信息。日志信息包括panic value和函数调用的堆栈跟踪信息。
- 不是所有的panic异常都来自运行时，直接调用内置的panic函数也会引发panic异常；panic函数接受任何值作为参数。
- 调用panic函数引发的panic异常：
  
  ```
  func TestA() {
    fmt.Println("func TestA()")
  }
  
  func TestB() {
    panic("func TestB(): panic")
  }
  
  func TestC() {
    fmt.Println("func TestC()")
  }
  
  func main() {
    TestA()
    TestB()//TestB()发生异常，中断程序
    TestC()
  }
  ```
- 内置的panic函数引发的panic异常：
  
  ```
  func TestA() {
    fmt.Println("func TestA()")
  }
  
  func TestB(x int) {
    var a [10]int
    a[x] = 222 //x值为11时，数组越界
  }
  
  func TestC() {
    fmt.Println("func TestC()")
  }
  
  func main() {
    TestA()
    TestB(11)//TestB()发生异常，中断程序
    TestC()
  }
  ```
- ### 2.8.3.  recover处理
- 运行时panic异常一旦被引发就会导致程序崩溃。这当然不是我们愿意看到的，因为谁也不能保证程序不会发生任何运行时错误。
- 不过，Go语言为我们提供了专用于“拦截”运行时panic的内建函数——recover。它可以是当前的程序从运行时panic的状态中恢复并重新获得流程控制权。
  
  ```
  func recover() interface{}
  ```
- 如果调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。
- 示例代码：
  
  ```
  func TestA() {
    fmt.Println("func TestA()")
  }
  
  func TestB() (err error) {
    defer func() { //在发生异常时，设置恢复
        if x := recover(); x != nil {
            //panic value被附加到错误信息中；
       //并用err变量接收错误信息，返回给调用者。
            err = fmt.Errorf("internal error: %v", x)
        }
    }()
  
    panic("func TestB(): panic")
  }
  
  func TestC() {
    fmt.Println("func TestC()")
  }
  
  func main() {
    TestA()
    err := TestB()
    fmt.Println(err)
    TestC()
  
    /*
        运行结果：
        func TestA()
        internal error: func TestB(): panic
        func TestC()
    */
  }
  ```
- 延迟调用中引发的错误，可被后续延迟调用捕获，但仅最后⼀个错误可被捕获：
  
  ```
  func test() {
    defer func() {
        fmt.Println(recover())
    }()
  
    defer func() {
        panic("defer panic")
    }()
  
    panic("test panic")
  }
  
  func main() {
    test()
    //运行结果：defer panic
  }
  ```
- # 3.  面向对象编程
- 对于面向对象编程的支持Go 语言设计得非常简洁而优雅。因为，Go语言并没有沿袭传统面向对象编程中的诸多概念，比如继承(不支持继承，尽管匿名字段的内存布局和行为类似继承，但它并不是继承)、虚函数、构造函数和析构函数、隐藏的this指针等。
- 尽管Go语言中没有封装、继承、多态这些概念，但同样通过别的方式实现这些特性：
	- 封装：通过方法实现
	- 继承：通过匿名字段实现
	- 多态：通过接口实现
- 在面向对象编程中，一个对象其实也就是一个简单的值或者一个变量，在这个对象中会包含一些函数，**这种带有接收者的函数，我们称为方法(method)**。本质上，一个方法则是一个和特殊类型关联的函数。
- 一个面向对象的程序会用方法来表达其属性和对应的操作，这样使用这个对象的用户就不需要直接去操作对象，而是借助方法来做这些事情。
- 在Go语言中，可以给任意自定义类型（包括内置类型，但不包括指针类型）添加相应的方法。
- ⽅法总是绑定对象实例，并隐式将实例作为第⼀实参 (receiver)，方法的语法如下：
	- **func** (receiver ReceiverType) funcName(parameters) (results)
	- 参数 receiver 可任意命名。如⽅法中未曾使⽤，可省略参数名。
	- 参数 receiver 类型可以是 T 或 *T。基类型 T 不能是接⼝或指针。
	- 不支持重载方法，也就是说，不能定义名字相同但是不同参数的方法。
- ## 3.1.  匿名组合
- 匿名字段
	- 一般情况下，定义结构体的时候是字段名与其类型一一对应，实际上Go支持只提供类型，而不写字段名的方式，也就是匿名字段，也称为嵌入字段。
	- 当匿名字段也是一个结构体的时候，那么这个结构体所拥有的全部字段都被隐式地引入了当前定义的这个结构体。
	  
	  ```
	  //人
	  type Person struct {
	   name string
	   sex  byte
	   age  int
	  }
	  //学生
	  type Student struct {
	   Person // 匿名字段，那么默认Student就包含了Person的所有字段
	   id     int
	   addr   string
	  }
	  ```
- 初始化
  
  ```
  //人
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //学生
  type Student struct {
    Person // 匿名字段，那么默认Student就包含了Person的所有字段
    id     int
    addr   string
  }
  
  func main() {
    //顺序初始化
    s1 := Student{Person{"mike", 'm', 18}, 1, "sz"}
    //s1 = {Person:{name:mike sex:109 age:18} id:1 addr:sz}
    fmt.Printf("s1 = %+v\n", s1)
  
    //s2 := Student{"mike", 'm', 18, 1, "sz"} //err
  
    //部分成员初始化1
    s3 := Student{Person: Person{"lily", 'f', 19}, id: 2}
    //s3 = {Person:{name:lily sex:102 age:19} id:2 addr:}
    fmt.Printf("s3 = %+v\n", s3)
  
    //部分成员初始化2
    s4 := Student{Person: Person{name: "tom"}, id: 3}
    //s4 = {Person:{name:tom sex:0 age:0} id:3 addr:}
    fmt.Printf("s4 = %+v\n", s4)
  }
  ```
- 成员操作
  
  ```
  var s1 Student //变量声明
    //给成员赋值
    s1.name = "mike" //等价于 s1.Person.name = "mike"
    s1.sex = 'm'
    s1.age = 18
    s1.id = 1
    s1.addr = "sz"
    fmt.Println(s1) //{{mike 109 18} 1 sz}
  
    var s2 Student //变量声明
    s2.Person = Person{"lily", 'f', 19}
    s2.id = 2
    s2.addr = "bj"
    fmt.Println(s2) //{{lily 102 19} 2 bj}
  ```
- 同名字段
  
  ```
  //人
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //学生
  type Student struct {
    Person // 匿名字段，那么默认Student就包含了Person的所有字段
    id     int
    addr   string
    name   string //和Person中的name同名
  }
  func main() {
    var s Student //变量声明
  
    //给Student的name，还是给Person赋值？
    s.name = "mike"
  
    //{Person:{name: sex:0 age:0} id:0 addr: name:mike}
    fmt.Printf("%+v\n", s)
  
    //默认只会给最外层的成员赋值
    //给匿名同名成员赋值，需要显示调用
    s.Person.name = "yoyo"
    //Person:{name:yoyo sex:0 age:0} id:0 addr: name:mike}
    fmt.Printf("%+v\n", s)
  }
  ```
- 其他匿名字段
	- 非结构体类型
	- 所有的内置类型和自定义类型都是可以作为匿名字段的：
	  
	  ```
	  type mystr string //自定义类型
	  
	  type Person struct {
	   name string
	   sex  byte
	   age  int
	  }
	  
	  type Student struct {
	   Person // 匿名字段，结构体类型
	   int     // 匿名字段，内置类型
	   mystr   // 匿名字段，自定义类型
	  }
	  
	  func main() {
	   //初始化
	   s1 := Student{Person{"mike", 'm', 18}, 1, "bj"}
	  
	   //{Person:{name:mike sex:109 age:18} int:1 mystr:bj}
	   fmt.Printf("%+v\n", s1)
	  
	   //成员的操作，打印结果：mike, m, 18, 1, bj
	   fmt.Printf("%s, %c, %d, %d, %s\n", s1.name, s1.sex, s1.age, s1.int, s1.mystr)
	  }
	  ```
	- 结构体指针类型
	  
	  ```
	  type Person struct { //人
	   name string
	   sex  byte
	   age  int
	  }
	  
	  type Student struct { //学生
	   *Person // 匿名字段，结构体指针类型
	   id      int
	   addr    string
	  }
	  
	  func main() {
	   //初始化
	   s1 := Student{&Person{"mike", 'm', 18}, 1, "bj"}
	  
	   //{Person:0xc0420023e0 id:1 addr:bj}
	   fmt.Printf("%+v\n", s1)
	   //mike, m, 18
	   fmt.Printf("%s, %c, %d\n", s1.name, s1.sex, s1.age)
	  
	   //声明变量
	   var s2 Student
	   s2.Person = new(Person) //分配空间
	   s2.name = "yoyo"
	   s2.sex = 'f'
	   s2.age = 20
	  
	   s2.id = 2
	   s2.addr = "sz"
	  
	   //yoyo 102 20 2 20
	   fmt.Println(s2.name, s2.sex, s2.age, s2.id, s2.age)
	  }
	  ```
- ## 3.2.  匿名字段
- 方法的继承
	- 如果匿名字段实现了一个方法，那么包含这个匿名字段的struct也能调用该方法。
	  
	  ```
	  type Person struct {
	   name string
	   sex  byte
	   age  int
	  }
	  
	  //Person定义了方法
	  func (p *Person) PrintInfo() {
	   fmt.Printf("%s,%c,%d\n", p.name, p.sex, p.age)
	  }
	  
	  type Student struct {
	   Person // 匿名字段，那么Student包含了Person的所有字段
	   id     int
	   addr   string
	  }
	  
	  func main() {
	   p := Person{"mike", 'm', 18}
	   p.PrintInfo()
	  
	   s := Student{Person{"yoyo", 'f', 20}, 2, "sz"}
	   s.PrintInfo()
	  }
	  ```
- 方法的重写
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //Person定义了方法
  func (p *Person) PrintInfo() {
    fmt.Printf("Person: %s,%c,%d\n", p.name, p.sex, p.age)
  }
  
  type Student struct {
    Person // 匿名字段，那么Student包含了Person的所有字段
    id     int
    addr   string
  }
  
  //Student定义了方法
  func (s *Student) PrintInfo() {
    fmt.Printf("Student：%s,%c,%d\n", s.name, s.sex, s.age)
  }
  
  func main() {
    p := Person{"mike", 'm', 18}
    p.PrintInfo() //Person: mike,m,18
  
    s := Student{Person{"yoyo", 'f', 20}, 2, "sz"}
    s.PrintInfo()        //Student：yoyo,f,20
    s.Person.PrintInfo() //Person: yoyo,f,20
  }
  ```
- ## 3.3.  为类型添加方法
- 基础类型作为接收者
	- 通过上面的例子可以看出，面向对象只是换了一种语法形式来表达。方法是函数的语法糖，因为receiver其实就是方法所接收的第1个参数。
	- 注意：虽然方法的名字一模一样，但是如果接收者不一样，那么方法就不一样。
	  
	  ```
	  type MyInt int //自定义类型，给int改名为MyInt
	  
	  //在函数定义时，在其名字之前放上一个变量，即是一个方法
	  func (a MyInt) Add(b MyInt) MyInt { //面向对象
	   return a + b
	  }
	  
	  //传统方式的定义
	  func Add(a, b MyInt) MyInt { //面向过程
	   return a + b
	  }
	  
	  func main() {
	   var a MyInt = 1
	   var b MyInt = 1
	  
	   //调用func (a MyInt) Add(b MyInt)
	   fmt.Println("a.Add(b) = ", a.Add(b)) //a.Add(b) =  2
	  
	   //调用func Add(a, b MyInt)
	   fmt.Println("Add(a, b) = ", Add(a, b)) //Add(a, b) =  2
	  }
	  ```
- 结构体作为接收者
	- 方法里面可以访问接收者的字段，调用方法通过点( **.** )访问，就像struct里面访问字段一样：
	  
	  ```
	  type Person struct {
	   name string
	   sex  byte
	   age  int
	  }
	  
	  func (p Person) PrintInfo() { //给Person添加方法
	   fmt.Println(p.name, p.sex, p.age)
	  }
	  
	  func main() {
	   p := Person{"mike", 'm', 18} //初始化
	   p.PrintInfo() //调用func (p Person) PrintInfo()
	  }
	  ```
- ## 3.4.  值语义与引用语义
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //指针作为接收者，引用语义
  func (p *Person) SetInfoPointer() {
    //给成员赋值
    (*p).name = "yoyo"
    p.sex = 'f'
    p.age = 22
  }
  
  //值作为接收者，值语义
  func (p Person) SetInfoValue() {
    //给成员赋值
    p.name = "yoyo"
    p.sex = 'f'
    p.age = 22
  }
  
  func main() {
    //指针作为接收者，引用语义
    p1 := Person{"mike", 'm', 18} //初始化
    fmt.Println("函数调用前 = ", p1)   //函数调用前 =  {mike 109 18}
    (&p1).SetInfoPointer()
    fmt.Println("函数调用后 = ", p1) //函数调用后 =  {yoyo 102 22}
  
    fmt.Println("==========================")
  
    p2 := Person{"mike", 'm', 18} //初始化
    //值作为接收者，值语义
    fmt.Println("函数调用前 = ", p2) //函数调用前 =  {mike 109 18}
    p2.SetInfoValue()
    fmt.Println("函数调用后 = ", p2) //函数调用后 =  {mike 109 18}
  }
  ```
- ## 3.5.  方法集
- 类型的方法集是指可以被该类型的值调用的所有方法的集合。
- 用实例实例 value 和 pointer 调用方法（含匿名字段）不受⽅法集约束，编译器编总是查找全部方法，并自动转换 receiver 实参。
- 一个指向自定义类型的值的指针，它的方法集由该类型定义的所有方法组成，无论这些方法接受的是一个值还是一个指针。
- 如果在指针上调用一个接受值的方法，Go语言会聪明地将该指针解引用，并将指针所指的底层值作为方法的接收者。
- 类型 *T ⽅法集包含全部 receiver T + *T ⽅法：
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //指针作为接收者，引用语义
  func (p *Person) SetInfoPointer() {
    (*p).name = "yoyo"
    p.sex = 'f'
    p.age = 22
  }
  
  //值作为接收者，值语义
  func (p Person) SetInfoValue() {
    p.name = "xxx"
    p.sex = 'm'
    p.age = 33
  }
  
  func main() {
    //p 为指针类型
    var p *Person = &Person{"mike", 'm', 18}
    p.SetInfoPointer() //func (p) SetInfoPointer()
  
    p.SetInfoValue()    //func (*p) SetInfoValue()
    (*p).SetInfoValue() //func (*p) SetInfoValue()
  }
  ```
- 一个自定义类型值的方法集则由为该类型定义的接收者类型为值类型的方法组成，但是不包含那些接收者类型为指针的方法。
- 但这种限制通常并不像这里所说的那样，因为如果我们只有一个值，仍然可以调用一个接收者为指针类型的方法，这可以借助于Go语言传值的地址能力实现。
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  //指针作为接收者，引用语义
  func (p *Person) SetInfoPointer() {
    (*p).name = "yoyo"
    p.sex = 'f'
    p.age = 22
  }
  
  //值作为接收者，值语义
  func (p Person) SetInfoValue() {
    p.name = "xxx"
    p.sex = 'm'
    p.age = 33
  }
  
  func main() {
    //p 为普通值类型
    var p Person = Person{"mike", 'm', 18}
    (&p).SetInfoPointer() //func (&p) SetInfoPointer()
    p.SetInfoPointer()    //func (&p) SetInfoPointer()
    
    p.SetInfoValue()      //func (p) SetInfoValue()
    (&p).SetInfoValue()   //func (*&p) SetInfoValue()
  ```
- ### 3.5.1.  表达式
- 类似于我们可以对函数进行赋值和传递一样，方法也可以进行赋值和传递。
- 根据调用者不同，方法分为两种表现形式：方法值和方法表达式。两者都可像普通函数那样赋值和传参，区别在于方法值绑定实例，⽽方法表达式则须显式传参。
- 方法值
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  func (p *Person) PrintInfoPointer() {
    fmt.Printf("%p, %v\n", p, p)
  }
  
  func (p Person) PrintInfoValue() {
    fmt.Printf("%p, %v\n", &p, p)
  }
  
  func main() {
    p := Person{"mike", 'm', 18}
    p.PrintInfoPointer() //0xc0420023e0, &{mike 109 18}
  
    pFunc1 := p.PrintInfoPointer //方法值，隐式传递 receiver
    pFunc1()                     //0xc0420023e0, &{mike 109 18}
  
    pFunc2 := p.PrintInfoValue
    pFunc2() //0xc042048420, {mike 109 18}
  }
  ```
- 方法表达式
  
  ```
  type Person struct {
    name string
    sex  byte
    age  int
  }
  
  func (p *Person) PrintInfoPointer() {
    fmt.Printf("%p, %v\n", p, p)
  }
  
  func (p Person) PrintInfoValue() {
    fmt.Printf("%p, %v\n", &p, p)
  }
  
  func main() {
    p := Person{"mike", 'm', 18}
    p.PrintInfoPointer() //0xc0420023e0, &{mike 109 18}
  
    //方法表达式， 须显式传参
    //func pFunc1(p *Person))
    pFunc1 := (*Person).PrintInfoPointer
    pFunc1(&p) //0xc0420023e0, &{mike 109 18}
  
    pFunc2 := Person.PrintInfoValue
    pFunc2(p) //0xc042002460, {mike 109 18}
  }
  ```
- ## 3.6.  接口
- 在Go语言中，接口(interface)是一个自定义类型，接口类型具体描述了一系列方法的集合。
- 接口类型是一种抽象的类型，它不会暴露出它所代表的对象的内部值的结构和这个对象支持的基础操作的集合，它们只会展示出它们自己的方法。因此接口类型不能将其实例化。
- Go通过接口实现了鸭子类型(duck-typing)：“当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子”。我们并不关心对象是什么类型，到底是不是鸭子，只关心行为。
- ### 3.6.1.  接口规范
- 接⼝命名习惯以 er 结尾
- 接口只有方法声明，没有实现，没有数据字段
- 接口可以匿名嵌入其它接口，或嵌入到结构中
- 接口是用来定义行为的类型。这些被定义的行为不由接口直接实现，而是通过方法由用户定义的类型实现，一个实现了这些方法的具体类型是这个接口类型的实例。
- 如果用户定义的类型实现了某个接口类型声明的一组方法，那么这个用户定义的类型的值就可以赋给这个接口类型的值。这个赋值会把用户定义的类型的值存入接口类型的值。
  
  ```
  type Humaner interface {
    SayHi()
  }
  
  type Student struct { //学生
    name  string
    score float64
  }
  
  //Student实现SayHi()方法
  func (s *Student) SayHi() {
    fmt.Printf("Student[%s, %f] say hi!!\n", s.name, s.score)
  }
  
  type Teacher struct { //老师
    name  string
    group string
  }
  
  //Teacher实现SayHi()方法
  func (t *Teacher) SayHi() {
    fmt.Printf("Teacher[%s, %s] say hi!!\n", t.name, t.group)
  }
  
  type MyStr string
  
  //MyStr实现SayHi()方法
  func (str MyStr) SayHi() {
    fmt.Printf("MyStr[%s] say hi!!\n", str)
  }
  
  //普通函数，参数为Humaner类型的变量i
  func WhoSayHi(i Humaner) {
    i.SayHi()
  }
  
  func main() {
    s := &Student{"mike", 88.88}
    t := &Teacher{"yoyo", "Go语言"}
    var tmp MyStr = "测试"
  
    s.SayHi()   //Student[mike, 88.880000] say hi!!
    t.SayHi()   //Teacher[yoyo, Go语言] say hi!!
    tmp.SayHi() //MyStr[测试] say hi!!
  
    //多态，调用同一接口，不同表现
    WhoSayHi(s)   //Student[mike, 88.880000] say hi!!
    WhoSayHi(t)   //Teacher[yoyo, Go语言] say hi!!
    WhoSayHi(tmp) //MyStr[测试] say hi!!
  
    x := make([]Humaner, 3)
    //这三个都是不同类型的元素，但是他们实现了interface同一个接口
    x[0], x[1], x[2] = s, t, tmp
    for _, value := range x {
        value.SayHi()
    }
    /*
        Student[mike, 88.880000] say hi!!
        Teacher[yoyo, Go语言] say hi!!
        MyStr[测试] say hi!!
    
  }
  ```
- ### 3.6.2.  接口组合
- 接口嵌入
	- 如果一个interface1作为interface2的一个嵌入字段，那么interface2隐式的包含了interface1里面的方法。
	  
	  ```
	  type Humaner interface {
	   SayHi()
	  }
	  
	  type Personer interface {
	   Humaner //这里想写了SayHi()一样
	   Sing(lyrics string)
	  }
	  
	  type Student struct { //学生
	   name  string
	   score float64
	  }
	  
	  //Student实现SayHi()方法
	  func (s *Student) SayHi() {
	   fmt.Printf("Student[%s, %f] say hi!!\n", s.name, s.score)
	  }
	  
	  //Student实现Sing()方法
	  func (s *Student) Sing(lyrics string) {
	   fmt.Printf("Student sing[%s]!!\n", lyrics)
	  }
	  
	  func main() {
	   s := &Student{"mike", 88.88}
	  
	   var i2 Personer
	   i2 = s
	   i2.SayHi()     //Student[mike, 88.880000] say hi!!
	   i2.Sing("学生哥") //Student sing[学生哥]!!
	  }
	  ```
- 接口转换
	- 超集接⼝对象可转换为⼦集接⼝，反之出错：
	  
	  ```
	  type Humaner interface {
	   SayHi()
	  }
	  
	  type Personer interface {
	   Humaner //这里像写了SayHi()一样
	   Sing(lyrics string)
	  }
	  
	  type Student struct { //学生
	   name  string
	   score float64
	  }
	  
	  //Student实现SayHi()方法
	  func (s *Student) SayHi() {
	   fmt.Printf("Student[%s, %f] say hi!!\n", s.name, s.score)
	  }
	  
	  //Student实现Sing()方法
	  func (s *Student) Sing(lyrics string) {
	   fmt.Printf("Student sing[%s]!!\n", lyrics)
	  }
	  
	  func main() {
	   //var i1 Humaner = &Student{"mike", 88.88}
	   //var i2 Personer = i1 //err
	  
	   //Personer为超集，Humaner为子集
	   var i1 Personer = &Student{"mike", 88.88}
	   var i2 Humaner = i1
	   i2.SayHi() //Student[mike, 88.880000] say hi!!
	  }
	  ```
- ### 3.6.3.  空接口
- 空接口(interface{})不包含任何的方法，正因为如此，所有的类型都实现了空接口，因此空接口可以存储任意类型的数值。它有点类似于C语言的void *类型。
  
  ```
  var v1 interface{} = 1     // 将int类型赋值给interface{}
    var v2 interface{} = "abc" // 将string类型赋值给interface{}
    var v3 interface{} = &v2   // 将*interface{}类型赋值给interface{}
    var v4 interface{} = struct{ X int }{1}
    var v5 interface{} = &struct{ X int }{1}
  ```
- 当函数可以接受任意的对象实例时，我们会将其声明为interface{}，最典型的例子是标准库fmt中PrintXXX系列的函数，例如：
  
  ```
  func Printf(fmt string, args ...interface{})
    func Println(args ...interface{})
  ```
- ### 3.6.4.  类型查询
- 我们知道interface的变量里面可以存储任意类型的数值(该类型实现了interface)。那么我们怎么反向知道这个变量里面实际保存了的是哪个类型的对象呢？目前常用的有两种方法：
	- comma-ok断言
	- switch测试
- comma-ok断言
	- Go语言里面有一个语法，可以直接判断是否是该类型的变量： value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。
	- 如果element里面确实存储了T类型的数值，那么ok返回true，否则返回false。
	  
	  ```
	  type Element interface{}
	  
	  type Person struct {
	   name string
	   age  int
	  }
	  
	  func main() {
	   list := make([]Element, 3)
	   list[0] = 1       // an int
	   list[1] = "Hello" // a string
	   list[2] = Person{"mike", 18}
	  
	   for index, element := range list {
	       if value, ok := element.(int); ok {
	           fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
	       } else if value, ok := element.(string); ok {
	           fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
	       } else if value, ok := element.(Person); ok {
	           fmt.Printf("list[%d] is a Person and its value is [%s, %d]\n", index, value.name, value.age)
	       } else {
	           fmt.Printf("list[%d] is of a different type\n", index)
	       }
	   }
	  
	   /*  打印结果：
	   list[0] is an int and its value is 1
	   list[1] is a string and its value is Hello
	   list[2] is a Person and its value is [mike, 18]
	   */
	  }
	  ```
- switch测试
  
  ```
  type Element interface{}
  
  type Person struct {
    name string
    age  int
  }
  
  func main() {
    list := make([]Element, 3)
    list[0] = 1       //an int
    list[1] = "Hello" //a string
    list[2] = Person{"mike", 18}
  
    for index, element := range list {
        switch value := element.(type) {
        case int:
            fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
        case string:
            fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
        case Person:
            fmt.Printf("list[%d] is a Person and its value is [%s, %d]\n", index, value.name, value.age)
        default:
            fmt.Println("list[%d] is of a different type", index)
        }
    }
  }
  ```
-