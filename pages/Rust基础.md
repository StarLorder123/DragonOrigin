# 1.  Rust环境搭建
- 安装Rust编译工具
- Rust编译工具：[https://www.rust-lang.org/zh-CN/tools/install](https://www.rust-lang.org/zh-CN/tools/install)
- Rust 的编译工具依赖 C 语言的编译工具，这意味着你的电脑上至少已经存在一个 C 语言的编译环境。如果你使用的是 Linux 系统，往往已经具备了 GCC 或 clang。如果你使用的是 macOS，需要安装 Xcode。如果你是用的是 Windows 操作系统，你需要安装 Visual Studio 2013 或以上的环境（需要 C/C++ 支持）以使用 MSVC 或安装 MinGW + GCC 编译环境（Cygwin 还没有测试）。
- ## 1.1.  安装Rust编译工具
- Rust 编译工具推荐使用刚才从上方链接中下载的 Rustup 安装。下载好的 Rustup 在 Windows 上是一个可执行程序 rustup-init.exe。（在其他平台上应该是 **rustup-init.sh** ）。
- 现在执行 rustup-init 文件：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1706845042011-2711e544-bd12-4631-9f66-095921606d17.png)
- 上图显示的是一个命令行安装向导。
- **如果你已经安装 MSVC （推荐），那么安装过程会非常的简单，输入 1 并回车，直接进入第二步**。
- 如果你安装的是 MinGW，那么你需要输入 2 （自定义安装），然后系统会询问你 Default host triple? ，请将上图中 **default host triple** 的 "msvc" 改为 "gnu" 再输入安装程序：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1706845226784-9c139c6e-6314-4c02-8649-79e3a69dce43.png)
- 其它属性都默认。
- 设置完所有选项，会回到安装向导界面（第一张图），这是我们输入 1 并回车即可。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1706845265353-08558fbc-05bc-4937-9cc9-639a5056c027.png)
- 进行到这一步就完成了 Rust 的安装，可以通过以下命令测试：
  
  ```
  rustc -V
  ```
- ## 1.2.  在vscode中安装插件
- 在vscode的插件市场中添加rust-analyzer和Native Debug两个扩展
- # 2.  变量绑定与解构
- Rust中的变量既支持灵活性又支持安全性。
- 关键字：Rust 语言有一些关键字（keywords），和其他语言一样，这些关键字都是被保留给 Rust 语言使用的，因此，它们不能被用作变量或函数的名称。
- ## 2.1.  变量绑定
- 在其它语言中，我们用 var a = "hello world" 的方式给 a 赋值，也就是把等式右边的 "hello world" 字符串赋值给变量 a ，而在 Rust 中，我们这样写： let a = "hello world" ，同时给这个过程起了另一个名字：变量绑定。
- 这里就涉及 Rust 最核心的原则——所有权，简单来讲，任何内存对象都是有主人的，而且一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人（在这种情况下，该对象之前的主人就会丧失对该对象的所有权）
- ## 2.2.  变量可变性
- Rust 的变量在默认情况下是不可变的。前文提到，这是 Rust 团队为我们精心设计的语言特性之一，让我们编写的代码更安全，性能也更好。当然你可以通过 mut 关键字让变量变为可变的，让设计更灵活。
- 如果变量 a 不可变，那么一旦为它绑定值，就不能再修改 a。举个例子，在我们的工程目录下使用 cargo new variables 新建一个项目，叫做 variables 。
- 然后在新建的 variables 目录下，编辑 src/main.rs ，改为下面代码：
  
  ```
  fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
  }
  ```
- 保存文件，再使用 cargo run 运行它，迎面而来的是一条错误提示：
  
  ```
  $ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
  error[E0384]: cannot assign twice to immutable variable `x`
  --> src/main.rs:4:5
  |
  2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
  3 |     println!("The value of x is: {}", x);
  4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
  
  error: aborting due to previous error
  ```
- 具体的错误原因是 cannot assign twice to immutable variable x（无法对不可变的变量进行重复赋值），因为我们想为不可变的 x 变量再次赋值。
- 这种错误是为了避免无法预期的错误发生在我们的变量上：一个变量往往被多处代码所使用，其中一部分代码假定该变量的值永远不会改变，而另外一部分代码却无情的改变了这个值，在实际开发过程中，这个错误是很难被发现的，特别是在多线程编程中。
- 这种规则让我们的代码变得非常清晰，只有你想让你的变量改变时，它才能改变，这样就不会造成心智上的负担，也给别人阅读代码带来便利。
- 但是可变性也非常重要，否则我们就要像 ClojureScript 那样，每次要改变，就要重新生成一个对象，在拥有大量对象的场景，性能会变得非常低下，内存拷贝的成本异常的高。
- 在 Rust 中，可变性很简单，只要在变量名前加一个 mut 即可, 而且这种显式的声明方式还会给后来人传达这样的信息：嗯，这个变量在后面代码部分会发生改变。
- 为了让变量声明为可变,将 src/main.rs 改为以下内容：
  
  ```
  fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
  }
  ```
- 选择可变还是不可变，更多的还是取决于你的使用场景，例如不可变可以带来安全性，但是丧失了灵活性和性能（如果你要改变，就要重新创建一个新的变量，这里涉及到内存对象的再分配）。而可变变量最大的好处就是使用上的灵活性和性能上的提升。
- ## 2.3.  变量解构
- let 表达式不仅仅用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容：
  
  ```
  fn main() {
    let (a, mut b): (bool,bool) = (true, false);
    // a = true,不可变; b = false，可变
    println!("a = {:?}, b = {:?}", a, b);
  
    b = true;
    assert_eq!(a, b);
  }
  ```
- 在 Rust 1.59 版本后，我们可以在赋值语句的左式中使用元组、切片和结构体模式了。
  
  ```
  struct Struct {
    e: i32
  }
  
  fn main() {
    let (a, b, c, d, e);
  
    (a, b) = (1, 2);
    // _ 代表匹配一个值，但是我们不关心具体的值是什么，因此没有使用一个变量名而是使用了 _
    [c, .., d, _] = [1, 2, 3, 4, 5];
    Struct { e, .. } = Struct { e: 5 };
  
    assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
  }
  ```
- 这种使用方式跟之前的 let 保持了一致性，但是 let 会重新绑定，而这里仅仅是对之前绑定的变量进行再赋值。
- 需要注意的是，使用 += 的赋值语句还不支持解构式赋值。
- ## 2.4.  使用下划线开头忽略未使用的变量
- 如果你创建了一个变量却不在任何地方使用它，Rust 通常会给你一个警告，因为这可能会是个 BUG。但是有时创建一个不会被使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头：
  
  ```
  fn main() {
    let _x = 5;
    let y = 10;
  }
  ```
- 使用 cargo run 运行下试试:
  
  ```
  warning: unused variable: `y`
  --> src/main.rs:3:9
  |
  3 |     let y = 10;
  |         ^ help: 如果 y 故意不被使用，请添加一个下划线前缀: `_y`
  |
  = note: `#[warn(unused_variables)]` on by default
  ```
- 可以看到，两个变量都是只有声明，没有使用，但是编译器却独独给出了 y 未被使用的警告，充分说明了 _ 变量名前缀在这里发挥的作用。
- 值得注意的是，这里编译器还很善意的给出了提示( Rust 的编译器非常强大，这里的提示只是小意思 ): 将 y 修改 _y 即可。这里就不再给出代码，留给大家手动尝试并观察下运行结果。
- ## 2.5.  变量与常量的区别
- 变量的值不能更改可能让你想起其他另一个很多语言都有的编程概念：常量(constant)。与不可变变量一样，常量也是绑定到一个常量名且不允许更改的值，但是常量和变量之间存在一些差异：
	- 常量不允许使用 mut。常量不仅仅默认不可变，而且自始至终不可变，因为常量在编译完成后，已经确定它的值。
	- 常量使用 const 关键字而不是 let 关键字来声明，并且值的类型必须标注。
- 我们将在下一节数据类型中介绍，因此现在暂时无需关心细节。
- 下面是一个常量声明的例子，其常量名为 MAX_POINTS，值设置为 100,000。（Rust 常量的命名约定是全部字母都使用大写，并使用下划线分隔单词，另外对数字字面量可插入下划线以提高可读性）：
  
  ```
  const MAX_POINTS: u32 = 100_000;
  ```
- 常量可以在任意作用域内声明，包括全局作用域，在声明的作用域内，常量在程序运行的整个过程中都有效。对于需要在多处代码共享一个不可变的值时非常有用，例如游戏中允许玩家赚取的最大点数或光速。
- ## 2.6.  变量屏蔽
- Rust 允许声明相同的变量名，在后面声明的变量会遮蔽掉前面声明的，如下所示：
  
  ```
  fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;
  
    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
  
    println!("The value of x is: {}", x);
  }
  ```
- 这个程序首先将数值 5 绑定到 x，然后通过重复使用 let x = 来遮蔽之前的 x，并取原来的值加上 1，所以 x 的值变成了 6。第三个 let 语句同样遮蔽前面的 x，取之前的值并乘上 2，得到的 x 最终值为 12。当运行此程序，将输出以下内容：
  
  ```
  $ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
   ...
  The value of x in the inner scope is: 12
  The value of x is: 6
  ```
- 这和 mut 变量的使用是不同的，第二个 let 生成了完全不同的新变量，两个变量只是恰好拥有同样的名称，涉及一次内存对象的再分配 ，而 mut 声明的变量，可以修改同一个内存地址上的值，并不会发生内存对象的再分配，性能要更好。
- 变量遮蔽的用处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字。
- 例如，假设有一个程序要统计一个空格字符串的空格数量：
  
  ```
  // 字符串类型
  let spaces = "   ";
  // usize数值类型
  let spaces = spaces.len();
  ```
- 这种结构是允许的，因为第一个 spaces 变量是一个字符串类型，第二个 spaces 变量是一个全新的变量且和第一个具有相同的变量名，且是一个数值类型。所以变量遮蔽可以帮我们节省些脑细胞，不用去想如 spaces_str 和 spaces_num 此类的变量名；相反我们可以重复使用更简单的 spaces 变量名。如果你不用 let :
  
  ```
  let mut spaces = "   ";
  spaces = spaces.len();
  ```
- 运行一下，你就会发现编译器报错：
  
  ```
  $ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
  error[E0308]: mismatched types
  --> src/main.rs:3:14
  |
  3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`
  
  error: aborting due to previous error
  ```
- 显然，Rust 对类型的要求很严格，不允许将整数类型 usize 赋值给字符串类型。usize 是一种 CPU 相关的整数类型，在数值类型中有详细介绍。
- # 3.  基本类型
- Rust 每个值都有其确切的数据类型，总的来说可以分为两类：基本类型和复合类型。 基本类型意味着它们往往是一个最小化原子类型，无法解构为其它类型(一般意义上来说)，由以下组成：
	- 数值类型: 有符号整数 (i8, i16, i32, i64, isize)、 无符号整数 (u8, u16, u32, u64, usize) 、浮点数 (f32, f64)、以及有理数、复数
	- 字符串：字符串字面量和字符串切片 &str
	- 布尔类型： true和false
	- 字符类型: 表示单个 Unicode 字符，存储为 4 个字节
	- 单元类型: 即 () ，其唯一的值也是 ()、
- ## 3.1.  类型推导与标注
- 与 Python、JavaScript 等动态语言不同，Rust 是一门静态类型语言，也就是编译器必须在编译期知道我们所有变量的类型，但这不意味着你需要为每个变量指定类型，因为 Rust 编译器很聪明，它可以根据变量的值和上下文中的使用方式来自动推导出变量的类型，同时编译器也不够聪明，在某些情况下，它无法推导出变量类型，需要手动去给予一个类型标注，关于这一点在 Rust 语言初印象中有过展示。
- 来看段代码：
  
  ```
  let guess = "42".parse().expect("Not a number!");
  ```
- 先忽略 .parse().expect.. 部分，这段代码的目的是将字符串 "42" 进行解析，而编译器在这里无法推导出我们想要的类型：整数？浮点数？字符串？因此编译器会报错：
  
  ```
  $ cargo build
   Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations)
  error[E0282]: type annotations needed
  --> src/main.rs:2:9
  |
  2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^ consider giving `guess` a type
  ```
- 因此我们需要提供给编译器更多的信息，例如给 guess 变量一个显式的类型标注：let guess: i32 = ... 或者 "42".parse::<i32>()。
- ## 3.2.  数值类型
- Rust 使用一个相对传统的语法来创建整数（1，2，...）和浮点数（1.0，1.1，...）。整数、浮点数的运算和你在其它语言上见过的一致，都是通过常见的运算符来完成。
- ### 3.2.1.  整型类型
- 整数是没有小数部分的数字。之前使用过的 i32 类型，表示有符号的 32 位整数（ i 是英文单词 integer 的首字母，与之相反的是 u，代表无符号 unsigned 类型）。下表显示了 Rust 中的内置的整数类型：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1719446391376-a9d1e5a9-b8c6-4ce5-b02b-88d76913fdc8.png)
- 类型定义的形式统一为：有无符号 + 类型大小(位数)。无符号数表示数字只能取正数和0，而有符号则表示数字可以取正数、负数还有0。就像在纸上写数字一样：当要强调符号时，数字前面可以带上正号或负号；然而，当很明显确定数字为正数时，就不需要加上正号了。有符号数字以补码形式存储。
- 每个有符号类型规定的数字范围是 -(2n - 1) ~ 2n - 1 - 1，其中 n 是该定义形式的位长度。因此 i8 可存储数字范围是 -(27) ~ 27 - 1，即 -128 ~ 127。无符号类型可以存储的数字范围是 0 ~ 2n - 1，所以 u8 能够存储的数字为 0 ~ 28 - 1，即 0 ~ 255。
- 此外，isize 和 usize 类型取决于程序运行的计算机 CPU 类型： 若 CPU 是 32 位的，则这两个类型是 32 位的，同理，若 CPU 是 64 位，那么它们则是 64 位。
- 整形字面量可以用下表的形式书写：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1719446426091-c430c7a9-7fed-4e1a-8f09-569114d0d31e.png)
- Rust 整型默认使用 i32，例如 let i = 1，那 i 就是 i32 类型，因此你可以首选它，同时该类型也往往是性能最好的。isize 和 usize 的主要应用场景是用作集合的索引。
- ### 3.2.2.  浮点数类型
- 浮点类型数字 是带有小数点的数字，在 Rust 中浮点类型数字也有两种基本类型： f32 和 f64，分别为 32 位和 64 位大小。默认浮点类型是 f64，在现代的 CPU 中它的速度与 f32 几乎相同，但精度更高。
- 下面是一个演示浮点数的示例：
  
  ```
  fn main() {
    let x = 2.0; // f64
  
    let y: f32 = 3.0; // f32
  }
  ```
- 浮点数根据 IEEE-754 标准实现。f32 类型是单精度浮点型，f64 为双精度。
- 浮点数陷阱
- 浮点数由于底层格式的特殊性，导致了如果在使用浮点数时不够谨慎，就可能造成危险，有两个原因：
	- 浮点数往往是你想要数字的近似表达 浮点数类型是基于二进制实现的，但是我们想要计算的数字往往是基于十进制，例如 0.1 在二进制上并不存在精确的表达形式，但是在十进制上就存在。这种不匹配性导致一定的歧义性，更多的，虽然浮点数能代表真实的数值，但是由于底层格式问题，它往往受限于定长的浮点数精度，如果你想要表达完全精准的真实数字，只有使用无限精度的浮点数才行
	- 浮点数在某些特性上是反直觉的 例如大家都会觉得浮点数可以进行比较，对吧？是的，它们确实可以使用 >，>= 等进行比较，但是在某些场景下，这种直觉上的比较特性反而会害了你。因为 f32 ， f64 上的比较运算实现的是 std::cmp::PartialEq 特征(类似其他语言的接口)，但是并没有实现 std::cmp::Eq 特征，但是后者在其它数值类型上都有定义，说了这么多，可能大家还是云里雾里，用一个例子来举例：
- Rust 的 HashMap 数据结构，是一个 KV 类型的 Hash Map 实现，它对于 K 没有特定类型的限制，但是要求能用作 K 的类型必须实现了 std::cmp::Eq 特征，因此这意味着你无法使用浮点数作为 HashMap 的 Key，来存储键值对，但是作为对比，Rust 的整数类型、字符串类型、布尔类型都实现了该特征，因此可以作为 HashMap 的 Key。
- 为了避免上面说的两个陷阱，你需要遵守以下准则：
	- 避免在浮点数上测试相等性
	- 当结果在数学上可能存在未定义时，需要格外的小心
- 来看个小例子:
  
  ```
  fn main() {
  // 断言0.1 + 0.2与0.3相等
  assert!(0.1 + 0.2 == 0.3);
  }
  ```
- 你可能以为，这段代码没啥问题吧，实际上它会 panic(程序崩溃，抛出异常)，因为二进制精度问题，导致了 0.1 + 0.2 并不严格等于 0.3，它们可能在小数点 N 位后存在误差。
- 那如果非要进行比较呢？可以考虑用这种方式 (0.1_f64 + 0.2 - 0.3).abs() < 0.00001 ，具体小于多少，取决于你对精度的需求。
- 讲到这里，相信大家基本已经明白了，为什么操作浮点数时要格外的小心，但是还不够，下面再来一段代码，直接震撼你的灵魂：
  
  ```
  fn main() {
    let abc: (f32, f32, f32) = (0.1, 0.2, 0.3);
    let xyz: (f64, f64, f64) = (0.1, 0.2, 0.3);
  
    println!("abc (f32)");
    println!("   0.1 + 0.2: {:x}", (abc.0 + abc.1).to_bits());
    println!("         0.3: {:x}", (abc.2).to_bits());
    println!();
  
    println!("xyz (f64)");
    println!("   0.1 + 0.2: {:x}", (xyz.0 + xyz.1).to_bits());
    println!("         0.3: {:x}", (xyz.2).to_bits());
    println!();
  
    assert!(abc.0 + abc.1 == abc.2);
    assert!(xyz.0 + xyz.1 == xyz.2);
  }
  ```
- 对 f32 类型做加法时，0.1 + 0.2 的结果是 3e99999a，0.3 也是 3e99999a，因此 f32 下的 0.1 + 0.2 == 0.3 通过测试，但是到了 f64 类型时，结果就不一样了，因为 f64 精度高很多，因此在小数点非常后面发生了一点微小的变化，0.1 + 0.2 以 4 结尾，但是 0.3 以3结尾，这个细微区别导致 f64 下的测试失败了，并且抛出了异常。
- NaN
- 对于数学上未定义的结果，例如对负数取平方根 -42.1.sqrt() ，会产生一个特殊的结果：Rust 的浮点数类型使用 NaN (not a number)来处理这些情况。
- 所有跟 NaN 交互的操作，都会返回一个 NaN，而且 NaN 不能用来比较，下面的代码会崩溃：
  
  ```
  fn main() {
  let x = (-42.0_f32).sqrt();
  assert_eq!(x, x);
  }
  ```
- 出于防御性编程的考虑，可以使用 is_nan() 等方法，可以用来判断一个数值是否是 NaN ：
  
  ```
  fn main() {
    let x = (-42.0_f32).sqrt();
    if x.is_nan() {
        println!("未定义的数学行为")
    }
  }
  ```
- ### 3.2.3.  数字运算
- Rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和取模运算。下面代码各使用一条 let 语句来说明相应运算的用法：
  
  ```
  fn main() {
    // 加法
    let sum = 5 + 10;
  
    // 减法
    let difference = 95.5 - 4.3;
  
    // 乘法
    let product = 4 * 30;
  
    // 除法
    let quotient = 56.7 / 32.2;
  
    // 求余
    let remainder = 43 % 5;
  }
  ```
- 这些语句中的每个表达式都使用了数学运算符，并且计算结果为一个值，然后绑定到一个变量上。附录 B 中给出了 Rust 提供的所有运算符的列表。
- 再来看一个综合性的示例：
  
  ```
  fn main() {
  // 编译器会进行自动推导，给予twenty i32的类型
  let twenty = 20;
  // 类型标注
  let twenty_one: i32 = 21;
  // 通过类型后缀的方式进行类型标注：22是i32类型
  let twenty_two = 22i32;
  
  // 只有同样类型，才能运算
  let addition = twenty + twenty_one + twenty_two;
  println!("{} + {} + {} = {}", twenty, twenty_one, twenty_two, addition);
  
  // 对于较长的数字，可以用_进行分割，提升可读性
  let one_million: i64 = 1_000_000;
  println!("{}", one_million.pow(2));
  
  // 定义一个f32数组，其中42.0会自动被推导为f32类型
  let forty_twos = [
    42.0,
    42f32,
    42.0_f32,
  ];
  
  // 打印数组中第一个值，并控制小数位为2位
  println!("{:.2}", forty_twos[0]);
  }
  ```
- ### 3.2.4.  数位运算
- Rust的位运算基本上和其他语言一样
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1719715559404-cb143ec8-0f89-4cc4-b369-cbc4c76bd110.png)
  
  ```
  fn main() {
    // 二进制为00000010
    let a:i32 = 2;
    // 二进制为00000011
    let b:i32 = 3;
  
    println!("(a & b) value is {}", a & b);
  
    println!("(a | b) value is {}", a | b);
  
    println!("(a ^ b) value is {}", a ^ b);
  
    println!("(!b) value is {} ", !b);
  
    println!("(a << b) value is {}", a << b);
  
    println!("(a >> b) value is {}", a >> b);
  
    let mut a = a;
    // 注意这些计算符除了!之外都可以加上=进行赋值 (因为!=要用来判断不等于)
    a <<= b;
    println!("(a << b) value is {}", a);
  }
  ```
- ### 3.2.5.  序列（Range）
- Rust 提供了一个非常简洁的方式，用来生成连续的数值，例如 1..5，生成从 1 到 4 的连续数字，不包含 5 ；1..=5，生成从 1 到 5 的连续数字，包含 5，它的用途很简单，常常用于循环中：
  
  ```
  for i in 1..=5 {
    println!("{}",i);
  }
  ```
- 最终程序输出:
  
  ```
  1
  2
  3
  4
  5
  ```
- 序列只允许用于数字或字符类型，原因是：它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字值是 Rust 中仅有的可以用于判断是否为空的类型。如下是一个使用字符类型序列的例子：
  
  ```
  for i in 'a'..='z' {
    println!("{}",i);
  }
  ```
- ### 3.2.6.  有理数和复数
- Rust 中可以使用 As 来完成一个类型到另一个类型的转换，其最常用于将原始类型转换为其他原始类型，但是它也可以完成诸如将指针转换为地址、地址转换为指针以及将指针转换为其他指针等功能。
- Rust 的标准库相比其它语言，准入门槛较高，因此有理数和复数并未包含在标准库中：
	- 有理数和复数
	- 任意大小的整数和任意精度的浮点数
	- 固定精度的十进制小数，常用于货币相关的场景
- 好在社区已经开发出高质量的 Rust 数值库：num。
- 按照以下步骤来引入 num 库：
	- 创建新工程 cargo new complex-num && cd complex-num
	- 在 Cargo.toml 中的 [dependencies] 下添加一行 num = "0.4.0"
	- 将 src/main.rs 文件中的 main 函数替换为下面的代码
- 运行 cargo run
  
  ```
  use num::complex::Complex;
  
  fn main() {
   let a = Complex { re: 2.1, im: -1.2 };
   let b = Complex::new(11.1, 22.2);
   let result = a + b;
  
   println!("{} + {}i", result.re, result.im)
  }
  ```
- ## 3.3.  字符、布尔、单元类型
- ### 3.3.1.  字符类型(char)
- 字符，对于没有其它编程经验的新手来说可能不太好理解（没有编程经验敢来学 Rust 的绝对是好汉），但是你可以把它理解为英文中的字母，中文中的汉字。
- 下面的代码展示了几个颇具异域风情的字符：
  
  ```
  fn main() {
    let c = 'z';
    let z = 'ℤ';
    let g = '国';
    let heart_eyed_cat = '😻';
  }
  ```
- 如果大家是从有年代感的编程语言过来，可能会大喊一声：这 XX 叫字符？是的，在 Rust 语言中这些都是字符，Rust 的字符不仅仅是 ASCII，所有的 Unicode 值都可以作为 Rust 字符，包括单个的中文、日文、韩文、emoji 表情符号等等，都是合法的字符类型。Unicode 值的范围从 U+0000 ~ U+D7FF 和 U+E000 ~ U+10FFFF。不过“字符”并不是 Unicode 中的一个概念，所以人在直觉上对“字符”的理解和 Rust 的字符概念并不一致。
- 由于 Unicode 都是 4 个字节编码，因此字符类型也是占用 4 个字节：
  
  ```
  fn main() {
    let x = '中';
    println!("字符'中'占用了{}字节的内存大小",std::mem::size_of_val(&x));
  }
  ```
- 输出如下:
  
  ```
  $ cargo run
   Compiling ...
  
  字符'中'占用了4字节的内存大小
  ```
  
  注意，我们还没开始讲字符串，但是这里提前说一下，和一些语言不同，Rust 的字符只能用 '' 来表示， "" 是留给字符串的。
- ### 3.3.2.  布尔类型（bool）
- Rust 中的布尔类型有两个可能的值：true 和 false，布尔值占用内存的大小为 1 个字节：
  
  ```
  fn main() {
    let t = true;
  
    let f: bool = false; // 使用类型标注,显式指定f的类型
  
    if f {
        println!("这是段毫无意义的代码");
    }
  }
  ```
- 使用布尔类型的场景主要在于流程控制，例如上述代码的中的 if 就是其中之一。
- ### 3.3.3.  单元类型
- 单元类型就是 () ，对，你没看错，就是 () ，唯一的值也是 () ，一些读者读到这里可能就不愿意了，你也太敷衍了吧，管这叫类型？
- 只能说，再不起眼的东西，都有其用途，在目前为止的学习过程中，大家已经看到过很多次 fn main() 函数的使用吧？那么这个函数返回什么呢？
- 没错， main 函数就返回这个单元类型 ()，你不能说 main 函数无返回值，因为没有返回值的函数在 Rust 中是有单独的定义的：发散函数( diverge function )，顾名思义，无法收敛的函数。
- 例如常见的 println!() 的返回值也是单元类型 ()。
- 再比如，你可以用 () 作为 map 的值，表示我们不关注具体的值，只关注 key。 这种用法和 Go 语言的 struct{} 类似，可以作为一个值用来占位，但是完全不占用任何内存。
- ## 3.4.  语句和表达式
- ### 3.4.1.  语句
  
  ```
  let a = 8;
  let b: Vec<f64> = Vec::new();
  let (a, c) = ("hi", false);
  ```
- 以上都是语句，它们完成了一个具体的操作，但是并没有返回值，因此是语句。
- 由于 let 是语句，因此不能将 let 语句赋值给其它值，如下形式是错误的：
  
  ```
  let b = (let a = 8);
  ```
- 错误如下:
  
  ```
  error: expected expression, found statement (`let`) // 期望表达式，却发现`let`语句
  --> src/main.rs:2:13
  |
  2 |     let b = let a = 8;
  |             ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement `let`是一条语句
  
  error[E0658]: `let` expressions in this position are experimental
          // 下面的 `let` 用法目前是试验性的，在稳定版中尚不能使用
  --> src/main.rs:2:13
  |
  2 |     let b = let a = 8;
  |             ^^^^^^^^^
  |
  = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information
  = help: you can write `matches!(<expr>, <pattern>)` instead of `let <pattern> = <expr>`
  ```
- 以上的错误告诉我们 let 是语句，不是表达式，因此它不返回值，也就不能给其它变量赋值。但是该错误还透漏了一个重要的信息， let 作为表达式已经是试验功能了，也许不久的将来，我们在 stable rust 下可以这样使用。
- ### 3.4.2.  表达式
- 表达式会进行求值，然后返回一个值。例如 5 + 6，在求值后，返回值 11，因此它就是一条表达式。
- 表达式可以成为语句的一部分，例如 let y = 6 中，6 就是一个表达式，它在求值后返回一个值 6（有些反直觉，但是确实是表达式）。
- 调用一个函数是表达式，因为会返回一个值，调用宏也是表达式，用花括号包裹最终返回一个值的语句块也是表达式，总之，能返回值，它就是表达式:
  
  ```
  fn main() {
    let y = {
        let x = 3;
        x + 1
    };
  
    println!("The value of y is: {}", y);
  }
  ```
- 上面使用一个语句块表达式将值赋给 y 变量，语句块长这样：
  
  ```
  {
    let x = 3;
    x + 1
  }
  ```
- 该语句块是表达式的原因是：它的最后一行是表达式，返回了 x + 1 的值，注意 x + 1 不能以分号结尾，否则就会从表达式变成语句， 表达式不能包含分号。这一点非常重要，一旦你在表达式后加上分号，它就会变成一条语句，再也不会返回一个值，请牢记！
- 最后，表达式如果不返回任何值，会隐式地返回一个 () 。
  
  ```
  fn main() {
    assert_eq!(ret_unit_type(), ())
  }
  
  fn ret_unit_type() {
    let x = 1;
    // if 语句块也是一个表达式，因此可以用于赋值，也可以直接返回
    // 类似三元运算符，在Rust里我们可以这样写
    let y = if x % 2 == 1 {
        "odd"
    } else {
        "even"
    };
    // 或者写成一行
    let z = if x % 2 == 1 { "odd" } else { "even" };
  }
  ```
- ## 3.5.  函数
- Rust 的函数我们在之前已经见过不少，跟其他语言几乎没有什么区别。因此本章的学习之路将轻松和愉快，骚年们，请珍惜这种愉快，下一章你将体验到不一样的 Rust。
- 在函数界，有一个函数只闻其名不闻其声，可以止小孩啼！在程序界只有 hello,world! 可以与之媲美，它就是 add 函数：
  
  ```
  fn add(i: i32, j: i32) -> i32 {
   i + j
  }
  ```
- 该函数如此简单，但是又是如此的五脏俱全，声明函数的关键字 fn ,函数名 add()，参数 i 和 j，参数类型和返回值类型都是 i32，总之一切那么的普通，但是又那么的自信，直到你看到了下面这张图：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1720130962633-b390bed5-56c6-43cc-abea-1166f1501679.png)
- ### 3.5.1.  函数参数
- Rust 是强类型语言，因此需要你为每一个函数参数都标识出它的具体类型，例如：
  
  ```
  fn main() {
    another_function(5, 6.1);
  }
  
  fn another_function(x: i32, y: f32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
  }
  ```
- another_function 函数有两个参数，其中 x 是 i32 类型，y 是 f32 类型，然后在该函数内部，打印出这两个值。这里去掉 x 或者 y 的任何一个的类型，都会报错：
  
  ```
  fn main() {
    another_function(5, 6.1);
  }
  
  fn another_function(x: i32, y) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
  }
  ```
- 错误如下:
  
  ```
  error: expected one of `:`, `@`, or `|`, found `)`
  --> src/main.rs:5:30
  |
  5 | fn another_function(x: i32, y) {
  |                              ^ expected one of `:`, `@`, or `|` // 期待以下符号之一 `:`, `@`, or `|`
  |
  = note: anonymous parameters are removed in the 2018 edition (see RFC 1685)
    // 匿名参数在 Rust 2018 edition 中就已经移除
  help: if this is a parameter name, give it a type // 如果y是一个参数名，请给予它一个类型
  |
  5 | fn another_function(x: i32, y: TypeName) {
  |                             ~~~~~~~~~~~
  help: if this is a type, explicitly ignore the parameter name // 如果y是一个类型，请使用_忽略参数名
  |
  5 | fn another_function(x: i32, _: y) {
  |                             ~~~~
  ```
- ### 3.5.2.  函数返回
- 在 Rust 中函数就是表达式，因此我们可以把函数的返回值直接赋给调用者。
- 函数的返回值就是函数体最后一条表达式的返回值，当然我们也可以使用 return 提前返回，下面的函数使用最后一条表达式来返回一个值：
  
  ```
  fn plus_five(x:i32) -> i32 {
    x + 5
  }
  
  fn main() {
    let x = plus_five(5);
  
    println!("The value of x is: {}", x);
  }
  ```
- x + 5 是一条表达式，求值后，返回一个值，因为它是函数的最后一行，因此该表达式的值也是函数的返回值。
- 再来看两个重点：
	- let x = plus_five(5)，说明我们用一个函数的返回值来初始化 x 变量，因此侧面说明了在 Rust 中函数也是表达式，这种写法等同于 let x = 5 + 5;
	- x + 5 没有分号，因为它是一条表达式，这个在上一节中我们也有详细介绍
- 再来看一段代码，同时使用 return 和表达式作为返回值：
  
  ```
  fn plus_or_minus(x:i32) -> i32 {
    if x > 5 {
        return x - 5
    }
  
    x + 5
  }
  
  fn main() {
    let x = plus_or_minus(5);
  
    println!("The value of x is: {}", x);
  }
  ```
