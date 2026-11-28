# 1.  构建单线程Web服务器
- 在开始之前先来简单回顾下构建所需的网络协议: HTTP 和 TCP。这两种协议都是请求-应答模式的网络协议，意味着在客户端发起请求后，服务器会监听并处理进入的请求，最后给予应答，至于这个过程怎么进行，取决于具体的协议定义。
- 与 HTTP 有所不同， TCP 是一个底层协议，它仅描述客户端传递了信息给服务器，至于这个信息长什么样，怎么解析处理，则不在该协议的职责范畴内。
- 而 HTTP 协议是更高层的通信协议，一般来说都基于 TCP 来构建 (HTTP/3 是基于 UDP 构建的协议)，更高层的协议也意味着它会对传输的信息进行解析处理。
- 更加深入的学习网络协议并不属于本书的范畴，因此让我们从如何读取 TCP 传输的字节流开始吧。
- ## 1.1.  监听TCP连接
- 创建一个新的项目：
  
  ```
  $ cargo new hello
     Created binary (application) `hello` project
  $ cd hello
  ```
- 接下来，使用 std::net 模块监听进入的请求连接，IP和端口是 127.0.0.1:7878 。
  
  ```
  use std::net::TcpListener;
  
  fn main() {
    // 监听地址: 127.0.0.1:7878
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
  
    for stream in listener.incoming() {
        let stream = stream.unwrap();
  
        println!("Connection established!");
    }
  }
  ```
- 选择 7878 端口的原因是，80 和 8080 往往都被 HTTP 服务器霸占，因此我们需要选择一个跟已经监听的端口不冲突的。
- bind 非常类似 new ，它返回一个 TcpListener 实例，只不过我们一般都说 "绑定到某个端口"，因此 bind 这个名称会更合适。
- unwrap 的使用是因为 bind 返回 Result<T,E>，毕竟监听是有可能报错的，例如：如果要监听 80 端口往往需要管理员权限；监听了同样的端口，等等。
- incoming 会返回一个迭代器，它每一次迭代都会返回一个新的连接 stream(客户端发起，web服务器监听接收)，因此，接下来要做的就是从 stream 中读取数据，然后返回处理后的结果。
- 细心的同学可能会注意到，代码中对 stream 还进行了一次 unwrap 处理，原因在于我们并不是在迭代一个一个连接，而是在迭代处理一个一个请求建立连接的尝试，而这种尝试可能会失败！例如，操作系统的最大连接数限制。
- 现在，启动服务器，然后在你的浏览器中，访问地址 127.0.0.1:7878，这时应该会看到一条错误信息，类似: "Connection reset"，毕竟我们的服务器目前只是接收了连接，并没有回复任何数据。
  
  ```
  $ cargo run
   Running `target/debug/hello`
  Connection established!
  Connection established!
  Connection established!
  ```
- 无论浏览器怎么摆烂，我们的服务器还是成功打出了信息：TCP 连接已经成功建立。
- 可能大家会疑问，为啥在浏览器访问一次，可能会在终端打印出多次请求建立的信息，难道不是应该一一对应吗？原因在于当 stream 超出作用域时，会触发 drop 的扫尾工作，其中包含了关闭连接。但是，浏览器可能会存在自动重试的情况，因此还会重新建立连接，最终打印了多次。
- 由于 listener.incoming 会在当前阻塞式监听，也就是 main 线程会被阻塞，我们最后需要通过 ctrl + c 来结束程序进程。
- ## 1.2.  读取请求
- 连接建立后，就可以开始读取客户端传来的数据:
  
  ```
  use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
  };
  
  fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
  
    for stream in listener.incoming() {
        let stream = stream.unwrap();
  
        handle_connection(stream);
    }
  }
  
  fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
  
    println!("Request: {:#?}", http_request);
  }
  ```
- 这段代码有几点值得注意:
	- 引入 std::io::prelude 和 std::io::BufReader 是引入相应的特征和类型，帮助我们读取和写入数据
	- BufReader 可以实现缓冲区读取，底层其实是基于 std::io::Read 实现
	- 可以使用 lines 方法来获取一个迭代器，可以对传输的内容流进行按行迭代读取，要使用该方法，必须先引入 std::io::BufRead
	- 最后使用 collect 消费掉迭代器，最终客户端发来的请求数据被存到 http_request 这个动态数组中
- 大家可能会比较好奇，该如何判断客户端发来的 HTTP 数据是否读取完成，答案就在于客户端会在请求数据的结尾附上两个换行符，当我们检测到某一行字符串为空时，就意味着请求数据已经传输完毕，可以 collect 了。
- 来运行下试试:
  
  ```
  $ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
  Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
  ]
  ```
- ## 1.3.  请求应答
- 应答的格式与请求相差不大，其中 Status-Code 是最重要的，它用于告诉客户端，当前的请求是否成功，若失败，大概是什么原因，它就是著名的 HTTP 状态码，常用的有 200: 请求成功，404 目标不存在，等等。
- 下面将该应答发送回客户端:
  
  ```
  fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
  
    let response = "HTTP/1.1 200 OK\r\n\r\n";
  
    stream.write_all(response.as_bytes()).unwrap();
  }
  ```
