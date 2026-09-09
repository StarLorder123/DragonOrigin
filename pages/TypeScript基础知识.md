# 1.  第一个TypeScript脚本
- 安装Node.js和npm安装包管理器
- 全局安装TypeScript。以下这条命令会在全局安装一个TypeScript的解释器。
  
  ```
  npm install -g typescript
  ```
- 在项目文件夹中创建一个新的文件夹，并在终端中导航到该文件夹。然后运行以下命令来初始化一个新的TypeScript项目
  
  ```
  tsc --init
  ```
	- 在执行了以上的命令之后，这个文件夹内会生成一个tsconfig.json的文件。
- 创建一个名为 greet.ts 的文件。在文件中输入代码：
  
  ```
  function greet(name:string){
    console.log(`Hello, ${name}!`);
  }
  
  greet("John");
  ```
- 使用TypeScript编译器（tsc）将TypeScript代码编译成JavaScript代码。在命令行中输入tsc命令。
	- 输入了tsc命令之后，可以看到文件夹下出现了一个greet.js的文件，这也就意味着TypeScript语言最后还是会转化为JavaScript语言进行解释和执行的。而且转化过后的js文件是使用严格模式执行的。
- 使用node greet.js 执行该文件。
- # 2.  TypeScript基础语法
- ## 2.1.  基本数据类型
  
  | 数据类型 | 关键字 | 说明 |
  | 任意类型 | any | 声明为 any 的变量可以赋予任意类型的值。 |
  | 数字类型 | number | 双精度 64 位浮点值。它可以用来表示整数和分数。 |
  | 字符串类型 | string | 一个字符系列，使用单引号（**'**）或双引号（**"**）来表示字符串类型。反引号（**`**）来定义多行文本和内嵌表达式。 |
  | 布尔类型 | boolean | 表示逻辑值：true 和 false。 |
  | 数组类型 | 无 | 声明变量为数组。 |
  | 元组 | 无 | 元组类型用来表示已知元素数量和类型的数组，各元素的类型不必相同，对应位置的类型需要相同。 |
  | 枚举 | enum | 枚举类型用于定义数值集合。 |
  | void | void | 用于标识方法返回值的类型，表示该方法没有返回值。 |
  | null | null | 表示对象值缺失。 |
  | undefined | undefined | 用于初始化变量为一个未定义的值 |
  | never | never | never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。 |
- ## 2.2.  变量声明
- TypeScript 变量的命名规则：
	- 变量名称可以包含数字和字母。
	- 除了下划线 _ 和美元 $ 符号外，不能包含其他特殊字符，包括空格。
	- 变量名不能以数字开头。
- 类型断言
	- 类型断言可以用来手动指定一个值的类型，即允许变量从一种类型更改为另一种类型。
	- 当 S 类型是 T 类型的子集，或者 T 类型是 S 类型的子集时，S 能被成功断言成 T。这是为了在进行类型断言时提供额外的安全性，完全毫无根据的断言是危险的，如果你想这么做，你可以使用 any。
	- 它之所以不被称为**类型转换**，是因为转换通常意味着某种运行时的支持。但是，类型断言纯粹是一个编译时语法，同时，它也是一种为编译器提供关于如何分析代码的方法。
- 变量作用域
	- 变量作用域指定了变量定义的位置。程序中变量的可用性由变量作用域决定。
	- TypeScript 有以下几种作用域：
		- **全局作用域** − 全局变量定义在程序结构的外部，它可以在你代码的任何位置使用。
		- **类作用域** − 这个变量也可以称为 **字段**。类变量声明在一个类里头，但在类的方法外面。 该变量可以通过类的对象来访问。类变量也可以是静态的，静态的变量可以通过类名直接访问。
		- **局部作用域** − 局部变量，局部变量只能在声明它的一个代码块（如：方法）中使用。
- ## 2.3.  运算符
- ### 2.3.1.  算术运算符
  
  | 运算符 | 描述 | 例子 |
  | + | 加法 | x=y+2 |
  | - | 减法 | x=y-2 |
  | * | 乘法 | x=y*2 |
  | / | 除法 | x=y/2 |
  | % | 取模（余数） | x=y%2 |
  | ++ | 自增 | x=++y |
  | -- | 自减 | x=--y |
- ### 2.3.2.  关系运算符
  
  | 运算符 | 描述 | 比较 | 返回值 |
  | == | 等于 | x==8 | *false* |
  | != |  不等于 | x!=8 | *true* |
  | > |  大于 | x>8 | *false* |
  | < |  小于 | x<8 | *true* |
  | >= |  大于或等于 | x>=8 | *false* |
  | <= |  小于或等于 | x<=8 | *true* |
- ### 2.3.3.  逻辑运算符
  
  | 运算符 | 描述 | 例子 |
  | && | and | (x < 10 && y > 1) 为 true |
  | || | or | (x==5 || y==5) 为 false |
  | ! | not | !(x==y) 为 true |