- plus_or_minus 函数根据传入 x 的大小来决定是做加法还是减法，若 x > 5 则通过 return 提前返回 x - 5 的值,否则返回 x + 5 的值。
- ### 3.5.3.  函数特殊的返回
- 单元类型 ()，是一个零长度的元组。它没啥作用，但是可以用来表达一个函数没有返回值：
	- 函数没有返回值，那么返回一个 ()
	- 通过 ; 结尾的语句返回一个 ()
- 例如下面的 report 函数会隐式返回一个 ()：
  
  ```
  use std::fmt::Debug;
  
  fn report<T: Debug>(item: T) {
  println!("{:?}", item);
  
  }
  ```
- 与上面的函数返回值相同，但是下面的函数显式的返回了 ()：
  
  ```
  fn clear(text: &mut String) -> () {
  *text = String::from("");
  }
  ```
- 在实际编程中，你会经常在错误提示中看到该 () 的身影出没，假如你的函数需要返回一个 u32 值，但是如果你不幸的以 表达式; 的语句形式作为函数的最后一行代码，就会报错：
  
  ```
  fn add(x:u32,y:u32) -> u32 {
    x + y;
  }
  ```
- 错误如下：
  
  ```
  error[E0308]: mismatched types // 类型不匹配
  --> src/main.rs:6:24
  |
  6 | fn add(x:u32,y:u32) -> u32 {
  |    ---                 ^^^ expected `u32`, found `()` // 期望返回u32,却返回()
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
  7 |     x + y;
  |          - help: consider removing this semicolon
  ```
- 还记得我们在语句与表达式中讲过的吗？只有表达式能返回值，而 ; 结尾的是语句，在 Rust 中，一定要严格区分表达式和语句的区别，这个在其它语言中往往是被忽视的点。
- 永不返回的发散函数 !
- 当用 ! 作函数返回类型的时候，表示该函数永不返回( diverge function )，特别的，这种语法往往用做会导致程序崩溃的函数：
  
  ```
  fn dead_end() -> ! {
  panic!("你已经到了穷途末路，崩溃吧！");
  }
  ```
- 下面的函数创建了一个无限循环，该循环永不跳出，因此函数也永不返回：
  
  ```
  fn forever() -> ! {
  loop {
    //...
  };
  }
  ```
- ## 3.6.  堆和栈
- 栈和堆是编程语言最核心的数据结构，但是在很多语言中，你并不需要深入了解栈与堆。 但对于 Rust 这样的系统编程语言，值是位于栈上还是堆上非常重要, 因为这会影响程序的行为和性能。
- 栈和堆的核心目标就是为程序在运行时提供可供使用的内存空间。
- ### 3.6.1.  栈
- 栈按照顺序存储值并以相反顺序取出值，这也被称作后进先出。想象一下一叠盘子：当增加更多盘子时，把它们放在盘子堆的顶部，当需要盘子时，再从顶部拿走。不能从中间也不能从底部增加或拿走盘子！
- 增加数据叫做进栈，移出数据则叫做出栈。
- 因为上述的实现方式，栈中的所有数据都必须占用已知且固定大小的内存空间，假设数据大小是未知的，那么在取出数据时，你将无法取到你想要的数据。
- ### 3.6.2.  堆
- 与栈不同，对于大小未知或者可能变化的数据，我们需要将它存储在堆上。
- 当向堆上放入数据时，需要请求一定大小的内存空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的指针, 该过程被称为在堆上分配内存，有时简称为 “分配”(allocating)。
- 接着，该指针会被推入栈中，因为指针的大小是已知且固定的，在后续使用过程中，你将通过栈中的指针，来获取数据在堆上的实际内存位置，进而访问该数据。
- 由上可知，堆是一种缺乏组织的数据结构。想象一下去餐馆就座吃饭: 进入餐馆，告知服务员有几个人，然后服务员找到一个够大的空桌子（堆上分配的内存空间）并领你们过去。如果有人来迟了，他们也可以通过桌号（栈上的指针）来找到你们坐在哪。
- ### 3.6.3.  性能区别
- 在栈上分配内存比在堆上分配内存要快，因为入栈时操作系统无需进行函数调用（或更慢的系统调用）来分配新的空间，只需要将新数据放入栈顶即可。相比之下，在堆上分配内存则需要更多的工作，这是因为操作系统必须首先找到一块足够存放数据的内存空间，接着做一些记录为下一次分配做准备，如果当前进程分配的内存页不足时，还需要进行系统调用来申请更多内存。 因此，处理器在栈上分配数据会比在堆上分配数据更加高效。
- ### 3.6.4.  所有权与堆栈
- 当你的代码调用一个函数时，传递给函数的参数（包括可能指向堆上数据的指针和函数的局部变量）依次被压入栈中，当函数调用结束时，这些值将被从栈中按照相反的顺序依次移除。
- 因为堆上的数据缺乏组织，因此跟踪这些数据何时分配和释放是非常重要的，否则堆上的数据将产生内存泄漏 —— 这些数据将永远无法被回收。这就是 Rust 所有权系统为我们提供的强大保障。
- 对于其他很多编程语言，你确实无需理解堆栈的原理，但是在 Rust 中，明白堆栈的原理，对于我们理解所有权的工作原理会有很大的帮助。
- ### 3.6.5.  所有权原则
- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)
- 变量作用域
	- 作用域是一个变量在程序中有效的范围, 假如有这样一个变量：
	  
	  ```
	  let s = "hello";
	  ```
	- 变量 s 绑定到了一个字符串字面值，该字符串字面值是硬编码到程序代码中的。s 变量从声明的点开始直到当前作用域的结束都是有效的：
	  
	  ```
	  {                      // s 在这里无效，它尚未声明
	   let s = "hello";   // 从此处起，s 是有效的
	  
	   // 使用 s
	  }                      // 此作用域已结束，s不再有效
	  ```
	- 简而言之，s 从创建开始就有效，然后有效期持续到它离开作用域为止，可以看出，就作用域来说，Rust 语言跟其他编程语言没有区别。
- 简单介绍String类型
	- 之前提到过，本章会用 String 作为例子，因此这里会进行一下简单的介绍，具体的 String 学习请参见 String 类型。
	- 我们已经见过字符串字面值 let s ="hello"，s 是被硬编码进程序里的字符串值（类型为 &str ）。字符串字面值是很方便的，但是它并不适用于所有场景。原因有二：
		- 字符串字面值是不可变的，因为被硬编码到程序代码中
		- 并非所有字符串的值都能在编写代码时得知
	- 例如，字符串是需要程序运行时，通过用户动态输入然后存储在内存中的，这种情况，字符串字面值就完全无用武之地。 为此，Rust 为我们提供动态字符串类型: String, 该类型被分配到堆上，因此可以动态伸缩，也就能存储在编译时大小未知的文本。
	- 可以使用下面的方法基于字符串字面量来创建 String 类型：
	  
	  ```
	  let s = String::from("hello");
	  ```
	- :: 是一种调用操作符，这里表示调用 String 模块中的 from 方法，由于 String 类型存储在堆上，因此它是动态的，你可以这样修改：
	  
	  ```
	  let mut s = String::from("hello");
	  
	  s.push_str(", world!"); // push_str() 在字符串后追加字面值
	  
	  println!("{}", s); // 将打印 `hello, world!`
	  ```
	- 言归正传，了解 String 后，一起来看看关于所有权的交互。
- ### 3.6.6.  变量绑定背后的数据交互
- 转移所有权
	- 先来看一段代码：
	  
	  ```
	  let x = 5;
	  let y = x;
	  ```
	- 这段代码并没有发生所有权的转移，原因很简单： 代码首先将 5 绑定到变量 x，接着拷贝 x 的值赋给 y，最终 x 和 y 都等于 5，因为整数是 Rust 基本数据类型，是固定大小的简单值，因此这两个值都是通过自动拷贝的方式来赋值的，都被存在栈中，完全无需在堆上分配内存。
	- 整个过程中的赋值都是通过值拷贝的方式完成(发生在栈中)，因此并不需要所有权转移。
	- 然后再来看一段代码：
	  
	  ```
	  let s1 = String::from("hello");
	  let s2 = s1;
	  ```
	- 对于基本类型（存储在栈上），Rust 会自动拷贝，但是 String 不是基本类型，而且是存储在堆上的，因此不能自动拷贝。
	- 实际上， String 类型是一个复杂类型，由存储在栈中的堆指针、字符串长度、字符串容量共同组成，其中堆指针是最重要的，它指向了真实存储字符串内容的堆内存，至于长度和容量，如果你有 Go 语言的经验，这里就很好理解：容量是堆内存分配空间的大小，长度是目前已经使用的大小。
	- 总之 String 类型指向了一个堆上的空间，这里存储着它的真实数据，下面对上面代码中的 let s2 = s1 分成两种情况讨论：
		- 拷贝 String 和存储在堆上的字节数组 如果该语句是拷贝所有数据(深拷贝)，那么无论是 String 本身还是底层的堆上数据，都会被全部拷贝，这对于性能而言会造成非常大的影响
		- 只拷贝 String 本身 这样的拷贝非常快，因为在 64 位机器上就拷贝了 8字节的指针、8字节的长度、8字节的容量，总计 24 字节，但是带来了新的问题，还记得我们之前提到的所有权规则吧？其中有一条就是：一个值只允许有一个所有者，而现在这个值（堆上的真实字符串数据）有了两个所有者：s1 和 s2。
	- 假定一个值可以拥有两个所有者，会发生什么呢？
		- 当变量离开作用域后，Rust 会自动调用 drop 函数并清理变量的堆内存。不过由于两个 String 变量指向了同一位置。这就有了一个问题：当 s1 和 s2 离开作用域，它们都会尝试释放相同的内存。这是一个叫做 二次释放（double free） 的错误，也是之前提到过的内存安全性 BUG 之一。两次释放（相同）内存会导致内存污染，它可能会导致潜在的安全漏洞。
		- 因此，Rust 这样解决问题：当 s1 被赋予 s2 后，Rust 认为 s1 不再有效，因此也无需在 s1 离开作用域后 drop 任何东西，这就是把所有权从 s1 转移给了 s2，s1 在被赋予 s2 后就马上失效了。
	- 再来看看，在所有权转移后再来使用旧的所有者，会发生什么：
	  
	  ```
	  let s1 = String::from("hello");
	  let s2 = s1;
	  
	  println!("{}, world!", s1);
	  ```
	- 由于 Rust 禁止你使用无效的引用，你会看到以下的错误：
	  
	  ```
	  error[E0382]: borrow of moved value: `s1`
	  --> src/main.rs:5:28
	  |
	  2 |     let s1 = String::from("hello");
	  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
	  3 |     let s2 = s1;
	  |              -- value moved here
	  4 |
	  5 |     println!("{}, world!", s1);
	  |                            ^^ value borrowed here after move
	  |
	  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
	  help: consider cloning the value if the performance cost is acceptable
	  |
	  3 |     let s2 = s1.clone();
	  |                ++++++++
	  
	  For more information about this error, try `rustc --explain E0382`.
	  ```
	- 如果你在其他语言中听说过术语 浅拷贝(shallow copy) 和 深拷贝(deep copy)，那么拷贝指针、长度和容量而不拷贝数据听起来就像浅拷贝，但是又因为 Rust 同时使第一个变量 s1 无效了，因此这个操作被称为 移动(move)，而不是浅拷贝。上面的例子可以解读为 s1 被移动到了 s2 中。那么具体发生了什么，用一张图简单说明：
	- ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1720571026065-f9498c34-fd52-4c54-a8ec-11642a48383d.png)
	- 这样就解决了我们之前的问题，s1 不再指向任何数据，只有 s2 是有效的，当 s2 离开作用域，它就会释放内存。 相信此刻，你应该明白了，为什么 Rust 称呼 let a = b 为变量绑定了吧？
	- 再来看一段代码:
	  
	  ```
	  fn main() {
	   let x: &str = "hello, world";
	   let y = x;
	   println!("{},{}",x,y);
	  }
	  ```
	- 这段代码，大家觉得会否报错？如果参考之前的 String 所有权转移的例子，那这段代码也应该报错才是，但是实际上呢？
	- 这段代码和之前的 String 有一个本质上的区别：在 String 的例子中 s1 持有了通过String::from("hello") 创建的值的所有权，而这个例子中，x 只是引用了存储在二进制中的字符串 "hello, world"，并没有持有所有权。
	- 因此 let y = x 中，仅仅是对该引用进行了拷贝，此时 y 和 x 都引用了同一个字符串。如果还不理解也没关系，当学习了下一章节 "引用与借用" 后，大家自然而言就会理解。
- 克隆（深拷贝）
	- ust 永远也不会自动创建数据的 “深拷贝”。因此，任何自动的复制都不是深拷贝，可以被认为对运行时性能影响较小。
	- 如果我们确实需要深度复制 String 中堆上的数据，而不仅仅是栈上的数据，可以使用一个叫做 clone 的方法。
	  
	  ```
	  let s1 = String::from("hello");
	  let s2 = s1.clone();
	  
	  println!("s1 = {}, s2 = {}", s1, s2);
	  ```
	- 这段代码能够正常运行，说明 s2 确实完整的复制了 s1 的数据。
	- 如果代码性能无关紧要，例如初始化程序时或者在某段时间只会执行寥寥数次时，你可以使用 clone 来简化编程。但是对于执行较为频繁的代码(热点路径)，使用 clone 会极大的降低程序性能，需要小心使用！
- 拷贝（浅拷贝）
	- 浅拷贝只发生在栈上，因此性能很高，在日常编程中，浅拷贝无处不在。
	- 再回到之前看过的例子:
	  
	  ```
	  let x = 5;
	  let y = x;
	  
	  println!("x = {}, y = {}", x, y);
	  ```
	- 但这段代码似乎与我们刚刚学到的内容相矛盾：没有调用 clone，不过依然实现了类似深拷贝的效果 —— 没有报所有权的错误。
	- 原因是像整型这样的基本类型在编译时是已知大小的，会被存储在栈上，所以拷贝其实际的值是快速的。这意味着没有理由在创建变量 y 后使 x 无效（x、y 都仍然有效）。换句话说，这里没有深浅拷贝的区别，因此这里调用 clone 并不会与通常的浅拷贝有什么不同，我们可以不用管它（可以理解成在栈上做了深拷贝）。
	- Rust 有一个叫做 Copy 的特征，可以用在类似整型这样在栈中存储的类型。如果一个类型拥有 Copy 特征，一个旧的变量在被赋值给其他变量后仍然可用，也就是赋值的过程即是拷贝的过程。
	- 那么什么类型是可 Copy 的呢？可以查看给定类型的文档来确认，这里可以给出一个通用的规则： 任何基本类型的组合可以 Copy ，不需要分配内存或某种形式资源的类型是可以 Copy 的。如下是一些 Copy 的类型：
		- 所有整数类型，比如 u32
		- 布尔类型，bool，它的值是 true 和 false
		- 所有浮点数类型，比如 f64
		- 字符类型，char
		- 元组，当且仅当其包含的类型也都是 Copy 的时候。比如，(i32, i32) 是 Copy 的，但 (i32, String) 就不是
		- 不可变引用 &T ，例如转移所有权中的最后一个例子，但是注意: 可变引用 &mut T 是不可以 Copy的
- ### 3.6.7.  函数的传值和返回
- 将值传递给函数，一样会发生 移动 或者 复制，就跟 let 语句一样，下面的代码展示了所有权、作用域的规则：
  
  ```
  fn main() {
    let s = String::from("hello");  // s 进入作用域
  
    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效
  
    let x = 5;                      // x 进入作用域
  
    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x
  
  } // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作
  
  fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
  } // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放
  
  fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
  } // 这里，some_integer 移出作用域。不会有特殊操作
  ```
- 你可以尝试在 takes_ownership 之后，再使用 s，看看如何报错？例如添加一行 println!("在move进函数后继续使用s: {}",s);。
- 同样的，函数返回值也有所有权，例如:
  
  ```
  fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1
  
    let s2 = String::from("hello");     // s2 进入作用域
  
    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
  } // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃
  
  fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数
  
    let some_string = String::from("hello"); // some_string 进入作用域.
  
    some_string                              // 返回 some_string 并移出给调用的函数
  }
  
  // takes_and_gives_back 将传入字符串并返回该值
  fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域
  
    a_string  // 返回 a_string 并移出给调用的函数
  }
  ```
- 所有权很强大，避免了内存的不安全性，但是也带来了一个新麻烦： 总是把一个值传来传去来使用它。 传入一个函数，很可能还要从该函数传出去，结果就是语言表达变得非常啰嗦，幸运的是，Rust 提供了新功能解决这个问题。
- # 4.  复合类型
- ## 4.1.  字符串
- ### 4.1.1.  切片
- 切片并不是 Rust 独有的概念，在 Go 语言中就非常流行，它允许你引用集合中部分连续的元素序列，而不是引用整个集合。
- 对于字符串而言，切片就是对 String 类型中某一部分的引用，它看起来像这样：
  
  ```
  let s = String::from("hello world");
  
  let hello = &s[0..5];
  let world = &s[6..11];
  ```
- hello 没有引用整个 String s，而是引用了 s 的一部分内容，通过 [0..5] 的方式来指定。
- 这就是创建切片的语法，使用方括号包括的一个序列：[开始索引..终止索引]，其中开始索引是切片中第一个元素的索引位置，而终止索引是最后一个元素后面的索引位置，也就是这是一个 右半开区间。在切片数据结构内部会保存开始的位置和切片的长度，其中长度是通过 终止索引 - 开始索引 的方式计算得来的。
- 对于 let world = &s[6..11]; 来说，world 是一个切片，该切片的指针指向 s 的第 7 个字节(索引从 0 开始, 6 是第 7 个字节)，且该切片的长度是 5 个字节。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1720882653779-866d570a-3597-4c70-ad33-2190e2e52cb6.png)
- 在使用 Rust 的 .. range 序列语法时，如果你想从索引 0 开始，可以使用如下的方式，这两个是等效的：
  
  ```
  let s = String::from("hello");
  
  let slice = &s[0..2];
  let slice = &s[..2];
  ```
- 同样的，如果你的切片想要包含 String 的最后一个字节，则可以这样使用：
  
  ```
  let s = String::from("hello");
  
  let len = s.len();
  
  let slice = &s[4..len];
  let slice = &s[4..];
  ```
- 你也可以截取完整的 String 切片：
  
  ```
  let s = String::from("hello");
  
  let len = s.len();
  
  let slice = &s[0..len];
  let slice = &s[..];
  ```
  
  在对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置，也就是 UTF-8 字符的边界，例如中文在 UTF-8 中占用三个字节，下面的代码就会崩溃：
  
  let s = "中国人";
  
  let a = &s[0..2];
  
  println!("{}",a);
  
  因为我们只取 s 字符串的前两个字节，但是本例中每个汉字占用三个字节，因此没有落在边界处，也就是连 中 字都取不完整，此时程序会直接崩溃退出，如果改成 &s[0..3]，则可以正常通过编译。 因此，当你需要对字符串做切片索引操作时，需要格外小心这一点, 关于该如何操作 UTF-8 字符串，参见这里。
- 字符串切片的类型标识是 &str，因此我们可以这样声明一个函数，输入 String 类型，返回它的切片: fn first_word(s: &String) -> &str 。
- 有了切片就可以写出这样的代码：
  
  ```
  fn main() {
    let mut s = String::from("hello world");
  
    let word = first_word(&s);
  
    s.clear(); // error!
  
    println!("the first word is: {}", word);
  }
  fn first_word(s: &String) -> &str {
    &s[..1]
  }
  ```
- 编译器报错如下：
  
  ```
  error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
  16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
  17 |
  18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
  19 |
  20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here
  ```
- 回忆一下借用的规则：当我们已经有了可变借用时，就无法再拥有不可变的借用。因为 clear 需要清空改变 String，因此它需要一个可变借用（利用 VSCode 可以看到该方法的声明是 pub fn clear(&mut self) ，参数是对自身的可变借用 ）；而之后的 println! 又使用了不可变借用，也就是在 s.clear() 处可变借用与不可变借用试图同时生效，因此编译无法通过。
- 从上述代码可以看出，Rust 不仅让我们的 API 更加容易使用，而且也在编译期就消除了大量错误！
- ### 4.1.2.  什么是字符串？
- 字符串是由字符组成的连续集合，但是在上一节中我们提到过，Rust 中的字符是 Unicode 类型，因此每个字符占据 4 个字节内存空间，但是在字符串中不一样，字符串是 UTF-8 编码，也就是字符串中的字符所占的字节数是变化的(1 - 4)，这样有助于大幅降低字符串所占用的内存空间。
- Rust 在语言级别，只有一种字符串类型： str，它通常是以引用类型出现 &str，也就是上文提到的字符串切片。虽然语言级别只有上述的 str 类型，但是在标准库里，还有多种不同用途的字符串类型，其中使用最广的即是 String 类型。
- str 类型是硬编码进可执行文件，也无法被修改，但是 String 则是一个可增长、可改变且具有所有权的 UTF-8 编码字符串，当 Rust 用户提到字符串时，往往指的就是 String 类型和 &str 字符串切片类型，这两个类型都是 UTF-8 编码。
- 除了 String 类型的字符串，Rust 的标准库还提供了其他类型的字符串，例如 OsString， OsStr， CsString 和 CsStr 等，注意到这些名字都以 String 或者 Str 结尾了吗？它们分别对应的是具有所有权和被借用的变量。
- ### 4.1.3.  String和&str的转换
- 在之前的代码中，已经见到好几种从 &str 类型生成 String 类型的操作：
	- String::from("hello,world")
	- "hello,world".to_string()
- 那么如何将 String 类型转为 &str 类型呢？答案很简单，取引用即可：
  
  ```
  fn main() {
    let s = String::from("hello,world!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
  }
  
  fn say_hello(s: &str) {
    println!("{}",s);
  }
  ```
- 实际上这种灵活用法是因为 deref 隐式强制转换，具体我们会在 Deref 特征进行详细讲解。
- ### 4.1.4.  字符串索引
- 在其它语言中，使用索引的方式访问字符串的某个字符或者子串是很正常的行为，但是在 Rust 中就会报错：
  
  ```
  let s1 = String::from("hello");
   let h = s1[0];
  ```
- 该代码会产生如下错误：
  
  ```
  3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`
  ```
- 字符串的底层的数据存储格式实际上是[ u8 ]，一个字节数组。对于 let hello = String::from("Hola"); 这行代码来说，Hola 的长度是 4 个字节，因为 "Hola" 中的每个字母在 UTF-8 编码中仅占用 1 个字节，但是对于下面的代码呢？
  
  ```
  let hello = String::from("中国人");
  ```
- 你该字符串多长，你可能会说 3，但是实际上是 9 个字节的长度，因为大部分常用汉字在 UTF-8 中的长度是 3 个字节，因此这种情况下对 hello 进行索引，访问 &hello[0] 没有任何意义，因为你取不到 中 这个字符，而是取到了这个字符三个字节中的第一个字节，这是一个非常奇怪而且难以理解的返回值。
- 现在看一下用梵文写的字符串 “नमस्ते”, 它底层的字节数组如下形式：
  
  ```
  [224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
  224, 165, 135]
  ```
- 长度是 18 个字节，这也是计算机最终存储该字符串的形式。如果从字符的形式去看，则是：
  
  ```
  ['न', 'म', 'स', '्', 'त', 'े']
  ```
- 但是这种形式下，第四和六两个字母根本就不存在，没有任何意义，接着再从字母串的形式去看：
  
  ```
  ["न", "म", "स्", "ते"]
  ```
- 可以看出来 Rust 提供了不同的字符串展现方式，这样程序可以挑选自己想要的方式去使用，而无需去管字符串从人类语言角度看长什么样。
- 还有一个原因导致了 Rust 不允许去索引字符串：因为索引操作，我们总是期望它的性能表现是 O(1)，然而对于 String 类型来说，无法保证这一点，因为 Rust 可能需要从 0 开始去遍历字符串来定位合法的字符。
- ### 4.1.5.  操作字符串
- 由于 String 是可变字符串，下面介绍 Rust 字符串的修改，添加，删除等常用方法：
- 追加（push）
	- 在字符串尾部可以使用 push() 方法追加字符 char，也可以使用 push_str() 方法追加字符串字面量。这两个方法都是在原有的字符串上追加，并不会返回新的字符串。由于字符串追加操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut s = String::from("Hello ");
	  
	   s.push_str("rust");
	   println!("追加字符串 push_str() -> {}", s);
	  
	   s.push('!');
	   println!("追加字符 push() -> {}", s);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  追加字符串 push_str() -> Hello rust
	  追加字符 push() -> Hello rust!
	  ```