- 由于 write_all 方法接受 &[u8] 类型作为参数，这里需要用 as_bytes 将字符串转换为字节数组。
- 重新启动服务器，然后再观察下浏览器中的输出，这次应该不再有报错，而是一个空白页面，因为没有返回任何具体的数据( message-body )，上面只是一条最简单的符合 HTTP 格式的数据。
- ## 1.4.  返回HTML页面
- 空白页面显然会让人不知所措，那就返回一个简单的 HTML 页面，给用户打给招呼。
- 在项目的根目录下创建 hello.html 文件并写入如下内容：
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
  </html>
  ```
- 新修改的代码中，读取了新增 HTML 的内容，并按照 HTTP 格式，将内容传回给客户端。
  
  ```
  use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
  };
  // --snip--
  
  fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
  
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();
  
    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
  
    stream.write_all(response.as_bytes()).unwrap();
  }
  ```
- ## 1.5.  验证请求和选择性应答
- 我们要针对用户的不同请求给出相应的不同回复，让场景模拟更加真实。
  
  ```
  fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();
  
    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();
  
        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );
  
        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // some other request
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();
  
        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );
  
        stream.write_all(response.as_bytes()).unwrap();
    }
  }
  ```
- 注意迭代器方法 next 的使用，原因在于我们只需要读取第一行，判断具体的 HTTP METHOD 是什么。
- 接着判断了用户是否请求了 / 根路径，如果是，返回之前的 hello.html 页面；如果不是...尚未实现。
- 重新运行服务器，如果你继续访问 127.0.0.1:7878 ，那么看到的依然是 hello.html 页面，因为默认访问根路径，但是一旦换一个路径访问，例如 127.0.0.1:7878/something-else，那你将继续看到之前看过多次的连接错误。
- 在根路径下创建 404.html并填入下面内容:
  
  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="utf-8">
    <title>你好!</title>
  </head>
  <body>
    <h1>很抱歉!</h1>
    <p>由于运维删库跑路，我们的数据全部丢失，总监也已经准备跑路，88</p>
  </body>
  </html>
  ```
- 最后，上面的代码其实有很多重复，可以提取出来进行简单重构:
  
  ```
  // --snip--
  
  fn handle_connection(mut stream: TcpStream) {
    // --snip--
  
    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };
  
    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();
  
    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
  
    stream.write_all(response.as_bytes()).unwrap();
  }
  ```
- # 2.  构建多线程Web服务器
- ## 2.1.  使用线程池改善吞吐
- 线程池包含一组已生成的线程，它们时刻等待着接收并处理新的任务。当程序接收到新任务时，它会将线程池中的一个线程指派给该任务，在该线程忙着处理时，新来的任务会交给池中剩余的线程进行处理。最终，当执行任务的线程处理完后，它会被重新放入到线程池中，准备处理新任务。
- 假设线程池中包含 N 个线程，那么可以推断出，服务器将拥有并发处理 N 个请求连接的能力，从而增加服务器的吞吐量。
- 同时，我们将限制线程池中的线程数量，以保护服务器免受拒绝服务攻击（DoS）的影响：如果针对每个请求创建一个新线程，那么一个人向我们的服务器发出1000万个请求，会直接耗尽资源，导致后续用户的请求无法被处理，这也是拒绝服务名称的来源。
- 因此，还需对线程池进行一定的架构设计，首先是设定最大线程数的上限，其次维护一个请求队列。池中的线程去队列中依次弹出请求并处理。这样就可以同时并发处理 N 个请求，其中 N 是线程数。
- 但聪明的读者可能会想到，假如每个请求依然耗时很长，那请求队列依然会堆积，后续的用户请求还是需要等待较长的时间，毕竟你也就 N 个线程，但总归比单线程要强 N 倍吧 :D
- 当然，线程池依然是较为传统的提升吞吐方法，比较新的有：单线程异步 IO，例如 redis；多线程异步 IO，例如 Rust 的主流 web 框架。事实上，大家在下一个实战项目中，会看到相关技术的应用。
- 为每一个请求生成一个线程
  
  ```
  fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
  
    for stream in listener.incoming() {
        let stream = stream.unwrap();
  
        thread::spawn(|| {
            handle_connection(stream);
        });
    }
  }
  ```
- ## 2.2.  限制创建线程的数量
- 原则上，我们希望在上面代码的基础上，尽量少的去修改，下面是一个假想的线程池 API 实现:
  
  ```
  fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);
  
    for stream in listener.incoming() {
        let stream = stream.unwrap();
  
        pool.execute(|| {
            handle_connection(stream);
        });
    }
  }
  ```
- 代码跟之前的类似，也非常简洁明了， ThreadPool::new(4) 创建一个包含 4 个线程的线程池，接着通过 pool.execute 去分发执行请求。
- 显然，上面的代码无法编译，下面来逐步实现。
- ## 2.3.  使用编译器驱动的方式开发ThreadPool
- Rust中的一项内容——使用编译器驱动开发
- 检查之前的代码：
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
  error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
  11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`
  
  For more information about this error, try `rustc --explain E0433`.
  error: could not compile `hello` due to previous error
  ```
- 看起来我们需要实现 ThreadPool 类型。看起来，还需要添加一个库包，未来线程池的代码都将在这个独立的包中完成，甚至于未来你要实现其它的服务，也可以复用这个多线程库包。
- 创建 src/lib.rs 文件并写入如下代码:
  
  ```
  pub struct ThreadPool;
  ```
- 接着在 main.rs 中引入:
  
  ```
  // main.rs
  use hello::ThreadPool;
  ```
- 编译后依然报错:
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
  error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
  12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`
  
  For more information about this error, try `rustc --explain E0599`.
  error: could not compile `hello` due to previous error
  ```
- 好，继续实现 new 函数 :
  
  ```
  pub struct ThreadPool;
  
  impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
  }
  ```
- 继续检查：
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
  error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
  17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`
  
  For more information about this error, try `rustc --explain E0599`.
  error: could not compile `hello` due to previous error
  ```