- ### 2.3.4.  位运算符
  
  | 运算符 | 描述 | 例子 | 类似于 |
  | & | AND，按位与处理两个长度相同的二进制数，两个相应的二进位都为 1，该位的结果值才为 1，否则为 0。 | x = 5 & 1 | 0101 & 0001 |
  | | | OR，按位或处理两个长度相同的二进制数，两个相应的二进位中只要有一个为 1，该位的结果值为 1。 | x = 5 | 1 | 0101 | 0001 |
  | ~ | 取反，取反是一元运算符，对一个二进制数的每一位执行逻辑反操作。使数字 1 成为 0，0 成为 1。 | x = ~ 5 |  ~0101 |
  | ^ | 异或，按位异或运算，对等长二进制模式按位或二进制数的每一位执行逻辑异按位或操作。操作的结果是如果某位不同则该位为 1，否则该位为 0。 | x = 5 ^ 1 | 0101 ^ 0001 |
  | << | 左移，把 << 左边的运算数的各二进位全部左移若干位，由 << 右边的数指定移动的位数，高位丢弃，低位补 0。 | x = 5 << 1 | 0101 << 1 |
  | >> | 右移，把 >> 左边的运算数的各二进位全部右移若干位，>> 右边的数指定移动的位数。 | x = 5 >> 1 | 0101 >> 1 |
  | >>> | 无符号右移，与有符号右移位类似，除了左边一律使用0 补位。 | x = 2 >>> 1 | 0010 >>> 1 |
- ### 2.3.5.  赋值运算符
  
  | 运算符 | 例子 | 实例 | x 值 |
  | = (赋值) | x = y | x = y | x = 5 |
  | += (先进行加运算后赋值) | x += y | x = x + y | x = 15 |
  | -= (先进行减运算后赋值) | x -= y | x = x - y | x = 5 |
  | *= (先进行乘运算后赋值) | x *= y | x = x * y | x = 50 |
  | /= (先进行除运算后赋值) | x /= y | x = x / y | x = 2 |
- ### 2.3.6.  三元运算符
- 三元运算有 3 个操作数，并且需要判断布尔表达式的值。该运算符的主要是决定哪个值应该赋值给变量。
	- Test ? expr1 : expr2
	- Test − 指定的条件语句
	- expr1 − 如果条件语句 Test 返回 true 则返回该值
	- expr2 − 如果条件语句 Test 返回 false 则返回该值
- ### 2.3.7.  类型运算符
- typeof 运算符。typeof 是一元运算符，返回操作数的数据类型。
- instanceof 运算符用于判断对象是否为指定的类型
- ## 2.4.  条件语句
- 条件语句用于基于不同的条件来执行不同的动作。
- TypeScript 条件语句是通过一条或多条语句的执行结果（True 或 False）来决定执行的代码块。可以通过下图来简单了解条件语句的执行过程:
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1696118231962-fa84a33f-b215-45de-8799-a977d1c6cd57.png)
- 在 TypeScript 中，我们可使用以下条件语句：
	- **if 语句** - 只有当指定条件为 true 时，使用该语句来执行代码
	- **if...else 语句** - 当条件为 true 时执行代码，当条件为 false 时执行其他代码
	- **if...else if....else 语句**- 使用该语句来选择多个代码块之一来执行
	- **switch 语句** - 使用该语句来选择多个代码块之一来执行
- if语句的表达式如下：
  
  ```
  if(boolean_expression){
    # 在布尔表达式 boolean_expression 为 true 执行
  }
  ```
- if...else 语句的表达如下：
  
  ```
  if(boolean_expression){
   # 在布尔表达式 boolean_expression 为 true 执行
  }else{
   # 在布尔表达式 boolean_expression 为 false 执行
  }
  ```
- if...else if ... else 语句的表达式如下：
  
  ```
  if(boolean_expression 1) {
    # 在布尔表达式 boolean_expression 1 为 true 执行
  } else if( boolean_expression 2) {
    # 在布尔表达式 boolean_expression 2 为 true 执行
  } else if( boolean_expression 3) {
    # 在布尔表达式 boolean_expression 3 为 true 执行
  } else {
    # 布尔表达式的条件都为 false 时执行
  }
  ```
- switch...case 语句的表达式如下：
  
  ```
  switch(expression){
    case constant-expression  :
       statement(s);
       break; /* 可选的 */
    case constant-expression  :
       statement(s);
       break; /* 可选的 */
  
    /* 您可以有任意数量的 case 语句 */
    default : /* 可选的 */
       statement(s);
  }
  ```
- **switch** 语句必须遵循下面的规则：
	- **switch** 语句中的 **expression** 是一个要被比较的表达式，可以是任何类型，包括基本数据类型（如 number、string、boolean）、对象类型（如 object、Array、Map）以及自定义类型（如 class、interface、enum）等。
	- 在一个 switch 中可以有任意数量的 case 语句。每个 case 后跟一个要比较的值和一个冒号。
	- case 的 **constant-expression** 必须与 switch 中的变量 expression 具有相同或兼容的数据类型。
	- 当被测试的变量等于 case 中的常量时，case 后跟的语句将被执行，直到遇到 **break** 语句为止。
	- 当遇到 **break** 语句时，switch 终止，控制流将跳转到 switch 语句后的下一行。
	- 不是每一个 case 都需要包含 **break**。如果 case 语句不包含 **break**，控制流将会 *继续* 后续的 case，直到遇到 break 为止。
	- 一个 **switch** 语句可以有一个可选的 **default** case，出现在 switch 的结尾。default 关键字则表示当表达式的值与所有 case 值都不匹配时执行的代码块。default case 中的 **break** 语句不是必需的
- ## 2.5.  循环语句
- ### 2.5.1.  for循环
- 语法格式如下：
  
  ```
  for ( init; condition; increment ){
    statement(s);
  }
  ```