- 插入（insert）
	- 可以使用 insert() 方法插入单个字符 char，也可以使用 insert_str() 方法插入字符串字面量，与 push() 方法不同，这俩方法需要传入两个参数，第一个参数是字符（串）插入位置的索引，第二个参数是要插入的字符（串），索引从 0 开始计数，如果越界则会发生错误。由于字符串插入操作要修改原来的字符串，则该字符串必须是可变的，即字符串变量必须由 mut 关键字修饰。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut s = String::from("Hello rust!");
	   s.insert(5, ',');
	   println!("插入字符 insert() -> {}", s);
	   s.insert_str(6, " I like");
	   println!("插入字符串 insert_str() -> {}", s);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  插入字符 insert() -> Hello, rust!
	  插入字符串 insert_str() -> Hello, I like rust!
	  ```
- 替换（replace）
	- 如果想要把字符串中的某个字符串替换成其它的字符串，那可以使用 replace() 方法。与替换有关的方法有三个。
	- replace
	- 该方法可适用于 String 和 &str 类型。replace() 方法接收两个参数，第一个参数是要被替换的字符串，第二个参数是新的字符串。该方法会替换所有匹配到的字符串。该方法是返回一个新的字符串，而不是操作原来的字符串。
	  
	  ```
	  fn main() {
	   let string_replace = String::from("I like rust. Learning rust is my favorite!");
	   let new_string_replace = string_replace.replace("rust", "RUST");
	   dbg!(new_string_replace);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  new_string_replace = "I like RUST. Learning RUST is my favorite!"
	  ```
	- replacen
	- 该方法可适用于 String 和 &str 类型。replacen() 方法接收三个参数，前两个参数与 replace() 方法一样，第三个参数则表示替换的个数。该方法是返回一个新的字符串，而不是操作原来的字符串。
	  
	  ```
	  fn main() {
	   let string_replace = "I like rust. Learning rust is my favorite!";
	   let new_string_replacen = string_replace.replacen("rust", "RUST", 1);
	   dbg!(new_string_replacen);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  new_string_replacen = "I like RUST. Learning rust is my favorite!"
	  ```
	- replace_range
	- 该方法仅适用于 String 类型。replace_range 接收两个参数，第一个参数是要替换字符串的范围（Range），第二个参数是新的字符串。该方法是直接操作原来的字符串，不会返回新的字符串。该方法需要使用 mut 关键字修饰。
	  
	  ```
	  fn main() {
	   let mut string_replace_range = String::from("I like rust!");
	   string_replace_range.replace_range(7..8, "R");
	   dbg!(string_replace_range);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  string_replace_range = "I like Rust!"
	  ```
- 删除（delete）
	- 与字符串删除相关的方法有 4 个，他们分别是 pop()，remove()，truncate()，clear()。这四个方法仅适用于 String 类型。
	- 1、 pop —— 删除并返回字符串的最后一个字符
	- 该方法是直接操作原来的字符串。但是存在返回值，其返回值是一个 Option 类型，如果字符串为空，则返回 None。 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut string_pop = String::from("rust pop 中文!");
	   let p1 = string_pop.pop();
	   let p2 = string_pop.pop();
	   dbg!(p1);
	   dbg!(p2);
	   dbg!(string_pop);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  p1 = Some(
	  '!',
	  )
	  p2 = Some(
	  '文',
	  )
	  string_pop = "rust pop 中"
	  ```
	- 2、 remove —— 删除并返回字符串中指定位置的字符
	- 该方法是直接操作原来的字符串。但是存在返回值，其返回值是删除位置的字符串，只接收一个参数，表示该字符起始索引位置。remove() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut string_remove = String::from("测试remove方法");
	   println!(
	       "string_remove 占 {} 个字节",
	       std::mem::size_of_val(string_remove.as_str())
	   );
	   // 删除第一个汉字
	   string_remove.remove(0);
	   // 下面代码会发生错误
	   // string_remove.remove(1);
	   // 直接删除第二个汉字
	   // string_remove.remove(3);
	   dbg!(string_remove);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  string_remove 占 18 个字节
	  string_remove = "试remove方法"
	  ```
	- 3、truncate —— 删除字符串中从指定位置开始到结尾的全部字符
	- 该方法是直接操作原来的字符串。无返回值。该方法 truncate() 方法是按照字节来处理字符串的，如果参数所给的位置不是合法的字符边界，则会发生错误。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut string_truncate = String::from("测试truncate");
	   string_truncate.truncate(3);
	   dbg!(string_truncate);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  string_truncate = "测"
	  ```
	- 4、clear —— 清空字符串
	- 该方法是直接操作原来的字符串。调用后，删除字符串中的所有字符，相当于 truncate() 方法参数为 0 的时候。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let mut string_clear = String::from("string clear");
	   string_clear.clear();
	   dbg!(string_clear);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  string_clear = ""
	  ```
- 连接 (Concatenate)
	- 1、使用 + 或者 += 连接字符串
	- 使用 + 或者 += 连接字符串，要求右边的参数必须为字符串的切片引用（Slice）类型。其实当调用 + 的操作符时，相当于调用了 std::string 标准库中的 add() 方法，这里 add() 方法的第二个参数是一个引用的类型。因此我们在使用 + 时， 必须传递切片引用类型。不能直接传递 String 类型。+ 是返回一个新的字符串，所以变量声明可以不需要 mut 关键字修饰。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let string_append = String::from("hello ");
	   let string_rust = String::from("rust");
	   // &string_rust会自动解引用为&str
	   let result = string_append + &string_rust;
	   let mut result = result + "!"; // `result + "!"` 中的 `result` 是不可变的
	   result += "!!!";
	  
	   println!("连接字符串 + -> {}", result);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  连接字符串 + -> hello rust!!!!
	  ```
	- add() 方法的定义：
	  
	  ```
	  fn add(self, s: &str) -> String
	  ```
	- 因为该方法涉及到更复杂的特征功能，因此我们这里简单说明下：
	  
	  ```
	  fn main() {
	   let s1 = String::from("hello,");
	   let s2 = String::from("world!");
	   // 在下句中，s1的所有权被转移走了，因此后面不能再使用s1
	   let s3 = s1 + &s2;
	   assert_eq!(s3,"hello,world!");
	   // 下面的语句如果去掉注释，就会报错
	   // println!("{}",s1);
	  }
	  ```
	- self 是 String 类型的字符串 s1，该函数说明，只能将 &str 类型的字符串切片添加到 String 类型的 s1 上，然后返回一个新的 String 类型，所以 let s3 = s1 + &s2; 就很好解释了，将 String 类型的 s1 与 &str 类型的 s2 进行相加，最终得到 String 类型的 s3。
	- 由此可推，以下代码也是合法的：
	  
	  ```
	  let s1 = String::from("tic");
	  let s2 = String::from("tac");
	  let s3 = String::from("toe");
	  
	  // String = String + &str + &str + &str + &str
	  let s = s1 + "-" + &s2 + "-" + &s3;
	  ```
	- String + &str返回一个 String，然后再继续跟一个 &str 进行 + 操作，返回一个 String 类型，不断循环，最终生成一个 s，也是 String 类型。
	- s1 这个变量通过调用 add() 方法后，所有权被转移到 add() 方法里面， add() 方法调用后就被释放了，同时 s1 也被释放了。再使用 s1 就会发生错误。这里涉及到所有权转移（Move）的相关知识。
	- 2、使用 format! 连接字符串
	- format! 这种方式适用于 String 和 &str 。format! 的用法与 print! 的用法类似，详见格式化输出。
	- 示例代码如下：
	  
	  ```
	  fn main() {
	   let s1 = "hello";
	   let s2 = String::from("rust");
	   let s = format!("{} {}!", s1, s2);
	   println!("{}", s);
	  }
	  ```
	- 代码运行结果：
	  
	  ```
	  hello rust!
	  ```
- ### 4.1.6.  字符串转义
- 我们可以通过转义的方式 \ 输出 ASCII 和 Unicode 字符。
  
  ```
  fn main() {
    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);
  
    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";
  
    println!(
        "Unicode character {} (U+211D) is called {}",
        unicode_codepoint, character_name
    );
  
    // 换行了也会保持之前的字符串格式
    // 使用\忽略换行符
    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
  }
  ```
- 当然，在某些情况下，可能你会希望保持字符串的原样，不要转义：
  
  ```
  fn main() {
    println!("{}", "hello \\x52\\x75\\x73\\x74");
    let raw_str = r"Escapes don't work here: \x3F \u{211D}";
    println!("{}", raw_str);
  
    // 如果字符串包含双引号，可以在开头和结尾加 #
    let quotes = r#"And then I said: "There is no escape!""#;
    println!("{}", quotes);
  
    // 如果还是有歧义，可以继续增加，没有限制
    let longer_delimiter = r###"A string with "# in it. And even "##!"###;
    println!("{}", longer_delimiter);
  }
  ```
- ### 4.1.7.  操作UTF-8字符串
- 如果你想要以 Unicode 字符的方式遍历字符串，最好的办法是使用 chars 方法，例如：
  
  ```
  for c in "中国人".chars() {
    println!("{}", c);
  }
  ```
- 这种方式是返回字符串的底层字节数组表现形式：
  
  ```
  for b in "中国人".bytes() {
    println!("{}", b);
  }
  ```
- 想要准确的从 UTF-8 字符串中获取子串是较为复杂的事情，例如想要从 holla中国人नमस्ते 这种变长的字符串中取出某一个子串，使用标准库你是做不到的。 你需要在 crates.io 上搜索 utf8 来寻找想要的功能。
- 可以考虑尝试下这个库：utf8_slice。
- 那么问题来了，为啥 String 可变，而字符串字面值 str 却不可以？
- 就字符串字面值来说，我们在编译时就知道其内容，最终字面值文本被直接硬编码进可执行文件中，这使得字符串字面值快速且高效，这主要得益于字符串字面值的不可变性。不幸的是，我们不能为了获得这种性能，而把每一个在编译时大小未知的文本都放进内存中（你也做不到！），因为有的字符串是在程序运行得过程中动态生成的。
- 对于 String 类型，为了支持一个可变、可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容，这些都是在程序运行时完成的：
	- 首先向操作系统请求内存来存放 String 对象
	- 在使用完成后，将内存释放，归还给操作系统
- 其中第一部分由 String::from 完成，它创建了一个全新的 String。
- 重点来了，到了第二部分，就是百家齐放的环节，在有垃圾回收 GC 的语言中，GC 来负责标记并清除这些不再使用的内存对象，这个过程都是自动完成，无需开发者关心，非常简单好用；但是在无 GC 的语言中，需要开发者手动去释放这些内存对象，就像创建对象需要通过编写代码来完成一样，未能正确释放对象造成的后果简直不可估量。
- 对于 Rust 而言，安全和性能是写到骨子里的核心特性，如果使用 GC，那么会牺牲性能；如果使用手动管理内存，那么会牺牲安全，这该怎么办？为此，Rust 的开发者想出了一个无比惊艳的办法：变量在离开作用域后，就自动释放其占用的内存
- 与其它系统编程语言的 free 函数相同，Rust 也提供了一个释放内存的函数： drop，但是不同的是，其它语言要手动调用 free 来释放每一个变量占用的内存，而 Rust 则在变量离开作用域时，自动调用 drop 函数: 上面代码中，Rust 在结尾的 } 处自动调用 drop。
- ## 4.2.  元组
- 元组是由多种类型组合到一起形成的，因此它是复合类型，元组的长度是固定的，元组中元素的顺序也是固定的。
- 可以通过以下语法创建一个元组：
  
  ```
  fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
  }
  ```
- 变量 tup 被绑定了一个元组值 (500, 6.4, 1)，该元组的类型是 (i32, f64, u8)，看到没？元组是用括号将多个类型组合到一起，简单吧？
- 可以使用模式匹配或者 . 操作符来获取元组中的值。
- ### 4.2.1.  用模式匹配解构元组
  
  ```
  fn main() {
    let tup = (500, 6.4, 1);
  
    let (x, y, z) = tup;
  
    println!("The value of y is: {}", y);
  }
  ```
- 上述代码首先创建一个元组，然后将其绑定到 tup 上，接着使用 let (x, y, z) = tup; 来完成一次模式匹配，因为元组是 (n1, n2, n3) 形式的，因此我们用一模一样的 (x, y, z) 形式来进行匹配，元组中对应的值会绑定到变量 x， y， z上。这就是解构：用同样的形式把一个复杂对象中的值匹配出来。
- ### 4.2.2.  用 . 来访问元组
- 模式匹配可以让我们一次性把元组中的值全部或者部分获取出来，如果只想要访问某个特定元素，那模式匹配就略显繁琐，对此，Rust 提供了 . 的访问方式：
  
  ```
  fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);
  
    let five_hundred = x.0;
  
    let six_point_four = x.1;
  
    let one = x.2;
  }
  ```
- 和其它语言的数组、字符串一样，元组的索引从 0 开始。
- ### 4.2.3.  元组的使用示例
- 元组在函数返回值场景很常用，例如下面的代码，可以使用元组返回多个值：
  
  ```
  fn main() {
    let s1 = String::from("hello");
  
    let (s2, len) = calculate_length(s1);
  
    println!("The length of '{}' is {}.", s2, len);
  }
  
  fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度
  
    (s, length)
  }
  ```
- calculate_length 函数接收 s1 字符串的所有权，然后计算字符串的长度，接着把字符串所有权和字符串长度再返回给 s2 和 len 变量。
- 在其他语言中，可以用结构体来声明一个三维空间中的点，例如 Point(10, 20, 30)，虽然使用 Rust 元组也可以做到：(10, 20, 30)，但是这样写有个非常重大的缺陷。
- ## 4.3.  结构体
- 结构体跟之前讲过的元组有些相像：都是由多种类型组合而成。但是与元组不同的是，结构体可以为内部的每个字段起一个富有含义的名称。因此结构体更加灵活更加强大，你无需依赖这些字段的顺序来访问和解析它们。
- ### 4.3.1.  结构体语法
- 定义结构体
	- 通过关键字 struct 定义
	- 一个清晰明确的结构体 名称
	- 几个有名字的结构体 字段
- 例如, 以下结构体定义了某网站的用户：
  
  ```
  struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
  }
  ```
- 该结构体名称是 User，拥有 4 个字段，且每个字段都有对应的字段名及类型声明，例如 username 代表了用户名，是一个可变的 String 类型。
- 创建结构体实例
	- 为了使用上述结构体，我们需要创建 User 结构体的实例：
	  
	  ```
	  let user1 = User {
	       email: String::from("someone@example.com"),
	       username: String::from("someusername123"),
	       active: true,
	       sign_in_count: 1,
	   };
	  ```
- 有几点值得注意:
	- 初始化实例时，每个字段都需要进行初始化
	- 初始化时的字段顺序不需要和结构体定义时的顺序一致
- 访问结构体字段
	- 通过 . 操作符即可访问结构体实例内部的字段值，也可以修改它们：
	  
	  ```
	  let mut user1 = User {
	       email: String::from("someone@example.com"),
	       username: String::from("someusername123"),
	       active: true,
	       sign_in_count: 1,
	   };
	  
	   user1.email = String::from("anotheremail@example.com");
	  ```
	- 需要注意的是，必须要将结构体实例声明为可变的，才能修改其中的字段，Rust 不支持将某个结构体某个字段标记为可变。
- 简化结构体创建
	- 下面的函数类似一个构建函数，返回了 User 结构体的实例：
	  
	  ```
	  fn build_user(email: String, username: String) -> User {
	   User {
	       email: email,
	       username: username,
	       active: true,
	       sign_in_count: 1,
	   }
	  }
	  ```
	- 它接收两个字符串参数： email 和 username，然后使用它们来创建一个 User 结构体，并且返回。可以注意到这两行： email: email 和 username: username，非常的扎眼，因为实在有些啰嗦，如果你从 TypeScript 过来，肯定会鄙视 Rust 一番，不过好在，它也不是无可救药：
	  
	  ```
	  fn build_user(email: String, username: String) -> User {
	   User {
	       email,
	       username,
	       active: true,
	       sign_in_count: 1,
	   }
	  }
	  ```
	- 如上所示，当函数参数和结构体字段同名时，可以直接使用缩略的方式进行初始化，跟 TypeScript 中一模一样。
- 结构体更新语法
	- 在实际场景中，有一种情况很常见：根据已有的结构体实例，创建新的结构体实例，例如根据已有的 user1 实例来构建 user2：
	  
	  ```
	  let user2 = User {
	       active: user1.active,
	       username: user1.username,
	       email: String::from("another@example.com"),
	       sign_in_count: user1.sign_in_count,
	   };
	  ```
	- 如果你从 TypeScript 过来，肯定觉得啰嗦爆了：竟然手动把 user1 的三个字段逐个赋值给 user2，好在 Rust 为我们提供了 结构体更新语法：
	  
	  ```
	  let user2 = User {
	       email: String::from("another@example.com"),
	       ..user1
	   };
	  ```
	- 因为 user2 仅仅在 email 上与 user1 不同，因此我们只需要对 email 进行赋值，剩下的通过结构体更新语法 ..user1 即可完成。
	- .. 语法表明凡是我们没有显式声明的字段，全部从 user1 中自动获取。需要注意的是 ..user1 必须在结构体的尾部使用。
	- 结构体更新语法跟赋值语句 = 非常相像，因此在上面代码中，user1 的部分字段所有权被转移到 user2 中：username 字段发生了所有权转移，作为结果，user1 无法再被使用。
	- 聪明的读者肯定要发问了：明明有三个字段进行了自动赋值，为何只有 username 发生了所有权转移？
	- 仔细回想一下所有权那一节的内容，我们提到了 Copy 特征：实现了 Copy 特征的类型无需所有权转移，可以直接在赋值时进行 数据拷贝，其中 bool 和 u64 类型就实现了 Copy 特征，因此 active 和 sign_in_count 字段在赋值给 user2 时，仅仅发生了拷贝，而不是所有权转移。
	- 值得注意的是：username 所有权被转移给了 user2，导致了 user1 无法再被使用，但是并不代表 user1 内部的其它字段不能被继续使用，例如：
	  
	  ```
	  let user1 = User {
	   email: String::from("someone@example.com"),
	   username: String::from("someusername123"),
	   active: true,
	   sign_in_count: 1,
	  };
	  let user2 = User {
	   active: user1.active,
	   username: user1.username,
	   email: String::from("another@example.com"),
	   sign_in_count: user1.sign_in_count,
	  };
	  println!("{}", user1.active);
	  // 下面这行会报错
	  println!("{:?}", user1);
	  ```
- ### 4.3.2.  结构体的内存排列
- 先来看以下代码：
  
  ```
  #[derive(Debug)]
  struct File {
   name: String,
   data: Vec<u8>,
  }
  
  fn main() {
   let f1 = File {
     name: String::from("f1.txt"),
     data: Vec::new(),
   };
  
   let f1_name = &f1.name;
   let f1_length = &f1.data.len();
  
   println!("{:?}", f1);
   println!("{} is {} bytes long", f1_name, f1_length);
  }
  ```
- 上面定义的 File 结构体在内存中的排列如下图所示：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1721951660698-ca9920a6-cf9a-4178-91a7-69c73613339a.png)
- 从图中可以清晰地看出 File 结构体两个字段 name 和 data 分别拥有底层两个 [u8] 数组的所有权(String 类型的底层也是 [u8] 数组)，通过 ptr 指针指向底层数组的内存地址，这里你可以把 ptr 指针理解为 Rust 中的引用类型。
- 该图片也侧面印证了：把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段。
- ### 4.3.3.  元组结构体
- 结构体必须要有名称，但是结构体的字段可以没有名称，这种结构体长得很像元组，因此被称为元组结构体，例如：
  
  ```
  struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);
  
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
  ```
- 元组结构体在你希望有一个整体名称，但是又不关心里面字段的名称时将非常有用。例如上面的 Point 元组结构体，众所周知 3D 点是 (x, y, z) 形式的坐标点，因此我们无需再为内部的字段逐一命名为：x, y, z。
- ### 4.3.4.  单元结构体
- 还记得之前讲过的基本没啥用的单元类型吧？单元结构体就跟它很像，没有任何字段和属性，但是好在，它还挺有用。
- 如果你定义一个类型，但是不关心该类型的内容, 只关心它的行为时，就可以使用 单元结构体：
  
  ```
  struct AlwaysEqual;
  
  let subject = AlwaysEqual;
  
  // 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
  impl SomeTrait for AlwaysEqual {
  
  }
  ```
- ### 4.3.5.  结构体数据的所有权
- 在之前的 User 结构体的定义中，有一处细节：我们使用了自身拥有所有权的 String 类型而不是基于引用的 &str 字符串切片类型。这是一个有意而为之的选择：因为我们想要这个结构体拥有它所有的数据，而不是从其它地方借用数据。
- 你也可以让 User 结构体从其它对象借用数据，不过这么做，就需要引入生命周期(lifetimes)这个新概念（也是一个复杂的概念），简而言之，生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小。
- 总之，如果你想在结构体中使用一个引用，就必须加上生命周期，否则就会报错：
  
  ```
  struct User {
    username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
  }
  
  fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
  }
  ```
- 编译器会抱怨它需要生命周期标识符：
  
  ```
  error[E0106]: missing lifetime specifier
  --> src/main.rs:2:15
  |
  2 |     username: &str,
  |               ^ expected named lifetime parameter // 需要一个生命周期
  |
  help: consider introducing a named lifetime parameter // 考虑像下面的代码这样引入一个生命周期
  |
  1 ~ struct User<'a> {
  2 ~     username: &'a str,
  |
  
  error[E0106]: missing lifetime specifier
  --> src/main.rs:3:12
  |
  3 |     email: &str,
  |            ^ expected named lifetime parameter
  |
  help: consider introducing a named lifetime parameter
  |
  1 ~ struct User<'a> {
  2 |     username: &str,
  3 ~     email: &'a str,
  |
  ```
- ## 4.4.  枚举
- 枚举(enum 或 enumeration)允许你通过列举可能的成员来定义一个枚举类型，例如扑克牌花色：
  
  ```
  enum PokerSuit {
  Clubs,
  Spades,
  Diamonds,
  Hearts,
  }
  ```
- 如果在此之前你没有在其它语言中使用过枚举，那么可能需要花费一些时间来理解这些概念，一旦上手，就会发现枚举的强大，甚至对它爱不释手，枚举虽好，可不要滥用哦。
- 再回到之前创建的 PokerSuit，扑克总共有四种花色，而这里我们枚举出所有的可能值，这也正是 枚举 名称的由来。
- 任何一张扑克，它的花色肯定会落在四种花色中，而且也只会落在其中一个花色上，这种特性非常适合枚举的使用，因为枚举值只可能是其中某一个成员。抽象来看，四种花色尽管是不同的花色，但是它们都是扑克花色这个概念，因此当某个函数处理扑克花色时，可以把它们当作相同的类型进行传参。
- 细心的读者应该注意到，我们对之前的 枚举类型 和 枚举值 进行了重点标注，这是因为对于新人来说容易混淆相应的概念，总而言之： 枚举类型是一个类型，它会包含所有可能的枚举成员, 而枚举值是该类型中的具体某个成员的实例。
- ### 4.4.1.  枚举值
- 现在来创建 PokerSuit 枚举类型的两个成员实例：
  
  ```
  let heart = PokerSuit::Hearts;
  let diamond = PokerSuit::Diamonds;
  ```
- 我们通过 :: 操作符来访问 PokerSuit 下的具体成员，从代码可以清晰看出，heart 和 diamond 都是 PokerSuit 枚举类型的，接着可以定义一个函数来使用它们：
  
  ```
  fn main() {
    let heart = PokerSuit::Hearts;
    let diamond = PokerSuit::Diamonds;
  
    print_suit(heart);
    print_suit(diamond);
  }
  
  fn print_suit(card: PokerSuit) {
    // 需要在定义 enum PokerSuit 的上面添加上 #[derive(Debug)]，否则会报 card 没有实现 Debug
    println!("{:?}",card);
  }
  ```
- print_suit 函数的参数类型是 PokerSuit，因此我们可以把 heart 和 diamond 传给它，虽然 heart 是基于 PokerSuit 下的 Hearts 成员实例化的，但是它是货真价实的 PokerSuit 枚举类型。
- 接下来，我们想让扑克牌变得更加实用，那么需要给每张牌赋予一个值：A(1)-K(13)，这样再加上花色，就是一张真实的扑克牌了，例如红心 A。
- 目前来说，枚举值还不能带有值，因此先用结构体来实现：
  
  ```
  enum PokerSuit {
    Clubs,
    Spades,
    Diamonds,
    Hearts,
  }
  
  struct PokerCard {
    suit: PokerSuit,
    value: u8
  }
  
  fn main() {
   let c1 = PokerCard {
       suit: PokerSuit::Clubs,
       value: 1,
   };
   let c2 = PokerCard {
       suit: PokerSuit::Diamonds,
       value: 12,
   };
  }
  ```
- 这段代码很好的完成了它的使命，通过结构体 PokerCard 来代表一张牌，结构体的 suit 字段表示牌的花色，类型是 PokerSuit 枚举类型，value 字段代表扑克牌的数值。
- 可以吗？可以！好吗？说实话，不咋地，因为还有简洁得多的方式来实现：
  
  ```
  enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(u8),
    Hearts(u8),
  }
  
  fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds(13);
  }
  ```
- 直接将数据信息关联到枚举成员上，省去近一半的代码，这种实现是不是更优雅？
- 不仅如此，同一个枚举类型下的不同成员还能持有不同的数据类型，例如让某些花色打印 1-13 的字样，另外的花色打印上 A-K 的字样：
  
  ```
  enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(char),
    Hearts(char),
  }
  
  fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds('A');
  }
  ```
- 回想一下，遇到这种不同类型的情况，再用我们之前的结构体实现方式，可行吗？也许可行，但是会复杂很多。
- 再来看一个来自标准库中的例子：
  
  ```
  struct Ipv4Addr {
    // --snip--
  }
  
  struct Ipv6Addr {
    // --snip--
  }
  
  enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
  }
  ```
- 这个例子跟我们之前的扑克牌很像，只不过枚举成员包含的类型更复杂了，变成了结构体：分别通过 Ipv4Addr 和 Ipv6Addr 来定义两种不同的 IP 数据。
- 从这些例子可以看出，任何类型的数据都可以放入枚举成员中: 例如字符串、数值、结构体甚至另一个枚举。
- 增加一些挑战？先看以下代码：
  
  ```
  enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
  }
  
  fn main() {
    let m1 = Message::Quit;
    let m2 = Message::Move{x:1,y:1};
    let m3 = Message::ChangeColor(255,255,0);
  }
  ```
- 该枚举类型代表一条消息，它包含四个不同的成员：
	- Quit 没有任何关联数据
	- Move 包含一个匿名结构体
	- Write 包含一个 String 字符串
	- ChangeColor 包含三个 i32
- 当然，我们也可以用结构体的方式来定义这些消息：
  
  ```
  struct QuitMessage; // 单元结构体
  struct MoveMessage {
    x: i32,
    y: i32,
  }
  struct WriteMessage(String); // 元组结构体
  struct ChangeColorMessage(i32, i32, i32); // 元组结构体
  ```
- 由于每个结构体都有自己的类型，因此我们无法在需要同一类型的地方进行使用，例如某个函数它的功能是接受消息并进行发送，那么用枚举的方式，就可以接收不同的消息，但是用结构体，该函数无法接受 4 个不同的结构体作为参数。
- 而且从代码规范角度来看，枚举的实现更简洁，代码内聚性更强，不像结构体的实现，分散在各个地方。
- ### 4.4.2.  同一化类型
- 最后，再用一个实际项目中的简化片段，来结束枚举类型的语法学习。
- 例如我们有一个 WEB 服务，需要接受用户的长连接，假设连接有两种：TcpStream 和 TlsStream，但是我们希望对这两个连接的处理流程相同，也就是用同一个函数来处理这两个连接，代码如下：
  
  ```
  fn new (stream: TcpStream) {
  let mut s = stream;
  if tls {
    s = negotiate_tls(stream)
  }
  
  // websocket是一个WebSocket<TcpStream>或者
  //   WebSocket<native_tls::TlsStream<TcpStream>>类型
  websocket = WebSocket::from_raw_socket(
    s, ......)
  }
  ```
- 此时，枚举类型就能帮上大忙：
  
  ```
  enum Websocket {
  Tcp(Websocket<TcpStream>),
  Tls(Websocket<native_tls::TlsStream<TcpStream>>),
  }
  ```
- ### 4.4.3.  Option 枚举用于处理空值
- 在其它编程语言中，往往都有一个 null 关键字，该关键字用于表明一个变量当前的值为空（不是零值，例如整型的零值是 0），也就是不存在值。当你对这些 null 进行操作时，例如调用一个方法，就会直接抛出null 异常，导致程序的崩溃，因此我们在编程时需要格外的小心去处理这些 null 空值。
  
  Tony Hoare， null 的发明者，曾经说过一段非常有名的话：
  
  我称之为我十亿美元的错误。当时，我在使用一个面向对象语言设计第一个综合性的面向引用的类型系统。我的目标是通过编译器的自动检查来保证所有引用的使用都应该是绝对安全的。不过在设计过程中，我未能抵抗住诱惑，引入了空引用的概念，因为它非常容易实现。就是因为这个决策，引发了无数错误、漏洞和系统崩溃，在之后的四十多年中造成了数十亿美元的苦痛和伤害。
- 尽管如此，空值的表达依然非常有意义，因为空值表示当前时刻变量的值是缺失的。有鉴于此，Rust 吸取了众多教训，决定抛弃 null，而改为使用 Option 枚举变量来表述这种结果。
- Option 枚举包含两个成员，一个成员表示含有值：Some(T), 另一个表示没有值：None，定义如下：
  
  ```
  enum Option<T> {
    Some(T),
    None,
  }
  ```
- 其中 T 是泛型参数，Some(T)表示该枚举成员的数据类型是 T，换句话说，Some 可以包含任何类型的数据。
- Option<T> 枚举是如此有用以至于它被包含在了 prelude（prelude 属于 Rust 标准库，Rust 会将最常用的类型、函数等提前引入其中，省得我们再手动引入）之中，你不需要将其显式引入作用域。另外，它的成员 Some 和 None 也是如此，无需使用 Option:: 前缀就可直接使用 Some 和 None。总之，不能因为 Some(T) 和 None 中没有 Option:: 的身影，就否认它们是 Option 下的卧龙凤雏。
- 再来看以下代码：
  
  ```
  let some_number = Some(5);
  let some_string = Some("a string");
  
  let absent_number: Option<i32> = None;
  ```
- 如果使用 None 而不是 Some，需要告诉 Rust Option<T> 是什么类型的，因为编译器只通过 None 值无法推断出 Some 成员保存的值的类型。
- 当有一个 Some 值时，我们就知道存在一个值，而这个值保存在 Some 中。当有个 None 值时，在某种意义上，它跟空值具有相同的意义：并没有一个有效的值。那么，Option<T> 为什么就比空值要好呢？
- 简而言之，因为 Option<T> 和 T（这里 T 可以是任何类型）是不同的类型，例如，这段代码不能编译，因为它尝试将 Option<i8>(Option<T>) 与 i8(T) 相加：
  
  ```
  let x: i8 = 5;
  let y: Option<i8> = Some(5);
  
  let sum = x + y;
  ```
- 如果运行这些代码，将得到类似这样的错误信息：
  
  ```
  error[E0277]: the trait bound `i8: std::ops::Add<std::option::Option<i8>>` is
  not satisfied
  -->
  |
  5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + std::option::Option<i8>`
  |
  ```
- 很好！事实上，错误信息意味着 Rust 不知道该如何将 Option<i8> 与 i8 相加，因为它们的类型不同。当在 Rust 中拥有一个像 i8 这样类型的值时，编译器确保它总是有一个有效的值，我们可以放心使用而无需做空值检查。只有当使用 Option<i8>（或者任何用到的类型）的时候才需要担心可能没有值，而编译器会确保我们在使用值之前处理了为空的情况。
- 换句话说，在对 Option<T> 进行 T 的运算之前必须将其转换为 T。通常这能帮助我们捕获到空值最常见的问题之一：期望某值不为空但实际上为空的情况。
- 不再担心会错误的使用一个空值，会让你对代码更加有信心。为了拥有一个可能为空的值，你必须要显式的将其放入对应类型的 Option<T> 中。接着，当使用这个值时，必须明确的处理值为空的情况。只要一个值不是 Option<T> 类型，你就 可以 安全的认定它的值不为空。这是 Rust 的一个经过深思熟虑的设计决策，来限制空值的泛滥以增加 Rust 代码的安全性。
- 那么当有一个 Option<T> 的值时，如何从 Some 成员中取出 T 的值来使用它呢？Option<T> 枚举拥有大量用于各种情况的方法：你可以查看它的文档。熟悉 Option<T> 的方法将对你的 Rust 之旅非常有用。
- 总的来说，为了使用 Option<T> 值，需要编写处理每个成员的代码。你想要一些代码只当拥有 Some(T) 值时运行，允许这些代码使用其中的 T。也希望一些代码在值为 None 时运行，这些代码并没有一个可用的 T 值。match 表达式就是这么一个处理枚举的控制流结构：它会根据枚举的成员运行不同的代码，这些代码可以使用匹配到的值中的数据。
- 这里先简单看一下 match 的大致模样，在模式匹配中，我们会详细讲解：
  
  ```
  fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
  }
  
  let five = Some(5);
  let six = plus_one(five);
  let none = plus_one(None);
  ```
- plus_one 通过 match 来处理不同 Option 的情况。
- ## 4.5.  数组
- 使用最广的数据结构之一就是数组，在 Rust 中，最常用的数组有两种，第一种是速度很快但是长度固定的 array，第二种是可动态增长的但是有性能损耗的 Vector，在本书中，我们称 array 为数组，Vector 为动态数组。
- 不知道你们发现没，这两个数组的关系跟 &str 与 String 的关系很像，前者是长度固定的字符串切片，后者是可动态增长的字符串。其实，在 Rust 中无论是 String 还是 Vector，它们都是 Rust 的高级类型：集合类型，在后面章节会有详细介绍。
- 对于本章节，我们的重点还是放在数组 array 上。数组的具体定义很简单：将多个类型相同的元素依次组合在一起，就是一个数组。结合上面的内容，可以得出数组的三要素：
	- 长度固定
	- 元素必须有相同的类型
	- 依次线性排列
- 这里再啰嗦一句，我们这里说的数组是 Rust 的基本类型，是固定长度的，这点与其他编程语言不同，其它编程语言的数组往往是可变长度的，与 Rust 中的动态数组 Vector 类似。
- ### 4.5.1.  创建数组
- 在 Rust 中，数组是这样定义的：
  
  ```
  fn main() {
    let a = [1, 2, 3, 4, 5];
  }
  ```
- 数组语法跟 JavaScript 很像，也跟大多数编程语言很像。由于它的元素类型大小固定，且长度也是固定，因此数组 array 是存储在栈上，性能也会非常优秀。与此对应，动态数组 Vector 是存储在堆上，因此长度可以动态改变。当你不确定是使用数组还是动态数组时，那就应该使用后者，具体见动态数组 Vector。
- 举个例子，在需要知道一年中各个月份名称的程序中，你很可能希望使用的是数组而不是动态数组。因为月份是固定的，它总是只包含 12 个元素：
  
  ```
  let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
  ```
- 在一些时候，还需要为数组声明类型，如下所示：
  
  ```
  let a: [i32; 5] = [1, 2, 3, 4, 5];
  ```
- 这里，数组类型是通过方括号语法声明，i32 是元素类型，分号后面的数字 5 是数组长度，数组类型也从侧面说明了数组的元素类型要统一，长度要固定。
- 还可以使用下面的语法初始化一个某个值重复出现 N 次的数组：
  
  ```
  let a = [3; 5];
  ```
- a 数组包含 5 个元素，这些元素的初始化值为 3，聪明的读者已经发现，这种语法跟数组类型的声明语法其实是保持一致的：[3; 5] 和 [类型; 长度]。
- 在元素重复的场景，这种写法要简单的多，否则你就得疯狂敲击键盘：let a = [3, 3, 3, 3, 3];，不过老板可能很喜欢你的这种疯狂编程的状态。
- ### 4.5.2.  访问数组元素
- 因为数组是连续存放元素的，因此可以通过索引的方式来访问存放其中的元素：
  
  ```
  fn main() {
    let a = [9, 8, 7, 6, 5];
  
    let first = a[0]; // 获取a数组第一个元素
    let second = a[1]; // 获取第二个元素
  }
  ```
- 与许多语言类似，数组的索引下标是从 0 开始的。此处，first 获取到的值是 9，second 是 8。
- 越界访问
	- 如果使用超出数组范围的索引访问数组元素，会怎么样？下面是一个接收用户的控制台输入，然后将其作为索引访问数组元素的例子：
	  
	  ```
	  use std::io;
	  
	  fn main() {
	   let a = [1, 2, 3, 4, 5];
	  
	   println!("Please enter an array index.");
	  
	   let mut index = String::new();
	   // 读取控制台的输出
	   io::stdin()
	       .read_line(&mut index)
	       .expect("Failed to read line");
	  
	   let index: usize = index
	       .trim()
	       .parse()
	       .expect("Index entered was not a number");
	  
	   let element = a[index];
	  
	   println!(
	       "The value of the element at index {} is: {}",
	       index, element
	   );
	  }
	  ```
- 使用 cargo run 来运行代码，因为数组只有 5 个元素，如果我们试图输入 5 去访问第 6 个元素，则会访问到不存在的数组元素，最终程序会崩溃退出：
  
  ```
  Please enter an array index.
  5
  thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 5', src/main.rs:19:19
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
  ```
- 这就是数组访问越界，访问了数组中不存在的元素，导致 Rust 运行时错误。程序因此退出并显示错误消息，未执行最后的 println! 语句。
- 当你尝试使用索引访问元素时，Rust 将检查你指定的索引是否小于数组长度。如果索引大于或等于数组长度，Rust 会出现 panic。这种检查只能在运行时进行，比如在上面这种情况下，编译器无法在编译期知道用户运行代码时将输入什么值。
- 这种就是 Rust 的安全特性之一。在很多系统编程语言中，并不会检查数组越界问题，你会访问到无效的内存地址获取到一个风马牛不相及的值，最终导致在程序逻辑上出现大问题，而且这种问题会非常难以检查。
- 数组元素为非基础类型
- 学习了上面的知识，很多朋友肯定觉得已经学会了Rust的数组类型，但现实会给我们一记重锤，实际开发中还会碰到一种情况，就是数组元素是非基本类型的，这时候大家一定会这样写。
  
  ```
  let array = [String::from("rust is good!"); 8];
  
  println!("{:#?}", array);
  ```
- 有些还没有看过特征的小伙伴，有可能不太明白这个报错，不过这个目前可以不提，我们就拿之前所学的所有权知识，就可以思考明白，前面几个例子都是Rust的基本类型，而基本类型在Rust中赋值是以Copy的形式，这时候你就懂了吧，let array=[3;5]底层就是不断的Copy出来的，但很可惜复杂类型都没有深拷贝，只能一个个创建。
- 接着就有小伙伴会这样写。
  
  ```
  let array = [String::from("rust is good!"),String::from("rust is good!"),String::from("rust is good!")];
  
  println!("{:#?}", array);
  ```
- 正确的写法，应该调用std::array::from_fn
  
  ```
  let array: [String; 8] = std::array::from_fn(|_i| String::from("rust is good!"));
  
  println!("{:#?}", array);
  ```
- ### 4.5.3.  数组切片
- 切片 这个概念，它允许你引用集合中的部分连续片段，而不是整个集合，对于数组也是，数组切片允许我们引用数组的一部分：
  
  ```
  let a: [i32; 5] = [1, 2, 3, 4, 5];
  
  let slice: &[i32] = &a[1..3];
  
  assert_eq!(slice, &[2, 3]);
  ```
- 上面的数组切片 slice 的类型是&[i32]，与之对比，数组的类型是[i32;5]，简单总结下切片的特点：
	- 切片的长度可以与数组不同，并不是固定的，而是取决于你使用时指定的起始和结束位置
	- 创建切片的代价非常小，因为切片只是针对底层数组的一个引用
	- 切片类型[T]拥有不固定的大小，而切片引用类型&[T]则具有固定的大小，因为 Rust 很多时候都需要固定大小数据类型，因此&[T]更有用,&str字符串切片也同理
	  
	  ```
	  fn main() {
	  // 编译器自动推导出one的类型
	  let one             = [1, 2, 3];
	  // 显式类型标注
	  let two: [u8; 3]    = [1, 2, 3];
	  let blank1          = [0; 3];
	  let blank2: [u8; 3] = [0; 3];
	  
	  // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
	  let arrays: [[u8; 3]; 4]  = [one, two, blank1, blank2];
	  
	  // 借用arrays的元素用作循环中
	  for a in &arrays {
	   print!("{:?}: ", a);
	   // 将a变成一个迭代器，用于循环
	   // 你也可以直接用for n in a {}来进行循环
	   for n in a.iter() {
	     print!("\t{} + 10 = {}", n, n+10);
	   }
	  
	   let mut sum = 0;
	   // 0..a.len,是一个 Rust 的语法糖，其实就等于一个数组，元素是从0,1,2一直增加到到a.len-1
	   for i in 0..a.len() {
	     sum += a[i];
	   }
	   println!("\t({:?} = {})", a, sum);
	  }
	  }
	  ```
- 做个总结，数组虽然很简单，但是其实还是存在几个要注意的点：
	- 数组类型容易跟数组切片混淆，[T;n]描述了一个数组的类型，而[T]描述了切片的类型， 因为切片是运行期的数据结构，它的长度无法在编译期得知，因此不能用[T;n]的形式去描述
	- [u8; 3]和[u8; 4]是不同的类型，数组的长度也是类型的一部分
	- 在实际开发中，使用最多的是数组切片[T]，我们往往通过引用的方式去使用&[T]，因为后者有固定的类型大小
- # 5.  流程控制
- ## 5.1.  if和if-else
- 总之，只要你拥有其它语言的编程经验，就一定会有以下认知：if else 表达式根据条件执行不同的代码分支：
  
  ```
  if condition == true {
    // A...
  } else {
    // B...
  }
  ```
- 该代码读作：若 condition 的值为 true，则执行 A 代码，否则执行 B 代码。
- 先看下面代码：
  
  ```
  fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };
  
    println!("The value of number is: {}", number);
  }
  ```
- 以上代码有以下几点要注意：
	- if 语句块是表达式，这里我们使用 if 表达式的返回值来给 number 进行赋值：number 的值是 5
	- 用 if 来赋值时，要保证每个分支返回的类型一样(事实上，这种说法不完全准确，见这里)，此处返回的 5 和 6 就是同一个类型，如果返回类型不一致就会报错
	  
	  ```
	  error[E0308]: if and else have incompatible types
	  --> src/main.rs:4:18
	  |
	  4 |       let number = if condition {
	  |  __________________^
	  5 | |         5
	  6 | |     } else {
	  7 | |         "six"
	  8 | |     };
	  | |_____^ expected integer, found &str // 期望整数类型，但却发现&str字符串切片
	  |
	  = note: expected type `{integer}`
	            found type `&str`
	  ```
- 使用 else if 来处理多重条件
- 可以将 else if 与 if、else 组合在一起实现更复杂的条件分支判断：
  
  ```
  fn main() {
    let n = 6;
  
    if n % 4 == 0 {
        println!("number is divisible by 4");
    } else if n % 3 == 0 {
        println!("number is divisible by 3");
    } else if n % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
  }
  ```
- 程序执行时，会按照自上至下的顺序执行每一个分支判断，一旦成功，则跳出 if 语句块，最终本程序会匹配执行 else if n % 3 == 0 的分支，输出 "number is divisible by 3"。
- 有一点要注意，就算有多个分支能匹配，也只有第一个匹配的分支会被执行！
- 如果代码中有大量的 else if 会让代码变得极其丑陋，不过不用担心，下一章的 match 专门用以解决多分支模式匹配的问题。
- ## 5.2.  for循环
- for 循环是 Rust 的大杀器：
  
  ```
  fn main() {
    for i in 1..=5 {
        println!("{}", i);
    }
  }
  ```
- 以上代码循环输出一个从 1 到 5 的序列，简单粗暴，核心就在于 for 和 in 的联动，语义表达如下：
  
  ```
  for 元素 in 集合 {
  // 使用元素干一些你懂我不懂的事情
  }
  ```
- 这个语法跟 JavaScript 还蛮像，应该挺好理解。
- 注意，使用 for 时我们往往使用集合的引用形式，除非你不想在后面的代码中继续使用该集合（比如我们这里使用了 container 的引用）。如果不使用引用的话，所有权会被转移（move）到 for 语句块中，后面就无法再使用这个集合了)：
  
  ```
  for item in &container {
  // ...
  }
  ```
- 对于实现了 copy 特征的数组(例如 [i32; 10] )而言， for item in arr 并不会把 arr 的所有权转移，而是直接对其进行了拷贝，因此循环之后仍然可以使用 arr 。
- 如果想在循环中，修改该元素，可以使用 mut 关键字：
  
  ```
  for item in &mut collection {
  // ...
  }
  ```
- 总结如下：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1722733585148-34b71c21-b782-4934-9ed5-98e6d510959e.png)
- 两种循环方式优劣对比
- 以下代码，使用了两种循环方式：
  
  ```
  // 第一种
  let collection = [1, 2, 3, 4, 5];
  for i in 0..collection.len() {
  let item = collection[i];
  // ...
  }
  
  // 第二种
  for item in collection {
  
  }
  ```
- 第一种方式是循环索引，然后通过索引下标去访问集合，第二种方式是直接循环集合中的元素，优劣如下：
	- 性能：第一种使用方式中 collection[index] 的索引访问，会因为边界检查(Bounds Checking)导致运行时的性能损耗 —— Rust 会检查并确认 index 是否落在集合内，但是第二种直接迭代的方式就不会触发这种检查，因为编译器会在编译时就完成分析并证明这种访问是合法的
	- 安全：第一种方式里对 collection 的索引访问是非连续的，存在一定可能性在两次访问之间，collection 发生了变化，导致脏数据产生。而第二种直接迭代的方式是连续访问，因此不存在这种风险( 由于所有权限制，在访问过程中，数据并不会发生变化)。
- 由于 for 循环无需任何条件限制，也不需要通过索引来访问，因此是最安全也是最常用的，通过与下面的 while 的对比，我们能看到为什么 for 会更加安全。
- continue用于跳过当前当次的循环，开始下次的循环，break用于直接跳出当前整个循环
- ## 5.3.  while循环
- 如果你需要一个条件来循环，当该条件为 true 时，继续循环，条件为 false，跳出循环，那么 while 就非常适用：
  
  ```
  fn main() {
    let mut n = 0;
  
    while n <= 5  {
        println!("{}!", n);
  
        n = n + 1;
    }
  
    println!("我出来了！");
  }
  ```
- 该 while 循环，只有当 n 小于等于 5 时，才执行，否则就立刻跳出循环，因此在上述代码中，它会先从 0 开始，满足条件，进行循环，然后是 1，满足条件，进行循环，最终到 6 的时候，大于 5，不满足条件，跳出 while 循环，执行 我出来了 的打印，然后程序结束：
  
  ```
  0!
  1!
  2!
  3!
  4!
  5!
  我出来了！
  ```
- 可以看出，for 并不会使用索引去访问数组，因此更安全也更简洁，同时避免 运行时的边界检查，性能更高。
- ## 5.4.  loop循环
- 对于循环而言，loop 循环毋庸置疑，是适用面最高的，它可以适用于所有循环场景（虽然能用，但是在很多场景下， for 和 while 才是最优选择），因为 loop 就是一个简单的无限循环，你可以在内部实现逻辑通过 break 关键字来控制循环何时结束。
- 使用 loop 循环一定要打起精神，否则你会写出下面的跑满你一个 CPU 核心的疯子代码：
  
  ```
  fn main() {
    loop {
        println!("again!");
    }
  }
  ```
- 当使用 loop 时，必不可少的伙伴是 break 关键字，它能让循环在满足某个条件时跳出：
  
  ```
  fn main() {
    let mut counter = 0;
  
    let result = loop {
        counter += 1;
  
        if counter == 10 {
            break counter * 2;
        }
    };
  
    println!("The result is {}", result);
  }
  ```
- 这里有几点值得注意：
	- break 可以单独使用，也可以带一个返回值，有些类似 return
	- loop 是一个表达式，因此可以返回一个值
- # 6.  模式匹配
- ## 6.1.  match和if let
- 在 Rust 中，模式匹配最常用的就是 match 和 if let，本章节将对两者及相关的概念进行详尽介绍。
- 先来看一个关于 match 的简单例子：
  
  ```
  enum Direction {
    East,
    West,
    North,
    South,
  }
  
  fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => println!("West"),
    };
  }
  ```
- 这里我们想去匹配 dire 对应的枚举类型，因此在 match 中用三个匹配分支来完全覆盖枚举变量 Direction 的所有成员类型，有以下几点值得注意：
	- match 的匹配必须要穷举出所有可能，因此这里用 _ 来代表未列出的所有可能性
	- match 的每一个分支都必须是一个表达式，且所有分支的表达式最终返回值的类型必须相同
	- X | Y，类似逻辑运算符 或，代表该分支可以匹配 X 也可以匹配 Y，只要满足一个即可
- 其实 match 跟其他语言中的 switch 非常像，_ 类似于 switch 中的 default。
- ### 6.1.1.  match匹配
- 首先来看看 match 的通用形式：
  
  ```
  match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
  }
  ```
- 该形式清晰的说明了何为模式，何为模式匹配：将模式与 target 进行匹配，即为模式匹配，而模式匹配不仅仅局限于 match，后面我们会详细阐述。
- match 允许我们将一个值与一系列的模式相比较，并根据相匹配的模式执行对应的代码，下面让我们来一一详解，先看一个例子：
  
  ```
  enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
  }
  
  fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny =>  {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
  }
  ```
- value_in_cents 函数根据匹配到的硬币，返回对应的美分数值。match 后紧跟着的是一个表达式，跟 if 很像，但是 if 后的表达式必须是一个布尔值，而 match 后的表达式返回值可以是任意类型，只要能跟后面的分支中的模式匹配起来即可，这里的 coin 是枚举 Coin 类型。
- 接下来是 match 的分支。一个分支有两个部分：一个模式和针对该模式的处理代码。第一个分支的模式是 Coin::Penny，其后的 => 运算符将模式和将要运行的代码分开。这里的代码就仅仅是表达式 1，不同分支之间使用逗号分隔。
- 当 match 表达式执行时，它将目标值 coin 按顺序依次与每一个分支的模式相比较，如果模式匹配了这个值，那么模式之后的代码将被执行。如果模式并不匹配这个值，将继续执行下一个分支。
- 每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 match 表达式的返回值。如果分支有多行代码，那么需要用 {} 包裹，同时最后一行代码需要是一个表达式。
- 模式绑定
- 模式匹配的另外一个重要功能是从模式中取出绑定的值，例如：
  
  ```
  #[derive(Debug)]
  enum UsState {
    Alabama,
    Alaska,
    // --snip--
  }
  
  enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), // 25美分硬币
  }
  ```
- 其中 Coin::Quarter 成员还存放了一个值：美国的某个州（因为在 1999 年到 2008 年间，美国在 25 美分(Quarter)硬币的背后为 50 个州印刷了不同的标记，其它硬币都没有这样的设计）。
- 接下来，我们希望在模式匹配中，获取到 25 美分硬币上刻印的州的名称：
  
  ```
  fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
  }
  ```
- 上面代码中，在匹配 Coin::Quarter(state) 模式时，我们把它内部存储的值绑定到了 state 变量上，因此 state 变量就是对应的 UsState 枚举类型。
- 例如有一个印了阿拉斯加州标记的 25 分硬币：Coin::Quarter(UsState::Alaska), 它在匹配时，state 变量将被绑定 UsState::Alaska 的枚举值。
- 再来看一个更复杂的例子：
  
  ```
  enum Action {
    Say(String),
    MoveTo(i32, i32),
    ChangeColorRGB(u16, u16, u16),
  }
  
  fn main() {
    let actions = [
        Action::Say("Hello Rust".to_string()),
        Action::MoveTo(1,2),
        Action::ChangeColorRGB(255,255,0),
    ];
    for action in actions {
        match action {
            Action::Say(s) => {
                println!("{}", s);
            },
            Action::MoveTo(x, y) => {
                println!("point from (0, 0) move to ({}, {})", x, y);
            },
            Action::ChangeColorRGB(r, g, _) => {
                println!("change color into '(r:{}, g:{}, b:0)', 'b' has been ignored",
                    r, g,
                );
            }
        }
    }
  }
  ```
- 穷尽匹配
- 在文章的开头，我们简单总结过 match 的匹配必须穷尽所有情况，下面来举例说明，例如：
  
  ```
  enum Direction {
    East,
    West,
    North,
    South,
  }
  
  fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
    };
  }
  ```
- _ 通配符
- 当我们不想在匹配时列出所有值的时候，可以使用 Rust 提供的一个特殊模式，例如，u8 可以拥有 0 到 255 的有效的值，但是我们只关心 1、3、5 和 7 这几个值，不想列出其它的 0、2、4、6、8、9 一直到 255 的值。那么, 我们不必一个一个列出所有值, 因为可以使用特殊的模式 _ 替代：
  
  ```
  let some_u8_value = 0u8;
  match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
  }
  ```
- 通过将 _ 其放置于其他分支后，_ 将会匹配所有遗漏的值。() 表示返回单元类型与所有分支返回值的类型相同，所以当匹配到 _ 后，什么也不会发生。
- 除了_通配符，用一个变量来承载其他情况也是可以的。
  
  ```
  #[derive(Debug)]
  enum Direction {
    East,
    West,
    North,
    South,
  }
  
  fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        other => println!("other direction: {:?}", other),
    };
  }
  ```
- 然而，在某些场景下，我们其实只关心某一个值是否存在，此时 match 就显得过于啰嗦。
- ### 6.1.2.  if let匹配
- 有时会遇到只有一个模式的值需要被处理，其它值直接忽略的场景，如果用 match 来处理就要写成下面这样：
  
  ```
  let v = Some(3u8);
    match v {
        Some(3) => println!("three"),
        _ => (),
    }
  ```
- 我们只想要对 Some(3) 模式进行匹配, 不想处理任何其他 Some<u8> 值或 None 值。但是为了满足 match 表达式（穷尽性）的要求，写代码时必须在处理完这唯一的成员后加上 _ => ()，这样会增加不少无用的代码。
- 俗话说“杀鸡焉用牛刀”，我们完全可以用 if let 的方式来实现：
  
  ```
  if let Some(3) = v {
    println!("three");
  }
  ```
- 这两种匹配对于新手来说，可能有些难以抉择，但是只要记住一点就好：当你只要匹配一个条件，且忽略其他条件时就用 if let ，否则都用 match。
- ### 6.1.3.  matches!宏
- Rust 标准库中提供了一个非常实用的宏：matches!，它可以将一个表达式跟模式进行匹配，然后返回匹配的结果 true or false。
- 例如，有一个动态数组，里面存有以下枚举：
  
  ```
  enum MyEnum {
    Foo,
    Bar
  }
  
  fn main() {
    let v = vec![MyEnum::Foo,MyEnum::Bar,MyEnum::Foo];
  }
  ```
- 现在如果想对 v 进行过滤，只保留类型是 MyEnum::Foo 的元素，你可能想这么写：
  
  ```
  v.iter().filter(|x| x == MyEnum::Foo);
  ```
- 但是，实际上这行代码会报错，因为你无法将 x 直接跟一个枚举成员进行比较。好在，你可以使用 match 来完成，但是会导致代码更为啰嗦，是否有更简洁的方式？答案是使用 matches!：
  
  ```
  v.iter().filter(|x| matches!(x, MyEnum::Foo));
  ```
- 很简单也很简洁，再来看看更多的例子：
  
  ```
  let foo = 'f';
  assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));
  
  let bar = Some(4);
  assert!(matches!(bar, Some(x) if x > 2));
  ```
- ### 6.1.4.  变量遮蔽
- 无论是 match 还是 if let，这里都是一个新的代码块，而且这里的绑定相当于新变量，如果你使用同名变量，会发生变量遮蔽:
  
  ```
  fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}",age);
   if let Some(age) = age {
       println!("匹配出来的age是{}",age);
   }
  
   println!("在匹配后，age是{:?}",age);
  }
  ```
- cargo run 运行后输出如下：
  
  ```
  在匹配前，age是Some(30)
  匹配出来的age是30
  在匹配后，age是Some(30)
  ```
- 可以看出在 if let 中，= 右边 Some(i32) 类型的 age 被左边 i32 类型的新 age 遮蔽了，该遮蔽一直持续到 if let 语句块的结束。因此第三个 println! 输出的 age 依然是 Some(i32) 类型。
- 对于 match 类型也是如此:
  
  ```
  fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}",age);
   match age {
       Some(age) =>  println!("匹配出来的age是{}",age),
       _ => ()
   }
   println!("在匹配后，age是{:?}",age);
  }
  ```
- 需要注意的是，match 中的变量遮蔽其实不是那么的容易看出，因此要小心！其实这里最好不要使用同名，避免难以理解，如下。
  
  ```
  fn main() {
   let age = Some(30);
   println!("在匹配前，age是{:?}", age);
   match age {
       Some(x) =>  println!("匹配出来的age是{}", x),
       _ => ()
   }
   println!("在匹配后，age是{:?}", age);
  }
  ```
- ## 6.2.  解构Option
- 在枚举那章，提到过 Option 枚举，它用来解决 Rust 中变量是否有值的问题，定义如下：
  
  ```
  enum Option<T> {
    Some(T),
    None,
  }
  ```
- 简单解释就是：一个变量要么有值：Some(T), 要么为空：None。
- 那么现在的问题就是该如何去使用这个 Option 枚举类型，根据我们上一节的经验，可以通过 match 来实现。
  
  因为 Option，Some，None 都包含在 prelude 中，因此你可以直接通过名称来使用它们，而无需以 Option::Some 这种形式去使用，总之，千万不要因为调用路径变短了，就忘记 Some 和 None 也是 Option 底下的枚举成员！
- 匹配 Option<T>
- 使用 Option<T>，是为了从 Some 中取出其内部的 T 值以及处理没有值的情况，为了演示这一点，下面一起来编写一个函数，它获取一个 Option<i32>，如果其中含有一个值，将其加一；如果其中没有值，则函数返回 None 值：
  
  ```
  fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
  }
  
  let five = Some(5);
  let six = plus_one(five);
  let none = plus_one(None);
  ```
- plus_one 接受一个 Option<i32> 类型的参数，同时返回一个 Option<i32> 类型的值(这种形式的函数在标准库内随处所见)，在该函数的内部处理中，如果传入的是一个 None ，则返回一个 None 且不做任何处理；如果传入的是一个 Some(i32)，则通过模式绑定，把其中的值绑定到变量 i 上，然后返回 i+1 的值，同时用 Some 进行包裹。
- 为了进一步说明，假设 plus_one 函数接受的参数值 x 是 Some(5)，来看看具体的分支匹配情况：
- 传入参数 Some(5)
  
  ```
  None => None
  ```
- 首先是匹配 None 分支，因为值 Some(5) 并不匹配模式 None，所以继续匹配下一个分支。
- Some(i) => Some(i + 1),
- Some(5) 与 Some(i) 匹配吗？当然匹配！它们是相同的成员。i 绑定了 Some 中包含的值，因此 i 的值是 5。接着匹配分支的代码被执行，最后将 i 的值加一并返回一个含有值 6 的新 Some。
- 传入参数 None
- 接着考虑下 plus_one 的第二个调用，这次传入的 x 是 None， 我们进入 match 并与第一个分支相比较。
- None => None,
- 匹配上了！接着程序继续执行该分支后的代码：返回表达式 None 的值，也就是返回一个 None，因为第一个分支就匹配到了，其他的分支将不再比较。
- ## 6.3.  模式使用场景
- 模式是 Rust 中的特殊语法，它用来匹配类型中的结构和数据，它往往和 match 表达式联用，以实现强大的模式匹配能力。模式一般由以下内容组合而成：
	- 字面值
	- 解构的数组、枚举、结构体或者元组
	- 变量
	- 通配符
	- 占位符
- match 分支
  
  ```
  match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
  }
  ```
- 如上所示，match 的每个分支就是一个模式，因为 match 匹配是穷尽式的，因此我们往往需要一个特殊的模式 _，来匹配剩余的所有情况：
  
  ```
  match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    _ => EXPRESSION,
  }
  ```
- if let 分支
- if let 往往用于匹配一个模式，而忽略剩下的所有模式的场景：
  
  ```
  if let PATTERN = SOME_VALUE {
  
  }
  ```
- while let 条件循环
- 一个与 if let 类似的结构是 while let 条件循环，它允许只要模式匹配就一直进行 while 循环。下面展示了一个使用 while let 的例子：
  
  ```
  // Vec是动态数组
  let mut stack = Vec::new();
  
  // 向数组尾部插入元素
  stack.push(1);
  stack.push(2);
  stack.push(3);
  
  // stack.pop从数组尾部弹出元素
  while let Some(top) = stack.pop() {
    println!("{}", top);
  }
  ```
- 这个例子会打印出 3、2 接着是 1。pop 方法取出动态数组的最后一个元素并返回 Some(value)，如果动态数组是空的，将返回 None，对于 while 来说，只要 pop 返回 Some 就会一直不停的循环。一旦其返回 None，while 循环停止。我们可以使用 while let 来弹出栈中的每一个元素。
- 你也可以用 loop + if let 或者 match 来实现这个功能，但是会更加啰嗦。
- for 循环
  
  ```
  let v = vec!['a', 'b', 'c'];
  
  for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
  }
  ```
- 这里使用 enumerate 方法产生一个迭代器，该迭代器每次迭代会返回一个 (索引，值) 形式的元组，然后用 (index,value) 来匹配。
- let 语句
  
  ```
  let PATTERN = EXPRESSION;
  ```
- 是的， 该语句我们已经用了无数次了，它也是一种模式匹配：
  
  ```
  let x = 5;
  ```
- 这其中，x 也是一种模式绑定，代表将匹配的值绑定到变量 x 上。因此，在 Rust 中,变量名也是一种模式，只不过它比较朴素很不起眼罢了。
  
  ```
  let (x, y, z) = (1, 2, 3);
  ```
- 上面将一个元组与模式进行匹配(模式和值的类型必需相同！)，然后把 1, 2, 3 分别绑定到 x, y, z 上。
- 模式匹配要求两边的类型必须相同，否则就会导致下面的报错：
  
  ```
  let (x, y) = (1, 2, 3);
  error[E0308]: mismatched types
  --> src/main.rs:4:5
  |
  4 | let (x, y) = (1, 2, 3);
  |     ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |     |
  |     expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`
  For more information about this error, try `rustc --explain E0308`.
  error: could not compile `playground` due to previous error
  ```