- 这个方法类似于 thread::spawn，用于将闭包中的任务交给某个空闲的线程去执行。
- 其实这里有一个小难点：execute 的参数是一个闭包，回忆下之前学过的内容，闭包作为参数时可以由三个特征进行约束: Fn、FnMut 和 FnOnce，选哪个就成为一个问题。由于 execute 在实现上类似 thread::spawn，我们可以参考下后者的签名如何声明。
  
  ```
  pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
  ```
- 可以看出，spawn 选择 FnOnce 作为 F 闭包的特征约束，原因是闭包作为任务只需被线程执行一次即可。
- F 还有一个特征约束 Send ，也可以照抄过来，毕竟闭包需要从一个线程传递到另一个线程，至于生命周期约束 'static，是因为我们并不知道线程需要多久时间来执行该任务。
  
  ```
  impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
  }
  ```
- 在理解 spawn 后，就可以轻松写出如上的 execute 实现，注意这里的 FnOnce() 跟 spawn 有所不同，原因是要 execute 传入的闭包没有参数也没有返回值。
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
  ```
- 成功编译，但在浏览器访问依然会报之前类似的错误，下面来实现 execute。
- ## 2.4.  new还是build
- 关于 ThreadPool 的构造函数，存在两个选择 new 和 build。
- new 往往用于简单初始化一个实例，而 build 往往会完成更加复杂的构建工作，例如入门实战中的 Config::build。
- 在这个项目中，我们并不需要在初始化线程池的同时创建相应的线程，因此 new 是更适合的选择:
  
  ```
  impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        ThreadPool
    }
  
    // --snip--
  }
  ```
- 这里有两点值得注意:
	- usize 类型包含 0，但是创建没有任何线程的线程池显然是无意义的，因此做一下 assert! 验证
	- ThreadPool 拥有不错的文档注释，甚至包含了可能 panic 的情况，通过 cargo doc --open 可以访问文档注释
- ## 2.5.  存储线程
- 创建 ThreadPool 后，下一步就是存储具体的线程，既然要存放线程，一个绕不过去的问题就是：用什么类型来存放，例如假如使用 Vec<T> 来存储，那这个 T 应该是什么？
- 估计还得探索下 thread::spawn 的签名，毕竟它生成并返回一个线程:
  
  ```
  pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
  ```
- 看起来 JoinHandle<T> 是我们需要的，这里的 T 是传入的闭包任务所返回的，我们的任务无需任何返回，因此 T 直接使用 () 即可。
  
  ```
  use std::thread;
  
  pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
  }
  
  impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        let mut threads = Vec::with_capacity(size);
  
        for _ in 0..size {
            // create some threads and store them in the vector
        }
  
        ThreadPool { threads }
    }
    // --snip--
  }
  ```
- 如上所示，最终我们使用 Vec<thread::JoinHandle<()>> 来存储线程，同时设定了容量上限 with_capacity(size)，该方法还可以提前分配好内存空间，比 Vec::new 的性能要更好一点。
- ## 2.6.  将代码从TreadPool发送到线程中
- 上面的代码留下一个未实现的 for 循环，用于创建和存储线程。
- 学过多线程一章后，大家应该知道 thread::spawn 虽然是生成线程最好的方式，但是它会立即执行传入的任务，然而，在我们的使用场景中，创建线程和执行任务明显是要分离的，因此标准库看起来不再适合。
- 可以考虑创建一个 Worker 结构体，作为 ThreadPool 和任务线程联系的桥梁，它的任务是获得将要执行的代码，然后在具体的线程中去执行。想象一个场景：一个餐馆，Worker 等待顾客的点餐，然后将具体的点餐信息传递给厨房，感觉类似服务员？
- 引入 Worker 后，就无需再存储 JoinHandle<()> 实例，直接存储 Worker 实例：该实例内部会存储 JoinHandle<()>。下面是新的线程池创建流程:
  
  ```
  use std::thread;
  
  pub struct ThreadPool {
    workers: Vec<Worker>,
  }
  
  impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        let mut workers = Vec::with_capacity(size);
  
        for id in 0..size {
            workers.push(Worker::new(id));
        }
  
        ThreadPool { workers }
    }
    // --snip--
  }
  
  struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
  }
  
  impl Worker {
    fn new(id: usize) -> Worker {
        // 尚未实现..
        let thread = thread::spawn(|| {});
        // 每个 `Worker` 都拥有自己的唯一 id
        Worker { id, thread }
    }
  }
  ```
- 由于外部调用者无需知道 Worker 的存在，因此这里使用了私有的声明。
- ## 2.7.  将请求发送给线程
- 在上面的代码中， thread::spawn(|| {}) 还没有给予实质性的内容，现在一起来完善下。
- 首先 Worker 结构体需要从线程池 TreadPool 的队列中获取待执行的代码，对于这类场景，消息传递非常适合：我们将使用消息通道( channel )作为任务队列。
  
  ```
  use std::{sync::mpsc, thread};
  
  pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
  }
  
  struct Job;
  
  impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        let (sender, receiver) = mpsc::channel();
  
        let mut workers = Vec::with_capacity(size);
  
        for id in 0..size {
            workers.push(Worker::new(id));
        }
  
        ThreadPool { workers, sender }
    }
    // --snip--
  }
  ```
- 阅读过之前内容的同学应该知道，消息通道有发送端和接收端，其中线程池 ThreadPool 持有发送端，通过 execute 方法来发送任务。那么问题来了，谁持有接收端呢？答案是 Worker，它的内部线程将接收任务，然后进行处理。
  
  ```
  impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        let (sender, receiver) = mpsc::channel();
  
        let mut workers = Vec::with_capacity(size);
  
        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }
  
        ThreadPool { workers, sender }
    }
    // --snip--
  }
  
  // --snip--
  
  impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });
  
        Worker { id, thread }
    }
  }
  ```
- receiver 并没有实现 Copy，因此它的所有权在第一次循环中，就被传入到第一个 Worker 实例中，后续自然无法再使用。
- 报错就解决呗，但 Rust 中的 channel 实现是 mpsc，即多生产者单消费者，因此我们无法通过克隆消费者的方式来修复这个错误。当然，发送多条消息给多个接收者也不在考虑范畴，该怎么办？似乎陷入了绝境。
- 雪上加霜的是，就算 receiver 可以克隆，但是你得保证同一个时间只有一个receiver 能接收消息，否则一个任务可能同时被多个 Worker 执行，因此多个线程需要安全的共享和使用 receiver，等等，安全的共享？听上去 Arc 这个多所有权结构非常适合，互斥使用？貌似 Mutex 很适合，结合一下，Arc<Mutex<T>>，这不就是我们之前见过多次的线程安全类型吗？
- 总之，Arc 允许多个 Worker 同时持有 receiver，而 Mutex 可以确保一次只有一个 Worker 能从 receiver 接收消息。
  
  ```
  use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
  };
  // --snip--
  
  impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
  
        let (sender, receiver) = mpsc::channel();
  
        let receiver = Arc::new(Mutex::new(receiver));
  
        let mut workers = Vec::with_capacity(size);
  
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
  
        ThreadPool { workers, sender }
    }
  
    // --snip--
  }
  
  // --snip--
  
  impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
  }
  ```
- 修改后，每一个 Worker 都可以安全的持有 receiver，同时不必担心一个任务会被重复执行多次，完美！
- ## 2.8.  实现execute方法
- 首先，需要为一个很长的类型创建一个别名, 有多长呢？
  
  ```
  // --snip--
  
  type Job = Box<dyn FnOnce() + Send + 'static>;
  
  impl ThreadPool {
    // --snip--
  
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
  
        self.sender.send(job).unwrap();
    }
  }
  
  // --snip--
  ```
- 创建别名的威力暂时还看不到，敬请期待。总之，这里的工作很简单，将传入的任务包装成 Job 类型后，发送出去。
- 但是还没完，接收的代码也要完善下:
  
  ```
  // --snip--
  
  impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
  
            println!("Worker {id} got a job; executing.");
  
            job();
        });
  
        Worker { id, thread }
    }
  }
  ```
- 修改后，就可以不停地循环去接收任务，最后进行执行。还可以看到因为之前 Job 别名的引入， new 函数的签名才没有过度复杂，否则你将看到的是 fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Box<dyn FnOnce() + Send + 'static>>>>) -> Worker ，感受下类型别名的威力吧 :D
- lock() 方法可以获得一个 Mutex 锁，至于为何使用 unwrap，难道获取锁还能失败？没错，假如当前持有锁的线程 panic 了，那么这些等待锁的线程就会获取一个错误，因此 通过 unwrap 来让当前等待的线程 panic 是一个不错的解决方案，当然你还可以换成 expect。
- 一旦获取到锁里的内容 mpsc::Receiver<Job>> 后，就可以调用其上的 recv 方法来接收消息，依然是一个 unwrap，原因在于持有发送端的线程可能会被关闭，这种情况下直接 panic 也是不错的。
- recv 的调用过程是阻塞的，意味着若没有任何任务，那当前的调用线程将一直等待，直到接收到新的任务。Mutex<T> 可以同一个任务只会被一个 Worker 获取，不会被重复执行。
  
  ```
  $ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
  warning: field is never read: `workers`
  --> src/lib.rs:7:5
  |
  7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default
  
  warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
  48 |     id: usize,
   |     ^^^^^^^^^
  
  warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
  49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  
  warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
  Worker 0 got a job; executing.
  Worker 2 got a job; executing.
  Worker 1 got a job; executing.
  Worker 3 got a job; executing.
  Worker 0 got a job; executing.
  Worker 2 got a job; executing.
  Worker 1 got a job; executing.
  Worker 3 got a job; executing.
  Worker 0 got a job; executing.
  Worker 2 got a job; executing.
  ```
- 终于，程序如愿运行起来，我们的线程池可以并发处理任务了！从打印的数字可以看到，只有 4 个线程去执行任务，符合我们对线程池的要求，这样再也不用担心系统的线程资源会被消耗殆尽了！
- ## 2.9.  为线程池实现Drop
- 当线程池被 drop 时，需要等待所有的子线程完成它们的工作，然后再退出，下面是一个初步尝试:
  
  ```
  impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
  
            worker.thread.join().unwrap();
        }
    }
  }
  ```
- 这里通过实现 Drop 特征来为线程池添加资源收尾工作，代码比较简单，就是依次调用每个线程的 join 方法。编译下试试：
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
  error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
  52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ ------ `worker.thread` moved due to this method call
   |             |
   |             move occurs because `worker.thread` has type `JoinHandle<()>`, which does not implement the `Copy` trait
   |
  note: this function takes ownership of the receiver `self`, which moves `worker.thread`
  
  For more information about this error, try `rustc --explain E0507`.
  error: could not compile `hello` due to previous error
  ```