- 下面是 for 循环的控制流程解析：
- **init** 会首先被执行，且只会执行一次。这一步允许您声明并初始化任何循环控制变量。您也可以不在这里写任何语句，只要有一个分号出现即可。
- 接下来，会判断 **condition**。如果为 true，则执行循环主体。如果为 false，则不执行循环主体，且控制流会跳转到紧接着 for 循环的下一条语句。
- 在执行完 for 循环主体后，控制流会跳回上面的 **increment** 语句。该语句允许您更新循环控制变量。该语句可以留空，只要在条件后有一个分号出现即可。
- 条件再次被判断。如果为 true，则执行循环，这个过程会不断重复（循环主体，然后增加步值，再然后重新判断条件）。在条件变为 false 时，for 循环终止。
- ### 2.5.2.  for ... in ... 循环
- 语法格式如下所示：
  
  ```
  for (var val in list) { 
    //语句 
  }
  ```
- val 需要为 string 或 any 类型。
- ### 2.5.3.  for…of 、forEach、every 和 some 循环
- for ... of 循环语法
  
  ```
  let someArray = [1, "string", false];
  
  for (let entry of someArray) {
    console.log(entry); // 1, "string", false
  }
  ```
- forEach 循环语法
  
  ```
  let list = [4, 5, 6];
  list.forEach((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
  });
  ```
- every和some语法
  
  ```
  let list = [4, 5, 6];
  list.every((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
    return true; // Continues
    // Return false will quit the iteration
  });
  ```
- ### 2.5.4.  while循环
- while 语句在给定条件为 true 时，重复执行语句或语句组。循环主体执行之前会先测试条件。
- 语法如下
  
  ```
  while(condition)
  {
   statement(s);
  }
  ```
- statement(s) 可以是一个单独的语句，也可以是几个语句组成的代码块。condition 可以是任意的表达式，当条件为 true 时执行循环。 当条件为 false 时，程序流将退出循环。*while* 循环的关键点是循环可能一次都不会执行。当条件为 false 时，会跳过循环主体，直接执行紧接着 while 循环的下一条语句。
- ### 2.5.5.  do ... while 循环
- **do...while** 循环是在循环的尾部检查它的条件。语法如下所示
  
  ```
  do
  {
   statement(s);
  }while( condition );
  ```
- 条件表达式出现在循环的尾部，所以循环中的 statement(s) 会在条件被测试之前至少执行一次。如果条件为 true，控制流会跳转回上面的 do，然后重新执行循环中的 statement(s)。这个过程会不断重复，直到给定条件变为 false 为止。
- ### 2.5.6.  break和continue
- **break** 语句有以下两种用法：
	- 当 **break** 语句出现在一个循环内时，循环会立即终止，且程序流将继续执行紧接着循环的下一条语句。
	- 它可用于终止 **switch** 语句中的一个 case。
- 如果您使用的是嵌套循环（即一个循环内嵌套另一个循环），break 语句会停止执行最内层的循环，然后开始执行该块之后的下一行代码。
- **continue** 语句有点像 **break** 语句。但它不是强制终止，continue 会跳过当前循环中的代码，强迫开始下一次循环。
	- 对于 **for** 循环，**continue** 语句执行后自增语句仍然会执行。对于 **while** 和 **do...while** 循环，**continue** 语句重新执行条件判断语句。
- ### 2.5.7.  无限循环
- 无限循环就是一直在运行不会停止的循环。 for 和 while 循环都可以创建无限循环。
- for 创建无限循环语法格式：
  
  ```
  for(;;) { 
   // 语句
  }
  ```
- while 创建无限循环语法格式
  
  ```
  while(true) { 
   // 语句
  }
  ```
- ## 2.6.  函数
- 函数是一组一起执行一个任务的语句。
- 函数声明告诉编译器函数的名称、返回类型和参数。函数定义提供了函数的实际主体。
- ### 2.6.1.  函数定义
- 语法格式如下所示：
  
  ```
  function function_name()
  {
    // 执行代码
  }
  ```
- ### 2.6.2.  函数调用
- 函数只有通过调用才可以执行函数内的代码。语法格式如下所示：
  
  ```
  function_name()
  ```
- ### 2.6.3.  函数返回值
- 语法格式如下所示：
  
  ```
  function function_name():return_type { 
    // 语句
    return value; 
  }
  ```
- return_type 是返回值的类型。
- return 关键词后跟着要返回的结果。
- 一般情况下，一个函数只有一个 return 语句。
- 返回值的类型需要与函数定义的返回类型(return_type)一致。
- ### 2.6.4.  带参函数
- 在调用函数时，您可以向其传递值，这些值被称为参数。这些参数可以在函数中使用。
- 可以向函数发送多个参数，每个参数使用逗号 **,** 分隔。语法格式如下所示：
  
  ```
  function func_name( param1 [:datatype], param2 [:datatype]) {   
  }
  ```
- ### 2.6.5.  可选参数和默认参数
- 在 TypeScript 函数里，如果我们定义了参数，则我们必须传入这些参数，除非将这些参数设置为可选，可选参数使用问号标识 ？。可选参数必须跟在必需参数后面。
- 默认参数。可以设置参数的默认值，这样在调用函数的时候，如果不传入该参数的值，则使用默认参数，语法格式为：
  
  ```
  function function_name(param1[:type],param2[:type] = default_value) { 
  }
  ```