- 对于元组来说，元素个数也是类型的一部分！
- 函数参数
- 函数参数也是模式：
  
  ```
  fn foo(x: i32) {
    // 代码
  }
  ```
- 其中 x 就是一个模式，你还可以在参数中匹配元组：
  
  ```
  fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
  }
  
  fn main() {
    let point = (3, 5);
    print_coordinates(&point);
  }
  ```
- &(3, 5) 会匹配模式 &(x, y)，因此 x 得到了 3，y 得到了 5。
- let 和 if let
- 对于以下代码，编译器会报错：
  
  ```
  let Some(x) = some_option_value;
  ```
- 因为右边的值可能不为 Some，而是 None，这种时候就不能进行匹配，也就是上面的代码遗漏了 None 的匹配。
- 类似 let , for和match 都必须要求完全覆盖匹配，才能通过编译( 不可驳模式匹配 )。
- 但是对于 if let，就可以这样使用：
  
  ```
  if let Some(x) = some_option_value {
    println!("{}", x);
  }
  ```
- 因为 if let 允许匹配一种模式，而忽略其余的模式( 可驳模式匹配 )。
- ## 6.4.  全模式列表
- 在本书中我们已领略过许多不同类型模式的例子，本节的目标就是把这些模式语法都罗列出来，方便大家检索查阅（模式匹配在我们的开发中会经常用到）。
- ### 6.4.1.  匹配字面值
  
  ```
  let x = 1;
  
  match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
  }
  ```
- 这段代码会打印 one 因为 x 的值是 1，如果希望代码获得特定的具体值，那么这种语法很有用。
- ### 6.4.2.  匹配命名变量
- 在 match 中，我们有讲过变量遮蔽的问题，这个在匹配命名变量时会遇到：
  
  ```
  fn main() {
    let x = Some(5);
    let y = 10;
  
    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }
  
    println!("at the end: x = {:?}, y = {:?}", x, y);
  }
  ```
- 让我们看看当 match 语句运行的时候发生了什么。第一个匹配分支的模式并不匹配 x 中定义的值，所以代码继续执行。
- 第二个匹配分支中的模式引入了一个新变量 y，它会匹配任何 Some 中的值。因为这里的 y 在 match 表达式的作用域中，而不是之前 main 作用域中，所以这是一个新变量，不是开头声明为值 10 的那个 y。这个新的 y 绑定会匹配任何 Some 中的值，在这里是 x 中的值。因此这个 y 绑定了 x 中 Some 内部的值。这个值是 5，所以这个分支的表达式将会执行并打印出 Matched，y = 5。
- 如果 x 的值是 None 而不是 Some(5)，头两个分支的模式不会匹配，所以会匹配模式 _。这个分支的模式中没有引入变量 x，所以此时表达式中的 x 会是外部没有被遮蔽的 x，也就是 None。
- 一旦 match 表达式执行完毕，其作用域也就结束了，同理内部 y 的作用域也结束了。最后的 println! 会打印 at the end: x = Some(5), y = 10。
- 如果你不想引入变量遮蔽，可以使用另一个变量名而非 y，或者使用匹配守卫(match guard)的方式，稍后在匹配守卫提供的额外条件中会讲解。
- ### 6.4.3.  单分支多模式
- 在 match 表达式中，可以使用 | 语法匹配多个模式，它代表 或的意思。例如，如下代码将 x 的值与匹配分支相比较，第一个分支有 或 选项，意味着如果 x 的值匹配此分支的任何一个模式，它就会运行：
  
  ```
  let x = 1;
  
  match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
  }
  ```
- 上面的代码会打印 one or two。
- ### 6.4.4.  通过序列 ..= 匹配值的范围
- 在数值类型中我们有讲到一个序列语法，该语法不仅可以用于循环中，还能用于匹配模式。
- ..= 语法允许你匹配一个闭区间序列内的值。在如下代码中，当模式匹配任何在此序列内的值时，该分支会执行：
  
  ```
  let x = 5;
  
  match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
  }
  ```
- 如果 x 是 1、2、3、4 或 5，第一个分支就会匹配。这相比使用 | 运算符表达相同的意思更为方便；相比 1..=5，使用 | 则不得不指定 1 | 2 | 3 | 4 | 5 这五个值，而使用 ..= 指定序列就简短的多，比如希望匹配比如从 1 到 1000 的数字的时候！
- 序列只允许用于数字或字符类型，原因是：它们可以连续，同时编译器在编译期可以检查该序列是否为空，字符和数字值是 Rust 中仅有的可以用于判断是否为空的类型。
- 如下是一个使用字符类型序列的例子：
  
  ```
  let x = 'c';
  
  match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
  }
  ```
- Rust 知道 'c' 位于第一个模式的序列内，所以会打印出 early ASCII letter。
- ### 6.4.5.  解构并分解值
- 解构结构体
- 下面代码展示了如何用 let 解构一个带有两个字段 x 和 y 的结构体 Point：
  
  ```
  struct Point {
    x: i32,
    y: i32,
  }
  
  fn main() {
    let p = Point { x: 0, y: 7 };
  
    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
  }
  ```
- 这段代码创建了变量 a 和 b 来匹配结构体 p 中的 x 和 y 字段，这个例子展示了模式中的变量名不必与结构体中的字段名一致。不过通常希望变量名与字段名一致以便于理解变量来自于哪些字段。
- 因为变量名匹配字段名是常见的，同时因为 let Point { x: x, y: y } = p; 中 x 和 y 重复了，所以对于匹配结构体字段的模式存在简写：只需列出结构体字段的名称，则模式创建的变量会有相同的名称。下例与上例有着相同行为的代码，不过 let 模式创建的变量为 x 和 y 而不是 a 和 b：
  
  ```
  struct Point {
    x: i32,
    y: i32,
  }
  
  fn main() {
    let p = Point { x: 0, y: 7 };
  
    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
  }
  ```
- 这段代码创建了变量 x 和 y，与结构体 p 中的 x 和 y 字段相匹配。其结果是变量 x 和 y 包含结构体 p 中的值。
- 也可以使用字面值作为结构体模式的一部分进行解构，而不是为所有的字段创建变量。这允许我们测试一些字段为特定值的同时创建其他字段的变量。
- 下文展示了固定某个字段的匹配方式：
  
  ```
  fn main() {
    let p = Point { x: 0, y: 7 };
  
    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
  }
  ```
- 首先是 match 第一个分支，指定匹配 y 为 0 的 Point； 然后第二个分支在第一个分支之后，匹配 y 不为 0，x 为 0 的 Point; 最后一个分支匹配 x 不为 0，y 也不为 0 的 Point。
- 在这个例子中，值 p 因为其 x 包含 0 而匹配第二个分支，因此会打印出 On the y axis at 7。
- 解构枚举
- 下面代码以 Message 枚举为例，编写一个 match 使用模式解构每一个内部值：
  
  ```
  enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
  }
  
  fn main() {
    let msg = Message::ChangeColor(0, 160, 255);
  
    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
  }
  ```
- 这里老生常谈一句话，模式匹配一样要类型相同，因此匹配 Message::Move{1,2} 这样的枚举值，就必须要用 Message::Move{x,y} 这样的同类型模式才行。
- 这段代码会打印出 Change the color to red 0, green 160, and blue 255。尝试改变 msg 的值来观察其他分支代码的运行。
- 对于像 Message::Quit 这样没有任何数据的枚举成员，不能进一步解构其值。只能匹配其字面值 Message::Quit，因此模式中没有任何变量。
- 对于另外两个枚举成员，就用相同类型的模式去匹配出对应的值即可。
- 解构嵌套的结构体和枚举
- 目前为止，所有的例子都只匹配了深度为一级的结构体或枚举。 match 也可以匹配嵌套的项！
- 例如使用下面的代码来同时支持 RGB 和 HSV 色彩模式：
  
  ```
  enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
  }
  
  enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
  }
  
  fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
  
    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }
  }
  ```
- match 第一个分支的模式匹配一个 Message::ChangeColor 枚举成员，该枚举成员又包含了一个 Color::Rgb 的枚举成员，最终绑定了 3 个内部的 i32 值。
- 解构结构体和元组
- 我们甚至可以用复杂的方式来混合、匹配和嵌套解构模式。如下是一个复杂结构体的例子，其中结构体和元组嵌套在元组中，并将所有的原始类型解构出来：
  
  ```
  struct Point {
     x: i32,
     y: i32,
  }
  
  let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
  ```
- 这种将复杂类型分解匹配的方式，可以让我们单独得到感兴趣的某个值。
- 解构数组
- 对于数组，我们可以用类似元组的方式解构，分为两种情况：
- 定长数组
  
  ```
  let arr: [u16; 2] = [114, 514];
  let [x, y] = arr;
  
  assert_eq!(x, 114);
  assert_eq!(y, 514);
  ```
- 不定长数组
  
  ```
  let arr: &[u16] = &[114, 514];
  
  if let [x, ..] = arr {
    assert_eq!(x, &114);
  }
  
  if let &[.., y] = arr {
    assert_eq!(y, 514);
  }
  
  let arr: &[u16] = &[];
  
  assert!(matches!(arr, [..]));
  assert!(!matches!(arr, [x, ..]));
  ```
- ## 6.5.  忽略模式中的值
- 有时忽略模式中的一些值是很有用的，比如在 match 中的最后一个分支使用 _ 模式匹配所有剩余的值。 你也可以在另一个模式中使用 _ 模式，使用一个以下划线开始的名称，或者使用 .. 忽略所剩部分的值。
- 使用 _ 忽略整个值
- 虽然 _ 模式作为 match 表达式最后的分支特别有用，但是它的作用还不限于此。例如可以将其用于函数参数中：
  
  ```
  fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
  }
  
  fn main() {
    foo(3, 4);
  }
  ```
- 这段代码会完全忽略作为第一个参数传递的值 3，并会打印出 This code only uses the y parameter: 4。
- 大部分情况当你不再需要特定函数参数时，最好修改签名不再包含无用的参数。在一些情况下忽略函数参数会变得特别有用，比如实现特征时，当你需要特定类型签名但是函数实现并不需要某个参数时。此时编译器就不会警告说存在未使用的函数参数，就跟使用命名参数一样。
- 使用嵌套的 _ 忽略部分值
- 可以在一个模式内部使用 _ 忽略部分值：
  
  ```
  let mut setting_value = Some(5);
  let new_setting_value = Some(10);
  
  match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
  }
  
  println!("setting is {:?}", setting_value);
  ```