- 这里的报错很明显，worker.thread 试图拿走所有权，但是 worker 仅仅是一个可变借用，显然是不可行的。
- 目前来看，只能将 thread 从 worker 中移动出来，一个可行的尝试:
  
  ```
  struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
  }
  ```
- 对于 Option 类型，可以使用 take 方法拿走内部值的所有权，同时留下一个 None 在风中孤独凌乱。继续尝试编译驱动开发模式:
  
  ```
  $ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
  error[E0599]: no method named `join` found for enum `Option` in the current scope
  --> src/lib.rs:52:27
   |
  52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in `Option<JoinHandle<()>>`
   |
  note: the method `join` exists on the type `JoinHandle<()>`
  help: consider using `Option::expect` to unwrap the `JoinHandle<()>` value, panicking if the value is an `Option::None`
   |
  52 |             worker.thread.expect("REASON").join().unwrap();
   |                          +++++++++++++++++
  
  error[E0308]: mismatched types
  --> src/lib.rs:72:22
   |
  72 |         Worker { id, thread }
   |                      ^^^^^^ expected enum `Option`, found struct `JoinHandle`
   |
   = note: expected enum `Option<JoinHandle<()>>`
            found struct `JoinHandle<_>`
  help: try wrapping the expression in `Some`
   |
  72 |         Worker { id, thread: Some(thread) }
   |                      +++++++++++++      +
  ```