- 剩余参数。剩余参数语法允许我们将一个不确定数量的参数作为一个数组传入。示例：
  
  ```
  function addNumbers(...nums:number[]) {  
    var i;   
    var sum:number = 0; 
    
    for(i = 0;i<nums.length;i++) { 
       sum = sum + nums[i]; 
    } 
    console.log("和为：",sum) 
  } 
  addNumbers(1,2,3) 
  addNumbers(10,10,10,10,10)
  ```
- ### 2.6.6.  匿名函数
- 匿名函数是一个没有函数名的函数。匿名函数在程序运行时动态声明，除了没有函数名外，其他的与标准函数一样。
- 代码格式如下：
  
  ```
  var res = function( [arguments] ) { ... }
  ```
- ## 2.7.  常用数据类型
- ### 2.7.1.  Number类型
  
  | 序号 | 属性 & 描述 |
  | 1. | **MAX_VALUE**可表示的最大的数，MAX_VALUE 属性值接近于 1.79E+308。大于 MAX_VALUE 的值代表 "Infinity"。 |
  | 2. | **MIN_VALUE**可表示的最小的数，即最接近 0 的正数 (实际上不会变成 0)。最大的负数是 -MIN_VALUE，MIN_VALUE 的值约为 5e-324。小于 MIN_VALUE ("underflow values") 的值将会转换为 0。 |
  | 3. | **NaN**非数字值（Not-A-Number）。 |
  | 4. | **NEGATIVE_INFINITY**负无穷大，溢出时返回该值。该值小于 MIN_VALUE。 |
  | 5. | **POSITIVE_INFINITY**正无穷大，溢出时返回该值。该值大于 MAX_VALUE。 |
  | 6. | **prototype**Number 对象的静态属性。使您有能力向对象添加属性和方法。 |
  | 7. | **constructor**返回对创建此对象的 Number 函数的引用。 |
- Number对象的方法
  
  | 序号 | 方法 & 描述 | 实例 |
  | 1. | toExponential()把对象的值转换为指数计数法。 | //toExponential()  var num1 = 1225.30  var val = num1.toExponential();  console.log(val) // 输出： 1.2253e+3 |
  | 2. | toFixed()把数字转换为字符串，并对小数点指定位数。 | var num3 = 177.234  console.log("num3.toFixed() 为 "+num3.toFixed())    // 输出：177 console.log("num3.toFixed(2) 为 "+num3.toFixed(2))  // 输出：177.23 console.log("num3.toFixed(6) 为 "+num3.toFixed(6))  // 输出：177.234000 |
  | 3. | toLocaleString()把数字转换为字符串，使用本地数字格式顺序。 | var num = new Number(177.1234);  console.log( num.toLocaleString());  // 输出：177.1234 |
  | 4. | toPrecision()把数字格式化为指定的长度。 | var num = new Number(7.123456);  console.log(num.toPrecision());  // 输出：7.123456  console.log(num.toPrecision(1)); // 输出：7 console.log(num.toPrecision(2)); // 输出：7.1 |
  | 5. | toString()把数字转换为字符串，使用指定的基数。数字的基数是 2 ~ 36 之间的整数。若省略该参数，则使用基数 10。 | var num = new Number(10);  console.log(num.toString());  // 输出10进制：10 console.log(num.toString(2)); // 输出2进制：1010 console.log(num.toString(8)); // 输出8进制：12 |
  | 6. | valueOf()返回一个 Number 对象的原始数字值。 | var num = new Number(10);  console.log(num.valueOf()); // 输出：10 |
- ### 2.7.2.  String类型
- 下表列出了 String 对象支持的属性：
  
  | 序号 | 属性 & 描述 | 实例 |
  | 1. | constructor对创建该对象的函数的引用。 | var str = new String( "This is string" );  console.log("str.constructor is:" + str.constructor)输出结果：str.constructor is:function String() { [native code] } |
  | 2. | length返回字符串的长度。 | var uname = new String("Hello World")  console.log("Length "+uname.length)  // 输出 11 |
  | 3. | prototype允许您向对象添加属性和方法。 | function employee(id:number,name:string) {      this.id = id      this.name = name   }   var emp = new employee(123,"admin")   employee.prototype.email="admin@runoob.com" // 添加属性 email  console.log("员工号: "+emp.id)   console.log("员工姓名: "+emp.name)   console.log("员工邮箱: "+emp.email) |