- 这段代码会打印出 Can't overwrite an existing customized value 接着是 setting is Some(5)。
- 第一个匹配分支，我们不关心里面的值，只关心元组中两个元素的类型，因此对于 Some 中的值，直接进行忽略。 剩下的形如 (Some(_),None)，(None, Some(_)), (None,None) 形式，都由第二个分支 _ 进行分配。
- 还可以在一个模式中的多处使用下划线来忽略特定值，如下所示，这里忽略了一个五元元组中的第二和第四个值：
  
  ```
  let numbers = (2, 4, 8, 16, 32);
  
  match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
  }
  ```
- 老生常谈：模式匹配一定要类型相同，因此匹配 numbers 元组的模式，也必须有五个值（元组中元素的数量也属于元组类型的一部分）。
- 这会打印出 Some numbers: 2, 8, 32, 值 4 和 16 会被忽略。
- 使用下划线开头忽略未使用的变量
- 如果你创建了一个变量却不在任何地方使用它，Rust 通常会给你一个警告，因为这可能会是个 BUG。但是有时创建一个不会被使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头：
  
  ```
  fn main() {
    let _x = 5;
    let y = 10;
  }
  ```
- 这里得到了警告说未使用变量 y，至于 x 则没有警告。
- 注意, 只使用 _ 和使用以下划线开头的名称有些微妙的不同：比如 _x 仍会将值绑定到变量，而 _ 则完全不会绑定。
  
  ```
  let s = Some(String::from("Hello!"));
  
  if let Some(_s) = s {
    println!("found a string");
  }
  
  println!("{:?}", s);
  ```
- s 是一个拥有所有权的动态字符串，在上面代码中，我们会得到一个错误，因为 s 的值会被转移给 _s，在 println! 中再次使用 s 会报错：
  
  ```
  error[E0382]: borrow of partially moved value: `s`
  --> src/main.rs:8:22
  |
  4 |     if let Some(_s) = s {
  |                 -- value partially moved here
  ...
  8 |     println!("{:?}", s);
  |                      ^ value borrowed here after partial move
  ```
- 只使用下划线本身，则并不会绑定值，因为 s 没有被移动进 _：
  
  ```
  let s = Some(String::from("Hello!"));
  
  if let Some(_) = s {
    println!("found a string");
  }
  
  println!("{:?}", s);
  ```
- 用 .. 忽略剩余值
- 对于有多个部分的值，可以使用 .. 语法来只使用部分值而忽略其它值，这样也不用再为每一个被忽略的值都单独列出下划线。.. 模式会忽略模式中剩余的任何没有显式匹配的值部分。
  
  ```
  struct Point {
    x: i32,
    y: i32,
    z: i32,
  }
  
  let origin = Point { x: 0, y: 0, z: 0 };
  
  match origin {
    Point { x, .. } => println!("x is {}", x),
  }
  ```
- 这里列出了 x 值，接着使用了 .. 模式来忽略其它字段，这样的写法要比一一列出其它字段，然后用 _ 忽略简洁的多。
- 还可以用 .. 来忽略元组中间的某些值：
  
  ```
  fn main() {
    let numbers = (2, 4, 8, 16, 32);
  
    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
  }
  ```
- 这里用 first 和 last 来匹配第一个和最后一个值。.. 将匹配并忽略中间的所有值。
- 然而使用 .. 必须是无歧义的。如果期望匹配和忽略的值是不明确的，Rust 会报错。下面代码展示了一个带有歧义的 .. 例子，因此不能编译：
  
  ```
  fn main() {
    let numbers = (2, 4, 8, 16, 32);
  
    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
  }
  ```
- 如果编译上面的例子，会得到下面的错误：
  
  ```
  error: `..` can only be used once per tuple pattern // 每个元组模式只能使用一个 `..`
  --> src/main.rs:5:22
  |
  5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here // 上一次使用在这里
  
  error: could not compile `world_hello` due to previous error              ^^
  ```
- Rust 无法判断，second 应该匹配 numbers 中的第几个元素，因此这里使用两个 .. 模式，是有很大歧义的！
- ### 6.5.1.  匹配守卫提供的额外条件
- 匹配守卫（match guard）是一个位于 match 分支模式之后的额外 if 条件，它能为分支模式提供更进一步的匹配条件。
- 这个条件可以使用模式中创建的变量：
  
  ```
  let num = Some(4);
  
  match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
  }
  ```
- 这个例子会打印出 less than five: 4。当 num 与模式中第一个分支匹配时，Some(4) 可以与 Some(x) 匹配，接着匹配守卫检查 x 值是否小于 5，因为 4 小于 5，所以第一个分支被选择。
- 相反如果 num 为 Some(10)，因为 10 不小于 5 ，所以第一个分支的匹配守卫为假。接着 Rust 会前往第二个分支，因为这里没有匹配守卫所以会匹配任何 Some 成员。
- 模式中无法提供类如 if x < 5 的表达能力，我们可以通过匹配守卫的方式来实现。
- 在之前，我们提到可以使用匹配守卫来解决模式中变量覆盖的问题，那里 match 表达式的模式中新建了一个变量而不是使用 match 之外的同名变量。内部变量覆盖了外部变量，意味着此时不能够使用外部变量的值，下面代码展示了如何使用匹配守卫修复这个问题。
  
  ```
  fn main() {
    let x = Some(5);
    let y = 10;
  
    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),
        _ => println!("Default case, x = {:?}", x),
    }
  
    println!("at the end: x = {:?}, y = {}", x, y);
  }
  ```
- 现在这会打印出 Default case, x = Some(5)。现在第二个匹配分支中的模式不会引入一个覆盖外部 y 的新变量 y，这意味着可以在匹配守卫中使用外部的 y。相比指定会覆盖外部 y 的模式 Some(y)，这里指定为 Some(n)。此新建的变量 n 并没有覆盖任何值，因为 match 外部没有变量 n。
- 匹配守卫 if n == y 并不是一个模式所以没有引入新变量。这个 y 正是 外部的 y 而不是新的覆盖变量 y，这样就可以通过比较 n 和 y 来表达寻找一个与外部 y 相同的值的概念了。
- 也可以在匹配守卫中使用 或 运算符 | 来指定多个模式，同时匹配守卫的条件会作用于所有的模式。下面代码展示了匹配守卫与 | 的优先级。这个例子中看起来好像 if y 只作用于 6，但实际上匹配守卫 if y 作用于 4、5 和 6 ，在满足 x 属于 4 | 5 | 6 后才会判断 y 是否为 true：
  
  ```
  let x = 4;
  let y = false;
  
  match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
  }
  ```
- 这个匹配条件表明此分支只匹配 x 值为 4、5 或 6 同时 y 为 true 的情况。
- 虽然在第一个分支中，x 匹配了模式 4 ，但是对于匹配守卫 if y 来说，因为 y 是 false，因此该守卫条件的值永远是 false，也意味着第一个分支永远无法被匹配。
- 下面的文字图解释了匹配守卫作用于多个模式时的优先级规则，第一张是正确的：
  
  ```
  (4 | 5 | 6) if y => ...
  ```
- 而第二张图是错误的
  
  ```
  4 | 5 | (6 if y) => ...
  ```
- 可以通过运行代码时的情况看出这一点：如果匹配守卫只作用于由 | 运算符指定的值列表的最后一个值，这个分支就会匹配且程序会打印出 yes。
- ### 6.5.2.  @绑定
- @（读作 at）运算符允许为一个字段绑定另外一个变量。下面例子中，我们希望测试 Message::Hello 的 id 字段是否位于 3..=7 范围内，同时也希望能将其值绑定到 id_variable 变量中以便此分支中相关的代码可以使用它。我们可以将 id_variable 命名为 id，与字段同名，不过出于示例的目的这里选择了不同的名称。
  
  ```
  enum Message {
    Hello { id: i32 },
  }
  
  let msg = Message::Hello { id: 5 };
  
  match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
  }
  ```
- 上例会打印出 Found an id in range: 5。通过在 3..=7 之前指定 id_variable @，我们捕获了任何匹配此范围的值并同时将该值绑定到变量 id_variable 上。
- 第二个分支只在模式中指定了一个范围，id 字段的值可以是 10、11 或 12，不过这个模式的代码并不知情也不能使用 id 字段中的值，因为没有将 id 值保存进一个变量。
- 最后一个分支指定了一个没有范围的变量，此时确实拥有可以用于分支代码的变量 id，因为这里使用了结构体字段简写语法。不过此分支中没有像头两个分支那样对 id 字段的值进行测试：任何值都会匹配此分支。
- 当你既想要限定分支范围，又想要使用分支的变量时，就可以用 @ 来绑定到一个新的变量上，实现想要的功能。
- @前绑定后解构(Rust 1.56 新增)
- 使用 @ 还可以在绑定新变量的同时，对目标进行解构：
  
  ```
  #[derive(Debug)]
  struct Point {
    x: i32,
    y: i32,
  }
  
  fn main() {
    // 绑定新变量 `p`，同时对 `Point` 进行解构
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);
  
    let point = Point {x: 10, y: 5};
    if let p @ Point {x: 10, y} = point {
        println!("x is 10 and y is {} in {:?}", y, p);
    } else {
        println!("x was not 10 :(");
    }
  }
  ```
- @新特性(Rust 1.53 新增)
- 考虑下面一段代码:
  
  ```
  fn main() {
    match 1 {
        num @ 1 | 2 => {
            println!("{}", num);
        }
        _ => {}
    }
  }
  ```
- 编译不通过，是因为 num 没有绑定到所有的模式上，只绑定了模式 1，你可能会试图通过这个方式来解决：
  
  ```
  num @ (1 | 2)
  ```
- 但是，如果你用的是 Rust 1.53 之前的版本，那这种写法会报错，因为编译器不支持。
- 至此，模式匹配的内容已经全部完结，复杂但是详尽，想要一次性全部记住属实不易，因此读者可以先留一个印象，等未来需要时，再来翻阅寻找具体的模式实现方式。
- ## 6.6.  方法Method
- 从面向对象语言过来的同学对于方法肯定不陌生，class 里面就充斥着方法的概念。在 Rust 中，方法的概念也大差不差，往往和对象成对出现：
  
  ```
  object.method()
  ```
- ### 6.6.1.  定义方法
- Rust 使用 impl 来定义方法，例如以下代码：
  
  ```
  struct Circle {
    x: f64,
    y: f64,
    radius: f64,
  }
  
  impl Circle {
    // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法往往用于初始化当前结构体的实例
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }
  
    // Circle的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
  }
  ```
- 我们这里先不详细展开讲解，只是先建立对方法定义的大致印象。下面的图片将 Rust 方法定义与其它语言的方法定义做了对比：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1724110736216-f01c99a4-dc52-4e0c-8764-3f361ea6ba8f.png)
- 可以看出，其它语言中所有定义都在 class 中，但是 Rust 的对象定义和方法定义是分离的，这种数据和使用分离的方式，会给予使用者极高的灵活度。
- 再来看一个例子：
  
  ```
  #[derive(Debug)]
  struct Rectangle {
    width: u32,
    height: u32,
  }
  
  impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
  }
  
  fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
  
    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
  }
  ```
- 该例子定义了一个 Rectangle 结构体，并且在其上定义了一个 area 方法，用于计算该矩形的面积。
- impl Rectangle {} 表示为 Rectangle 实现方法(impl 是实现 implementation 的缩写)，这样的写法表明 impl 语句块中的一切都是跟 Rectangle 相关联的。
- self、&self 和 &mut self
- 接下来的内容非常重要，请大家仔细看。在 area 的签名中，我们使用 &self 替代 rectangle: &Rectangle，&self 其实是 self: &Self 的简写（注意大小写）。在一个 impl 块内，Self 指代被实现方法的结构体类型，self 指代此类型的实例，换句话说，self 指代的是 Rectangle 结构体实例，这样的写法会让我们的代码简洁很多，而且非常便于理解：我们为哪个结构体实现方法，那么 self 就是指代哪个结构体的实例。
- 需要注意的是，self 依然有所有权的概念：
- self 表示 Rectangle 的所有权转移到该方法中，这种形式用的较少
- &self 表示该方法对 Rectangle 的不可变借用
- &mut self 表示可变借用
- 总之，self 的使用就跟函数参数一样，要严格遵守 Rust 的所有权规则。
- 回到上面的例子中，选择 &self 的理由跟在函数中使用 &Rectangle 是相同的：我们并不想获取所有权，也无需去改变它，只是希望能够读取结构体中的数据。如果想要在方法中去改变当前的结构体，需要将第一个参数改为 &mut self。仅仅通过使用 self 作为第一个参数来使方法获取实例的所有权是很少见的，这种使用方式往往用于把当前的对象转成另外一个对象时使用，转换完后，就不再关注之前的对象，且可以防止对之前对象的误调用。
- 简单总结下，使用方法代替函数有以下好处：
- 不用在函数签名中重复书写 self 对应的类型
- 代码的组织性和内聚性更强，对于代码维护和阅读来说，好处巨大
- 方法名跟结构体字段名相同
- 在 Rust 中，允许方法名跟结构体的字段名相同：
  
  ```
  impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
  }
  
  fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
  
    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
  }
  ```
- 当我们使用 rect1.width() 时，Rust 知道我们调用的是它的方法，如果使用 rect1.width，则是访问它的字段。
- 一般来说，方法跟字段同名，往往适用于实现 getter 访问器，例如:
  
  ```
  pub struct Rectangle {
    width: u32,
    height: u32,
  }
  
  impl Rectangle {
    pub fn new(width: u32, height: u32) -> Self {
        Rectangle { width, height }
    }
    pub fn width(&self) -> u32 {
        return self.width;
    }
  }
  
  fn main() {
    let rect1 = Rectangle::new(30, 50);
  
    println!("{}", rect1.width());
  }
  ```
- 用这种方式，我们可以把 Rectangle 的字段设置为私有属性，只需把它的 new 和 width 方法设置为公开可见，那么用户就可以创建一个矩形，同时通过访问器 rect1.width() 方法来获取矩形的宽度，因为 width 字段是私有的，当用户访问 rect1.width 字段时，就会报错。注意在此例中，Self 指代的就是被实现方法的结构体 Rectangle。
  
  -> 运算符到哪去了？
  
  在 C/C++ 语言中，有两个不同的运算符来调用方法：. 直接在对象上调用方法，而 -> 在一个对象的指针上调用方法，这时需要先解引用指针。换句话说，如果 object 是一个指针，那么 object->something() 和 (*object).something() 是一样的。
  
  Rust 并没有一个与 -> 等效的运算符；相反，Rust 有一个叫 自动引用和解引用的功能。方法调用是 Rust 中少数几个拥有这种行为的地方。
  
  他是这样工作的：当使用 object.something() 调用方法时，Rust 会自动为 object 添加 &、&mut 或 * 以便使 object 与方法签名匹配。也就是说，这些代码是等价的：
  
  p1.distance(&p2);
  
  (&p1).distance(&p2);
  
  第一行看起来简洁的多。这种自动引用的行为之所以有效，是因为方法有一个明确的接收者———— self 的类型。在给出接收者和方法名的前提下，Rust 可以明确地计算出方法是仅仅读取（&self），做出修改（&mut self）或者是获取所有权（self）。事实上，Rust 对方法接收者的隐式借用让所有权在实践中更友好。
- ### 6.6.2.  多参方法
- 方法和函数一样，可以使用多个参数：
  
  ```
  impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
  
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
  }
  
  fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };
  
    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
  }
  ```
- ### 6.6.3.  关联函数
- 现在大家可以思考一个问题，如何为一个结构体定义一个构造器方法？也就是接受几个参数，然后构造并返回该结构体的实例。其实答案在开头的代码片段中就给出了，很简单，参数中不包含 self 即可。
- 这种定义在 impl 中且没有 self 的函数被称之为关联函数： 因为它没有 self，不能用 f.read() 的形式调用，因此它是一个函数而不是方法，它又在 impl 中，与结构体紧密关联，因此称为关联函数。
- 在之前的代码中，我们已经多次使用过关联函数，例如 String::from，用于创建一个动态字符串。
  
  ```
  impl Rectangle {
    fn new(w: u32, h: u32) -> Rectangle {
        Rectangle { width: w, height: h }
    }
  }
  ```
- Rust 中有一个约定俗成的规则，使用 new 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 new 作为关键字。
- 因为是函数，所以不能用 . 的方式来调用，我们需要用 :: 来调用，例如 let sq = Rectangle::new(3, 3);。这个方法位于结构体的命名空间中：:: 语法用于关联函数和模块创建的命名空间。
- ### 6.6.4.  多个 impl 定义
- Rust 允许我们为一个结构体定义多个 impl 块，目的是提供更多的灵活性和代码组织性，例如当方法多了后，可以把相关的方法组织在同一个 impl 块中，那么就可以形成多个 impl 块，各自完成一块儿目标：
  
  ```
  impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
  }
  
  impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
  }
  ```
- 当然，就这个例子而言，我们没必要使用两个 impl 块，这里只是为了演示方便。
- ### 6.6.5.  为枚举实现方法
- 枚举类型之所以强大，不仅仅在于它好用、可以同一化类型，还在于，我们可以像结构体一样，为枚举实现方法：
  
  ```
  #![allow(unused)]
  enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
  }
  
  impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
  }
  
  fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
  }
  ```
- 除了结构体和枚举，我们还能为特征(trait)实现方法，这将在下一章进行讲解，在此之前，先来看看泛型。
- ## 6.7.  泛型和特征
- ### 6.7.1.  泛型详解
- 上面代码的 T 就是泛型参数，实际上在 Rust 中，泛型参数的名称你可以任意起，但是出于惯例，我们都用 T ( T 是 type 的首字母)来作为首选，这个名称越短越好，除非需要表达含义，否则一个字母是最完美的。
- 使用泛型参数，有一个先决条件，必需在使用前对其进行声明：
  
  ```
  fn largest<T>(list: &[T]) -> T {
  ```
- 该泛型函数的作用是从列表中找出最大的值，其中列表中的元素类型为 T。首先 largest<T> 对泛型参数 T 进行了声明，然后才在函数参数中进行使用该泛型参数 list: &[T] （还记得 &[T] 类型吧？这是数组切片）。
- 总之，我们可以这样理解这个函数定义：函数 largest 有泛型类型 T，它有个参数 list，其类型是元素为 T 的数组切片，最后，该函数返回值的类型也是 T。
- 下面是一个错误的泛型函数的实现：
  
  ```
  fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];
  
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
  
    largest
  }
  
  fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
  
    let result = largest(&number_list);
    println!("The largest number is {}", result);
  
    let char_list = vec!['y', 'm', 'a', 'q'];
  
    let result = largest(&char_list);
    println!("The largest char is {}", result);
  }
  ```
- 运行后报错：
  
  ```
  error[E0369]: binary operation `>` cannot be applied to type `T` // `>`操作符不能用于类型`T`
  --> src/main.rs:5:17
  |
  5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
  help: consider restricting type parameter `T` // 考虑对T进行类型上的限制 :
  |
  1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++
  ```
- 因为 T 可以是任何类型，但不是所有的类型都能进行比较，因此上面的错误中，编译器建议我们给 T 添加一个类型限制：使用 std::cmp::PartialOrd 特征（Trait）对 T 进行限制，特征在下一节会详细介绍，现在你只要理解，该特征的目的就是让类型实现可比较的功能。
- 还记得我们一开始的 add 泛型函数吗？如果你运行它，会得到以下的报错：
  
  ```
  error[E0369]: cannot add `T` to `T` // 无法将 `T` 类型跟 `T` 类型进行相加
  --> src/main.rs:2:7
  |
  2 |     a + b
  |     - ^ - T
  |     |
  |     T
  |
  help: consider restricting type parameter `T`
  |
  1 | fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
  |         +++++++++++++++++++++++++++
  ```
- 同样的，不是所有 T 类型都能进行相加操作，因此我们需要用 std::ops::Add<Output = T> 对 T 进行限制：
  
  ```
  fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
  }
  ```
- 进行如上修改后，就可以正常运行。
- ### 6.7.2.  结构体中使用泛型
- 结构体中的字段类型也可以用泛型来定义，下面代码定义了一个坐标点 Point，它可以存放任何类型的坐标值：
  
  ```
  struct Point<T> {
    x: T,
    y: T,
  }
  
  fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
  }
  ```
- 这里有两点需要特别的注意：
- 提前声明，跟泛型函数定义类似，首先我们在使用泛型参数之前必需要进行声明 Point<T>，接着就可以在结构体的字段类型中使用 T 来替代具体的类型
- x 和 y 是相同的类型
- 第二点非常重要，如果使用不同的类型，那么它会导致下面代码的报错：
  
  ```
  struct Point<T> {
    x: T,
    y: T,
  }
  
  fn main() {
    let p = Point{x: 1, y :1.1};
  }
  ```
- 错误如下：
  
  ```
  error[E0308]: mismatched types //类型不匹配
  --> src/main.rs:7:28
  |
  7 |     let p = Point{x: 1, y :1.1};
  |                            ^^^ expected integer, found floating-point number //期望y是整数，但是却是浮点数
  ```
- 当把 1 赋值给 x 时，变量 p 的 T 类型就被确定为整数类型，因此 y 也必须是整数类型，但是我们却给它赋予了浮点数，因此导致报错。
- 如果想让 x 和 y 既能类型相同，又能类型不同，就需要使用不同的泛型参数：
  
  ```
  struct Point<T,U> {
    x: T,
    y: U,
  }
  fn main() {
    let p = Point{x: 1, y :1.1};
  }
  ```
- 切记，所有的泛型参数都要提前声明：Point<T,U> ! 但是如果你的结构体变成这鬼样：struct Woo<T,U,V,W,X>，那么你需要考虑拆分这个结构体，减少泛型参数的个数和代码复杂度。
- ### 6.7.3.  枚举中使用泛型
- 提到枚举类型，Option 永远是第一个应该被想起来的，在之前的章节中，它也多次出现：
  
  ```
  enum Option<T> {
    Some(T),
    None,
  }
  ```
- Option<T> 是一个拥有泛型 T 的枚举类型，它第一个成员是 Some(T)，存放了一个类型为 T 的值。得益于泛型的引入，我们可以在任何一个需要返回值的函数中，去使用 Option<T> 枚举类型来做为返回值，用于返回一个任意类型的值 Some(T)，或者没有值 None。
- 对于枚举而言，卧龙凤雏永远是绕不过去的存在：如果是 Option 是卧龙，那么 Result 就一定是凤雏，得两者可得天下：
  
  ```
  enum Result<T, E> {
    Ok(T),
    Err(E),
  }
  ```
- 这个枚举和 Option 一样，主要用于函数返回值，与 Option 用于值的存在与否不同，Result 关注的主要是值的正确性。
- 如果函数正常运行，则最后返回一个 Ok(T)，T 是函数具体的返回值类型，如果函数异常运行，则返回一个 Err(E)，E 是错误类型。例如打开一个文件：如果成功打开文件，则返回 Ok(std::fs::File)，因此 T 对应的是 std::fs::File 类型；而当打开文件时出现问题时，返回 Err(std::io::Error)，E 对应的就是 std::io::Error 类型。
- ### 6.7.4.  方法中使用泛型
- 上一章中，我们讲到什么是方法以及如何在结构体和枚举上定义方法。方法上也可以使用泛型：
  
  ```
  struct Point<T> {
    x: T,
    y: T,
  }
  
  impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
  }
  
  fn main() {
    let p = Point { x: 5, y: 10 };
  
    println!("p.x = {}", p.x());
  }
  ```
- 使用泛型参数前，依然需要提前声明：impl<T>，只有提前声明了，我们才能在Point<T>中使用它，这样 Rust 就知道 Point 的尖括号中的类型是泛型而不是具体类型。需要注意的是，这里的 Point<T> 不再是泛型声明，而是一个完整的结构体类型，因为我们定义的结构体就是 Point<T> 而不再是 Point。
- 除了结构体中的泛型参数，我们还能在该结构体的方法中定义额外的泛型参数，就跟泛型函数一样：
  
  ```
  struct Point<T, U> {
    x: T,
    y: U,
  }
  
  impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
  }
  
  fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};
  
    let p3 = p1.mixup(p2);
  
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
  }
  ```
- 这个例子中，T,U 是定义在结构体 Point 上的泛型参数，V,W 是单独定义在方法 mixup 上的泛型参数，它们并不冲突，说白了，你可以理解为，一个是结构体泛型，一个是函数泛型。
- 为具体的泛型类型实现方法
- 对于 Point<T> 类型，你不仅能定义基于 T 的方法，还能针对特定的具体类型，进行方法定义：
  
  ```
  impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
  }
  ```
- 这段代码意味着 Point<f32> 类型会有一个方法 distance_from_origin，而其他 T 不是 f32 类型的 Point<T> 实例则没有定义此方法。这个方法计算点实例与坐标(0.0, 0.0) 之间的距离，并使用了只能用于浮点型的数学运算符。
- 这样我们就能针对特定的泛型类型实现某个特定的方法，对于其它泛型类型则没有定义该方法。
- ### 6.7.5.  const泛型
- 在之前的泛型中，可以抽象为一句话：针对类型实现的泛型，所有的泛型都是为了抽象不同的类型，那有没有针对值的泛型？可能很多同学感觉很难理解，值怎么使用泛型？不急，我们先从数组讲起。
- 在数组那节，有提到过很重要的一点：[i32; 2] 和 [i32; 3] 是不同的数组类型，比如下面的代码：
  
  ```
  fn display_array(arr: [i32; 3]) {
    println!("{:?}", arr);
  }
  fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);
  
    let arr: [i32;2] = [1,2];
    display_array(arr);
  }
  ```
- 运行后报错：
  
  ```
  error[E0308]: mismatched types // 类型不匹配
  --> src/main.rs:10:19
   |
  10 |     display_array(arr);
   |                   ^^^ expected an array with a fixed size of 3 elements, found one with 2 elements
                          // 期望一个长度为3的数组，却发现一个长度为2的
  ```
- 结合代码和报错，可以很清楚的看出，[i32; 3] 和 [i32; 2] 确实是两个完全不同的类型，因此无法用同一个函数调用。
- 首先，让我们修改代码，让 display_array 能打印任意长度的 i32 数组：
  
  ```
  fn display_array(arr: &[i32]) {
    println!("{:?}", arr);
  }
  fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);
  
    let arr: [i32;2] = [1,2];
    display_array(&arr);
  }
  ```
- 很简单，只要使用数组切片，然后传入 arr 的不可变引用即可。
- 接着，将 i32 改成所有类型的数组：
  
  ```
  fn display_array<T: std::fmt::Debug>(arr: &[T]) {
    println!("{:?}", arr);
  }
  fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);
  
    let arr: [i32;2] = [1,2];
    display_array(&arr);
  }
  ```
- 也不难，唯一要注意的是需要对 T 加一个限制 std::fmt::Debug，该限制表明 T 可以用在 println!("{:?}", arr) 中，因为 {:?} 形式的格式化输出需要 arr 实现该特征。
- 通过引用，我们可以很轻松的解决处理任何类型数组的问题，但是如果在某些场景下引用不适宜用或者干脆不能用呢？你们知道为什么以前 Rust 的一些数组库，在使用的时候都限定长度不超过 32 吗？因为它们会为每个长度都单独实现一个函数，简直。。。毫无人性。难道没有什么办法可以解决这个问题吗？
- 好在，现在咱们有了 const 泛型，也就是针对值的泛型，正好可以用于处理数组长度的问题：
  
  ```
  fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
  }
  fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);
  
    let arr: [i32; 2] = [1, 2];
    display_array(arr);
  }
  ```
- 如上所示，我们定义了一个类型为 [T; N] 的数组，其中 T 是一个基于类型的泛型参数，这个和之前讲的泛型没有区别，而重点在于 N 这个泛型参数，它是一个基于值的泛型参数！因为它用来替代的是数组的长度。
- N 就是 const 泛型，定义的语法是 const N: usize，表示 const 泛型 N ，它基于的值类型是 usize。
- 在泛型参数之前，Rust 完全不适合复杂矩阵的运算，自从有了 const 泛型，一切即将改变。
- 假设我们某段代码需要在内存很小的平台上工作，因此需要限制函数参数占用的内存大小，此时就可以使用 const 泛型表达式来实现：
  
  ```
  // 目前只能在nightly版本下使用
  #![allow(incomplete_features)]
  #![feature(generic_const_exprs)]
  
  fn something<T>(val: T)
  where
    Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
    //       ^-----------------------------^ 这里是一个 const 表达式，换成其它的 const 表达式也可以
  {
    //
  }
  
  fn main() {
    something([0u8; 0]); // ok
    something([0u8; 512]); // ok
    something([0u8; 1024]); // 编译错误，数组长度是1024字节，超过了768字节的参数长度限制
  }
  
  // ---
  
  pub enum Assert<const CHECK: bool> {
    //
  }
  
  pub trait IsTrue {
    //
  }
  
  impl IsTrue for Assert<true> {
    //
  }
  ```
- ### 6.7.6.  泛型的性能
- 在 Rust 中泛型是零成本的抽象，意味着你在使用泛型时，完全不用担心性能上的问题。
- 但是任何选择都是权衡得失的，既然我们获得了性能上的巨大优势，那么又失去了什么呢？Rust 是在编译期为泛型对应的多个类型，生成各自的代码，因此损失了编译速度和增大了最终生成文件的大小。
- 具体来说：
- Rust 通过在编译时进行泛型代码的 单态化(monomorphization)来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。
- 编译器所做的工作正好与我们创建泛型函数的步骤相反，编译器寻找所有泛型代码被调用的位置并针对具体类型生成代码。
- 让我们看看一个使用标准库中 Option 枚举的例子：
  
  ```
  let integer = Some(5);
  let float = Some(5.0);
  ```
- 当 Rust 编译这些代码的时候，它会进行单态化。编译器会读取传递给 Option<T> 的值并发现有两种 Option<T>：一种对应 i32 另一种对应 f64。为此，它会将泛型定义 Option<T> 展开为 Option_i32 和 Option_f64，接着将泛型定义替换为这两个具体的定义。
- 编译器生成的单态化版本的代码看起来像这样：
  
  ```
  enum Option_i32 {
    Some(i32),
    None,
  }
  
  enum Option_f64 {
    Some(f64),
    None,
  }
  
  fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
  }
  ```
- 我们可以使用泛型来编写不重复的代码，而 Rust 将会为每一个实例编译其特定类型的代码。这意味着在使用泛型时没有运行时开销；当代码运行，它的执行效率就跟好像手写每个具体定义的重复代码一样。这个单态化过程正是 Rust 泛型在运行时极其高效的原因。
- ## 6.8.  特征Trait
- 如果我们想定义一个文件系统，那么把该系统跟底层存储解耦是很重要的。文件操作主要包含四个：open 、write、read、close，这些操作可以发生在硬盘，可以发生在内存，还可以发生在网络IO甚至（...我实在编不下去了，大家来帮帮我）。总之如果你要为每一种情况都单独实现一套代码，那这种实现将过于繁杂，而且也没那个必要。
- 要解决上述问题，需要把这些行为抽象出来，就要使用 Rust 中的特征 trait 概念。可能你是第一次听说这个名词，但是不要怕，如果学过其他语言，那么大概率你听说过接口，没错，特征跟接口很类似。
- 在之前的代码中，我们也多次见过特征的使用，例如 #[derive(Debug)]，它在我们定义的类型(struct)上自动派生 Debug 特征，接着可以使用 println!("{:?}", x) 打印这个类型；再例如：
  
  ```
  fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
  }
  ```
- 通过 std::ops::Add 特征来限制 T，只有 T 实现了 std::ops::Add 才能进行合法的加法操作，毕竟不是所有的类型都能进行相加。
- 这些都说明一个道理，特征定义了一组可以被共享的行为，只要实现了特征，你就能使用这组行为。
- ### 6.8.1.  定义特征
- 如果不同的类型具有相同的行为，那么我们就可以定义一个特征，然后为这些类型实现该特征。定义特征是把一些方法组合在一起，目的是定义一个实现某些目标所必需的行为的集合。
- 例如，我们现在有文章 Post 和微博 Weibo 两种内容载体，而我们想对相应的内容进行总结，也就是无论是文章内容，还是微博内容，都可以在某个时间点进行总结，那么总结这个行为就是共享的，因此可以用特征来定义：
  
  ```
  pub trait Summary {
    fn summarize(&self) -> String;
  }
  ```