- 先来解决第二个类型不匹配的错误:
  
  ```
  impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
  
        Worker {
            id,
            thread: Some(thread),
        }
    }
  }
  ```
- 简单搞定，回头看看第一个错误，既然换了 Option，就可以用 take 拿走所有权:
  
  ```
  impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
  
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
  }
  ```
- 注意这种 if let 的写法，若 worker.thread 已经是 None，什么都不会发生，符合我们的预期; 若包含一个线程，那就拿走其所有权，然后调用 join。
- ## 2.10.  停止工作线程
- 虽然调用了 join ，但是目标线程依然不会停止，原因在于它们在无限的 loop 循环等待，看起来需要借用 channel 的 drop 机制：释放 sender发送端后，receiver 接收端会收到报错，然后再退出即可。
  
  ```
  pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
  }
  // --snip--
  impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --snip--
  
        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }
  
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
  
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
  }
  
  impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());
  
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
  
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
  }
  ```
- 上面做了两处改变:
	- 为 sender 增加 Option 封装，这样可以用 take 拿走所有权，跟之前的 thread 一样
	- 主动调用 drop 关闭发送端 sender
- 关闭 sender 后，将关闭对应的 channel，意味着不会再有任何消息被发送。随后，所有的处于无限 loop 的接收端将收到一个错误，我们根据错误再进行进一步的处理。
  
  ```
  impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();
  
            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");
  
                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });
  
        Worker {
            id,
            thread: Some(thread),
        }
    }
  }
  ```
- 为了快速验证代码是否正确，修改 main 函数，让其只接收前两个请求:
  
  ```
  fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);
  
    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();
  
        pool.execute(|| {
            handle_connection(stream);
        });
    }
  
    println!("Shutting down.");
  }
  ```
- take 是迭代器 Iterator 上的方法，会限制后续的迭代进行最多两次，然后就结束监听，随后 ThreadPool 也将超出作用域并自动触发 drop。
  
  ```
  $ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
  Worker 0 got a job; executing.
  Shutting down.
  Shutting down worker 0
  Worker 3 got a job; executing.
  Worker 1 disconnected; shutting down.
  Worker 2 disconnected; shutting down.
  Worker 3 disconnected; shutting down.
  Worker 0 disconnected; shutting down.
  Shutting down worker 1
  Shutting down worker 2
  Shutting down worker 3
  ```
- 可以看到，代码按照我们的设想如期运行，至此，一个基于线程池的简单 Web 服务器已经完成
- # 3.  文件搜索工具
- ## 3.1.  接收命令行参数
- 创建一个新的项目 minigrep ，该名字充分体现了我们的自信：就是不如 grep。
  
  ```
  cargo new minigrep
     Created binary (application) `minigrep` project
  $ cd minigrep
  ```
- 首先来思考下，如果要传入文件路径和待搜索的字符串，那这个命令该长啥样，我觉得大概率是这样:
  
  ```
  cargo run -- searchstring example-filename.txt
  ```
- “--” 告诉 cargo 后面的参数是给我们的程序使用的，而不是给 cargo 自己使用，例如 -- 前的 run 就是给它用的。
- 接下来就是在程序中读取传入的参数，这个很简单，下面代码就可以:
  
  ```
  // in main.rs
  use std::env;
  
  fn main() {
    let args: Vec<String> = env::args().collect();
    dbg!(args);
  }
  ```
- 首先通过 use 引入标准库中的 env 包，然后 env::args 方法会读取并分析传入的命令行参数，最终通过 collect 方法输出一个集合类型 Vector。
- 可能有同学疑惑，为啥不直接引入 args ，例如 use std::env::args ，这样就无需 env::args 来繁琐调用，直接args.collect() 即可。原因很简单，args 方法只会使用一次，啰嗦就啰嗦点吧，把相同的好名字让给 let args.. 这位大哥不好吗？毕竟人家要出场多次的。
- 不可信的输入
	- 所有的用户输入都不可信！不可信！不可信！
	- 重要的话说三遍，我们的命令行程序也是，用户会输入什么你根本就不知道，例如他输入了一个非 Unicode 字符，你能阻止吗？显然不能，但是这种输入会直接让我们的程序崩溃！
	- 原因是当传入的命令行参数包含非 Unicode 字符时， std::env::args 会直接崩溃，如果有这种特殊需求，建议大家使用 std::env::args_os，该方法产生的数组将包含 OsString 类型，而不是之前的 String 类型，前者对于非 Unicode 字符会有更好的处理。
	- 至于为啥我们不用，两个理由，你信哪个：1. 用户爱输入啥输入啥，反正崩溃了，他就知道自己错了 2. args_os 会引入额外的跨平台复杂性