- 下表列出了 String 对象支持的方法：
  
  | 序号 | 方法 & 描述 | 实例 |
  | 1. | charAt()返回在指定位置的字符。 | var str = new String("RUNOOB");  console.log("str.charAt(0) 为:" + str.charAt(0)); // R console.log("str.charAt(1) 为:" + str.charAt(1)); // U  console.log("str.charAt(2) 为:" + str.charAt(2)); // N  console.log("str.charAt(3) 为:" + str.charAt(3)); // O  console.log("str.charAt(4) 为:" + str.charAt(4)); // O  console.log("str.charAt(5) 为:" + str.charAt(5)); // B |
  | 2. | charCodeAt()返回在指定的位置的字符的 Unicode 编码。 | var str = new String("RUNOOB");  console.log("str.charCodeAt(0) 为:" + str.charCodeAt(0)); // 82 console.log("str.charCodeAt(1) 为:" + str.charCodeAt(1)); // 85  console.log("str.charCodeAt(2) 为:" + str.charCodeAt(2)); // 78  console.log("str.charCodeAt(3) 为:" + str.charCodeAt(3)); // 79  console.log("str.charCodeAt(4) 为:" + str.charCodeAt(4)); // 79 console.log("str.charCodeAt(5) 为:" + str.charCodeAt(5)); // 66 |
  | 3. | concat()连接两个或更多字符串，并返回新的字符串。 | var str1 = new String( "RUNOOB" );  var str2 = new String( "GOOGLE" );  var str3 = str1.concat( str2 );  console.log("str1 + str2 : "+str3) // RUNOOBGOOGLE |
  | 4. | indexOf()返回某个指定的字符串值在字符串中首次出现的位置。 | var str1 = new String( "RUNOOB" );  var index = str1.indexOf( "OO" );  console.log("查找的字符串位置 :" + index );  // 3 |
  | 5. | lastIndexOf()从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置。 | var str1 = new String( "This is string one and again string" );  var index = str1.lastIndexOf( "string" ); console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 29      index = str1.lastIndexOf( "one" );  console.log("lastIndexOf 查找到的最后字符串位置 :" + index ); // 15 |
  | 6. | localeCompare()用本地特定的顺序来比较两个字符串。 | var str1 = new String( "This is beautiful string" );    var index = str1.localeCompare( "This is beautiful string");   console.log("localeCompare first :" + index );  // 0 |
  | 7. | **match()**查找找到一个或多个正则表达式的匹配。 | var str="The rain in SPAIN stays mainly in the plain";  var n=str.match(/ain/g);  // ain,ain,ain |
  | 8. | replace()替换与正则表达式匹配的子串 | var re = /(\w+)\s(\w+)/;  var str = "zara ali";  var newstr = str.replace(re, "$2, $1");  console.log(newstr); // ali, zara |
  | 9. | search()检索与正则表达式相匹配的值 | var re = /apples/gi;  var str = "Apples are round, and apples are juicy."; if (str.search(re) == -1 ) {     console.log("Does not contain Apples" );  } else {     console.log("Contains Apples" );  }  |
  | 10. | slice()提取字符串的片断，并在新的字符串中返回被提取的部分。 |  |
  | 11. | split()把字符串分割为子字符串数组。 | var str = "Apples are round, and apples are juicy.";  var splitted = str.split(" ", 3);  console.log(splitted)  // [ 'Apples', 'are', 'round,' ] |
  | 12. | substr()从起始索引号提取字符串中指定数目的字符。 |  |
  | 13. | substring()提取字符串中两个指定的索引号之间的字符。 | var str = "RUNOOB GOOGLE TAOBAO FACEBOOK";  console.log("(1,2): "    + str.substring(1,2));   // U console.log("(0,10): "   + str.substring(0, 10)); // RUNOOB GOO console.log("(5): "      + str.substring(5));     // B GOOGLE TAOBAO FACEBOOK |
  | 14. | toLocaleLowerCase()根据主机的语言环境把字符串转换为小写，只有几种语言（如土耳其语）具有地方特有的大小写映射。 | var str = "Runoob Google";  console.log(str.toLocaleLowerCase( ));  // runoob google |
  | 15. | toLocaleUpperCase()据主机的语言环境把字符串转换为大写，只有几种语言（如土耳其语）具有地方特有的大小写映射。 | var str = "Runoob Google";  console.log(str.toLocaleUpperCase( ));  // RUNOOB GOOGLE |
  | 16. | toLowerCase()把字符串转换为小写。 | var str = "Runoob Google";  console.log(str.toLowerCase( ));  // runoob google |
  | 17. | toString()返回字符串。 | var str = "Runoob";  console.log(str.toString( )); // Runoob |
  | 18. | toUpperCase()把字符串转换为大写。 | var str = "Runoob Google";  console.log(str.toUpperCase( ));  // RUNOOB GOOGLE |
  | 19. | valueOf()返回指定字符串对象的原始值。 | var str = new String("Runoob");  console.log(str.valueOf( ));  // Runoob |
- ### 2.7.3.  Array类型
- TypeScript 声明数组的语法格式如下所示：
  
  ```
  var array_name[:datatype];        //声明 
  array_name = [val1,val2,valn..]   //初始化
  ```
- 或者直接在声明时初始化：
  
  ```
  var array_name[:datatype] = [val1,val2…valn]
  ```
- 也可以使用 Array 对象创建数组。
- Array 对象的构造函数接受以下两种值：
	- 表示数组大小的数值。
	- 初始化的数组列表，元素使用逗号分隔值。
	  
	  ```
	  var arr_names:number[] = new Array(4)  
	  
	  for(var i = 0; i<arr_names.length; i++) { 
	       arr_names[i] = i * 2 
	       console.log(arr_names[i]) 
	  }
	  ```