- 这里使用 trait 关键字来声明一个特征，Summary 是特征名。在大括号中定义了该特征的所有方法，在这个例子中是： fn summarize(&self) -> String。
- 特征只定义行为看起来是什么样的，而不定义行为具体是怎么样的。因此，我们只定义特征方法的签名，而不进行实现，此时方法签名结尾是 ;，而不是一个 {}。
- 接下来，每一个实现这个特征的类型都需要具体实现该特征的相应方法，编译器也会确保任何实现 Summary 特征的类型都拥有与这个签名的定义完全一致的 summarize 方法。
- ### 6.8.2.  为类型实现特征
- 因为特征只定义行为看起来是什么样的，因此我们需要为类型实现具体的特征，定义行为具体是怎么样的。
- 首先来为 Post 和 Weibo 实现 Summary 特征：
  
  ```
  pub trait Summary {
    fn summarize(&self) -> String;
  }
  pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
  }
  
  impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
  }
  
  pub struct Weibo {
    pub username: String,
    pub content: String
  }
  
  impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
  }
  ```
- 实现特征的语法与为结构体、枚举实现方法很像：impl Summary for Post，读作“为 Post 类型实现 Summary 特征”，然后在 impl 的花括号中实现该特征的具体方法。
- 接下来就可以在这个类型上调用特征的方法：
  
  ```
  fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好像微博没Tweet好用".to_string()};
  
    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
  }
  ```
- 孤儿规则
- 上面我们将 Summary 定义成了 pub 公开的。这样，如果他人想要使用我们的 Summary 特征，则可以引入到他们的包中，然后再进行实现。
- 关于特征实现与定义的位置，有一条非常重要的原则：如果你想要为类型 A 实现特征 T，那么 A 或者 T 至少有一个是在当前作用域中定义的！ 例如我们可以为上面的 Post 类型实现标准库中的 Display 特征，这是因为 Post 类型定义在当前的作用域中。同时，我们也可以在当前包中为 String 类型实现 Summary 特征，因为 Summary 定义在当前作用域中。
- 但是你无法在当前作用域中，为 String 类型实现 Display 特征，因为它们俩都定义在标准库中，其定义所在的位置都不在当前作用域，跟你半毛钱关系都没有，看看就行了。
- 该规则被称为孤儿规则，可以确保其它人编写的代码不会破坏你的代码，也确保了你不会莫名其妙就破坏了风马牛不相及的代码。
- 默认实现
- 你可以在特征中定义具有默认实现的方法，这样其它类型无需再实现该方法，或者也可以选择重载该方法：
  
  ```
  pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
  }
  ```
- 上面为 Summary 定义了一个默认实现，下面我们编写段代码来测试下：
  
  ```
  impl Summary for Post {}
  
  impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
  }
  ```
- 可以看到，Post 选择了默认实现，而 Weibo 重载了该方法，调用和输出如下：
  
  ```
  println!("{}",post.summarize());
    println!("{}",weibo.summarize());
  (Read more...)
  ```
- sunface发表了微博好像微博没Tweet好用
- 默认实现允许调用相同特征中的其他方法，哪怕这些方法没有默认实现。如此，特征可以提供很多有用的功能而只需要实现指定的一小部分内容。例如，我们可以定义 Summary 特征，使其具有一个需要实现的 summarize_author 方法，然后定义一个 summarize 方法，此方法的默认实现调用 summarize_author 方法：
  
  ```
  pub trait Summary {
    fn summarize_author(&self) -> String;
  
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
  }
  ```
- 为了使用 Summary，只需要实现 summarize_author 方法即可：
  
  ```
  impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
  }
  println!("1 new weibo: {}", weibo.summarize());
  ```
- weibo.summarize() 会先调用 Summary 特征默认实现的 summarize 方法，通过该方法进而调用 Weibo 为 Summary 实现的 summarize_author 方法，最终输出：1 new weibo: (Read more from @horse_ebooks...)。
- ### 6.8.3.  使用特征作为函数参数
- 之前提到过，特征如果仅仅是用来实现方法，那真的有些大材小用，现在我们来讲下，真正可以让特征大放光彩的地方。
- 现在，先定义一个函数，使用特征作为函数参数：
  
  ```
  pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
  }
  ```
- impl Summary，只能说想出这个类型的人真的是起名鬼才，简直太贴切了，顾名思义，它的意思是 实现了Summary特征 的 item 参数。
- 你可以使用任何实现了 Summary 特征的类型作为该函数的参数，同时在函数体内，还可以调用该特征的方法，例如 summarize 方法。具体的说，可以传递 Post 或 Weibo 的实例来作为参数，而其它类如 String 或者 i32 的类型则不能用做该函数的参数，因为它们没有实现 Summary 特征。
- ### 6.8.4.  特征约束
- 虽然 impl Trait 这种语法非常好理解，但是实际上它只是一个语法糖：
  
  ```
  pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
  }
  ```
- 真正的完整书写形式如上所述，形如 T: Summary 被称为特征约束。
- 在简单的场景下 impl Trait 这种语法糖就足够使用，但是对于复杂的场景，特征约束可以让我们拥有更大的灵活性和语法表现能力，例如一个函数接受两个 impl Summary 的参数：
  
  ```
  pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
  ```
- 如果函数两个参数是不同的类型，那么上面的方法很好，只要这两个类型都实现了 Summary 特征即可。但是如果我们想要强制函数的两个参数是同一类型呢？上面的语法就无法做到这种限制，此时我们只能使特征约束来实现：
  
  ```
  pub fn notify<T: Summary>(item1: &T, item2: &T) {}
  ```
- 泛型类型 T 说明了 item1 和 item2 必须拥有同样的类型，同时 T: Summary 说明了 T 必须实现 Summary 特征。
- 多重约束
- 除了单个约束条件，我们还可以指定多个约束条件，例如除了让参数实现 Summary 特征外，还可以让参数实现 Display 特征以控制它的格式化输出：
  
  ```
  pub fn notify(item: &(impl Summary + Display)) {}
  ```
- 除了上述的语法糖形式，还能使用特征约束的形式：
  
  ```
  pub fn notify<T: Summary + Display>(item: &T) {}
  ```
- 通过这两个特征，就可以使用 item.summarize 方法，以及通过 println!("{}", item) 来格式化输出 item。
- Where 约束
- 当特征约束变得很多时，函数的签名将变得很复杂：
  
  ```
  fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
  ```
- 严格来说，上面的例子还是不够复杂，但是我们还是能对其做一些形式上的改进，通过 where：
  
  ```
  fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
  {}
  ```
- 使用特征约束有条件地实现方法或特征
- 特征约束，可以让我们在指定类型 + 指定特征的条件下去实现方法，例如：
  
  ```
  use std::fmt::Display;
  
  struct Pair<T> {
    x: T,
    y: T,
  }
  
  impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
  }
  
  impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
  }
  ```
- cmp_display 方法，并不是所有的 Pair<T> 结构体对象都可以拥有，只有 T 同时实现了 Display + PartialOrd 的 Pair<T> 才可以拥有此方法。 该函数可读性会更好，因为泛型参数、参数、返回值都在一起，可以快速的阅读，同时每个泛型参数的特征也在新的代码行中通过特征约束进行了约束。
- 也可以有条件地实现特征, 例如，标准库为任何实现了 Display 特征的类型实现了 ToString 特征：
  
  ```
  impl<T: Display> ToString for T {
    // --snip--
  }
  ```
- 我们可以对任何实现了 Display 特征的类型调用由 ToString 定义的 to_string 方法。例如，可以将整型转换为对应的 String 值，因为整型实现了 Display：
  
  ```
  let s = 3.to_string();
  ```
- 使用特征约束有条件地实现方法或特征
- 特征约束，可以让我们在指定类型 + 指定特征的条件下去实现方法，例如：
  
  ```
  use std::fmt::Display;
  
  struct Pair<T> {
    x: T,
    y: T,
  }
  
  impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
  }
  
  impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
  }
  ```
- cmp_display 方法，并不是所有的 Pair<T> 结构体对象都可以拥有，只有 T 同时实现了 Display + PartialOrd 的 Pair<T> 才可以拥有此方法。 该函数可读性会更好，因为泛型参数、参数、返回值都在一起，可以快速的阅读，同时每个泛型参数的特征也在新的代码行中通过特征约束进行了约束。
- 也可以有条件地实现特征, 例如，标准库为任何实现了 Display 特征的类型实现了 ToString 特征：
  
  ```
  impl<T: Display> ToString for T {
    // --snip--
  }
  ```
- 我们可以对任何实现了 Display 特征的类型调用由 ToString 定义的 to_string 方法。例如，可以将整型转换为对应的 String 值，因为整型实现了 Display：
  
  ```
  let s = 3.to_string();
  ```
- ### 6.8.5.  函数返回中的 impl Trait
- 可以通过 impl Trait 来说明一个函数返回了一个类型，该类型实现了某个特征：
  
  ```
  fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
  }
  ```
- 因为 Weibo 实现了 Summary，因此这里可以用它来作为返回值。要注意的是，虽然我们知道这里是一个 Weibo 类型，但是对于 returns_summarizable 的调用者而言，他只知道返回了一个实现了 Summary 特征的对象，但是并不知道返回了一个 Weibo 类型。
- 这种 impl Trait 形式的返回值，在一种场景下非常非常有用，那就是返回的真实类型非常复杂，你不知道该怎么声明时(毕竟 Rust 要求你必须标出所有的类型)，此时就可以用 impl Trait 的方式简单返回。例如，闭包和迭代器就是很复杂，只有编译器才知道那玩意的真实类型，如果让你写出来它们的具体类型，估计内心有一万只草泥马奔腾，好在你可以用 impl Iterator 来告诉调用者，返回了一个迭代器，因为所有迭代器都会实现 Iterator 特征。
- 但是这种返回值方式有一个很大的限制：只能有一个具体的类型，例如：
  
  ```
  fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        }
    }
  }
  ```
- 以上的代码就无法通过编译，因为它返回了两个不同的类型 Post 和 Weibo。
  
  ```
  `if` and `else` have incompatible types
  expected struct `Post`, found struct `Weibo`
  ```
- 报错提示我们 if 和 else 返回了不同的类型。如果想要实现返回不同的类型，需要使用下一章节中的特征对象。
- ### 6.8.6.  largest函数
- 还记得上一节中的例子吧，当时留下一个疑问，该如何解决编译报错：
  
  ```
  error[E0369]: binary operation `>` cannot be applied to type `T` // 无法在 `T` 类型上应用`>`运算符
  --> src/main.rs:5:17
  |
  5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
  help: consider restricting type parameter `T` // 考虑使用以下的特征来约束 `T`
  |
  1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ^^^^^^^^^^^^^^^^^^^^^^
  ```
- 在 largest 函数体中我们想要使用大于运算符（>）比较两个 T 类型的值。这个运算符是标准库中特征 std::cmp::PartialOrd 的一个默认方法。所以需要在 T 的特征约束中指定 PartialOrd，这样 largest 函数可以用于内部元素类型可比较大小的数组切片。
- 由于 PartialOrd 位于 prelude 中所以并不需要通过 std::cmp 手动将其引入作用域。所以可以将 largest 的签名修改为如下：
  
  ```
  fn largest<T: PartialOrd>(list: &[T]) -> T {}
  ```
- 但是此时编译，又会出现新的错误：
  
  ```
  error[E0508]: cannot move out of type `[T]`, a non-copy slice
  --> src/main.rs:2:23
  |
  2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       help: consider using a reference instead: `&list[0]`
  
  error[E0507]: cannot move out of borrowed content
  --> src/main.rs:4:9
  |
  4 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
  ```
- 错误的核心是 cannot move out of type [T], a non-copy slice，原因是 T 没有实现 Copy 特性，因此我们只能把所有权进行转移，毕竟只有 i32 等基础类型才实现了 Copy 特性，可以存储在栈上，而 T 可以指代任何类型（严格来说是实现了 PartialOrd 特征的所有类型）。
- 因此，为了让 T 拥有 Copy 特性，我们可以增加特征约束：
  
  ```
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
  
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
  
    largest
  }
  
  fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
  
    let result = largest(&number_list);
    println!("The largest number is {}", result);
  
    let char_list = vec!['y', 'm', 'a', 'q'];
  
    let result = largest(&char_list);
    println!("The largest char is {}", result);
  }
  ```
- 如果并不希望限制 largest 函数只能用于实现了 Copy 特征的类型，我们可以在 T 的特征约束中指定 Clone 特征 而不是 Copy 特征。并克隆 list 中的每一个值使得 largest 函数拥有其所有权。使用 clone 函数意味着对于类似 String 这样拥有堆上数据的类型，会潜在地分配更多堆上空间，而堆分配在涉及大量数据时可能会相当缓慢。
- 另一种 largest 的实现方式是返回在 list 中 T 值的引用。如果我们将函数返回值从 T 改为 &T 并改变函数体使其能够返回一个引用，我们将不需要任何 Clone 或 Copy 的特征约束而且也不会有任何的堆分配。尝试自己实现这种替代解决方式吧！
- ### 6.8.7.  通过derive派生特征
- 在本书中，形如 #[derive(Debug)] 的代码已经出现了很多次，这种是一种特征派生语法，被 derive 标记的对象会自动实现对应的默认特征代码，继承相应的功能。
- 例如 Debug 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 println!("{:?}", s) 的形式打印该结构体的对象。
- 再如 Copy 特征，它也有一套自动实现的默认代码，当标记到一个类型上时，可以让这个类型自动实现 Copy 特征，进而可以调用 copy 方法，进行自我复制。
- 总之，derive 派生出来的是 Rust 默认给我们提供的特征，在开发过程中极大的简化了自己手动实现相应特征的需求，当然，如果你有特殊的需求，还可以自己手动重载该实现。
- 详细的 derive 列表参见附录-派生特征。
- ### 6.8.8.  引入特征
- 在一些场景中，使用 as 关键字做类型转换会有比较大的限制，因为你想要在类型转换上拥有完全的控制，例如处理转换错误，那么你将需要 TryInto：
  
  ```
  use std::convert::TryInto;
  
  fn main() {
  let a: i32 = 10;
  let b: u16 = 100;
  
  let b_ = b.try_into()
            .unwrap();
  
  if a < b_ {
    println!("Ten is less than one hundred.");
  }
  }
  ```
- 上面代码中引入了 std::convert::TryInto 特征，但是却没有使用它，可能有些同学会为此困惑，主要原因在于如果你要使用一个特征的方法，那么你需要将该特征引入当前的作用域中，我们在上面用到了 try_into 方法，因此需要引入对应的特征。
- 但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 std::prelude 模块提前引入到当前作用域中，其中包括了 std::convert::TryInto，你可以尝试删除第一行的代码 use ...，看看是否会报错。
- ## 6.9.  特征对象
- 在上一节中有一段代码无法通过编译:
  
  ```
  fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
           // ...
        }
    } else {
        Weibo {
            // ...
        }
    }
  }
  ```
- 其中 Post 和 Weibo 都实现了 Summary 特征，因此上面的函数试图通过返回 impl Summary 来返回这两个类型，但是编译器却无情地报错了，原因是 impl Trait 的返回值类型并不支持多种不同的类型返回，那如果我们想返回多种类型，该怎么办？
- 再来考虑一个问题：现在在做一款游戏，需要将多个对象渲染在屏幕上，这些对象属于不同的类型，存储在列表中，渲染的时候，需要循环该列表并顺序渲染每个对象，在 Rust 中该怎么实现？
- 聪明的同学可能已经能想到一个办法，利用枚举：
  
  ```
  #[derive(Debug)]
  enum UiObject {
    Button,
    SelectBox,
  }
  
  fn main() {
    let objects = [
        UiObject::Button,
        UiObject::SelectBox
    ];
  
    for o in objects {
        draw(o)
    }
  }
  
  fn draw(o: UiObject) {
    println!("{:?}",o);
  }
  ```
- Bingo，这个确实是一个办法，但是问题来了，如果你的对象集合并不能事先明确地知道呢？或者别人想要实现一个 UI 组件呢？此时枚举中的类型是有些缺少的，是不是还要修改你的代码增加一个枚举成员？
- 总之，在编写这个 UI 库时，我们无法知道所有的 UI 对象类型，只知道的是：
- UI 对象的类型不同
- 需要一个统一的类型来处理这些对象，无论是作为函数参数还是作为列表中的一员
- 需要对每一个对象调用 draw 方法
- 在拥有继承的语言中，可以定义一个名为 Component 的类，该类上有一个 draw 方法。其他的类比如 Button、Image 和 SelectBox 会从 Component 派生并因此继承 draw 方法。它们各自都可以覆盖 draw 方法来定义自己的行为，但是框架会把所有这些类型当作是 Component 的实例，并在其上调用 draw。不过 Rust 并没有继承，我们得另寻出路。
- ### 6.9.1.  特征对象定义
- 为了解决上面的所有问题，Rust 引入了一个概念 —— 特征对象。
- 在介绍特征对象之前，先来为之前的 UI 组件定义一个特征：
  
  ```
  pub trait Draw {
    fn draw(&self);
  }
  ```
- 只要组件实现了 Draw 特征，就可以调用 draw 方法来进行渲染。假设有一个 Button 和 SelectBox 组件实现了 Draw 特征：
  
  ```
  pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
  }
  
  impl Draw for Button {
    fn draw(&self) {
        // 绘制按钮的代码
    }
  }
  
  struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
  }
  
  impl Draw for SelectBox {
    fn draw(&self) {
        // 绘制SelectBox的代码
    }
  }
  ```
- 此时，还需要一个动态数组来存储这些 UI 对象：
  
  ```
  pub struct Screen {
    pub components: Vec<?>,
  }
  ```
- 注意到上面代码中的 ? 吗？它的意思是：我们应该填入什么类型，可以说就之前学过的内容里，你找不到哪个类型可以填入这里，但是因为 Button 和 SelectBox 都实现了 Draw 特征，那我们是不是可以把 Draw 特征的对象作为类型，填入到数组中呢？答案是肯定的。
- 特征对象指向实现了 Draw 特征的类型的实例，也就是指向了 Button 或者 SelectBox 的实例，这种映射关系是存储在一张表中，可以在运行时通过特征对象找到具体调用的类型方法。
- 可以通过 & 引用或者 Box<T> 智能指针的方式来创建特征对象。
- Box<T> 在后面章节会详细讲解，大家现在把它当成一个引用即可，只不过它包裹的值会被强制分配在堆上。
  
  ```
  trait Draw {
    fn draw(&self) -> String;
  }
  
  impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
  }
  
  impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
  }
  
  // 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
  fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
  }
  
  fn draw2(x: &dyn Draw) {
    x.draw();
  }
  
  fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;
  
    // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
    // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
    draw1(Box::new(x));
    // 基于 y 的值创建一个 Box<u8> 类型的智能指针
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
  }
  ```
- 上面代码，有几个非常重要的点：
	- draw1 函数的参数是 Box<dyn Draw> 形式的特征对象，该特征对象是通过 Box::new(x) 的方式创建的
	- draw2 函数的参数是 &dyn Draw 形式的特征对象，该特征对象是通过 &x 的方式创建的
	- dyn 关键字只用在特征对象的类型声明上，在创建时无需使用 dyn
- 因此，可以使用特征对象来代表泛型或具体的类型。
- 继续来完善之前的 UI 组件代码，首先来实现 Screen：
  
  ```
  pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
  }
  ```
- 其中存储了一个动态数组，里面元素的类型是 Draw 特征对象：Box<dyn Draw>，任何实现了 Draw 特征的类型，都可以存放其中。
- 再来为 Screen 定义 run 方法，用于将列表中的 UI 组件渲染在屏幕上：
  
  ```
  impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
  }
  ```
- 至此，我们就完成了之前的目标：在列表中存储多种不同类型的实例，然后将它们使用同一个方法逐一渲染在屏幕上！
- 再来看看，如果通过泛型实现，会如何：
  
  ```
  pub struct Screen<T: Draw> {
    pub components: Vec<T>,
  }
  
  impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
  }
  ```
- 上面的 Screen 的列表中，存储了类型为 T 的元素，然后在 Screen 中使用特征约束让 T 实现了 Draw 特征，进而可以调用 draw 方法。
- 但是这种写法限制了 Screen 实例的 Vec<T> 中的每个元素必须是 Button 类型或者全是 SelectBox 类型。如果只需要同质（相同类型）集合，更倾向于采用泛型+特征约束这种写法，因其实现更清晰，且性能更好(特征对象，需要在运行时从 vtable 动态查找需要调用的方法)。
- 现在来运行渲染下咱们精心设计的 UI 组件列表：
  
  ```
  fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };
  
    screen.run();
  }
  ```
- 上面使用 Box::new(T) 的方式来创建了两个 Box<dyn Draw> 特征对象，如果以后还需要增加一个 UI 组件，那么让该组件实现 Draw 特征，则可以很轻松的将其渲染在屏幕上，甚至用户可以引入我们的库作为三方库，然后在自己的库中为自己的类型实现 Draw 特征，然后进行渲染。
- 在动态类型语言中，有一个很重要的概念：鸭子类型(duck typing)，简单来说，就是只关心值长啥样，而不关心它实际是什么。当一个东西走起来像鸭子，叫起来像鸭子，那么它就是一只鸭子，就算它实际上是一个奥特曼，也不重要，我们就当它是鸭子。
- 在上例中，Screen 在 run 的时候，我们并不需要知道各个组件的具体类型是什么。它也不检查组件到底是 Button 还是 SelectBox 的实例，只要它实现了 Draw 特征，就能通过 Box::new 包装成 Box<dyn Draw> 特征对象，然后被渲染在屏幕上。
- 使用特征对象和 Rust 类型系统来进行类似鸭子类型操作的优势是，无需在运行时检查一个值是否实现了特定方法或者担心在调用时因为值没有实现方法而产生错误。如果值没有实现特征对象所需的特征， 那么 Rust 根本就不会编译这些代码：
  
  ```
  fn main() {
    let screen = Screen {
        components: vec![
            Box::new(String::from("Hi")),
        ],
    };
  
    screen.run();
  }
  ```
- 因为 String 类型没有实现 Draw 特征，编译器直接就会报错，不会让上述代码运行。如果想要 String 类型被渲染在屏幕上，那么只需要为其实现 Draw 特征即可，非常容易。
- 注意 dyn 不能单独作为特征对象的定义，例如下面的代码编译器会报错，原因是特征对象可以是任意实现了某个特征的类型，编译器在编译期不知道该类型的大小，不同的类型大小是不同的。
- 而 &dyn 和 Box<dyn> 在编译期都是已知大小，所以可以用作特征对象的定义。
  
  ```
  fn draw2(x: dyn Draw) {
    x.draw();
  }
  10 | fn draw2(x: dyn Draw) {
   |          ^ doesn't have a size known at compile-time
   |
   = help: the trait `Sized` is not implemented for `(dyn Draw + 'static)`
  help: function arguments must have a statically known size, borrowed types always have a known
  ```
- ### 6.9.2.  特征对象的动态分发
- 回忆一下泛型章节我们提到过的，泛型是在编译期完成处理的：编译器会为每一个泛型参数对应的具体类型生成一份代码，这种方式是静态分发(static dispatch)，因为是在编译期完成的，对于运行期性能完全没有任何影响。
- 与静态分发相对应的是动态分发(dynamic dispatch)，在这种情况下，直到运行时，才能确定需要调用什么方法。之前代码中的关键字 dyn 正是在强调这一“动态”的特点。
- 当使用特征对象时，Rust 必须使用动态分发。编译器无法知晓所有可能用于特征对象代码的类型，所以它也不知道应该调用哪个类型的哪个方法实现。为此，Rust 在运行时使用特征对象中的指针来知晓需要调用哪个方法。动态分发也阻止编译器有选择的内联方法代码，这会相应的禁用一些优化。
- 下面这张图很好的解释了静态分发 Box<T> 和动态分发 Box<dyn Trait> 的区别：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1725665962608-ef9fd4e9-577d-496f-883c-33f6a86cb58c.png)
- 结合上文的内容和这张图可以了解：
	- 特征对象大小不固定：这是因为，对于特征 Draw，类型 Button 可以实现特征 Draw，类型 SelectBox 也可以实现特征 Draw，因此特征没有固定大小
	- 几乎总是使用特征对象的引用方式，如 &dyn Draw、Box<dyn Draw>
		- 虽然特征对象没有固定大小，但它的引用类型的大小是固定的，它由两个指针组成（ptr 和 vptr），因此占用两个指针大小
		- 一个指针 ptr 指向实现了特征 Draw 的具体类型的实例，也就是当作特征 Draw 来用的类型的实例，比如类型 Button 的实例、类型 SelectBox 的实例
	- 另一个指针 vptr 指向一个虚表 vtable，vtable 中保存了类型 Button 或类型 SelectBox 的实例对于可以调用的实现于特征 Draw 的方法。当调用方法时，直接从 vtable 中找到方法并调用。之所以要使用一个 vtable 来保存各实例的方法，是因为实现了特征 Draw 的类型有多种，这些类型拥有的方法各不相同，当将这些类型的实例都当作特征 Draw 来使用时(此时，它们全都看作是特征 Draw 类型的实例)，有必要区分这些实例各自有哪些方法可调用
- 简而言之，当类型 Button 实现了特征 Draw 时，类型 Button 的实例对象 btn 可以当作特征 Draw 的特征对象类型来使用，btn 中保存了作为特征对象的数据指针（指向类型 Button 的实例数据）和行为指针（指向 vtable）。
- 一定要注意，此时的 btn 是 Draw 的特征对象的实例，而不再是具体类型 Button 的实例，而且 btn 的 vtable 只包含了实现自特征 Draw 的那些方法（比如 draw），因此 btn 只能调用实现于特征 Draw 的 draw 方法，而不能调用类型 Button 本身实现的方法和类型 Button 实现于其他特征的方法。也就是说，btn 是哪个特征对象的实例，它的 vtable 中就包含了该特征的方法。
- ### 6.9.3.  Self与self
- 在 Rust 中，有两个self，一个指代当前的实例对象，一个指代特征或者方法类型的别名：
  
  ```
  trait Draw {
    fn draw(&self) -> Self;
  }
  
  #[derive(Clone)]
  struct Button;
  impl Draw for Button {
    fn draw(&self) -> Self {
        return self.clone()
    }
  }
  
  fn main() {
    let button = Button;
    let newb = button.draw();
  }
  ```
- 上述代码中，self指代的就是当前的实例对象，也就是 button.draw() 中的 button 实例，Self 则指代的是 Button 类型。
- 当理解了 self 与 Self 的区别后，我们再来看看何为对象安全。
- ### 6.9.4.  特征对象的限制
- 不是所有特征都能拥有特征对象，只有对象安全的特征才行。当一个特征的所有方法都有如下属性时，它的对象才是安全的：
	- 方法的返回类型不能是 Self
	- 方法没有任何泛型参数
- 对象安全对于特征对象是必须的，因为一旦有了特征对象，就不再需要知道实现该特征的具体类型是什么了。如果特征方法返回了具体的 Self 类型，但是特征对象忘记了其真正的类型，那这个 Self 就非常尴尬，因为没人知道它是谁了。但是对于泛型类型参数来说，当使用特征时其会放入具体的类型参数：此具体类型变成了实现该特征的类型的一部分。而当使用特征对象时其具体类型被抹去了，故而无从得知放入泛型参数类型到底是什么。
- 标准库中的 Clone 特征就不符合对象安全的要求：
  
  ```
  pub trait Clone {
    fn clone(&self) -> Self;
  }
  ```
- 因为它的其中一个方法，返回了 Self 类型，因此它是对象不安全的。
- String 类型实现了 Clone 特征， String 实例上调用 clone 方法时会得到一个 String 实例。类似的，当调用 Vec<T> 实例的 clone 方法会得到一个 Vec<T> 实例。clone 的签名需要知道什么类型会代替 Self，因为这是它的返回值。
- 如果违反了对象安全的规则，编译器会提示你。例如，如果尝试使用之前的 Screen 结构体来存放实现了 Clone 特征的类型：
  
  ```
  pub struct Screen {
    pub components: Vec<Box<dyn Clone>>,
  }
  ```
- 将会得到如下错误：
  
  ```
  error[E0038]: the trait `std::clone::Clone` cannot be made into an object
  --> src/lib.rs:2:5
  |
  2 |     pub components: Vec<Box<dyn Clone>>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::clone::Clone`
  cannot be made into an object
  |
  = note: the trait cannot require that `Self : Sized`
  ```
- 这意味着不能以这种方式使用此特征作为特征对象。
- ### 6.9.5.  进一步深入特征
- 关联类型
- 关联类型是在特征定义的语句块中，申明一个自定义类型，这样就可以在特征的方法签名中使用该类型：
  
  ```
  pub trait Iterator {
    type Item;
  
    fn next(&mut self) -> Option<Self::Item>;
  }
  ```
- 以上是标准库中的迭代器特征 Iterator，它有一个 Item 关联类型，用于替代遍历的值的类型。
- 同时，next 方法也返回了一个 Item 类型，不过使用 Option 枚举进行了包裹，假如迭代器中的值是 i32 类型，那么调用 next 方法就将获取一个 Option<i32> 的值。
- 还记得 Self 吧？在之前的章节提到过， Self 用来指代当前调用者的具体类型，那么 Self::Item 就用来指代该类型实现中定义的 Item 类型：
  
  ```
  impl Iterator for Counter {
    type Item = u32;
  
    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
  }
  
  fn main() {
    let c = Counter{..}
    c.next()
  }
  ```
- 在上述代码中，我们为 Counter 类型实现了 Iterator 特征，变量 c 是特征 Iterator 的实例，也是 next 方法的调用者。 结合之前的黑体内容可以得出：对于 next 方法而言，Self 是调用者 c 的具体类型： Counter，而 Self::Item 是 Counter 中定义的 Item 类型: u32。
- 聪明的读者之所以聪明，是因为你们喜欢联想和举一反三，同时你们也喜欢提问：为何不用泛型，例如如下代码：
  
  ```
  pub trait Iterator<Item> {
    fn next(&mut self) -> Option<Item>;
  }
  ```
- 答案其实很简单，为了代码的可读性，当你使用了泛型后，你需要在所有地方都写 Iterator<Item>，而使用了关联类型，你只需要写 Iterator，当类型定义复杂时，这种写法可以极大的增加可读性：
  
  ```
  pub trait CacheableItem: Clone + Default + fmt::Debug + Decodable + Encodable {
  type Address: AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash;
  fn is_null(&self) -> bool;
  }
  ```
- 例如上面的代码，Address 的写法自然远比 AsRef<[u8]> + Clone + fmt::Debug + Eq + Hash 要简单的多，而且含义清晰。
- 再例如，如果使用泛型，你将得到以下的代码：
  
  ```
  trait Container<A,B> {
    fn contains(&self,a: A,b: B) -> bool;
  }
  
  fn difference<A,B,C>(container: &C) -> i32
  where
    C : Container<A,B> {...}
  ```
- 可以看到，由于使用了泛型，导致函数头部也必须增加泛型的声明，而使用关联类型，将得到可读性好得多的代码：
  
  ```
  trait Container{
    type A;
    type B;
    fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
  }
  
  fn difference<C: Container>(container: &C) {}
  ```
- 默认泛型类型参数
- 当使用泛型类型参数时，可以为其指定一个默认的具体类型，例如标准库中的 std::ops::Add 特征：
  
  ```
  trait Add<RHS=Self> {
    type Output;
  
    fn add(self, rhs: RHS) -> Self::Output;
  }
  ```
- 它有一个泛型参数 RHS，但是与我们以往的用法不同，这里它给 RHS 一个默认值，也就是当用户不指定 RHS 时，默认使用两个同样类型的值进行相加，然后返回一个关联类型 Output。
- 可能上面那段不太好理解，下面我们用代码来举例：
  
  ```
  use std::ops::Add;
  
  #[derive(Debug, PartialEq)]
  struct Point {
    x: i32,
    y: i32,
  }
  
  impl Add for Point {
    type Output = Point;
  
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
  }
  
  fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
  }
  ```
- 上面的代码主要干了一件事，就是为 Point 结构体提供 + 的能力，这就是运算符重载，不过 Rust 并不支持创建自定义运算符，你也无法为所有运算符进行重载，目前来说，只有定义在 std::ops 中的运算符才能进行重载。
- 跟 + 对应的特征是 std::ops::Add，我们在之前也看过它的定义 trait Add<RHS=Self>，但是上面的例子中并没有为 Point 实现 Add<RHS> 特征，而是实现了 Add 特征（没有默认泛型类型参数），这意味着我们使用了 RHS 的默认类型，也就是 Self。换句话说，我们这里定义的是两个相同的 Point 类型相加，因此无需指定 RHS。
- 与上面的例子相反，下面的例子，我们来创建两个不同类型的相加：
  
  ```
  use std::ops::Add;
  
  struct Millimeters(u32);
  struct Meters(u32);
  
  impl Add<Meters> for Millimeters {
    type Output = Millimeters;
  
    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
  }
  ```
- 这里，是进行 Millimeters + Meters 两种数据类型的 + 操作，因此此时不能再使用默认的 RHS，否则就会变成 Millimeters + Millimeters 的形式。使用 Add<Meters> 可以将 RHS 指定为 Meters，那么 fn add(self, rhs: RHS) 自然而言的变成了 Millimeters 和 Meters 的相加。
- 默认类型参数主要用于两个方面：
	- 减少实现的样板代码
	- 扩展类型但是无需大幅修改现有的代码
- 之前的例子就是第一点，虽然效果也就那样。在 + 左右两边都是同样类型时，只需要 impl Add 即可，否则你需要 impl Add<SOME_TYPE>，嗯，会多写几个字:)
- 对于第二点，也很好理解，如果你在一个复杂类型的基础上，新引入一个泛型参数，可能需要修改很多地方，但是如果新引入的泛型参数有了默认类型，情况就会好很多，添加泛型参数后，使用这个类型的代码需要逐个在类型提示部分添加泛型参数，就很麻烦；但是有了默认参数（且默认参数取之前的实现里假设的值的情况下）之后，原有的使用这个类型的代码就不需要做改动了。
- 调用同名方法
- 不同特征拥有同名的方法是很正常的事情，你没有任何办法阻止这一点；甚至除了特征上的同名方法外，在你的类型上，也有同名方法：
  
  ```
  trait Pilot {
    fn fly(&self);
  }
  
  trait Wizard {
    fn fly(&self);
  }
  
  struct Human;
  
  impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
  }
  
  impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
  }
  
  impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
  }
  ```