- collect 方法其实并不是std::env包提供的，而是迭代器自带的方法(env::args() 会返回一个迭代器)，它会将迭代器消费后转换成我们想要的集合类型，关于迭代器和 collect 的具体介绍，请参考这里。
- 最后，代码中使用 dbg! 宏来输出读取到的数组内容，来看看长啥样：
  
  ```
  $ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/minigrep`
  [src/main.rs:5] args = [
    "target/debug/minigrep",
  ]
  ```
  
  ```
  $ cargo run -- needle haystack
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 1.57s
     Running `target/debug/minigrep needle haystack`
  [src/main.rs:5] args = [
    "target/debug/minigrep",
    "needle",
    "haystack",
  ]
  ```
- 上面两个版本分别是无参数和两个参数，其中无参数版本实际上也会读取到一个字符串，仔细看，是不是长得很像我们的程序名，Bingo! env::args 读取到的参数中第一个就是程序的可执行路径名。
- ## 3.2.  存储读取到的参数
- 在编程中，给予清晰合理的变量名是一项基本功，咱总不能到处都是 args[1] 、args[2] 这样的糟糕代码吧。
- 因此我们需要两个变量来存储文件路径和待搜索的字符串:
  
  ```
  use std::env;
  
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    let query = &args[1];
    let file_path = &args[2];
  
    println!("Searching for {}", query);
    println!("In file {}", file_path);
  }
  ```
- 很简单的代码，来运行下:
  
  ```
  $ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep test sample.txt`
  Searching for test
  In file sample.txt
  ```
- 输出结果很清晰的说明了我们的目标：在文件 sample.txt 中搜索包含 test 字符串的内容。
- ## 3.3.  文件读取
- 接下来修改 main.rs 来读取文件内容：
  
  ```
  use std::env;
  use std::fs;
  
  fn main() {
    // --省略之前的内容--
    println!("In file {}", file_path);
  
    let contents = fs::read_to_string(file_path)
        .expect("Should have been able to read the file");
  
    println!("With text:\n{contents}");
  }
  ```