- 数组的一些方法：
  
  | 序号 | 方法 & 描述 | 实例 |
  | 1. | concat()连接两个或更多的数组，并返回结果。 | var alpha = ["a", "b", "c"];  var numeric = [1, 2, 3]; var alphaNumeric = alpha.concat(numeric);  console.log("alphaNumeric : " + alphaNumeric );    // a,b,c,1,2,3    |
  | 2. | every()检测数值元素的每个元素是否都符合条件。 | function isBigEnough(element, index, array) {          return (element >= 10);  }           var passed = [12, 5, 8, 130, 44].every(isBigEnough);  console.log("Test Value : " + passed ); // false |
  | 3. | filter()检测数值元素，并返回符合条件所有元素的数组。 | function isBigEnough(element, index, array) {     return (element >= 10);  }             var passed = [12, 5, 8, 130, 44].filter(isBigEnough);  console.log("Test Value : " + passed ); // 12,130,44 |
  | 4. | forEach()数组每个元素都执行一次回调函数。 | let num = [7, 8, 9]; num.forEach(function (value) {     console.log(value); }); 编译成 JavaScript 代码：var num = [7, 8, 9]; num.forEach(function (value) {     console.log(value);  // 7   8   9 }); |
  | 5. | indexOf()搜索数组中的元素，并返回它所在的位置。如果搜索不到，返回值 -1，代表没有此项。 | var index = [12, 5, 8, 130, 44].indexOf(8);  console.log("index is : " + index );  // 2 |
  | 6. | join()把数组的所有元素放入一个字符串。 | var arr = new Array("Google","Runoob","Taobao");             var str = arr.join();  console.log("str : " + str );  // Google,Runoob,Taobao            var str = arr.join(", ");  console.log("str : " + str );  // Google, Runoob, Taobao            var str = arr.join(" + ");  console.log("str : " + str );  // Google + Runoob + Taobao |
  | 7. | lastIndexOf()返回一个指定的字符串值最后出现的位置，在一个字符串中的指定位置从后向前搜索。 | var index = [12, 5, 8, 130, 44].lastIndexOf(8);  console.log("index is : " + index );  // 2 |
  | 8. | map()通过指定函数处理数组的每个元素，并返回处理后的数组。 | var numbers = [1, 4, 9];  var roots = numbers.map(Math.sqrt);  console.log("roots is : " + roots );  // 1,2,3 |
  | 9. | pop()删除数组的最后一个元素并返回删除的元素。 | var numbers = [1, 4, 9];             var element = numbers.pop();  console.log("element is : " + element );  // 9            var element = numbers.pop();  console.log("element is : " + element );  // 4 |
  | 10. | push()向数组的末尾添加一个或更多元素，并返回新的长度。 | var numbers = new Array(1, 4, 9);  var length = numbers.push(10);  console.log("new numbers is : " + numbers );  // 1,4,9,10  length = numbers.push(20);  console.log("new numbers is : " + numbers );  // 1,4,9,10,20 |
  | 11. | reduce()将数组元素计算为一个值（从左到右）。 | var total = [0, 1, 2, 3].reduce(function(a, b){ return a + b; });  console.log("total is : " + total );  // 6 |
  | 12. | reduceRight()将数组元素计算为一个值（从右到左）。 | var total = [0, 1, 2, 3].reduceRight(function(a, b){ return a + b; });  console.log("total is : " + total );  // 6 |
  | 13. | reverse()反转数组的元素顺序。 | var arr = [0, 1, 2, 3].reverse();  console.log("Reversed array is : " + arr );  // 3,2,1,0 |
  | 14. | shift()删除并返回数组的第一个元素。 | var arr = [10, 1, 2, 3].shift();  console.log("Shifted value is : " + arr );  // 10 |
  | 15. | slice()选取数组的的一部分，并返回一个新数组。 | var arr = ["orange", "mango", "banana", "sugar", "tea"];  console.log("arr.slice( 1, 2) : " + arr.slice( 1, 2) );  // mango console.log("arr.slice( 1, 3) : " + arr.slice( 1, 3) );  // mango,banana |
  | 16. | some()检测数组元素中是否有元素符合指定条件。 | function isBigEnough(element, index, array) {     return (element >= 10);             }             var retval = [2, 5, 8, 1, 4].some(isBigEnough); console.log("Returned value is : " + retval );  // false            var retval = [12, 5, 8, 1, 4].some(isBigEnough);  console.log("Returned value is : " + retval );  // true |
  | 17. | sort()对数组的元素进行排序。 | var arr = new Array("orange", "mango", "banana", "sugar");  var sorted = arr.sort();  console.log("Returned string is : " + sorted );  // banana,mango,orange,sugar |
  | 18. | splice()从数组中添加或删除元素。 | var arr = ["orange", "mango", "banana", "sugar", "tea"];   var removed = arr.splice(2, 0, "water");   console.log("After adding 1: " + arr );    // orange,mango,water,banana,sugar,tea  console.log("removed is: " + removed);             removed = arr.splice(3, 1);   console.log("After removing 1: " + arr );  // orange,mango,water,sugar,tea  console.log("removed is: " + removed);  // banana |
  | 19. | toString()把数组转换为字符串，并返回结果。 | var arr = new Array("orange", "mango", "banana", "sugar");          var str = arr.toString();  console.log("Returned string is : " + str );  // orange,mango,banana,sugar |
  | 20. | unshift()向数组的开头添加一个或更多元素，并返回新的长度。 | var arr = new Array("orange", "mango", "banana", "sugar");  var length = arr.unshift("water");  console.log("Returned array is : " + arr );  // water,orange,mango,banana,sugar  console.log("Length of the array is : " + length ); // 5 |
- ### 2.7.4.  Map对象
- Map 对象保存键值对，并且能够记住键的原始插入顺序。任何值(对象或者原始值) 都可以作为一个键或一个值。Map 是 ES6 中引入的一种新的数据结构。
- 创建Map
	- TypeScript 使用 Map 类型和 new 关键字来创建 Map：
	- let myMap = new Map();
- 初始化Map
	- let myMap = new Map([         ["key1", "value1"],         ["key2", "value2"]     ]);
- Map相关的函数与属性
	- **map.clear()** – 移除 Map 对象的所有键/值对 。
	- **map.set()** – 设置键值对，返回该 Map 对象。
	- **map.get()** – 返回键对应的值，如果不存在，则返回 undefined。
	- **map.has()** – 返回一个布尔值，用于判断 Map 中是否包含键对应的值。
	- **map.delete()** – 删除 Map 中的元素，删除成功返回 true，失败返回 false。
	- **map.size** – 返回 Map 对象键/值对的数量。
	- **map.keys()** - 返回一个 Iterator 对象， 包含了 Map 对象中每个元素的键 。
	- **map.values()** – 返回一个新的Iterator对象，包含了Map对象中每个元素的值 。
- 迭代Map
	- Map 对象中的元素是按顺序插入的，我们可以迭代 Map 对象，每一次迭代返回 [key, value] 数组。
	- TypeScript使用 **for...of** 来实现迭代：
	  
	  ```
	  let nameSiteMapping = new Map();
	  
	  nameSiteMapping.set("Google", 1);
	  nameSiteMapping.set("Runoob", 2);
	  nameSiteMapping.set("Taobao", 3);
	  
	  // 迭代 Map 中的 key
	  for (let key of nameSiteMapping.keys()) {
	  console.log(key);                  
	  }
	  
	  // 迭代 Map 中的 value
	  for (let value of nameSiteMapping.values()) {
	  console.log(value);                 
	  }
	  
	  // 迭代 Map 中的 key => value
	  for (let entry of nameSiteMapping.entries()) {
	  console.log(entry[0], entry[1]);   
	  }
	  
	  // 使用对象解析
	  for (let [key, value] of nameSiteMapping) {
	  console.log(key, value);            
	  }
	  ```
- ### 2.7.5.  TypeScript元组
- 创建元组的语法格式如下：
	- var tuple_name = [value1,value2,value3,…value n]
- 访问元组
- 元组中元素使用索引来访问，第一个元素的索引值为 0，第二个为 1，以此类推第 n 个为 n-1，语法格式如下:
	- tuple_name[index]
- 操作元组
- 使用以下两个函数向元组添加新元素或者删除元素：
	- push() 向元组添加元素，添加在最后面。
	- pop() 从元组中移除元素（最后一个），并返回移除的元素。
- 更新元组
- 元组是可变的，这意味着我们可以对元组进行更新操作：
  
  ```
  var mytuple = [10, "Runoob", "Taobao", "Google"]; // 创建一个元组
  console.log("元组的第一个元素为：" + mytuple[0]) 
  
  // 更新元组元素
  mytuple[0] = 121     
  console.log("元组中的第一个元素更新为："+ mytuple[0])
  ```
- 解构元组
	- 把元组元素赋值给变量
	  
	  ```
	  var a =[10,"Runoob"] 
	  var [b,c] = a 
	  console.log( b )    
	  console.log( c )
	  ```
- ### 2.7.6.  TypeScript联合类型
- 联合类型（Union Types）可以通过管道(|)将变量设置多种类型，赋值时可以根据设置的类型来赋值。
- **注意**：只能赋值指定的类型，如果赋值其它类型就会报错。
- 创建联合类型的语法格式如下：
  
  ```
  Type1|Type2|Type3
  ```
- ## 2.8.  接口
- 接口是一系列抽象方法的声明，是一些方法特征的集合，这些方法都应该是抽象的，需要由具体的类去实现，然后第三方就可以通过这组抽象方法调用，让具体的类执行具体的方法。
- TypeScript 接口定义如下：
  
  ```
  interface interface_name { 
  }
  ```
- 接口继承
- 接口继承就是说接口可以通过其他接口来扩展自己。
- Typescript 允许接口继承多个接口。
- 继承使用关键字 **extends**。
- 单接口继承语法格式：
  
  ```
  Child_interface_name extends super_interface_name
  ```
- ## 2.9.  类
- TypeScript 是面向对象的 JavaScript。
- 类描述了所创建的对象共同的属性和方法。
- TypeScript 支持面向对象的所有特性，比如 类、接口等。
- TypeScript 类定义方式如下：
  
  ```
  class class_name { 
    // 类作用域
  }
  ```
- 定义类的关键字为 class，后面紧跟类名，类可以包含以下几个模块（类的数据成员）：
	- **字段** − 字段是类里面声明的变量。字段表示对象的有关数据。
	- **构造函数** − 类实例化时调用，可以为类的对象分配内存。
	- **方法** − 方法为对象要执行的操作。
- 使用 new 关键字来实例化类的对象，语法格式如下：
  
  ```
  var object_name = new class_name([ arguments ])
  ```
- TypeScript 支持继承类，即我们可以在创建类的时候继承一个已存在的类，这个已存在的类称为父类，继承它的类称为子类。
- 类继承使用关键字 **extends**，子类除了不能继承父类的私有成员(方法和属性)和构造函数，其他的都可以继承。
- TypeScript 一次只能继承一个类，不支持继承多个类，但 TypeScript 支持多重继承（A 继承 B，B 继承 C）。
- 类继承后，子类可以对父类的方法重新定义，这个过程称之为方法的重写。
- static 关键字用于定义类的数据成员（属性和方法）为静态的，静态成员可以直接通过类名调用。
- 在类中，使用#+函数名，表示为类内的私有方法，不能被外部调用只能被内部调用。调用的时候，也需要将#带上。
- ## 2.10.  模块
- TypeScript 模块的设计理念是可以更换的组织代码。
- 模块是在其自身的作用域里执行，并不是在全局作用域，这意味着定义在模块里面的变量、函数和类等在模块外部是不可见的，除非明确地使用 export 导出它们。类似地，我们必须通过 import 导入其他模块导出的变量、函数、类等。
- 两个模块之间的关系是通过在文件级别上使用 import 和 export 建立的。
- 模块使用模块加载器去导入其它的模块。 在运行时，模块加载器的作用是在执行此模块代码前去查找并执行这个模块的所有依赖。 大家最熟知的JavaScript模块加载器是服务于 Node.js 的 CommonJS 和服务于 Web 应用的 Require.js。
- ## 2.11.  声明文件
- TypeScript 作为 JavaScript 的超集，在开发过程中不可避免要引用其他第三方的 JavaScript 的库。虽然通过直接引用可以调用库的类和方法，但是却无法使用TypeScript 诸如类型检查等特性功能。为了解决这个问题，需要将这些库里的函数和方法体去掉后只保留导出类型声明，而产生了一个描述 JavaScript 库和模块信息的声明文件。通过引用这个声明文件，就可以借用 TypeScript 的各种特性来使用库文件了。
- 示例
  
  ```
  declare module Runoob { 
   export class Calc { 
      doSum(limit:number) : number; 
   }
  }
  ```
  
  ```
  /// <reference path = "Calc.d.ts" /> 
  var obj = new Runoob.Calc(); 
  // obj.doSum("Hello"); // 编译错误
  console.log(obj.doSum(10));
  ```
- # 3.  构建TypeScript项目
- ## 3.1.  初始化项目
  
  ```
  mkdir ts-project && cd ts-project && npm init -y && npm i typescript -D && npx tsc --init
  ```
- 这条命令的意思是在当前目录下创建一个 ts-project 目录，然后进入 ts-project 目录执行 npm init -y 初始话目录产生 package.json 文件，之后运行 npm i typescript -D 在开发环境安装 typescript 包，之后执行 npx tsc --init 生成 tsconfig.json 文件
- 之后所有的操作都以 ts-project 为根目录
  
  ```
  mkdir src && touch src/index.ts
  ```
- 新建 src 目录作为项目的源码目录（开发目录），并在 src 目录下创建 index.ts 文件作为项目的入口文件
- ## 3.2.  设置 tsconfig.json
- 如果一个目录下存在 tsconfig.json 文件，那就意味着这个目录是 typescirpt 项目的根目录，tsconfig.json 文件中指定了用来编译项目的根文件和编译选项，使用 tsc --init 生成的 tsconfig.json 文件包含了大量的选项，其中大部分都被注释掉了，一般我们只需要配置如下内容即可：
  
  ```
  {
  "compileOnSave": true,
  "compilerOptions": {
    "target": "ES2018",
    "module": "commonjs",
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "inlineSourceMap":true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "stripInternal": true,
    "pretty": true,
    "declaration": true,
    "outDir": "lib",
    "baseUrl": "./",
    "paths": {
      "*": ["src/*"]
    }
  },
  "exclude": [
    "lib",
    "node_modules"
  ]
  }
  ```
- ## 3.3.  开发环境准备
  
  ```
  npm i @types/node -D
  ```
- 这个是 node.js 的类型定义包
  
  ```
  npm i ts-node-dev -D
  ```
- 在 package.json 的 scripts 中增加如下内容
  
  ```
  {
  "scripts": {
    "dev:comment": "启动开发环境",
    "dev": "ts-node-dev --respawn --transpile-only src/index.ts"
  }
  }
  ```
- 执行 npm run dev 即可启动开发环境，并且修改文件时可实时编译
- ## 3.4.  保证代码质量
- 代码质量对于一个系统的可维护性、可迭代性至关重要，特别是在多人协作一个大型项目中，如果没有把控代码质量的工具，每人一套编码风格，这样的系统在后期的维护难度可想而知，基本上会成为一个难以迭代升级的祖传系统，除了重写别无他法。
- 因此控制代码质量的工具应运而生，而 ESLint 当属其中的佼佼者，熬死了各路的竞争者；typescript 之前还在使用 TSLint，但在 2019 年 1 月 官方决定全面采用 ESLint 作为代码检查工具。
- 采用社区的开源配置方案 eslint-config-standard，简单直接，足以 hold 住大部分项目了
  
  ```
  npx eslint --init
  ```
- 以上流程走完以后在项目根目录会多出来一个 .eslintrc.js 文件，接下来在 package.json 的 scripts 中增加如下配置
  
  ```
  {
  "scripts": {
    "eslint:comment": "使用 ESLint 检查并自动修复 src 目录下所有扩展名为 .ts 的文件",
    "eslint": "eslint --fix src --ext .ts --max-warnings=0"
  }
  }
  ```
- ## 3.5.  写在最后
  
  参考资料：
  
  [https://juejin.cn/post/6901552013717438472#heading-10](https://juejin.cn/post/6901552013717438472#heading-10)