- 这里，不仅仅两个特征 Pilot 和 Wizard 有 fly 方法，就连实现那两个特征的 Human 单元结构体，也拥有一个同名方法 fly (这世界怎么了，非要这么卷吗？程序员何苦难为程序员，哎)。
- 既然代码已经不可更改，那下面我们来讲讲该如何调用这些 fly 方法。
- 优先调用类型上的方法
- 当调用 Human 实例的 fly 时，编译器默认调用该类型中定义的方法：
  
  ```
  fn main() {
    let person = Human;
    person.fly();
  }
  ```
- 这段代码会打印 *waving arms furiously*，说明直接调用了类型上定义的方法。
- 调用特征上的方法
- 为了能够调用两个特征的方法，需要使用显式调用的语法：
  
  ```
  fn main() {
    let person = Human;
    Pilot::fly(&person); // 调用Pilot特征上的方法
    Wizard::fly(&person); // 调用Wizard特征上的方法
    person.fly(); // 调用Human类型自身的方法
  }
  ```
- 运行后依次输出：
  
  ```
  This is your captain speaking.
  Up!
  *waving arms furiously*
  ```
- 因为 fly 方法的参数是 self，当显式调用时，编译器就可以根据调用的类型( self 的类型)决定具体调用哪个方法。
- 这个时候问题又来了，如果方法没有 self 参数呢？稍等，估计有读者会问：还有方法没有 self 参数？看到这个疑问，作者的眼泪不禁流了下来，大明湖畔的关联函数，你还记得嘛？
- 但是成年人的世界，就算再伤心，事还得做，咱们继续：
  
  ```
  trait Animal {
    fn baby_name() -> String;
  }
  
  struct Dog;
  
  impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
  }
  
  impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
  }
  
  fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
  }
  ```
- 就像人类妈妈会给自己的宝宝起爱称一样，狗妈妈也会。狗妈妈称呼自己的宝宝为Spot，其它动物称呼狗宝宝为puppy，这个时候假如有动物不知道该如何称呼狗宝宝，它需要查询一下。
- Dog::baby_name() 的调用方式显然不行，因为这只是狗妈妈对宝宝的爱称，可能你会想到通过下面的方式查询其他动物对狗狗的称呼：
  
  ```
  fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
  }
  ```
- 铛铛，无情报错了：
  
  ```
  error[E0283]: type annotations needed // 需要类型注释
  --> src/main.rs:20:43
   |
  20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot infer type // 无法推断类型
   |
   = note: cannot satisfy `_: Animal`
  ```
- 因为单纯从 Animal::baby_name() 上，编译器无法得到任何有效的信息：实现 Animal 特征的类型可能有很多，你究竟是想获取哪个动物宝宝的名称？狗宝宝？猪宝宝？还是熊宝宝？
- 此时，就需要使用完全限定语法。
- 完全限定语法
- 完全限定语法是调用函数最为明确的方式：
  
  ```
  fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
  }
  ```
- 在尖括号中，通过 as 关键字，我们向 Rust 编译器提供了类型注解，也就是 Animal 就是 Dog，而不是其他动物，因此最终会调用 impl Animal for Dog 中的方法，获取到其它动物对狗宝宝的称呼：puppy。
- 言归正题，完全限定语法定义为：
  
  ```
  <Type as Trait>::function(receiver_if_method, next_arg, ...);
  ```
- 上面定义中，第一个参数是方法接收器 receiver （三种 self），只有方法才拥有，例如关联函数就没有 receiver。
- 完全限定语法可以用于任何函数或方法调用，那么我们为何很少用到这个语法？原因是 Rust 编译器能根据上下文自动推导出调用的路径，因此大多数时候，我们都无需使用完全限定语法。只有当存在多个同名函数或方法，且 Rust 无法区分出你想调用的目标函数时，该用法才能真正有用武之地。
- 特征定义中的特征约束
- 有时，我们会需要让某个特征 A 能使用另一个特征 B 的功能(另一种形式的特征约束)，这种情况下，不仅仅要为类型实现特征 A，还要为类型实现特征 B 才行，这就是 supertrait (实在不知道该如何翻译，有大佬指导下嘛？)
- 例如有一个特征 OutlinePrint，它有一个方法，能够对当前的实现类型进行格式化输出：
  
  ```
  use std::fmt::Display;
  
  trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
  }
  ```
- 等等，这里有一个眼熟的语法: OutlinePrint: Display，感觉很像之前讲过的特征约束，只不过用在了特征定义中而不是函数的参数中，是的，在某种意义上来说，这和特征约束非常类似，都用来说明一个特征需要实现另一个特征，这里就是：如果你想要实现 OutlinePrint 特征，首先你需要实现 Display 特征。
- 想象一下，假如没有这个特征约束，那么 self.to_string 还能够调用吗（ to_string 方法会为实现 Display 特征的类型自动实现）？编译器肯定是不愿意的，会报错说当前作用域中找不到用于 &Self 类型的方法 to_string ：
  
  ```
  struct Point {
    x: i32,
    y: i32,
  }
  
  impl OutlinePrint for Point {}
  ```
- 因为 Point 没有实现 Display 特征，会得到下面的报错：
  
  ```
  error[E0277]: the trait bound `Point: std::fmt::Display` is not satisfied
  --> src/main.rs:20:6
   |
  20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ `Point` cannot be formatted with the default formatter;
  try using `:?` instead if you are using a format string
   |
   = help: the trait `std::fmt::Display` is not implemented for `Point`
  ```
- 既然我们有求于编译器，那只能选择满足它咯：
  
  ```
  use std::fmt;
  
  impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
  }
  ```
- 上面代码为 Point 实现了 Display 特征，那么 to_string 方法也将自动实现：最终获得字符串是通过这里的 fmt 方法获得的。
- 在外部类型上实现外部特征
- 在特征章节中，有提到孤儿规则，简单来说，就是特征或者类型必需至少有一个是本地的，才能在此类型上定义特征。
- 这里提供一个办法来绕过孤儿规则，那就是使用newtype 模式，简而言之：就是为一个元组结构体创建新类型。该元组结构体封装有一个字段，该字段就是希望实现特征的具体类型。
- 该封装类型是本地的，因此我们可以为此类型实现外部的特征。
- newtype 不仅仅能实现以上的功能，而且它在运行时没有任何性能损耗，因为在编译期，该类型会被自动忽略。
- 下面来看一个例子，我们有一个动态数组类型： Vec<T>，它定义在标准库中，还有一个特征 Display，它也定义在标准库中，如果没有 newtype，我们是无法为 Vec<T> 实现 Display 的：
  
  ```
  error[E0117]: only traits defined in the current crate can be implemented for arbitrary types
  --> src/main.rs:5:1
  |
  5 | impl<T> std::fmt::Display for Vec<T> {
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^------
  | |                             |
  | |                             Vec is not defined in the current crate
  | impl doesn't use only types from inside the current crate
  |
  = note: define and implement a trait or new type instead
  ```
- 编译器给了我们提示： define and implement a trait or new type instead，重新定义一个特征，或者使用 new type，前者当然不可行，那么来试试后者：
  
  ```
  use std::fmt;
  
  struct Wrapper(Vec<String>);
  
  impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
  }
  
  fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
  }
  ```
- 其中，struct Wrapper(Vec<String>) 就是一个元组结构体，它定义了一个新类型 Wrapper，代码很简单，相信大家也很容易看懂。
- 既然 new type 有这么多好处，它有没有不好的地方呢？答案是肯定的。注意到我们怎么访问里面的数组吗？self.0.join(", ")，是的，很啰嗦，因为需要先从 Wrapper 中取出数组: self.0，然后才能执行 join 方法。
- 类似的，任何数组上的方法，你都无法直接调用，需要先用 self.0 取出数组，然后再进行调用。
- 当然，解决办法还是有的，要不怎么说 Rust 是极其强大灵活的编程语言！Rust 提供了一个特征叫 Deref，实现该特征后，可以自动做一层类似类型转换的操作，可以将 Wrapper 变成 Vec<String> 来使用。这样就会像直接使用数组那样去使用 Wrapper，而无需为每一个操作都添加上 self.0。
- 同时，如果不想 Wrapper 暴露底层数组的所有方法，我们还可以为 Wrapper 去重载这些方法，实现隐藏的目的。
- # 7.  集合类型
- ## 7.1.  动态数组Vector
- 动态数组允许你存储多个值，这些值在内存中一个紧挨着另一个排列，因此访问其中某个元素的成本非常低。动态数组只能存储相同类型的元素，如果你想存储不同类型的元素，可以使用之前讲过的枚举类型或者特征对象。
- 总之，当我们想拥有一个列表，里面都是相同类型的数据时，动态数组将会非常有用。
- ### 7.1.1.  创建动态数组
- 在 Rust 中，有多种方式可以创建动态数组。
- Vec::new
- 使用 Vec::new 创建动态数组是最 rusty 的方式，它调用了 Vec 中的 new 关联函数：
  
  ```
  let v: Vec<i32> = Vec::new();
  ```
- 这里，v 被显式地声明了类型 Vec<i32>，这是因为 Rust 编译器无法从 Vec::new() 中得到任何关于类型的暗示信息，因此也无法推导出 v 的具体类型，但是当你向里面增加一个元素后，一切又不同了：
  
  ```
  let mut v = Vec::new();
  v.push(1);
  ```
- 此时，v 就无需手动声明类型，因为编译器通过 v.push(1)，推测出 v 中的元素类型是 i32，因此推导出 v 的类型是 Vec<i32>。
- 如果预先知道要存储的元素个数，可以使用 Vec::with_capacity(capacity) 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能
- vec![]
- 还可以使用宏 vec! 来创建数组，与 Vec::new 有所不同，前者能在创建同时给予初始化值：
  
  ```
  let v = vec![1, 2, 3];
  ```
- 同样，此处的 v 也无需标注类型，编译器只需检查它内部的元素即可自动推导出 v 的类型是 Vec<i32> （Rust 中，整数默认类型是 i32，在数值类型中有详细介绍）。
- 更新 Vector
- 向数组尾部添加元素，可以使用 push 方法：
  
  ```
  let mut v = Vec::new();
  v.push(1);
  ```
- 与其它类型一样，必须将 v 声明为 mut 后，才能进行修改。
- ### 7.1.2.  更新 Vector
- 向数组尾部添加元素，可以使用 push 方法：
  
  ```
  let mut v = Vec::new();
  v.push(1);
  ```
- 与其它类型一样，必须将 v 声明为 mut 后，才能进行修改。
- ### 7.1.3.  Vector 与其元素共存亡
- 跟结构体一样，Vector 类型在超出作用域范围后，会被自动删除：
  
  ```
  {
    let v = vec![1, 2, 3];
  
    // ...
  } // <- v超出作用域并在此处被删除
  ```
- 当 Vector 被删除后，它内部存储的所有内容也会随之被删除。目前来看，这种解决方案简单直白，但是当 Vector 中的元素被引用后，事情可能会没那么简单。
- ### 7.1.4.  从 Vector 中读取元素
- 读取指定位置的元素有两种方式可选：
- 通过下标索引访问。
- 使用 get 方法。
  
  ```
  let v = vec![1, 2, 3, 4, 5];
  
  let third: &i32 = &v[2];
  println!("第三个元素是 {}", third);
  
  match v.get(2) {
    Some(third) => println!("第三个元素是 {third}"),
    None => println!("去你的第三个元素，根本没有！"),
  }
  ```
- 和其它语言一样，集合类型的索引下标都是从 0 开始，&v[2] 表示借用 v 中的第三个元素，最终会获得该元素的引用。而 v.get(2) 也是访问第三个元素，但是有所不同的是，它返回了 Option<&T>，因此还需要额外的 match 来匹配解构出具体的值。
- 细心的同学会注意到这里使用了两种格式化输出的方式，其中第一种我们在之前已经见过，而第二种是后续新版本中引入的写法，也是更推荐的用法，具体介绍请参见格式化输出章节。
- 下标索引与 .get 的区别
- 这两种方式都能成功的读取到指定的数组元素，既然如此为什么会存在两种方法？何况 .get 还会增加使用复杂度，这就涉及到数组越界的问题了，让我们通过示例说明：
  
  ```
  let v = vec![1, 2, 3, 4, 5];
  
  let does_not_exist = &v[100];
  let does_not_exist = v.get(100);
  ```
- 运行以上代码，&v[100] 的访问方式会导致程序无情报错退出，因为发生了数组越界访问。 但是 v.get 就不会，它在内部做了处理，有值的时候返回 Some(T)，无值的时候返回 None，因此 v.get 的使用方式非常安全。
- 既然如此，为何不统一使用 v.get 的形式？因为实在是有些啰嗦，Rust 语言的设计者和使用者在审美这方面还是相当统一的：简洁即正义，何况性能上也会有轻微的损耗。
- 既然有两个选择，肯定就有如何选择的问题，答案很简单，当你确保索引不会越界的时候，就用索引访问，否则用 .get。例如，访问第几个数组元素并不取决于我们，而是取决于用户的输入时，用 .get 会非常适合，天知道那些可爱的用户会输入一个什么样的数字进来！
- ### 7.1.5.  同时借用多个数组元素
- 既然涉及到借用数组元素，那么很可能会遇到同时借用多个数组元素的情况，还记得在所有权和借用章节咱们讲过的借用规则嘛？如果记得，就来看看下面的代码 :)
  
  ```
  let mut v = vec![1, 2, 3, 4, 5];
  
  let first = &v[0];
  
  v.push(6);
  
  println!("The first element is: {first}");
  ```
- 先不运行，来推断下结果，首先 first = &v[0] 进行了不可变借用，v.push 进行了可变借用，如果 first 在 v.push 之后不再使用，那么该段代码可以成功编译（原因见引用的作用域）。
- 可是上面的代码中，first 这个不可变借用在可变借用 v.push 后被使用了，那么妥妥的，编译器就会报错：
  
  ```
  $ cargo run
  Compiling collections v0.1.0 (file:///projects/collections)
  error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable 无法对v进行可变借用，因此之前已经进行了不可变借用
  --> src/main.rs:6:5
  |
  4 |     let first = &v[0];
  |                  - immutable borrow occurs here // 不可变借用发生在此处
  5 |
  6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here // 可变借用发生在此处
  7 |
  8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here // 不可变借用在这里被使用
  
  For more information about this error, try `rustc --explain E0502`.
  error: could not compile `collections` due to previous error
  ```
- 其实，按理来说，这两个引用不应该互相影响的：一个是查询元素，一个是在数组尾部插入元素，完全不相干的操作，为何编译器要这么严格呢？
- 原因在于：数组的大小是可变的，当旧数组的大小不够用时，Rust 会重新分配一块更大的内存空间，然后把旧数组拷贝过来。这种情况下，之前的引用显然会指向一块无效的内存，这非常 rusty —— 对用户进行严格的教育。
- ### 7.1.6.  迭代遍历 Vector 中的元素
- 如果想要依次访问数组中的元素，可以使用迭代的方式去遍历数组，这种方式比用下标的方式去遍历数组更安全也更高效（每次下标访问都会触发数组边界检查）：
  
  ```
  let v = vec![1, 2, 3];
  for i in &v {
    println!("{i}");
  }
  ```
- 也可以在迭代过程中，修改 Vector 中的元素：
  
  ```
  let mut v = vec![1, 2, 3];
  for i in &mut v {
    *i += 10
  }
  ```
- ### 7.1.7.  存储不同类型的元素
- 在本节开头，有讲到数组的元素必须类型相同，但是也提到了解决方案：那就是通过使用枚举类型和特征对象来实现不同类型元素的存储。先来看看通过枚举如何实现：
  
  ```
  #[derive(Debug)]
  enum IpAddr {
    V4(String),
    V6(String)
  }
  fn main() {
    let v = vec![
        IpAddr::V4("127.0.0.1".to_string()),
        IpAddr::V6("::1".to_string())
    ];
  
    for ip in v {
        show_addr(ip)
    }
  }
  
  fn show_addr(ip: IpAddr) {
    println!("{:?}",ip);
  }
  ```
- 数组 v 中存储了两种不同的 ip 地址，但是这两种都属于 IpAddr 枚举类型的成员，因此可以存储在数组中。
- 再来看看特征对象的实现：
  
  ```
  trait IpAddr {
    fn display(&self);
  }
  
  struct V4(String);
  impl IpAddr for V4 {
    fn display(&self) {
        println!("ipv4: {:?}",self.0)
    }
  }
  struct V6(String);
  impl IpAddr for V6 {
    fn display(&self) {
        println!("ipv6: {:?}",self.0)
    }
  }
  
  fn main() {
    let v: Vec<Box<dyn IpAddr>> = vec![
        Box::new(V4("127.0.0.1".to_string())),
        Box::new(V6("::1".to_string())),
    ];
  
    for ip in v {
        ip.display();
    }
  }
  ```
- 比枚举实现要稍微复杂一些，我们为 V4 和 V6 都实现了特征 IpAddr，然后将它俩的实例用 Box::new 包裹后，存在了数组 v 中，需要注意的是，这里必须手动地指定类型：Vec<Box<dyn IpAddr>>，表示数组 v 存储的是特征 IpAddr 的对象，这样就实现了在数组中存储不同的类型。
- ### 7.1.8.  Vector常用方法
- 初始化 vec 的更多方式：
  
  ```
  fn main() {
    let v = vec![0; 3];   // 默认值为 0，初始长度为 3
    let v_from = Vec::from([0, 0, 0]);
    assert_eq!(v, v_from);
  }
  ```
- 动态数组意味着我们增加元素时，如果容量不足就会导致 vector 扩容（目前的策略是重新申请一块 2 倍大小的内存，再将所有元素拷贝到新的内存位置，同时更新指针数据），显然，当频繁扩容或者当元素数量较多且需要扩容时，大量的内存拷贝会降低程序的性能。
- 可以考虑在初始化时就指定一个实际的预估容量，尽量减少可能的内存拷贝：
  
  ```
  fn main() {
    let mut v = Vec::with_capacity(10);
    v.extend([1, 2, 3]);    // 附加数据到 v
    println!("Vector 长度是: {}, 容量是: {}", v.len(), v.capacity());
  
    v.reserve(100);        // 调整 v 的容量，至少要有 100 的容量
    println!("Vector（reserve） 长度是: {}, 容量是: {}", v.len(), v.capacity());
  
    v.shrink_to_fit();     // 释放剩余的容量，一般情况下，不会主动去释放容量
    println!("Vector（shrink_to_fit） 长度是: {}, 容量是: {}", v.len(), v.capacity());
  }
  ```
- Vector 常见的一些方法示例：
  
  ```
  let mut v =  vec![1, 2];
  assert!(!v.is_empty());         // 检查 v 是否为空
  
  v.insert(2, 3);                 // 在指定索引插入数据，索引值不能大于 v 的长度， v: [1, 2, 3] 
  assert_eq!(v.remove(1), 2);     // 移除指定位置的元素并返回, v: [1, 3]
  assert_eq!(v.pop(), Some(3));   // 删除并返回 v 尾部的元素，v: [1]
  assert_eq!(v.pop(), Some(1));   // v: []
  assert_eq!(v.pop(), None);      // 记得 pop 方法返回的是 Option 枚举值
  v.clear();                      // 清空 v, v: []
  
  let mut v1 = [11, 22].to_vec(); // append 操作会导致 v1 清空数据，增加可变声明
  v.append(&mut v1);              // 将 v1 中的所有元素附加到 v 中, v1: []
  v.truncate(1);                  // 截断到指定长度，多余的元素被删除, v: [11]
  v.retain(|x| *x > 10);          // 保留满足条件的元素，即删除不满足条件的元素
  
  let mut v = vec![11, 22, 33, 44, 55];
  // 删除指定范围的元素，同时获取被删除元素的迭代器, v: [11, 55], m: [22, 33, 44]
  let mut m: Vec<_> = v.drain(1..=3).collect();    
  
  let v2 = m.split_off(1);        // 指定索引处切分成两个 vec, m: [22], v2: [33, 44]
  ```
- 当然也可以像数组切片的方式获取 vec 的部分元素：
  
  ```
  fn main() {
    let v = vec![11, 22, 33, 44, 55];
    let slice = &v[1..=3];
    assert_eq!(slice, &[22, 33, 44]);
  }
  ```
- ### 7.1.9.  Vector 的排序
- 在 rust 里，实现了两种排序算法，分别为稳定的排序 sort 和 sort_by，以及非稳定排序 sort_unstable 和 sort_unstable_by。
- 当然，这个所谓的 非稳定 并不是指排序算法本身不稳定，而是指在排序过程中对相等元素的处理方式。在 稳定 排序算法里，对相等的元素，不会对其进行重新排序。而在 不稳定 的算法里则不保证这点。
- 总体而言，非稳定 排序的算法的速度会优于 稳定 排序算法，同时，稳定 排序还会额外分配原数组一半的空间。
- 整数数组的排序
- 以下是对整数列进行排序的例子。
  
  ```
  fn main() {
    let mut vec = vec![1, 5, 10, 2, 15];    
    vec.sort_unstable();    
    assert_eq!(vec, vec![1, 2, 5, 10, 15]);
  }
  ```
- 浮点数数组的排序
- 我们尝试使用上面的方法来对浮点数进行排序：
  
  ```
  fn main() {
    let mut vec = vec![1.0, 5.6, 10.3, 2.0, 15f32];    
    vec.sort_unstable();    
    assert_eq!(vec, vec![1.0, 2.0, 5.6, 10.3, 15f32]);
  }
  ```
- 结果，居然报错了，
  
  ```
  error[E0277]: the trait bound `f32: Ord` is not satisfied
    --> src/main.rs:29:13
     |
  29   |         vec.sort_unstable();
     |             ^^^^^^^^^^^^^ the trait `Ord` is not implemented for `f32`
     |
     = help: the following other types implement trait `Ord`:
               i128
               i16
               i32
               i64
               i8
               isize
               u128
               u16
             and 4 others
  note: required by a bound in `core::slice::<impl [T]>::sort_unstable`
    --> /home/keijack/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/slice/mod.rs:2635:12
     |
  2635 |         T: Ord,
     |            ^^^ required by this bound in `core::slice::<impl [T]>::sort_unstable`
  
  For more information about this error, try `rustc --explain E0277`.
  ```
- 原来，在浮点数当中，存在一个 NAN 的值，这个值无法与其他的浮点数进行对比，因此，浮点数类型并没有实现全数值可比较 Ord 的特性，而是实现了部分可比较的特性 PartialOrd。
- 如此，如果我们确定在我们的浮点数数组当中，不包含 NAN 值，那么我们可以使用 partial_cmp 来作为大小判断的依据。
  
  ```
  fn main() {
    let mut vec = vec![1.0, 5.6, 10.3, 2.0, 15f32];    
    vec.sort_unstable_by(|a, b| a.partial_cmp(b).unwrap());    
    assert_eq!(vec, vec![1.0, 2.0, 5.6, 10.3, 15f32]);
  }
  ```
- OK，现在可以正确执行了。
- 对结构体数组进行排序
- 有了上述浮点数排序的经验，我们推而广之，那么对结构体是否也可以使用这种自定义对比函数的方式来进行呢？马上来试一下：
  
  ```
  #[derive(Debug)]
  struct Person {
    name: String,
    age: u32,
  }
  
  impl Person {
    fn new(name: String, age: u32) -> Person {
        Person { name, age }
    }
  }
  
  fn main() {
    let mut people = vec![
        Person::new("Zoe".to_string(), 25),
        Person::new("Al".to_string(), 60),
        Person::new("John".to_string(), 1),
    ];
    // 定义一个按照年龄倒序排序的对比函数
    people.sort_unstable_by(|a, b| b.age.cmp(&a.age));
  
    println!("{:?}", people);
  }
  ```
- 执行后输出：
  
  ```
  [Person { name: "Al", age: 60 }, Person { name: "Zoe", age: 25 }, Person { name: "John", age: 1 }]
  ```
- 结果正确。
- 从上面我们学习过程当中，排序需要我们实现 Ord 特性，那么如果我们把我们的结构体实现了该特性，是否就不需要我们自定义对比函数了呢？
- 是，但不完全是，实现 Ord 需要我们实现 Ord、Eq、PartialEq、PartialOrd 这些属性。好消息是，你可以 derive 这些属性：
  
  ```
  #[derive(Debug, Ord, Eq, PartialEq, PartialOrd)]
  struct Person {
    name: String,
    age: u32,
  }
  
  impl Person {
    fn new(name: String, age: u32) -> Person {
        Person { name, age }
    }
  }
  
  fn main() {
    let mut people = vec![
        Person::new("Zoe".to_string(), 25),
        Person::new("Al".to_string(), 60),
        Person::new("Al".to_string(), 30),
        Person::new("John".to_string(), 1),
        Person::new("John".to_string(), 25),
    ];
  
    people.sort_unstable();
  
    println!("{:?}", people);
  }
  ```
- 执行输出
  
  ```
  [Person { name: "Al", age: 30 }, Person { name: "Al", age: 60 }, Person { name: "John", age: 1 }, Person { name: "John", age: 25 }, Person { name: "Zoe", age: 25 }]
  ```
- 需要 derive Ord 相关特性，需要确保你的结构体中所有的属性均实现了 Ord 相关特性，否则会发生编译错误。derive 的默认实现会依据属性的顺序依次进行比较，如上述例子中，当 Person 的 name 值相同，则会使用 age 进行比较。
- ## 7.2.  KV存储HashMap
- 和动态数组一样，HashMap 也是 Rust 标准库中提供的集合类型，但是又与动态数组不同，HashMap 中存储的是一一映射的 KV 键值对，并提供了平均复杂度为 O(1) 的查询方法，当我们希望通过一个 Key 去查询值时，该类型非常有用，以致于 Go 语言将该类型设置成了语言级别的内置特性。
- Rust 中哈希类型（哈希映射）为 HashMap<K,V>，在其它语言中，也有类似的数据结构，例如 hash map，map，object，hash table，字典 等等。
- ### 7.2.1.  创建 HashMap
- 跟创建动态数组 Vec 的方法类似，可以使用 new 方法来创建 HashMap，然后通过 insert 方法插入键值对。
- 使用 new 方法创建
  
  ```
  use std::collections::HashMap;
  
  // 创建一个HashMap，用于存储宝石种类和对应的数量
  let mut my_gems = HashMap::new();
  
  // 将宝石类型和对应的数量写入表中
  my_gems.insert("红宝石", 1);
  my_gems.insert("蓝宝石", 2);
  my_gems.insert("河边捡的误以为是宝石的破石头", 18);
  ```
- 很简单对吧？跟其它语言没有区别，聪明的同学甚至能够猜到该 HashMap 的类型：HashMap<&str,i32>。
- 但是还有一点，你可能没有注意，那就是使用 HashMap 需要手动通过 use ... 从标准库中引入到我们当前的作用域中来，仔细回忆下，之前使用另外两个集合类型 String 和 Vec 时，我们是否有手动引用过？答案是 No，因为 HashMap 并没有包含在 Rust 的 prelude 中（Rust 为了简化用户使用，提前将最常用的类型自动引入到作用域中）。
- 所有的集合类型都是动态的，意味着它们没有固定的内存大小，因此它们底层的数据都存储在内存堆上，然后通过一个存储在栈中的引用类型来访问。同时，跟其它集合类型一致，HashMap 也是内聚性的，即所有的 K 必须拥有同样的类型，V 也是如此。
- 跟 Vec 一样，如果预先知道要存储的 KV 对个数，可以使用 HashMap::with_capacity(capacity) 创建指定大小的 HashMap，避免频繁的内存分配和拷贝，提升性能。
- 使用迭代器和 collect 方法创建
- 在实际使用中，不是所有的场景都能 new 一个哈希表后，然后悠哉悠哉的依次插入对应的键值对，而是可能会从另外一个数据结构中，获取到对应的数据，最终生成 HashMap。
- 例如考虑一个场景，有一张表格中记录了足球联赛中各队伍名称和积分的信息，这张表如果被导入到 Rust 项目中，一个合理的数据结构是 Vec<(String, u32)> 类型，该数组中的元素是一个个元组，该数据结构跟表格数据非常契合：表格中的数据都是逐行存储，每一个行都存有一个 (队伍名称, 积分) 的信息。
- 但是在很多时候，又需要通过队伍名称来查询对应的积分，此时动态数组就不适用了，因此可以用 HashMap 来保存相关的队伍名称 -> 积分映射关系。 理想很丰满，现实很骨感，如何将 Vec<(String, u32)> 中的数据快速写入到 HashMap<String, u32> 中？
- 一个动动脚趾头就能想到的笨方法如下：
  
  ```
  fn main() {
    use std::collections::HashMap;
  
    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];
  
    let mut teams_map = HashMap::new();
    for team in &teams_list {
        teams_map.insert(&team.0, team.1);
    }
  
    println!("{:?}",teams_map)
  }
  ```
- 遍历列表，将每一个元组作为一对 KV 插入到 HashMap 中，很简单，但是……也不太聪明的样子，换个词说就是 —— 不够 rusty。
- 好在，Rust 为我们提供了一个非常精妙的解决办法：先将 Vec 转为迭代器，接着通过 collect 方法，将迭代器中的元素收集后，转成 HashMap：
  
  ```
  fn main() {
    use std::collections::HashMap;
  
    let teams_list = vec![
        ("中国队".to_string(), 100),
        ("美国队".to_string(), 10),
        ("日本队".to_string(), 50),
    ];
  
    let teams_map: HashMap<_,_> = teams_list.into_iter().collect();
    
    println!("{:?}",teams_map)
  }
  ```
- 代码很简单，into_iter 方法将列表转为迭代器，接着通过 collect 进行收集，不过需要注意的是，collect 方法在内部实际上支持生成多种类型的目标集合，因此我们需要通过类型标注 HashMap<_,_> 来告诉编译器：请帮我们收集为 HashMap 集合类型，具体的 KV 类型，麻烦编译器您老人家帮我们推导。
- 由此可见，Rust 中的编译器时而小聪明，时而大聪明，不过好在，它大聪明的时候，会自家人知道自己事，总归会通知你一声：
  
  ```
  error[E0282]: type annotations needed // 需要类型标注
  --> src/main.rs:10:9
   |
  10 |     let teams_map = teams_list.into_iter().collect();
   |         ^^^^^^^^^ consider giving `teams_map` a type // 给予 `teams_map` 一个具体的类型
  ```
- ### 7.2.2.  所有权转移
- HashMap 的所有权规则与其它 Rust 类型没有区别：
- 若类型实现 Copy 特征，该类型会被复制进 HashMap，因此无所谓所有权
- 若没实现 Copy 特征，所有权将被转移给 HashMap 中
- 例如我参选帅气男孩时的场景再现：
  
  ```
  fn main() {
    use std::collections::HashMap;
  
    let name = String::from("Sunface");
    let age = 18;
  
    let mut handsome_boys = HashMap::new();
    handsome_boys.insert(name, age);
  
    println!("因为过于无耻，{}已经被从帅气男孩名单中除名", name);
    println!("还有，他的真实年龄远远不止{}岁", age);
  }
  ```
- 运行代码，报错如下：
  
  ```
  error[E0382]: borrow of moved value: `name`
  --> src/main.rs:10:32
   |
  4  |     let name = String::from("Sunface");
   |         ---- move occurs because `name` has type `String`, which does not implement the `Copy` trait
  ...
  8  |     handsome_boys.insert(name, age);
   |                          ---- value moved here
  9  |
  10 |     println!("因为过于无耻，{}已经被除名", name);
   |                                            ^^^^ value borrowed here after move
  ```