- 首先，通过 use std::fs 引入文件操作包，然后通过 fs::read_to_string 读取指定的文件内容，最后返回的 contents 是 std::io::Result<String> 类型。
- 运行下试试，这里无需输入第二个参数，因为我们还没有实现查询功能:
  
  ```
  $ cargo run -- the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep the poem.txt`
  Searching for the
  In file poem.txt
  With text:
  I'm nobody! Who are you?
  Are you nobody, too?
  Then there's a pair of us - don't tell!
  They'd banish us, you know.
  
  How dreary to be somebody!
  How public, like a frog
  To tell your name the livelong day
  To an admiring bog!
  ```
- 虽然代码还有很多瑕疵，例如所有内容都在 main 函数，这个不符合软件工程，没有错误处理，功能不完善等。
- ## 3.4.  模块化和错误处理
- 可以得出 main 函数应该包含的功能:
	- 解析命令行参数
	- 初始化其它配置
	- 调用 lib.rs 中的 run 函数，以启动逻辑代码的运行
	- 如果 run 返回一个错误，需要对该错误进行处理
- ### 3.4.1.  分离命令行解析
- 根据之前的分析，我们需要将命令行解析的代码分离到一个单独的函数，然后将该函数放置在 main.rs 中：
  
  ```
  // in main.rs
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    let (query, file_path) = parse_config(&args);
  
    // --省略--
  }
  
  fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let file_path = &args[2];
  
    (query, file_path)
  }
  ```
- 经过分离后，之前的设计目标完美达成，即精简了 main 函数，又将配置相关的代码放在了 main.rs 文件里。
- ### 3.4.2.  聚合配置变量
- 前文提到，配置变量并不适合分散的到处都是，因此使用一个结构体来统一存放是非常好的选择，这样修改后，后续的使用以及未来的代码维护都将更加简单明了。
  
  ```
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    let config = parse_config(&args);
  
    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
  
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");
  
    // --snip--
  }
  
  struct Config {
    query: String,
    file_path: String,
  }
  
  fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();
  
    Config { query, file_path }
  }
  ```
- 值得注意的是，Config 中存储的并不是 &str 这样的引用类型，而是一个 String 字符串，也就是 Config 并没有去借用外部的字符串，而是拥有内部字符串的所有权。clone 方法的使用也可以佐证这一点。大家可以尝试不用 clone 方法，看看该如何解决相关的报错 :D
  
  clone 的得与失
- 在上面的代码中，除了使用 clone ，还有其它办法来达成同样的目的，但 clone 无疑是最简单的方法：直接完整的复制目标数据，无需被所有权、借用等问题所困扰，但是它也有其缺点，那就是有一定的性能损耗。
- 因此是否使用 clone 更多是一种性能上的权衡，对于上面的使用而言，由于是配置的初始化，因此整个程序只需要执行一次，性能损耗几乎是可以忽略不计的。
- 总之，判断是否使用 clone:
	- 是否严肃的项目，玩具项目直接用 clone 就行，简单不好吗？
	- 要看所在的代码路径是否是热点路径(hot path)，例如执行次数较多的显然就是热点路径，热点路径就值得去使用性能更好的实现方式
- 我们试着来优化下，通过构造函数来初始化一个 Config 实例，而不是直接通过函数返回实例，典型的，标准库中的 String::new 函数就是一个范例。
  
  ```
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    let config = Config::new(&args);
  
    // --snip--
  }
  
  // --snip--
  
  impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();
  
        Config { query, file_path }
    }
  }
  ```
- ### 3.4.3.  错误处理
- 报错信息对于开发者会很明确，但是对于使用者而言，就相当难理解了，下面一起来解决它。
- 还记得在错误处理章节，我们提到过 panic 的两种用法: 被动触发和主动调用嘛？上面代码的出现方式很明显是被动触发，这种报错信息是不可控的，下面我们先改成主动调用的方式:
  
  ```
  // in main.rs
  // --snip--
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!("not enough arguments");
        }
        // --snip--
  ```
- 目的很明确，一旦传入的参数数组长度小于 3，则报错并让程序崩溃推出，这样后续的数组访问就不会再越界了。
- 返回Result来替代直接panic
  
  ```
  impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
  
        let query = args[1].clone();
        let file_path = args[2].clone();
  
        Ok(Config { query, file_path })
    }
  }
  ```
- 这里的 Result 可能包含一个 Config 实例，也可能包含一条错误信息 &static str，不熟悉这种字符串类型的同学可以回头看看字符串章节，代码中的字符串字面量都是该类型，且拥有 'static 生命周期。
- 处理返回的Result
- 接下来就是在调用 build 函数时，对返回的 Result 进行处理了，目的就是给出准确且友好的报错提示, 为了让大家更好的回顾我们修改过的内容，这里给出整体代码:
  
  ```
  use std::env;
  use std::fs;
  use std::process;
  
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    // 对 build 返回的 `Result` 进行处理
    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });
  
    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
  
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");
  
    println!("With text:\n{contents}");
  }
  
  struct Config {
    query: String,
    file_path: String,
  }
  
  impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
  
        let query = args[1].clone();
        let file_path = args[2].clone();
  
        Ok(Config { query, file_path })
    }
  }
  ```
- 上面代码有几点值得注意:
	- 当 Result 包含错误时，我们不再调用 panic 让程序崩溃，而是通过 process::exit(1) 来终结进程，其中 1 是一个信号值(事实上非 0 值都可以)，通知调用我们程序的进程，程序是因为错误而退出的。
	- unwrap_or_else 是定义在 Result<T,E> 上的常用方法，如果 Result 是 Ok，那该方法就类似 unwrap：返回 Ok 内部的值；如果是 Err，就调用闭包中的自定义代码对错误进行进一步处理
- 综上可知，config 变量的值是一个 Config 实例，而 unwrap_or_else 闭包中的 err 参数，它的类型是 'static str，值是 "not enough arguments" 那个字符串字面量。
- ### 3.4.4.  分离主体逻辑
- 接下来可以继续精简 main 函数，那就是将主体逻辑( 例如业务逻辑 )从 main 中分离出去，这样 main 函数就保留主流程调用，非常简洁。
  
  ```
  // in main.rs
  fn main() {
    let args: Vec<String> = env::args().collect();
  
    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });
  
    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
  
    run(config);
  }
  
  fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");
  
    println!("With text:\n{contents}");
  }
  
  // --snip--
  ```
- 如上所示，main 函数仅保留主流程各个环节的调用，一眼看过去非常简洁清晰。
- ### 3.4.5.  使用？和特征对象来返回错误
- 答案就是 run 函数没有错误处理，因为在文章开头我们提到过，错误处理最好统一在一个地方完成，这样极其有利于后续的代码维护。
  
  ```
  //in main.rs
  use std::error::Error;
  
  // --snip--
  
  fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;
  
    println!("With text:\n{contents}");
  
    Ok(())
  }
  ```
- 值得注意的是这里的 Result<(), Box<dyn Error>> 返回类型，首先我们的程序无需返回任何值，但是为了满足 Result<T,E> 的要求，因此使用了 Ok(()) 返回一个单元类型 ()。
- 最重要的是 Box<dyn Error>， 如果按照顺序学到这里，大家应该知道这是一个Error 的特征对象(为了使用 Error，我们通过 use std::error::Error; 进行了引入)，它表示函数返回一个类型，该类型实现了 Error 特征，这样我们就无需指定具体的错误类型，否则你还需要查看 fs::read_to_string 返回的错误类型，然后复制到我们的 run 函数返回中，这么做一个是麻烦，最主要的是，一旦这么做，意味着我们无法在上层调用时统一处理错误，但是 Box<dyn Error> 不同，其它函数也可以返回这个特征对象，然后调用者就可以使用统一的方式来处理不同函数返回的 Box<dyn Error>。
- 明白了 Box<dyn Error> 的重要战略地位，接下来大家分析下，fs::read_to_string 返回的具体错误类型是怎么被转化为 Box<dyn Error> 的？其实原因在之前章节都有讲过，这里就不直接给出答案了，参见 ?-传播界的大明星。
- 运行代码看看效果：
  
  ```
  $ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
  warning: unused `Result` that must be used
  --> src/main.rs:19:5
   |
  19 |     run(config);
   |     ^^^^^^^^^^^^
   |
   = note: `#[warn(unused_must_use)]` on by default
   = note: this `Result` may be an `Err` variant, which should be handled
  
  warning: `minigrep` (bin "minigrep") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.71s
     Running `target/debug/minigrep the poem.txt`
  Searching for the
  In file poem.txt
  With text:
  I'm nobody! Who are you?
  Are you nobody, too?
  Then there's a pair of us - don't tell!
  They'd banish us, you know.
  
  How dreary to be somebody!
  How public, like a frog
  To tell your name the livelong day
  To an admiring bog!
  ```
- 没任何问题，不过 Rust 编译器也给出了善意的提示，那就是 Result 并没有被使用，这可能意味着存在错误的潜在可能性。
- ### 3.4.6.  处理返回的错误
  
  ```
  fn main() {
    // --snip--
  
    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
  
    if let Err(e) = run(config) {
        println!("Application error: {e}");
        process::exit(1);
    }
  }
  ```
- 先回忆下在 build 函数调用时，我们怎么处理错误的？然后与这里的方式做一下对比，是不是发现了一些区别？
- 没错 if let 的使用让代码变得更简洁，可读性也更加好，原因是，我们并不关注 run 返回的 Ok 值，因此只需要用 if let 去匹配是否存在错误即可。
- 好了，截止目前，代码看起来越来越美好了，距离我们的目标也只差一个：将主体逻辑代码分离到一个独立的文件 lib.rs 中。
- ### 3.4.7.  分离的逻辑代码放到库包中
- 首先，创建一个 src/lib.rs 文件，然后将所有的非 main 函数都移动到其中。代码大概类似：
  
  ```
  use std::error::Error;
  use std::fs;
  
  pub struct Config {
    pub query: String,
    pub file_path: String,
  }
  
  impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
    }
  }
  
  pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    // --snip--
  }
  ```
- 为了内容的简洁性，这里忽略了具体的实现，下一步就是在 main.rs 中引入 lib.rs 中定义的 Config 类型。
  
  ```
  use std::env;
  use std::process;
  
  use minigrep::Config;
  
  fn main() {
    // --snip--
    let args: Vec<String> = env::args().collect();
  
    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });
  
    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
  
    if let Err(e) = minigrep::run(config) {
        // --snip--
        println!("Application error: {e}");
        process::exit(1);
    }
  }
  ```
- 很明显，这里的 mingrep::run 的调用，以及 Config 的引入，跟使用其它第三方包已经没有任何区别，也意味着我们成功的将逻辑代码放置到一个独立的库包中，其它包只要引入和调用就行。
- ## 3.5.  测试驱动开发
- ### 3.5.1.  失败的测试用例
- 既然要添加测试，那之前的 println! 语句将没有大的用处，毕竟 println! 存在的目的就是为了让我们看到结果是否正确，而现在测试用例将取而代之。
- 接下来，在 lib.rs 文件中，添加 tests 模块和 test 函数:
  
  ```
  #[cfg(test)]
  mod tests {
    use super::*;
  
    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
  Rust:
  safe, fast, productive.
  Pick three.";
  
        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
  }
  ```
- 测试用例将在指定的内容中搜索 duct 字符串，目测可得：其中有一行内容是包含有目标字符串的。
- 但目前为止，还无法运行该测试用例，更何况还想幸灾乐祸的看其失败，原因是 search 函数还没有实现！毕竟是测试驱动、测试先行。
  
  ```
  // in lib.rs
  pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
  }
  ```
- 先添加一个简单的 search 函数实现，非常简单粗暴的返回一个空的数组，显而易见测试用例将成功通过，真是一个居心叵测的测试用例！
- 注意这里生命周期 'a 的使用，之前的章节有详细介绍，不太明白的同学可以回头看看。
- 喔，这么复杂的代码，都用上生命周期了！嘚瑟两下试试：
  
  ```
  $ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 0.97s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)
  
  running 1 test
  test tests::one_result ... FAILED
  
  failures:
  
  ---- tests::one_result stdout ----
  thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `["safe, fast, productive."]`,
  right: `[]`', src/lib.rs:44:9
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
  
  failures:
    tests::one_result
  
  test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
  
  error: test failed, to rerun pass `--lib`
  ```
- ### 3.5.2.  成功的测试用例
- 遍历迭代每一行
- Rust 提供了一个很便利的 lines 方法将目标字符串进行按行分割:
  
  ```
  // in lib.rs
  pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // do something with line
    }
  }
  ```
- 这里的 lines 返回一个迭代器，关于迭代器在后续章节会详细讲解，现在只要知道 for 可以遍历取出迭代器中的值即可。
- 在每一行中查询目标字符串
  
  ```
  // in lib.rs
  pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
  }
  ```
- 与之前的 lines 函数类似，Rust 的字符串还提供了 contains 方法，用于检查 line 是否包含待查询的 query。
- 接下来，只要返回合适的值，就可以完成 search 函数的编写。
- 存储匹配到的结果
- 简单，创建一个 Vec 动态数组，然后将查询到的每一个 line 推进数组中即可：
  
  ```
  // in lib.rs
  pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();
  
    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }
  
    results
  }
  ```
- 至此，search 函数已经完成了既定目标，为了检查功能是否正确，运行下我们之前编写的测试用例:
  
  ```
  $ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished test [unoptimized + debuginfo] target(s) in 1.22s
     Running unittests src/lib.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)
  
  running 1 test
  test tests::one_result ... ok
  
  test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
  
     Running unittests src/main.rs (target/debug/deps/minigrep-9cd200e5fac0fc94)
  
  running 0 tests
  
  test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
  
   Doc-tests minigrep
  
  running 0 tests
  
  test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
  ```
- 测试通过，意味着我们的代码也完美运行，接下来就是在 run 函数中大显身手了。
- 在run函数中调用search函数
  
  ```
  // in src/lib.rs
  pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;
  
    for line in search(&config.query, &contents) {
        println!("{line}");
    }
  
    Ok(())
  }
  ```
- 好，再运行下看看结果，看起来我们距离成功从未如此之近！
  
  ```
  $ cargo run -- frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38s
     Running `target/debug/minigrep frog poem.txt`
  How public, like a frog
  ```
- 酷！成功查询到包含 frog 的行，再来试试 body :
  
  ```
  $ cargo run -- body poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/minigrep body poem.txt`
  I'm nobody! Who are you?
  Are you nobody, too?
  How dreary to be somebody!
  ```