- 提示很清晰，name 是 String 类型，因此它受到所有权的限制，在 insert 时，它的所有权被转移给 handsome_boys，所以最后在使用时，会遇到这个无情但是意料之中的报错。
- 如果你使用引用类型放入 HashMap 中，请确保该引用的生命周期至少跟 HashMap 活得一样久：
  
  ```
  fn main() {
    use std::collections::HashMap;
  
    let name = String::from("Sunface");
    let age = 18;
  
    let mut handsome_boys = HashMap::new();
    handsome_boys.insert(&name, age);
  
    std::mem::drop(name);
    println!("因为过于无耻，{:?}已经被除名", handsome_boys);
    println!("还有，他的真实年龄远远不止{}岁", age);
  }
  ```
- 上面代码，我们借用 name 获取了它的引用，然后插入到 handsome_boys 中，至此一切都很完美。但是紧接着，就通过 drop 函数手动将 name 字符串从内存中移除，再然后就报错了：
  
  ```
  handsome_boys.insert(&name, age);
   |                          ----- borrow of `name` occurs here // name借用发生在此处
  9  |
  10 |     std::mem::drop(name);
   |                    ^^^^ move out of `name` occurs here // name的所有权被转移走
  11 |     println!("因为过于无耻，{:?}已经被除名", handsome_boys);
   |                                              ------------- borrow later used here // 所
  ```
- ### 7.2.3.  查询HashMap
- 通过 get 方法可以获取元素：
  
  ```
  use std::collections::HashMap;
  
  let mut scores = HashMap::new();
  
  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Yellow"), 50);
  
  let team_name = String::from("Blue");
  let score: Option<&i32> = scores.get(&team_name);
  ```
- 上面有几点需要注意：
- get 方法返回一个 Option<&i32> 类型：当查询不到时，会返回一个 None，查询到时返回 Some(&i32)
- &i32 是对 HashMap 中值的借用，如果不使用借用，可能会发生所有权的转移
- 还可以继续拓展下，上面的代码中，如果我们想直接获得值类型的 score 该怎么办，答案简约但不简单:
  
  ```
  let score: i32 = scores.get(&team_name).copied().unwrap_or(0);
  ```
- 这里留给大家一个小作业: 去官方文档中查询下 Option 的 copied 方法和 unwrap_or 方法的含义及该如何使用。
- 还可以通过循环的方式依次遍历 KV 对：
  
  ```
  use std::collections::HashMap;
  
  let mut scores = HashMap::new();
  
  scores.insert(String::from("Blue"), 10);
  scores.insert(String::from("Yellow"), 50);
  
  for (key, value) in &scores {
    println!("{}: {}", key, value);
  }
  ```
- 最终输出：
  
  ```
  Yellow: 50
  Blue: 10
  ```
- ### 7.2.4.  更新HashMap的值
- 更新值的时候，涉及多种情况，咱们在代码中一一进行说明：
  
  ```
  fn main() {
    use std::collections::HashMap;
  
    let mut scores = HashMap::new();
  
    scores.insert("Blue", 10);
  
    // 覆盖已有的值
    let old = scores.insert("Blue", 20);
    assert_eq!(old, Some(10));
  
    // 查询新插入的值
    let new = scores.get("Blue");
    assert_eq!(new, Some(&20));
  
    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(5);
    assert_eq!(*v, 5); // 不存在，插入5
  
    // 查询Yellow对应的值，若不存在则插入新值
    let v = scores.entry("Yellow").or_insert(50);
    assert_eq!(*v, 5); // 已经存在，因此50没有插入
  }
  ```
- 具体的解释在代码注释中已有，这里不再进行赘述。
- 在已有值的基础上更新
	- 另一个常用场景如下：查询某个 key 对应的值，若不存在则插入新值，若存在则对已有的值进行更新，例如在文本中统计词语出现的次数：
	  
	  ```
	  use std::collections::HashMap;
	  
	  let text = "hello world wonderful world";
	  
	  let mut map = HashMap::new();
	  // 根据空格来切分字符串(英文单词都是通过空格切分)
	  for word in text.split_whitespace() {
	   let count = map.entry(word).or_insert(0);
	   *count += 1;
	  }
	  
	  println!("{:?}", map);
	  ```
- 上面代码中，新建一个 map 用于保存词语出现的次数，插入一个词语时会进行判断：若之前没有插入过，则使用该词语作 Key，插入次数 0 作为 Value，若之前插入过则取出之前统计的该词语出现的次数，对其加一。
- 有两点值得注意：
	- or_insert 返回了 &mut v 引用，因此可以通过该可变引用直接修改 map 中对应的值
	- 使用 count 引用时，需要先进行解引用 *count，否则会出现类型不匹配
- ### 7.2.5.  哈希函数
- 为何叫哈希表，到底什么是哈希。
- 先来设想下，如果要实现 Key 与 Value 的一一对应，是不是意味着我们要能比较两个 Key 的相等性？例如 "a" 和 "b"，1 和 2，当这些类型做 Key 且能比较时，可以很容易知道 1 对应的值不会错误的映射到 2 上，因为 1 不等于 2。因此，一个类型能否作为 Key 的关键就是是否能进行相等比较，或者说该类型是否实现了 std::cmp::Eq 特征。
  
  f32 和 f64 浮点数，没有实现 std::cmp::Eq 特征，因此不可以用作 HashMap 的 Key。
- 好了，理解完这个，再来设想一点，若一个复杂点的类型作为 Key，那怎么在底层对它进行存储，怎么使用它进行查询和比较？ 是不是很棘手？好在我们有哈希函数：通过它把 Key 计算后映射为哈希值，然后使用该哈希值来进行存储、查询、比较等操作。
- 但是问题又来了，如何保证不同 Key 通过哈希后的两个值不会相同？如果相同，那意味着我们使用不同的 Key，却查到了同一个结果，这种明显是错误的行为。 此时，就涉及到安全性跟性能的取舍了。
- 若要追求安全，尽可能减少冲突，同时防止拒绝服务（Denial of Service, DoS）攻击，就要使用密码学安全的哈希函数，HashMap 就是使用了这样的哈希函数。反之若要追求性能，就需要使用没有那么安全的算法。
- 高性能三方库
	- 因此若性能测试显示当前标准库默认的哈希函数不能满足你的性能需求，就需要去 crates.io 上寻找其它的哈希函数实现，使用方法很简单：
	  
	  ```
	  use std::hash::BuildHasherDefault;
	  use std::collections::HashMap;
	  // 引入第三方的哈希函数
	  use twox_hash::XxHash64;
	  
	  // 指定HashMap使用第三方的哈希函数XxHash64
	  let mut hash: HashMap<_, _, BuildHasherDefault<XxHash64>> = Default::default();
	  hash.insert(42, "the answer");
	  assert_eq!(hash.get(&42), Some(&"the answer"));
	  ```
- 目前，HashMap 使用的哈希函数是 SipHash，它的性能不是很高，但是安全性很高。SipHash 在中等大小的 Key 上，性能相当不错，但是对于小型的 Key （例如整数）或者大型 Key （例如字符串）来说，性能还是不够好。若你需要极致性能，例如实现算法，可以考虑这个库：ahash。
- # 8.  认识生命周期
- 生命周期，简而言之就是引用的有效作用域。在大多数时候，我们无需手动的声明生命周期，因为编译器可以自动进行推导，用类型来类比下：
	- 就像编译器大部分时候可以自动推导类型 <-> 一样，编译器大多数时候也可以自动推导生命周期
	- 在多种类型存在时，编译器往往要求我们手动标明类型 <-> 当多个生命周期存在，且编译器无法推导出某个引用的生命周期时，就需要我们手动标明生命周期
- Rust 生命周期之所以难，是因为这个概念对于我们来说是全新的，没有其它编程语言的经验可以借鉴。当你觉得难的时候，不用过于担心，这个难对于所有人都是平等的，多点付出就能早点解决此拦路虎，同时本书也会尽力帮助大家减少学习难度(生命周期很可能是 Rust 中最难的部分)。
- ## 8.1.  悬垂指针和生命周期
- 生命周期的主要作用是避免悬垂引用，它会导致程序引用了本不该引用的数据：
  
  ```
  {
    let r;
  
    {
        let x = 5;
        r = &x;
    }
  
    println!("r: {}", r);
  }
  ```
- 这段代码有几点值得注意:
	- let r; 的声明方式貌似存在使用 null 的风险，实际上，当我们不初始化它就使用时，编译器会给予报错
	- r 引用了内部花括号中的 x 变量，但是 x 会在内部花括号 } 处被释放，因此回到外部花括号后，r 会引用一个无效的 x
- 此处 r 就是一个悬垂指针，它引用了提前被释放的变量 x，可以预料到，这段代码会报错：
  
  ```
  error[E0597]: `x` does not live long enough // `x` 活得不够久
  --> src/main.rs:7:17
   |
  7  |             r = &x;
   |                 ^^ borrowed value does not live long enough // 被借用的 `x` 活得不够久
  8  |         }
   |         - `x` dropped here while still borrowed // `x` 在这里被丢弃，但是它依然还在被借用
  9  |
  10 |         println!("r: {}", r);
   |                           - borrow later used here // 对 `x` 的借用在此处被使用
  ```
- 在这里 r 拥有更大的作用域，或者说活得更久。如果 Rust 不阻止该悬垂引用的发生，那么当 x 被释放后，r 所引用的值就不再是合法的，会导致我们程序发生异常行为，且该异常行为有时候会很难被发现。
- ## 8.2.  借用检查
- 为了保证 Rust 的所有权和借用的正确性，Rust 使用了一个借用检查器(Borrow checker)，来检查我们程序的借用正确性：
  
  ```
  {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
  }                         // ---------+
  ```
- 这段代码和之前的一模一样，唯一的区别在于增加了对变量生命周期的注释。这里，r 变量被赋予了生命周期 'a，x 被赋予了生命周期 'b，从图示上可以明显看出生命周期 'b 比 'a 小很多。
- 在编译期，Rust 会比较两个变量的生命周期，结果发现 r 明明拥有生命周期 'a，但是却引用了一个小得多的生命周期 'b，在这种情况下，编译器会认为我们的程序存在风险，因此拒绝运行。
- 如果想要编译通过，也很简单，只要 'b 比 'a 大就好。总之，x 变量只要比 r 活得久，那么 r 就能随意引用 x 且不会存在危险：
  
  ```
  {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
  }                         // ----------+
  ```
- 根据之前的结论，我们重新实现了代码，现在 x 的生命周期 'b 大于 r 的生命周期 'a，因此 r 对 x 的引用是安全的。
- 通过之前的内容，我们了解了何为生命周期，也了解了 Rust 如何利用生命周期来确保引用是合法的，下面来看看函数中的生命周期。
- ## 8.3.  函数中的生命周期
- 先来考虑一个例子 - 返回两个字符串切片中较长的那个，该函数的参数是两个字符串切片，返回值也是字符串切片：
  
  ```
  fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
  
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
  }
  fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
  }
  ```
- 这段 longest 实现，非常标准优美，就连多余的 return 和分号都没有，可是现实总是给我们重重一击：
  
  ```
  error[E0106]: missing lifetime specifier
  --> src/main.rs:9:33
  |
  9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter // 参数需要一个生命周期
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is
  borrowed from `x` or `y`
  = 帮助： 该函数的返回值是一个引用类型，但是函数签名无法说明，该引用是借用自 `x` 还是 `y`
  help: consider introducing a named lifetime parameter // 考虑引入一个生命周期
  |
  9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ^^^^    ^^^^^^^     ^^^^^^^     ^^^
  ```
- 喔，这真是一个复杂的提示，那感觉就好像是生命周期去非诚勿扰相亲，结果在初印象环节就 23 盏灯全灭。等等，先别急，如果你仔细阅读，就会发现，其实主要是编译器无法知道该函数的返回值到底引用 x 还是 y ，因为编译器需要知道这些，来确保函数调用后的引用生命周期分析。
- 不过说来尴尬，就这个函数而言，我们也不知道返回值到底引用哪个，因为一个分支返回 x，另一个分支返回 y...这可咋办？先来分析下。
- 我们在定义该函数时，首先无法知道传递给函数的具体值，因此到底是 if 还是 else 被执行，无从得知。其次，传入引用的具体生命周期也无法知道，因此也不能像之前的例子那样通过分析生命周期来确定引用是否有效。同时，编译器的借用检查也无法推导出返回值的生命周期，因为它不知道 x 和 y 的生命周期跟返回值的生命周期之间的关系是怎样的(说实话，人都搞不清，何况编译器这个大聪明)。
- 因此，这时就回到了文章开头说的内容：在存在多个引用时，编译器有时会无法自动推导生命周期，此时就需要我们手动去标注，通过为参数标注合适的生命周期来帮助编译器进行借用检查的分析。
- ## 8.4.  生命周期标注语法
- 生命周期标注并不会改变任何引用的实际作用域 -- 鲁迅
- 鲁迅说过的话，总是值得重点标注，当你未来更加理解生命周期时，你才会发现这句话的精髓和重要！现在先简单记住，标记的生命周期只是为了取悦编译器，让编译器不要难为我们，记住了吗？没记住，再回头看一遍，这对未来你遇到生命周期问题时会有很大的帮助！
- 在很多时候编译器是很聪明的，但是总有些时候，它会化身大聪明，自以为什么都很懂，然后去拒绝我们代码的执行，此时，就需要我们通过生命周期标注来告诉这个大聪明：别自作聪明了，听我的就好。
- 例如一个变量，只能活一个花括号，那么就算你给它标注一个活全局的生命周期，它还是会在前面的花括号结束处被释放掉，并不会真的全局存活。
- 生命周期的语法也颇为与众不同，以 ' 开头，名称往往是一个单独的小写字母，大多数人都用 'a 来作为生命周期的名称。 如果是引用类型的参数，那么生命周期会位于引用符号 & 之后，并用一个空格来将生命周期和引用参数分隔开:
  
  ```
  &i32        // 一个引用
  &'a i32     // 具有显式生命周期的引用
  &'a mut i32 // 具有显式生命周期的可变引用
  ```
- 一个生命周期标注，它自身并不具有什么意义，因为生命周期的作用就是告诉编译器多个引用之间的关系。例如，有一个函数，它的第一个参数 first 是一个指向 i32 类型的引用，具有生命周期 'a，该函数还有另一个参数 second，它也是指向 i32 类型的引用，并且同样具有生命周期 'a。此处生命周期标注仅仅说明，这两个参数 first 和 second 至少活得和'a 一样久，至于到底活多久或者哪个活得更久，抱歉我们都无法得知：
  
  ```
  fn useless<'a>(first: &'a i32, second: &'a i32) {}
  ```
- ### 8.4.1.  函数签名中的生命周期标注
- 继续之前的 longest 函数，从两个字符串切片中返回较长的那个：
  
  ```
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
  }
  ```
- 需要注意的点如下：
	- 和泛型一样，使用生命周期参数，需要先声明 <'a>
	- x、y 和返回值至少活得和 'a 一样久(因为返回值要么是 x，要么是 y)
- 该函数签名表明对于某些生命周期 'a，函数的两个参数都至少跟 'a 活得一样久，同时函数的返回引用也至少跟 'a 活得一样久。实际上，这意味着返回值的生命周期与参数生命周期中的较小值一致：虽然两个参数的生命周期都是标注了 'a，但是实际上这两个参数的真实生命周期可能是不一样的(生命周期 'a 不代表生命周期等于 'a，而是大于等于 'a)。
- 回忆下“鲁迅”说的话，再参考上面的内容，可以得出：在通过函数签名指定生命周期参数时，我们并没有改变传入引用或者返回引用的真实生命周期，而是告诉编译器当不满足此约束条件时，就拒绝编译通过。
- 因此 longest 函数并不知道 x 和 y 具体会活多久，只要知道它们的作用域至少能持续 'a 这么长就行。
- 当把具体的引用传给 longest 时，那生命周期 'a 的大小就是 x 和 y 的作用域的重合部分，换句话说，'a 的大小将等于 x 和 y 中较小的那个。由于返回值的生命周期也被标记为 'a，因此返回值的生命周期也是 x 和 y 中作用域较小的那个。
- 说实话，这段文字我写的都快崩溃了，不知道你们读起来如何，实在***太绕了。。那就干脆用一个例子来解释吧：
  
  ```
  fn main() {
    let string1 = String::from("long string is long");
  
    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
  }
  ```
- 在上例中，string1 的作用域直到 main 函数的结束，而 string2 的作用域到内部花括号的结束 }，那么根据之前的理论，'a 是两者中作用域较小的那个，也就是 'a 的生命周期等于 string2 的生命周期，同理，由于函数返回的生命周期也是 'a，可以得出函数返回的生命周期也等于 string2 的生命周期。
- 现在来验证下上面的结论：result 的生命周期等于参数中生命周期最小的，因此要等于 string2 的生命周期，也就是说，result 要活得和 string2 一样久，观察下代码的实现，可以发现这个结论是正确的！
- 因此，在这种情况下，通过生命周期标注，编译器得出了和我们肉眼观察一样的结论，而不再是一个蒙圈的大聪明。
- 再来看一个例子，该例子证明了 result 的生命周期必须等于两个参数中生命周期较小的那个:
  
  ```
  fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
  }
  ```
- Bang，错误冒头了：
  
  ```
  error[E0597]: `string2` does not live long enough
  --> src/main.rs:6:44
  |
  6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^ borrowed value does not live long enough
  7 |     }
  |     - `string2` dropped here while still borrowed
  8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here
  ```
- 在上述代码中，result 必须要活到 println!处，因为 result 的生命周期是 'a，因此 'a 必须持续到 println!。
- 在 longest 函数中，string2 的生命周期也是 'a，由此说明 string2 也必须活到 println! 处，可是 string2 在代码中实际上只能活到内部语句块的花括号处 }，小于它应该具备的生命周期 'a，因此编译出错。
- 作为人类，我们可以很清晰的看出 result 实际上引用了 string1，因为 string1 的长度明显要比 string2 长，既然如此，编译器不该如此矫情才对，它应该能认识到 result 没有引用 string2，让我们这段代码通过。只能说，作为尊贵的人类，编译器的发明者，你高估了这个工具的能力，它真的做不到！而且 Rust 编译器在调教上是非常保守的：当可能出错也可能不出错时，它会选择前者，抛出编译错误。
- 总之，显式的使用生命周期，可以让编译器正确的认识到多个引用之间的关系，最终帮我们提前规避可能存在的代码风险。
- ### 8.4.2.  深入思考生命周期标注
- 使用生命周期的方式往往取决于函数的功能，例如之前的 longest 函数，如果它永远只返回第一个参数 x，生命周期的标注该如何修改(该例子就是上面的小练习结果之一)?
  
  ```
  fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
  }
  ```
- 在此例中，y 完全没有被使用，因此 y 的生命周期与 x 和返回值的生命周期没有任何关系，意味着我们也不必再为 y 标注生命周期，只需要标注 x 参数和返回值即可。
- 函数的返回值如果是一个引用类型，那么它的生命周期只会来源于：
- 函数参数的生命周期
- 函数体中某个新建引用的生命周期
- 若是后者情况，就是典型的悬垂引用场景：
  
  ```
  fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
  }
  ```
- 上面的函数的返回值就和参数 x，y 没有任何关系，而是引用了函数体内创建的字符串，那么很显然，该函数会报错：
  
  ```
  error[E0515]: cannot return value referencing local variable `result` // 返回值result引用了本地的变量
  --> src/main.rs:11:5
   |
  11 |     result.as_str()
   |     ------^^^^^^^^^
   |     |
   |     returns a value referencing data owned by the current function
   |     `result` is borrowed here
  ```
- 主要问题就在于，result 在函数结束后就被释放，但是在函数结束后，对 result 的引用依然在继续。在这种情况下，没有办法指定合适的生命周期来让编译通过，因此我们也就在 Rust 中避免了悬垂引用。
- 那遇到这种情况该怎么办？最好的办法就是返回内部字符串的所有权，然后把字符串的所有权转移给调用者：
  
  ```
  fn longest<'a>(_x: &str, _y: &str) -> String {
    String::from("really long string")
  }
  
  fn main() {
   let s = longest("not", "important");
  }
  ```
- 至此，可以对生命周期进行下总结：生命周期语法用来将函数的多个引用参数和返回值的作用域关联到一起，一旦关联到一起后，Rust 就拥有充分的信息来确保我们的操作是内存安全的。
- ## 8.5.  结构体中的生命周期
- 不仅仅函数具有生命周期，结构体其实也有这个概念，只不过我们之前对结构体的使用都停留在非引用类型字段上。细心的同学应该能回想起来，之前为什么不在结构体中使用字符串字面量或者字符串切片，而是统一使用 String 类型？原因很简单，后者在结构体初始化时，只要转移所有权即可，而前者，抱歉，它们是引用，它们不能为所欲为。
- 既然之前已经理解了生命周期，那么意味着在结构体中使用引用也变得可能：只要为结构体中的每一个引用标注上生命周期即可：
  
  ```
  struct ImportantExcerpt<'a> {
    part: &'a str,
  }
  
  fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
  }
  ```
- ImportantExcerpt 结构体中有一个引用类型的字段 part，因此需要为它标注上生命周期。结构体的生命周期标注语法跟泛型参数语法很像，需要对生命周期参数进行声明 <'a>。该生命周期标注说明，结构体 ImportantExcerpt 所引用的字符串 str 必须比该结构体活得更久。
- 从 main 函数实现来看，ImportantExcerpt 的生命周期从第 4 行开始，到 main 函数末尾结束，而该结构体引用的字符串从第一行开始，也是到 main 函数末尾结束，可以得出结论结构体引用的字符串活得比结构体久，这符合了编译器对生命周期的要求，因此编译通过。
- 与之相反，下面的代码就无法通过编译：
  
  ```
  #[derive(Debug)]
  struct ImportantExcerpt<'a> {
    part: &'a str,
  }
  
  fn main() {
    let i;
    {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.').next().expect("Could not find a '.'");
        i = ImportantExcerpt {
            part: first_sentence,
        };
    }
    println!("{:?}",i);
  }
  ```
- 观察代码，可以看出结构体比它引用的字符串活得更久，引用字符串在内部语句块末尾 } 被释放后，println! 依然在外面使用了该结构体，因此会导致无效的引用，不出所料，编译报错：
  
  ```
  error[E0597]: `novel` does not live long enough
  --> src/main.rs:10:30
   |
  10 |         let first_sentence = novel.split('.').next().expect("Could not find a '.'");
   |                              ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
  ...
  14 |     }
   |     - `novel` dropped here while still borrowed
  15 |     println!("{:?}",i);
   |                     - borrow later used here
  ```
- ## 8.6.  生命周期消除
- 实际上，对于编译器来说，每一个引用类型都有一个生命周期，那么为什么我们在使用过程中，很多时候无需标注生命周期？例如：
  
  ```
  fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
  
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
  
    &s[..]
  }
  ```
- 该函数的参数和返回值都是引用类型，尽管我们没有显式的为其标注生命周期，编译依然可以通过。其实原因不复杂，编译器为了简化用户的使用，运用了生命周期消除大法。
- 对于 first_word 函数，它的返回值是一个引用类型，那么该引用只有两种情况：
	- 从参数获取
	- 从函数体内部新创建的变量获取
- 如果是后者，就会出现悬垂引用，最终被编译器拒绝，因此只剩一种情况：返回值的引用是获取自参数，这就意味着参数和返回值的生命周期是一样的。道理很简单，我们能看出来，编译器自然也能看出来，因此，就算我们不标注生命周期，也不会产生歧义。
- 实际上，在 Rust 1.0 版本之前，这种代码果断不给通过，因为 Rust 要求必须显式的为所有引用标注生命周期：
  
  ```
  fn first_word<'a>(s: &'a str) -> &'a str {
  ```
- 在写了大量的类似代码后，Rust 社区抱怨声四起，包括开发者自己都忍不了了，最终揭锅而起，这才有了我们今日的幸福。
- 生命周期消除的规则不是一蹴而就，而是伴随着 总结-改善 流程的周而复始，一步一步走到今天，这也意味着，该规则以后可能也会进一步增加，我们需要手动标注生命周期的时候也会越来越少，hooray!
- 在开始之前有几点需要注意：
	- 消除规则不是万能的，若编译器不能确定某件事是正确时，会直接判为不正确，那么你还是需要手动标注生命周期
	- 函数或者方法中，参数的生命周期被称为 输入生命周期，返回值的生命周期被称为 输出生命周期
- ### 8.6.1.  三条消除规则
- 编译器使用三条消除规则来确定哪些场景不需要显式地去标注生命周期。其中第一条规则应用在输入生命周期上，第二、三条应用在输出生命周期上。若编译器发现三条规则都不适用时，就会报错，提示你需要手动标注生命周期。
- 每一个引用参数都会获得独自的生命周期
- 例如一个引用参数的函数就有一个生命周期标注: fn foo<'a>(x: &'a i32)，两个引用参数的有两个生命周期标注:fn foo<'a, 'b>(x: &'a i32, y: &'b i32), 依此类推。
- 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期
- 例如函数 fn foo(x: &i32) -> &i32，x 参数的生命周期会被自动赋给返回值 &i32，因此该函数等同于 fn foo<'a>(x: &'a i32) -> &'a i32
- 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
- 拥有 &self 形式的参数，说明该函数是一个 方法，该规则让方法的使用便利度大幅提升。
- 规则其实很好理解，但是，爱思考的读者肯定要发问了，例如第三条规则，若一个方法，它的返回值的生命周期就是跟参数 &self 的不一样怎么办？总不能强迫我返回的值总是和 &self 活得一样久吧？! 问得好，答案很简单：手动标注生命周期，因为这些规则只是编译器发现你没有标注生命周期时默认去使用的，当你标注生命周期后，编译器自然会乖乖听你的话。
- 让我们假装自己是编译器，然后看下以下的函数该如何应用这些规则：
	- 例子 1
	  
	  ```
	  fn first_word(s: &str) -> &str { // 实际项目中的手写代码
	  ```
- 首先，我们手写的代码如上所示时，编译器会先应用第一条规则，为每个参数标注一个生命周期：
  
  ```
  fn first_word<'a>(s: &'a str) -> &str { // 编译器自动为参数添加生命周期
  ```
- 此时，第二条规则就可以进行应用，因为函数只有一个输入生命周期，因此该生命周期会被赋予所有的输出生命周期：
  
  ```
  fn first_word<'a>(s: &'a str) -> &'a str { // 编译器自动为返回值添加生命周期
  ```
- 此时，编译器为函数签名中的所有引用都自动添加了具体的生命周期，因此编译通过，且用户无需手动去标注生命周期，只要按照 fn first_word(s: &str) -> &str { 的形式写代码即可。
- 例子 2 再来看一个例子：
  
  ```
  fn longest(x: &str, y: &str) -> &str { // 实际项目中的手写代码
  ```
- 首先，编译器会应用第一条规则，为每个参数都标注生命周期：
  
  ```
  fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
  ```
- 但是此时，第二条规则却无法被使用，因为输入生命周期有两个，第三条规则也不符合，因为它是函数，不是方法，因此没有 &self 参数。在套用所有规则后，编译器依然无法为返回值标注合适的生命周期，因此，编译器就会报错，提示我们需要手动标注生命周期：
  
  ```
  error[E0106]: missing lifetime specifier
  --> src/main.rs:1:47
  |
  1 | fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
  |                       -------     -------     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
  note: these named lifetimes are available to use
  --> src/main.rs:1:12
  |
  1 | fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
  |            ^^  ^^
  help: consider using one of the available lifetimes here
  |
  1 | fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &'lifetime str {
  |                                                +++++++++
  ```
- 不得不说，Rust 编译器真的很强大，还贴心的给我们提示了该如何修改，虽然……好像……。它的提示貌似不太准确。这里我们更希望参数和返回值都是 'a 生命周期。
- ## 8.7.  方法中的生命周期
- 先来回忆下泛型的语法：
  
  ```
  struct Point<T> {
    x: T,
    y: T,
  }
  
  impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
  }
  ```
- 实际上，为具有生命周期的结构体实现方法时，我们使用的语法跟泛型参数语法很相似：
  
  ```
  struct ImportantExcerpt<'a> {
    part: &'a str,
  }
  
  impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
  }
  ```
- 其中有几点需要注意的：
	- impl 中必须使用结构体的完整名称，包括 <'a>，因为生命周期标注也是结构体类型的一部分！
	- 方法签名中，往往不需要标注生命周期，得益于生命周期消除的第一和第三规则
- 下面的例子展示了第三规则应用的场景：
  
  ```
  impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```
- 首先，编译器应用第一规则，给予每个输入参数一个生命周期:
  
  ```
  impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```
- 需要注意的是，编译器不知道 announcement 的生命周期到底多长，因此它无法简单的给予它生命周期 'a，而是重新声明了一个全新的生命周期 'b。
- 接着，编译器应用第三规则，将 &self 的生命周期赋给返回值 &str：
  
  ```
  impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'a str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```
- Bingo，最开始的代码，尽管我们没有给方法标注生命周期，但是在第一和第三规则的配合下，编译器依然完美的为我们亮起了绿灯。
- 在结束这块儿内容之前，再来做一个有趣的修改，将方法返回的生命周期改为'b：
  
  ```
  impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```
- 此时，编译器会报错，因为编译器无法知道 'a 和 'b 的关系。 &self 生命周期是 'a，那么 self.part 的生命周期也是 'a，但是好巧不巧的是，我们手动为返回值 self.part 标注了生命周期 'b，因此编译器需要知道 'a 和 'b 的关系。
- 有一点很容易推理出来：由于 &'a self 是被引用的一方，因此引用它的 &'b str 必须要活得比它短，否则会出现悬垂引用。因此说明生命周期 'b 必须要比 'a 小，只要满足了这一点，编译器就不会再报错：
  
  ```
  impl<'a: 'b, 'b> ImportantExcerpt<'a> {
    fn announce_and_return_part(&'a self, announcement: &'b str) -> &'b str {
        println!("Attention please: {}", announcement);
        self.part
    }
  }
  ```
- Bang，一个复杂的玩意儿被甩到了你面前，就问怕不怕？
- 就关键点稍微解释下：
	- 'a: 'b，是生命周期约束语法，跟泛型约束非常相似，用于说明 'a 必须比 'b 活得久
	- 可以把 'a 和 'b 都在同一个地方声明（如上），或者分开声明但通过 where 'a: 'b 约束生命周期关系，如下：
	  
	  ```
	  impl<'a> ImportantExcerpt<'a> {
	   fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str
	   where
	       'a: 'b,
	   {
	       println!("Attention please: {}", announcement);
	       self.part
	   }
	  }
	  ```
- 总之，实现方法比想象中简单：加一个约束，就能暗示编译器，尽管引用吧，反正我想引用的内容比我活得久，爱咋咋地，我怎么都不会引用到无效的内容！
- ## 8.8.  静态生命周期
- 在 Rust 中有一个非常特殊的生命周期，那就是 'static，拥有该生命周期的引用可以和整个程序活得一样久。
- 在之前我们学过字符串字面量，提到过它是被硬编码进 Rust 的二进制文件中，因此这些字符串变量全部具有 'static 的生命周期：
  
  ```
  let s: &'static str = "我没啥优点，就是活得久，嘿嘿";
  ```
- 这时候，有些聪明的小脑瓜就开始开动了：当生命周期不知道怎么标时，对类型施加一个静态生命周期的约束 T: 'static 是不是很爽？这样我和编译器再也不用操心它到底活多久了。
- 嗯，只能说，这个想法是对的，在不少情况下，'static 约束确实可以解决生命周期编译不通过的问题，但是问题来了：本来该引用没有活那么久，但是你非要说它活那么久，万一引入了潜在的 BUG 怎么办？
- 因此，遇到因为生命周期导致的编译不通过问题，首先想的应该是：是否是我们试图创建一个悬垂引用，或者是试图匹配不一致的生命周期，而不是简单粗暴的用 'static 来解决问题。
- 但是，话说回来，存在即合理，有时候，'static 确实可以帮助我们解决非常复杂的生命周期问题甚至是无法被手动解决的生命周期问题，那么此时就应该放心大胆的用，只要你确定：你的所有引用的生命周期都是正确的，只是编译器太笨不懂罢了。
- 总结下：
	- 生命周期 'static 意味着能和程序活得一样久，例如字符串字面量和特征对象
	- 实在遇到解决不了的生命周期标注问题，可以尝试 T: 'static，有时候它会给你奇迹
- # 9.  参考文章
  
  [https://course.rs/first-try/sth-you-should-not-do.html](https://course.rs/first-try/sth-you-should-not-do.html)
-