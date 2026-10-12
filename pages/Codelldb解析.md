- Codelldb是一个可以进行C/C++、Rust调试的、DAP Server的开源项目。
  
  Codelldb基于LLDB实现，提供了跨平台的调试支持（Windows、Linux、macOS）。 
  
  github仓库：[https://github.com/vadimcn/codelldb.git](https://github.com/vadimcn/codelldb.git)
- # 1.  编译
- 本文首先介绍的是如何在windows操作系统下进行codelldb的构建
- ## 1.1.  Windows
- ### 1.1.1.  准备windows SDK
- 直接去微软的官网，下载对应installer/build tools。选择C++应用构建即可。本质上来说，实际就是将windows上的C++工具链和windows SDK下载下来。
- 如果需要脱机安装，那么建议下载build tools即可。
- 首先是在 [https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/](https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/) 官网上下载生成工具的installer。
- 然后将installer移动到一个空文件夹，执行
  
  ```
  vs_buildtools.exe --layout D:\VS2022CppOfflineFull --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended --includeOptional --lang zh-CN
  ```
	- --layout参数可以自定义下载的路径
	- --add参数用于定义你需要下载什么包
	- --lang是语言
- 然后将整个下载的离线包移动到对应的安装主机上
- 直接执行
  
  ```
  vs_buildtools.exe --noWeb
  ```
- 然后会弹出一个安装框，选择对应的路径即可。然后执行安装
- ### 1.1.2.  Rust环境
- 编译还需要Rust环境
- 具体的下载方式可以查看[https://rust-lang.org/tools/install/](https://rust-lang.org/tools/install/)
- ### 1.1.3.  Python环境
- 编译还需要Python环境
- 具体的下载方式可以查看[https://www.python.org/downloads/](https://www.python.org/downloads/)
- ### 1.1.4.  构建流程
- 需要打开 x64 Native Tools Command Prompt for VS 2022 ，以确保具体的msvc中的相关环境变量可以加载进去。
- 然后进入到具体的根目录下。与此同时，需要将对应的LLDB_PACKAGE压缩包下载好，放到一个对应的位置
- 在根目录中执行 mkdir build & cd build
- 然后执行cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/toolchain-x86_64-windows-msvc.cmake -DLLDB_PACKAGE=xxx
	- LLDB_PACKAGE就是具体的lldb package.zip的位置
- 最后输入命令 cmake --build build --target adapter
- 最终产物在build中可以找到
- # 2.  解析
- ## 2.1.  架构
  
  ![](https://cdn.nlark.com/yuque/0/2025/svg/2713067/1757806023785-3ed72f18-00ac-4994-9595-8a5d7ef258a5.svg)
  
  `adapter/codelldb` 是 CodeLLDB 调试器适配器的核心组件，它实现了 Debug Adapter Protocol (DAP)，作为 VS Code 和 LLDB 调试器之间的桥梁。
- ### 2.1.1.  主要模块结构
- #### 2.1.1.1.  入口点和初始化
- bin/main.rs: 程序入口点，负责：
	- 解析命令行参数
	- 加载 liblldb 动态库
	- 启动调试服务器
	- 处理终端代理模式
- src/lib.rs: 库的主入口，包含：
	- 模块声明和导入
	- debug_server() 函数：核心服务器逻辑
	- 崩溃处理钩子
	- 测试初始化代码
- #### 2.1.1.2.  核心通信层
- dap_session.rs: DAP 会话管理
	- DAPSession 结构体：管理与 DAP 客户端的通信
	- 消息路由：请求、响应、事件的分发
	- 异步消息处理：使用 tokio 进行并发处理
- dap_codec.rs: DAP 协议编解码器
	- 处理 DAP 消息的序列化/反序列化
	- 帧分割和重组
- stdio_stream.rs: 标准输入输出流处理
	- 为 DAP 通信提供 stdio 传输层
- #### 2.1.1.3.   调试会话管理
- debug_session.rs: 调试会话核心逻辑
	- DebugSession 结构体：调试会话的主要状态管理
	- 请求处理：处理各种 DAP 请求（启动、断点、变量等）
	- 事件处理：处理 LLDB 调试事件
	- 异步响应机制
- #### 2.1.1.4.  调试功能子模块 ( debug_session/ )
- launch.rs: 启动和附加调试目标
	- 处理 launch 和 attach 请求
	- 目标创建和配置
	- 进程启动参数处理
- breakpoints.rs: 断点管理
	- 源码断点、指令断点、函数断点
	- 条件断点和日志断点
	- 异常断点处理
- variables.rs: 变量和作用域管理
	- 变量树结构
	- 变量值格式化
	- 作用域（局部变量、全局变量等）
- step_in.rs: 单步调试功能
	- 步入目标选择
	- 细粒度步进控制
- disassemble.rs: 反汇编功能
	- 指令级调试支持
- #### 2.1.1.5.  表达式处理 ( expressions/ )
- mod.rs: 表达式处理主模块
	- 支持多种表达式类型：Native、Simple、Python
	- 表达式预处理和格式化
- preprocess.rs: 表达式预处理
- hit_condition.rs: 命中条件解析
- expression_format.rs: 表达式格式化
- #### 2.1.1.6.  辅助功能模块
- python.rs: Python 集成
	- Python 脚本执行
	- 自定义可视化器支持
- terminal.rs: 终端管理
	- 调试目标的终端输入输出
- disassembly.rs: 反汇编支持
	- 指令级调试视图
- handles.rs: 句柄管理
	- DAP 对象引用管理
- cancellation.rs: 取消机制
	- 请求取消处理
- ### 2.1.2.  关键方法解析
- #### 2.1.2.1.  debug_server方法
  
  ```
  pub fn debug_server(matches: &ArgMatches) -> Result<(), Error>
  ```
- 接收命令行参数匹配结果
- 返回 Result<(), Error> 表示成功或失败
  
  **崩溃处理钩子**
  
  ```
  hook_crashes();
  ```
- 设置信号处理器，捕获程序崩溃并打印堆栈跟踪
  
  **适配器设置解析**
  
  ```
  let adapter_settings: AdapterSettings = match matches.value_of("settings") {
    Some(s) => match serde_json::from_str(s) {
        Ok(settings) => settings,
        Err(err) => {
            error!("{}", err);
            Default::default()
        }
    },
    None => Default::default(),
  };
  ```
- 从命令行参数中解析适配器配置
- 如果解析失败，使用默认配置并记录错误
  
  **LLDB 调试器初始化**
  
  ```
  SBDebugger::initialize();
  let debugger = SBDebugger::create(false);
  ```
- 初始化 LLDB 调试器库
- 创建调试器实例（false 表示不使用源码编辑器）
  
  **Python 启动命令执行**
  
  ```
  if let Ok(command) = std::env::var("CODELLDB_STARTUP") {
    debug!("Executing {}", command);
    let mut command_result = SBCommandReturnObject::new();
    debugger.command_interpreter().handle_command(&command, &mut command_result, false);
  }
  ```
- 检查 CODELLDB_STARTUP 环境变量
- 如果存在，执行其中的 LLDB 命令
  
  **Python 接口初始化**
  
  ```
  let python_interface = match python::initialize(&debugger, &adapter_dir) {
    Ok(python) => Some(python),
    Err(err) => {
        error!("Initialize Python interpreter: {}", err);
        None
    }
  };
  ```
- 初始化 Python 解释器接口
- 用于支持 Python 脚本和自定义可视化器
  
  **通信方式配置 (第70-78行)**
  
  ```
  let (use_stdio, port, connect) = if let Some(port) = matches.value_of("connect") {
    (false, port.parse()?, true)
  } else if let Some(port) = matches.value_of("port") {
    (false, port.parse()?, false)
  } else {
    (true, 0, false)
  };
  ```
  
  支持三种通信模式：
- **连接模式** (--connect PORT): 连接到指定端口
- **监听模式** (--port PORT): 在指定端口监听连接
- **标准输入输出模式** (默认): 使用 stdio 通信
  
  **异步运行时创建 (第80-84行)**
  
  ```
  let rt = tokio::runtime::Builder::new_multi_thread()
    .worker_threads(2)
    .enable_all()
    .build()
    .unwrap();
  ```
- 创建多线程 Tokio 运行时
- 使用 2 个工作线程
- 启用所有 I/O 和定时器功能
  
  **主要业务逻辑 (第86-119行)**
  
  **标准输入输出模式**
  
  ```
  if use_stdio {
    debug!("Starting on stdio");
    let stream = stdio_stream::StdioStream::new();
    let framed_stream = dap_codec::DAPCodec::new().framed(stream);
    run_debug_session(Box::new(framed_stream), &adapter_settings, &python_interface).await;
  }
  ```
- 创建标准输入输出流
- 使用 DAP 编解码器包装流
- 启动调试会话
  
  **TCP 连接模式**
  
  ```
  if connect {
    debug!("Connecting to {}", addr);
    let mut tcp_stream = TcpStream::connect(addr).await?;
    tcp_stream.set_nodelay(true).unwrap();
    if let Some(auth_token) = auth_token {
        let auth_header = format!("Auth-Token: {}\r\n", auth_token);
        tcp_stream.write_all(&auth_header.as_bytes()).await?;
    }
    let framed_stream = dap_codec::DAPCodec::new().framed(tcp_stream);
    run_debug_session(Box::new(framed_stream), &adapter_settings, &python_interface).await;
  }
  ```
- 连接到指定地址
- 设置 TCP_NODELAY 优化
- 可选的身份验证令牌发送
- 启动调试会话
  
  **TCP 监听模式**
  
  ```
  let listener = TcpListener::bind(&addr).await?;
  while {
    debug!("Listening on {}", listener.local_addr()?);
    let (tcp_stream, _) = listener.accept().await?;
    tcp_stream.set_nodelay(true).unwrap();
    let framed_stream = dap_codec::DAPCodec::new().framed(tcp_stream);
    run_debug_session(Box::new(framed_stream), &adapter_settings, &python_interface).await;
    multi_session
  } {}
  ```
- 绑定到指定地址并监听
- 循环接受连接
- 每个连接启动一个调试会话
- multi_session 控制是否支持多会话
  
  **清理阶段 (第121-125行)**
  
  ```
  rt.shutdown_timeout(Duration::from_millis(10));
  
  debug!("Exiting");
  #[cfg(not(windows))]
  SBDebugger::terminate();
  ```
- 关闭 Tokio 运行时（10ms 超时）
- 在非 Windows 平台上终止 LLDB 调试器
  
  **run_debug_session**** 函数**
  
  这是实际启动调试会话的异步函数：
  
  ```
  async fn run_debug_session(
    framed_stream: Box<dyn DAPChannel>,
    adapter_settings: &adapter_protocol::AdapterSettings,
    python_interface: &Option<Arc<python::PythonInterface>>,
  ) {
    debug!("New debug session");
    let (dap_session, dap_fut) = dap_session::DAPSession::new(framed_stream);
    let session_fut = debug_session::DebugSession::run(
        dap_session,
        adapter_settings.clone(),
        python_interface.as_ref().map(|i| i.clone()),
    );
    tokio::spawn(dap_fut);
    session_fut.await;
    debug!("End of the debug session");
  }
  ```
  
  **执行步骤：**
- 创建 DAP 会话和消息处理 future
- 创建调试会话 future
- 异步启动 DAP 消息处理器
- 等待调试会话完成
- #### 2.1.2.2.   handle_request  方法解析
  
  这是处理 DAP 请求的主入口方法，负责请求的预处理、取消处理和响应分发。
  
  **方法签名**
  
  ```
  fn handle_request(&mut self, seq: u32, request_args: RequestArguments, cancellation: cancellation::Receiver)
  ```
  
  **参数说明：**
- seq: 请求序列号，用于匹配请求和响应
- request_args: DAP 请求参数的枚举类型
- cancellation: 取消接收器，用于处理请求取消
  
  **执行流程**
  
  **第一步：取消检查 (第350-351行)**
  
  ```
  if cancellation.is_cancelled() {
    self.send_response(seq, Err("canceled".into()));
  }
  ```
- 检查请求是否已被取消
- 如果已取消，直接发送取消错误响应
  
  **第二步：Python 中断设置 (第353-355行)**
  
  ```
  if let Some(python) = &self.python {
    cancellation.add_callback(python.interrupt_sender())
  }
  ```
- 如果启用了 Python 支持，将 Python 中断回调添加到取消机制
- 当请求被取消时，会中断正在执行的 Python 代码
  
  **第三步：设置当前取消令牌 (第356行)**
  
  ```
  self.current_cancellation = cancellation;
  ```
- 保存当前请求的取消令牌，供子方法使用
  
  **第四步：未知命令处理 (第357-359行)**
  
  ```
  if let RequestArguments::unknown = request_args {
    info!("Received an unknown command");
    self.send_response(seq, Err("Not implemented.".into()));
  }
  ```
- 处理未知的 DAP 命令
- 返回 "Not implemented" 错误
  
  **第五步：请求处理和响应分发 (第361-378行)**
  
  ```
  let result = self.handle_request_args(request_args);
  self.current_cancellation = cancellation::dummy();
  match result {
    // Spawn async responses as tasks
    Err(err) if err.is::<AsyncResponse>() => {
        let self_ref = self.self_ref.clone();
        tokio::task::spawn_local(async move {
            let fut: std::pin::Pin<Box<_>> = err.downcast::<AsyncResponse>().unwrap().0.into();
            let result = fut.await;
            self_ref.map(|s| s.send_response(seq, result)).await;
        });
    }
    // Send synchronous results immediately
    _ => {
        self.send_response(seq, result);
    }
  }
  ```
  
  **响应处理机制：**
- **同步响应**: 大多数请求立即处理并返回结果
- **异步响应**: 某些请求需要异步处理（如 configurationDone）
	- 使用 AsyncResponse 包装 Future
	- 通过 tokio::task::spawn_local 异步执行
	- 完成后通过 self_ref 发送响应
- #### 2.1.2.3.   handle_request_args  方法解析
  
  这是一个巨大的模式匹配方法，负责将不同类型的 DAP 请求路由到相应的处理函数。
  
  **方法结构**
  
  ```
  #[rustfmt::skip]
  fn handle_request_args(&mut self, arguments: RequestArguments) -> Result<ResponseBody, Error>
  ```
- #[rustfmt::skip] 注解防止格式化工具重新排列代码
- 返回 Result<ResponseBody, Error>
  
  **请求分类处理**
  
  **基础会话管理请求 (第385-410行)**
  
  ```
  RequestArguments::_adapterSettings(args) => 
    self.handle_adapter_settings(args).map(|_| ResponseBody::_adapterSettings),
  RequestArguments::initialize(args) => 
    self.handle_initialize(args).map(|r| ResponseBody::initialize(r)),
  RequestArguments::launch(Either::First(args)) => 
    self.handle_launch(args),
  RequestArguments::attach(Either::First(args)) => 
    self.handle_attach(args),
  RequestArguments::configurationDone(_) => 
    self.handle_configuration_done().map(|_| ResponseBody::configurationDone),
  RequestArguments::disconnect(args) => 
    self.handle_disconnect(args).map(|_| ResponseBody::disconnect),
  ```
  
  **特殊处理：**
- launch 和 attach 支持两种格式：正确解析的参数和原始 JSON
- JSON 格式用于错误报告，调用 report_launch_cfg_error
  
  **调试功能请求 (第411-520行)**
  
  这部分被包装在 no_debug 检查中：
  
  ```
  _ => {
    if self.no_debug {
        bail!("Not supported in noDebug mode.")
    } else {
        // 调试功能请求处理
    }
  }
  ```
  
  **断点管理：**
  
  ```
  RequestArguments::setBreakpoints(args) => 
    self.handle_set_breakpoints(args).map(|r| ResponseBody::setBreakpoints(r)),
  RequestArguments::setInstructionBreakpoints(args) => 
    self.handle_set_instruction_breakpoints(args).map(|r| ResponseBody::setInstructionBreakpoints(r)),
  RequestArguments::setFunctionBreakpoints(args) => 
    self.handle_set_function_breakpoints(args).map(|r| ResponseBody::setFunctionBreakpoints(r)),
  ```
  
  **执行控制：**
  
  ```
  RequestArguments::pause(args) => 
    self.handle_pause(args).map(|_| ResponseBody::pause),
  RequestArguments::continue_(args) => 
    self.handle_continue(args).map(|r| ResponseBody::continue_(r)),
  RequestArguments::next(args) => 
    self.handle_next(args).map(|_| ResponseBody::next),
  RequestArguments::stepIn(args) => 
    self.handle_step_in(args).map(|_| ResponseBody::stepIn),
  ```
  
  **调试信息查询：**
  
  ```
  RequestArguments::threads(_) => 
    self.handle_threads().map(|r| ResponseBody::threads(r)),
  RequestArguments::stackTrace(args) => 
    self.handle_stack_trace(args).map(|r| ResponseBody::stackTrace(r)),
  RequestArguments::variables(args) => 
    self.handle_variables(args).map(|r| ResponseBody::variables(r)),
  ```
  
  **高级功能：**
  
  ```
  RequestArguments::evaluate(args) => 
    self.handle_evaluate(args),
  RequestArguments::readMemory(args) => 
    self.handle_read_memory(args).map(|r| ResponseBody::readMemory(r)),
  RequestArguments::disassemble(args) => 
    self.handle_disassemble(args).map(|r| ResponseBody::disassemble(r)),
  ```
- #### 2.1.2.4.   send_response  方法解析
  
  负责将处理结果转换为 DAP 响应并发送。
  
  **成功响应处理**
  
  ```
  Ok(body) => Response {
    request_seq: request_seq,
    success: true,
    result: ResponseResult::Success { body: body },
  }
  ```
  
  **错误响应处理**
  
  ```
  Err(err) => {
    let blamed = BlamedError::from(err);
    let (message, show) = match blamed.blame {
        Blame::Internal => (format!("Internal debugger error: {}", blamed.inner), true),
        Blame::User => (format!("{}", blamed.inner), true),
        Blame::Nobody => (format!("{}", blamed.inner), false),
    };
    // 错误日志记录和响应构建
  }
  ```
  
  **错误分类：**
- Blame::Internal: 内部调试器错误，显示给用户
- Blame::User: 用户错误（如配置错误），显示给用户
- Blame::Nobody: 正常情况（如进程未运行），不显示错误
  
  **异步响应机制 (****AsyncResponse****)**
  
  **设计目的**
  
  ```
  // AsyncResponse is used to "smuggle" futures out of request handlers
  // in the few cases when we need to respond asynchronously.
  struct AsyncResponse(pub Box<dyn Future<Output = Result<ResponseBody, Error>> + 'static>);
  ```
  
  **使用场景**
  
  主要用于需要等待其他异步操作完成的请求，如：
- configurationDone: 需要等待配置完成
- 某些需要与 LLDB 异步交互的操作
  
  **处理流程**
- 处理函数返回 AsyncResponse 错误
- handle_request 检测到异步响应
- 提取 Future 并异步执行
- 完成后发送响应
- ## 2.2.  启动流程
  
  ![](https://cdn.nlark.com/yuque/0/2025/svg/2713067/1757806099994-2aafbfcd-2b07-4842-97f3-b5193233dd25.svg)
- 插件的启动流程如上述图。
- 关键的代码路径
	- extension/main.ts 启动入口
	  
	  ```
	  sequenceDiagram
	   participant VSCode
	   participant Extension
	   participant Adapter
	   participant LLDB
	  
	   VSCode->>Extension: 开始调试会话
	   Extension->>Extension: createDebugAdapterDescriptor()
	   Extension->>Extension: startDebugAdapter()
	   Extension->>Extension: getAdapterDylibs() - 查找liblldb路径
	   Extension->>Adapter: adapter.start() - 启动DAP进程
	   Adapter->>LLDB: 加载liblldb动态库
	   Adapter-->>Extension: 返回子进程
	   Extension-->>VSCode: 返回适配器描述符
	  ```
	- extension/novsc/adapter.ts 构建启动参数
	  
	  ```
	  // extension/main.ts:343-384
	  async startDebugAdapter(
	  folder: WorkspaceFolder | undefined,
	  adapterSettings: AdapterSettings,
	  connectPort: number,
	  authToken: string
	  ): Promise<ChildProcess> {
	  let config = getExtensionConfig(folder);
	  let [liblldb] = await this.getAdapterDylibs(config);
	  
	  // 关键：调用adapter.start启动DAP服务器
	  let adapterProcess = await adapter.start(liblldb, {
	   extensionRoot: this.context.extensionPath,  // VS Code扩展根目录
	   extraEnv: adapterEnv,
	   workDir: workspace.rootPath,
	   port: connectPort,
	   connect: true,
	   authToken: authToken,
	   adapterSettings: adapterSettings,
	   verboseLogging: verboseLogging
	  });
	  
	  return adapterProcess;
	  }
	  ```
	- 启动子进程
	  
	  ```
	  // extension/novsc/adapter.ts:26-69
	  export async function getSpawnParams(
	   liblldb: string,
	   options: AdapterStartOptions
	  ): Promise<ProcessSpawnParams> {
	   // 关键：确定可执行文件路径
	   let executable = path.join(options.extensionRoot, 'adapter', 'codelldb');
	   
	   // 构建命令行参数
	   let args = ['--liblldb', liblldb, portAction, options.port.toString()];
	   if (options.authToken) {
	       args = args.concat(['--auth-token', options.authToken]);
	   }
	   if (options.adapterSettings) {
	       args = args.concat(['--settings', JSON.stringify(options.adapterSettings)]);
	   }
	   
	   return {
	       command: executable,  // 实际的codelldb可执行文件
	       args: args,
	       options: { env: env, cwd: workDir }
	   }
	  }
	  ```
- ## 2.3.  线程模型
  
  CodeLLDB Adapter的运行时线程模型采用了混合线程模型。结合了：
- **Tokio 异步运行时**（主要业务逻辑）
- **原生操作系统线程**（LLDB 事件监听和阻塞操作）
- **LocalSet 单线程执行器**（调试会话主循环）
  
  ```
  ┌─────────────────────────────────────────────────────────────┐
  │                        主线程                                │
  │  ┌─────────────────────────────────────────────────────┐    │
  │  │                Tokio Runtime                        │    │
  │  │  ┌─────────────────────────────────────────────┐    │    │
  │  │  │            LocalSet                         │    │    │
  │  │  │  ┌─────────────────────────────────────┐    │    │    │
  │  │  │  │        主事件循环                    │    │    │    │
  │  │  │  │  • DAP 请求处理                     │    │    │    │
  │  │  │  │  • LLDB 事件处理                    │    │    │    │
  │  │  │  │  • Python 事件处理                  │    │    │    │
  │  │  │  └─────────────────────────────────────┘    │    │    │
  │  │  └─────────────────────────────────────────────┘    │    │
  │  │                                                      │    │
  │  │  异步任务:                                           │    │
  │  │  • DAP 消息分发器                                    │    │
  │  │  • 控制台输出处理                                    │    │
  │  │  • 请求取消过滤                                      │    │
  │  └─────────────────────────────────────────────────────┘    │
  └─────────────────────────────────────────────────────────────┘
  
  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
  │  LLDB 事件线程   │    │  Python 线程    │    │   其他系统线程   │
  │  (原生线程)      │    │  (如果启用)      │    │  (终端等)        │
  └─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────────────┐
                    │      异步通道网络        │
                    │   (mpsc/broadcast)      │
                    └─────────────────────────┘
  ```
- ### 2.3.1.  线程类型
- #### 2.3.1.1.  主线程 (Main Thread)
  
  ```
  // lib.rs:80-84
  let rt = tokio::runtime::Builder::new_multi_thread()
    .worker_threads(2)
    .enable_all()
    .build()
    .unwrap();
  ```
  
  **职责：**
- 程序入口点
- 创建和管理 Tokio 运行时
- 初始化 LLDB 调试器
- 处理程序退出清理
- #### 2.3.1.2.  Tokio 工作线程池 (2个工作线程)
  
  ```
  rt.block_on(async { /* 主要异步逻辑 */ })
  ```
  
  **职责：**
- 执行异步 I/O 操作
- 处理网络连接 (TCP/stdio)
- 管理定时器和异步任务
- 提供异步运行时基础设施
- #### 2.3.1.3.  LocalSet 单线程执行器
  
  ```
  // debug_session.rs:196-238
  let local_set = tokio::task::LocalSet::new();
  local_set.spawn_local(async move {
    loop {
        tokio::select! {
            // 主事件循环
        }
    }
  });
  ```
  
  **职责：**
- **调试会话主事件循环**
- 处理 DAP 请求
- 处理 LLDB 事件
- 处理 Python 事件
- 确保调试会话状态的线程安全
- ### 2.3.2.  专用线程（按工作分类的线程）
- #### 2.3.2.1.  LLDB 事件监听线程
  
  ```
  // debug_event_listener.rs:40-63
  thread::spawn(move || {
    let mut event = SBEvent::new();
    loop {
        let received = event_listener.wait_for_event(1, &mut event);
        // 处理事件并发送到异步通道
    }
  });
  ```
  
  **特点：**
- **原生系统线程**（非 Tokio 管理）
- 阻塞式等待 LLDB 事件
- 通过 mpsc::channel 与异步世界通信
- 支持事件过滤（cork/uncork 机制）
- #### 2.3.2.2.  控制台输出处理线程
  
  ```
  // debug_session.rs:245-270
  tokio::spawn(async move {
    let mut con_data = [0u8; 1024];
    loop {
        if let Ok(count) = con_reader.read(&mut con_data).await {
            // 处理控制台输出
        }
    }
  });
  ```
  
  **职责：**
- 读取调试器控制台输出
- 格式化并发送到 DAP 客户端
- 异步处理，不阻塞主循环
- #### 2.3.2.3.  请求取消过滤线程
  
  ```
  // debug_session.rs:337
  tokio::spawn(filter);
  ```
  
  **职责：**
- 处理请求取消逻辑
- 管理取消令牌
- 过滤和路由请求
- #### 2.3.2.4.  DAP 消息分发线程
  
  ```
  // dap_session.rs:52-136
  let dispatcher = async move {
    loop {
        tokio::select! {
            // 处理输入消息
            // 处理输出消息
        }
    }
  };
  ```
  
  **职责：**
- DAP 协议消息的编解码
- 消息路由和分发
- 错误处理和恢复
- ### 2.3.3.  通信机制
- #### 2.3.3.1.  异步通道 (Tokio Channels)
  
  ```
  // 各种 mpsc 和 broadcast 通道
  mpsc::channel::<(u32, RequestArguments, cancellation::Receiver)>(100)
  broadcast::channel::<(u32, RequestArguments)>(100)
  ```
  
  **用途：**
- DAP 请求/响应传递
- 事件广播
- 取消信号传播
- #### 2.3.3.2.  跨线程通信
- **LLDB 事件**: 原生线程 → 异步通道 → 主事件循环
- **控制台输出**: 异步任务 → DAP 事件 → 客户端
- **Python 事件**: Python 解释器 → 异步通道 → 主事件循环
- ### 2.3.4.  线程安全机制
- #### 2.3.4.1.  共享状态管理
  
  ```
  // debug_session.rs:192-193
  let shared_session = Shared::new(debug_session);
  shared_session.try_map(|s| s.self_ref = Initialized(shared_session.clone())).unwrap();
  ```
  
  **特点：**
- 使用 Shared<T> 包装器管理共享状态
- 引用计数和生命周期管理
- 异步安全的状态访问
- #### 2.3.4.2.  LLDB 线程安全
  
  ```
  // debug_session.rs:121
  debugger.set_async_mode(true);
  
  // debug_session.rs:342-347
  fn with_sync_mode<F: Fn() -> R, R>(&self, func: F) -> R {
    self.debugger.set_async_mode(false);
    let result = func();
    self.debugger.set_async_mode(true);
    result
  }
  ```
  
  **机制：**
- LLDB 异步模式：允许多线程访问
- 同步模式包装：某些操作需要独占访问
- 自动模式切换
- #### 2.3.4.3.  取消机制
  
  ```
  // 支持请求取消的完整流程
  cancellation::Receiver → Python 中断 → 请求终止
  ```
- ### 2.3.5.  性能优化特性
- #### 2.3.5.1.  事件过滤
  
  ```
  // debug_event_listener.rs:68-80
  pub fn cork(&self) { /* 暂停事件处理 */ }
  pub fn uncork(&self) { /* 恢复事件处理 */ }
  ```
- #### 2.3.5.2.  异步批处理
- 使用 tokio::select! 实现高效的多路复用
- 非阻塞的事件处理
- 优先级处理（取消 > 事件 > 请求）
- #### 2.3.5.3.  内存管理
- 智能指针和引用计数
- 自动清理和资源释放
- 避免内存泄漏
- ## 2.4.  Extension部分
- ### 2.4.1.  调试会话启动流程
- #### 2.4.1.1.  **第一阶段：Extension 激活与注册**
  
  ```
  // main.ts - Extension 类构造函数
  constructor(context: ExtensionContext) {
    // 注册调试配置提供器 - 两次注册支持不同触发类型
    subscriptions.push(debug.registerDebugConfigurationProvider('lldb', {
        provideDebugConfigurations: (folder, token) =>
            this.provideDebugConfigurations(DebugConfigurationProviderTriggerKind.Initial, folder, token)
    }, DebugConfigurationProviderTriggerKind.Initial));
  
    subscriptions.push(debug.registerDebugConfigurationProvider('lldb', {
        provideDebugConfigurations: (folder, token) =>
            this.provideDebugConfigurations(DebugConfigurationProviderTriggerKind.Dynamic, folder, token),
        resolveDebugConfiguration: (folder, config, token) => this.resolveDebugConfiguration(folder, config, token)
    }, DebugConfigurationProviderTriggerKind.Dynamic));
  
    // 注册调试适配器描述符工厂 - 核心接口
    subscriptions.push(debug.registerDebugAdapterDescriptorFactory('lldb', this));
  }
  ```
- #### 2.4.1.2.  **第二阶段：调试配置解析**
  
  当用户启动调试时，VS Code 会调用 resolveDebugConfiguration 方法：
  
  ```
  async resolveDebugConfiguration(
    folder: WorkspaceFolder | undefined,
    launchConfig: DebugConfiguration,
    cancellation?: CancellationToken
  ): Promise<DebugConfiguration> {
    output.clear();
    
    // 1. 检查先决条件
    if (!await this.checkPrerequisites(folder))
        return undefined;
  
    // 2. 合并工作区设置
    let launchDefaults = getExtensionConfig(folder, 'launch');
    this.mergeWorkspaceSettings(launchConfig, launchDefaults);
  
    // 3. 扩展 dbgconfig 变量
    let dbgconfigConfig = getExtensionConfig(folder, 'dbgconfig');
    launchConfig = util.expandDbgConfig(launchConfig, dbgconfigConfig);
  
    // 4. 处理 Cargo 配置（Rust 项目）
    if (launchConfig.cargo != undefined) {
        let cargo = new Cargo(folder, cancellation);
        let program = await cargo.getProgramFromCargoConfig(launchConfig.cargo);
        delete launchConfig.cargo;
        
        // 扩展 ${cargo:program} 变量
        launchConfig = expandCargo(launchConfig, { program: program });
        
        if (launchConfig.program == undefined) {
            launchConfig.program = program;
        }
        
        // 添加 Rust 语言支持
        if (!launchConfig.sourceLanguages)
            launchConfig.sourceLanguages = [];
        launchConfig.sourceLanguages.push('rust');
    }
  
    // 5. 设置适配器设置
    launchConfig._adapterSettings = this.settingsManager.getAdapterSettings(folder);
  
    return launchConfig;
  }
  ```
- #### 2.4.1.3.  **第三阶段：调试适配器描述符创建**
  
  VS Code 调用 createDebugAdapterDescriptor 来创建调试适配器：
  
  ```
  async createDebugAdapterDescriptor(session: DebugSession, executable: DebugAdapterExecutable | undefined): Promise<DebugAdapterDescriptor> {
    // 1. 准备适配器设置
    let settings = this.settingsManager.getAdapterSettings(session.workspaceFolder);
    let adapterSettings: AdapterSettings = {
        evaluateForHovers: settings.evaluateForHovers,
        commandCompletions: settings.commandCompletions,
    };
    
    if (session.configuration.sourceLanguages) {
        adapterSettings.sourceLanguages = session.configuration.sourceLanguages;
        delete session.configuration.sourceLanguages;
    }
  
    // 2. 创建反向连接器 - 关键组件！
    let authToken = crypto.randomBytes(16).toString('base64');
    let connector = new ReverseAdapterConnector(authToken);
    let port = await connector.listen();
  
    try {
        // 3. 启动调试适配器进程
        await this.startDebugAdapter(session.workspaceFolder, adapterSettings, port, authToken);
        
        // 4. 等待适配器连接
        await connector.accept();
        
        // 5. 返回内联实现
        return new DebugAdapterInlineImplementation(connector);
    } catch (err) {
        this.analyzeStartupError(err);
        throw err;
    }
  }
  ```
- ### 2.4.2.  反向连接机制 (ReverseAdapterConnector)
  
  这是 CodeLLDB 的一个创新设计，适配器主动连接到 VS Code，而不是相反：
  
  ```
  export class ReverseAdapterConnector implements DebugAdapter {
    private server: net.Server = net.createServer();
    private connection: net.Socket;
    private authToken: string;
    private authenticated: boolean = false;
  
    async listen(port: number = 0): Promise<number> {
        // 监听本地端口，等待适配器连接
        return new Promise(resolve => {
            this.server.listen(port, '127.0.0.1', () => {
                let address = this.server.address() as net.AddressInfo;
                resolve(address.port);
            });
        });
    }
  
    async accept(): Promise<void> {
        // 接受适配器的连接
        return new Promise(resolve => {
            this.server.on('connection', socket => {
                this.server.close();
                this.connection = socket;
                socket.on('data', data => this.handleData(data));
                resolve();
            })
        });
    }
  
    private handleData(data: Buffer): void {
        // 处理调试协议消息，包含认证逻辑
        this.rawData.write(data);
        while (true) {
            if (this.contentLength < 0) {
                // 解析 HTTP 风格的头部
                let idx = this.rawData.contents.indexOf('\r\n\r\n')
                if (idx != -1) {
                    let header = this.rawData.head(idx).toString('utf8');
                    let lines = header.split('\r\n');
                    for (let i = 0; i < lines.length; i++) {
                        let pair = lines[i].split(/: +/, 2);
                        if (pair[0] == 'Content-Length') {
                            this.contentLength = parseInt(pair[1]);
                        } else if (pair[0] == 'Auth-Token') {
                            // 验证认证令牌
                            if (pair[1] == this.authToken) {
                                this.authenticated = true;
                            }
                        }
                    }
                    this.rawData.remove(idx + 4);
                    continue;
                }
            } else {
                // 处理 JSON 消息体
                if (!this.authenticated) {
                    this.dispose();
                    return;
                }
                
                if (this.rawData.length >= this.contentLength) {
                    let message = this.rawData.head(this.contentLength);
                    if (message.length > 0) {
                        try {
                            let msg = JSON.parse(message.toString('utf8')) as DebugProtocolMessage;
                            this.onDidSendMessageEmitter.fire(msg);
                        } catch (err) {
                            console.log('Error handling data: ' + err.toString());
                        }
                    }
                    this.rawData.remove(this.contentLength);
                    this.contentLength = -1;
                    continue;
                }
            }
            break;
        }
    }
  }
  ```
- ### 2.4.3.  调试适配器进程启动
  
  ```
  async startDebugAdapter(
    folder: WorkspaceFolder | undefined,
    adapterSettings: AdapterSettings,
    connectPort: number,
    authToken: string
  ): Promise<ChildProcess> {
    let config = getExtensionConfig(folder);
    let adapterEnv = config.get<any>('adapterEnv', {});
    let verboseLogging = config.get<boolean>('verboseLogging');
    
    // 配置 LLDB 原生 PDB 读取器
    if (config.get<boolean>('useNativePDBReader'))
        adapterEnv['LLDB_USE_NATIVE_PDB_READER'] = 'true';
    
    // 查找 liblldb 库
    let [liblldb] = await this.getAdapterDylibs(config);
  
    output.appendLine('Launching adapter');
    output.appendLine(`liblldb: ${liblldb}`);
    output.appendLine(`environment: ${inspect(adapterEnv)}`);
    output.appendLine(`settings: ${inspect(adapterSettings)}`);
  
    // 启动适配器进程
    let adapterProcess = await adapter.start(liblldb, {
        extensionRoot: this.context.extensionPath,
        extraEnv: adapterEnv,
        workDir: workspace.rootPath,
        port: connectPort,
        connect: true,  // 适配器连接到 VS Code
        authToken: authToken,
        adapterSettings: adapterSettings,
        verboseLogging: verboseLogging
    });
  
    // 记录进程输出
    util.logProcessOutput(adapterProcess, output);
  
    // 监听进程退出
    adapterProcess.on('exit', async (code, signal) => {
        output.appendLine(`Debug adapter exit code=${code}, signal=${signal}.`);
        if (code != 0) {
            let result = await window.showErrorMessage('Oops!  The debug adapter has terminated abnormally.', 'Open log');
            if (result != undefined) {
                output.show();
            }
        }
    });
    return adapterProcess;
  }
  ```
- ### 2.4.4.  Rust 适配器进程启动详情
  
  ```
  // novsc/adapter.ts
  export async function start(
    liblldb: string,
    options: AdapterStartOptions
  ): Promise<cp.ChildProcess> {
    let spawnParams = await getSpawnParams(liblldb, options);
    spawnParams.options.stdio = ['ignore', 'pipe', 'pipe'];
    return cp.spawn(spawnParams.command, spawnParams.args, spawnParams.options);
  }
  
  export async function getSpawnParams(
    liblldb: string,
    options: AdapterStartOptions
  ): Promise<ProcessSpawnParams> {
    let executable = path.join(options.extensionRoot, 'adapter', 'codelldb');
    let portAction = options.connect ? '--connect' : '--port';
    let args = ['--liblldb', liblldb, portAction, options.port.toString()];
    
    if (options.authToken) {
        args = args.concat(['--auth-token', options.authToken]);
    }
    if (options.adapterSettings) {
        args = args.concat(['--settings', JSON.stringify(options.adapterSettings)]);
    }
    
    let env = getAdapterEnv(options.extraEnv);
    env['RUST_TRACEBACK'] = '1';
    if (options.verboseLogging) {
        env['RUST_LOG'] = 'error,codelldb=debug';
    }
  
    // 处理 Apple Silicon 架构
    if (await isRosetta()) {
        args = ['--arm64', executable].concat(args);
        executable = 'arch';
    }
  
    return {
        command: executable,
        args: args,
        options: {
            env: env,
            cwd: options.workDir
        }
    }
  }
  ```
- ### 2.4.5.  Cargo 集成机制
  
  对于 Rust 项目，CodeLLDB 提供了深度的 Cargo 集成：
  
  ```
  export class Cargo {
    public async getProgramFromCargoConfig(cargoConfig: CargoConfig): Promise<string> {
        let artifacts: CompilationArtifact[] | Error = [];
  
        // 创建自定义执行任务
        let execution = new CustomExecution(async taskDef => {
            let outputEmitter = new EventEmitter<string>();
            let doneEmitter = new EventEmitter<number>();
            let pty: Pseudoterminal = {
                onDidWrite: outputEmitter.event,
                onDidClose: doneEmitter.event,
                open: async () => {
                    try {
                        let cargoArgs = taskDef.args || [];
                        let pos = cargoArgs.indexOf('--');
                        // 插入 JSON 消息格式参数
                        cargoArgs.splice(pos >= 0 ? pos : cargoArgs.length, 0, '--message-format=json', '--color=always');
  
                        outputEmitter.fire('Running `cargo ' + cargoArgs.join(' ') + '`...\r\n');
                        artifacts = await this.getCargoArtifacts(
                            taskDef.args,
                            taskDef.env,
                            taskDef.cwd,
                            message => outputEmitter.fire(message.replace(/\n/g, '\r\n'))
                        );
                        doneEmitter.fire(0);
                    } catch (err) {
                        artifacts = err;
                        let msg = formatError(err);
                        outputEmitter.fire(msg.replace(/\n/g, '\r\n'));
                        output.appendLine(msg);
                        doneEmitter.fire(1);
                    }
                },
                close: () => { }
            };
            return pty;
        });
  
        // 执行 Cargo 任务
        let task = new Task(cargoConfig, this.workspaceFolder, 'cargo', 'CodeLLDB', execution, cargoConfig.problemMatcher);
        task.presentationOptions.clear = true;
        let taskExecution = await tasks.executeTask(task);
  
        // 等待任务完成
        await new Promise<void>(resolve => {
            tasks.onDidEndTask(e => {
                if (e.execution == taskExecution) {
                    resolve();
                }
            });
        });
  
        if (artifacts instanceof Error)
            throw new ErrorWithCause('Cargo task failed.', { cause: artifacts });
  
        return this.getProgramFromArtifacts(artifacts, cargoConfig.filter);
    }
  }
  ```
- ### 2.4.6.  配置变量扩展机制
  
  CodeLLDB 支持多种配置变量的扩展：
  
  ```
  // expand.ts
  export function expandVariablesInObject(obj: any, expander: Expander): any {
    if (typeof obj == 'string' || obj instanceof String)
        return expandVariables(obj, expander);
  
    if (isScalarValue(obj))
        return obj;
  
    if (obj instanceof Array)
        return obj.map(v => expandVariablesInObject(v, expander));
  
    for (let prop of Object.keys(obj))
        obj[prop] = expandVariablesInObject(obj[prop], expander)
    return obj;
  }
  
  // 支持的变量类型：
  // ${env:VAR_NAME} - 环境变量
  // ${cargo:program} - Cargo 程序路径
  // ${dbgconfig:key} - 调试配置变量
  ```
- ### 2.4.7.  调试会话生命周期管理
  
  一旦调试会话建立，Extension 通过多个视图组件管理调试状态：
- **ModulesView**: 显示加载的模块
- **ExcludedCallersView**: 管理断点排除规则
- **WebviewManager**: 处理自定义 WebView 面板
- **AdapterSettingsManager**: 管理调试器设置
- ### 2.4.8.  与 Rust 适配器的通信
  
  从您提到的 debug_session.rs 文件中的 handle_configuration_done() 可以看出，Rust 适配器端会处理配置完成事件，这标志着调试会话正式开始。
- ### 2.4.9.   完整启动流程总结
- **Extension 激活** → 注册各种提供器和命令
- **用户启动调试** → VS Code 调用 resolveDebugConfiguration
- **配置解析** → 处理 Cargo、变量扩展、设置合并
- **适配器描述符创建** → 创建 ReverseAdapterConnector
- **适配器进程启动** → 启动 Rust 编写的 codelldb 进程
- **反向连接建立** → 适配器连接到 VS Code
- **认证和握手** → 验证 auth token
- **调试会话开始** → 发送 configurationDone 事件
  
  这个架构的优势在于：
- **解耦设计**：Extension 和适配器进程分离
- **安全性**：使用 auth token 认证
- **灵活性**：支持多种启动方式和配置
- **可扩展性**：模块化的功能组件
- **跨平台**：统一的接口处理不同平台差异
  
  ```
  // main.ts - Extension 类构造函数
  constructor(context: ExtensionContext) {
    // 注册调试配置提供器 - 两次注册支持不同触发类型
    subscriptions.push(debug.registerDebugConfigurationProvider('lldb', {
        provideDebugConfigurations: (folder, token) =>
            this.provideDebugConfigurations(DebugConfigurationProviderTriggerKind.Initial, folder, token)
    }, DebugConfigurationProviderTriggerKind.Initial));
  
    subscriptions.push(debug.registerDebugConfigurationProvider('lldb', {
        provideDebugConfigurations: (folder, token) =>
            this.provideDebugConfigurations(DebugConfigurationProviderTriggerKind.Dynamic, folder, token),
        resolveDebugConfiguration: (folder, config, token) => this.resolveDebugConfiguration(folder, config, token)
    }, DebugConfigurationProviderTriggerKind.Dynamic));
  
    // 注册调试适配器描述符工厂 - 核心接口
    subscriptions.push(debug.registerDebugAdapterDescriptorFactory('lldb', this));
  }
  ```
  
  ```
  async resolveDebugConfiguration(
    folder: WorkspaceFolder | undefined,
    launchConfig: DebugConfiguration,
    cancellation?: CancellationToken
  ): Promise<DebugConfiguration> {
    output.clear();
    
    // 1. 检查先决条件
    if (!await this.checkPrerequisites(folder))
        return undefined;
  
    // 2. 合并工作区设置
    let launchDefaults = getExtensionConfig(folder, 'launch');
    this.mergeWorkspaceSettings(launchConfig, launchDefaults);
  
    // 3. 扩展 dbgconfig 变量
    let dbgconfigConfig = getExtensionConfig(folder, 'dbgconfig');
    launchConfig = util.expandDbgConfig(launchConfig, dbgconfigConfig);
  
    // 4. 处理 Cargo 配置（Rust 项目）
    if (launchConfig.cargo != undefined) {
        let cargo = new Cargo(folder, cancellation);
        let program = await cargo.getProgramFromCargoConfig(launchConfig.cargo);
        delete launchConfig.cargo;
        
        // 扩展 ${cargo:program} 变量
        launchConfig = expandCargo(launchConfig, { program: program });
        
        if (launchConfig.program == undefined) {
            launchConfig.program = program;
        }
        
        // 添加 Rust 语言支持
        if (!launchConfig.sourceLanguages)
            launchConfig.sourceLanguages = [];
        launchConfig.sourceLanguages.push('rust');
    }
  
    // 5. 设置适配器设置
    launchConfig._adapterSettings = this.settingsManager.getAdapterSettings(folder);
  
    return launchConfig;
  }
  ```
  
  ```
  async createDebugAdapterDescriptor(session: DebugSession, executable: DebugAdapterExecutable | undefined): Promise<DebugAdapterDescriptor> {
    // 1. 准备适配器设置
    let settings = this.settingsManager.getAdapterSettings(session.workspaceFolder);
    let adapterSettings: AdapterSettings = {
        evaluateForHovers: settings.evaluateForHovers,
        commandCompletions: settings.commandCompletions,
    };
    
    if (session.configuration.sourceLanguages) {
        adapterSettings.sourceLanguages = session.configuration.sourceLanguages;
        delete session.configuration.sourceLanguages;
    }
  
    // 2. 创建反向连接器 - 关键组件！
    let authToken = crypto.randomBytes(16).toString('base64');
    let connector = new ReverseAdapterConnector(authToken);
    let port = await connector.listen();
  
    try {
        // 3. 启动调试适配器进程
        await this.startDebugAdapter(session.workspaceFolder, adapterSettings, port, authToken);
        
        // 4. 等待适配器连接
        await connector.accept();
        
        // 5. 返回内联实现
        return new DebugAdapterInlineImplementation(connector);
    } catch (err) {
        this.analyzeStartupError(err);
        throw err;
    }
  }
  ```
  
  ```
  export class ReverseAdapterConnector implements DebugAdapter {
    private server: net.Server = net.createServer();
    private connection: net.Socket;
    private authToken: string;
    private authenticated: boolean = false;
  
    async listen(port: number = 0): Promise<number> {
        // 监听本地端口，等待适配器连接
        return new Promise(resolve => {
            this.server.listen(port, '127.0.0.1', () => {
                let address = this.server.address() as net.AddressInfo;
                resolve(address.port);
            });
        });
    }
  
    async accept(): Promise<void> {
        // 接受适配器的连接
        return new Promise(resolve => {
            this.server.on('connection', socket => {
                this.server.close();
                this.connection = socket;
                socket.on('data', data => this.handleData(data));
                resolve();
            })
        });
    }
  
    private handleData(data: Buffer): void {
        // 处理调试协议消息，包含认证逻辑
        this.rawData.write(data);
        while (true) {
            if (this.contentLength < 0) {
                // 解析 HTTP 风格的头部
                let idx = this.rawData.contents.indexOf('\r\n\r\n')
                if (idx != -1) {
                    let header = this.rawData.head(idx).toString('utf8');
                    let lines = header.split('\r\n');
                    for (let i = 0; i < lines.length; i++) {
                        let pair = lines[i].split(/: +/, 2);
                        if (pair[0] == 'Content-Length') {
                            this.contentLength = parseInt(pair[1]);
                        } else if (pair[0] == 'Auth-Token') {
                            // 验证认证令牌
                            if (pair[1] == this.authToken) {
                                this.authenticated = true;
                            }
                        }
                    }
                    this.rawData.remove(idx + 4);
                    continue;
                }
            } else {
                // 处理 JSON 消息体
                if (!this.authenticated) {
                    this.dispose();
                    return;
                }
                
                if (this.rawData.length >= this.contentLength) {
                    let message = this.rawData.head(this.contentLength);
                    if (message.length > 0) {
                        try {
                            let msg = JSON.parse(message.toString('utf8')) as DebugProtocolMessage;
                            this.onDidSendMessageEmitter.fire(msg);
                        } catch (err) {
                            console.log('Error handling data: ' + err.toString());
                        }
                    }
                    this.rawData.remove(this.contentLength);
                    this.contentLength = -1;
                    continue;
                }
            }
            break;
        }
    }
  }
  ```
  
  ```
  async startDebugAdapter(
    folder: WorkspaceFolder | undefined,
    adapterSettings: AdapterSettings,
    connectPort: number,
    authToken: string
  ): Promise<ChildProcess> {
    let config = getExtensionConfig(folder);
    let adapterEnv = config.get<any>('adapterEnv', {});
    let verboseLogging = config.get<boolean>('verboseLogging');
    
    // 配置 LLDB 原生 PDB 读取器
    if (config.get<boolean>('useNativePDBReader'))
        adapterEnv['LLDB_USE_NATIVE_PDB_READER'] = 'true';
    
    // 查找 liblldb 库
    let [liblldb] = await this.getAdapterDylibs(config);
  
    output.appendLine('Launching adapter');
    output.appendLine(`liblldb: ${liblldb}`);
    output.appendLine(`environment: ${inspect(adapterEnv)}`);
    output.appendLine(`settings: ${inspect(adapterSettings)}`);
  
    // 启动适配器进程
    let adapterProcess = await adapter.start(liblldb, {
        extensionRoot: this.context.extensionPath,
        extraEnv: adapterEnv,
        workDir: workspace.rootPath,
        port: connectPort,
        connect: true,  // 适配器连接到 VS Code
        authToken: authToken,
        adapterSettings: adapterSettings,
        verboseLogging: verboseLogging
    });
  
    // 记录进程输出
    util.logProcessOutput(adapterProcess, output);
  
    // 监听进程退出
    adapterProcess.on('exit', async (code, signal) => {
        output.appendLine(`Debug adapter exit code=${code}, signal=${signal}.`);
        if (code != 0) {
            let result = await window.showErrorMessage('Oops!  The debug adapter has terminated abnormally.', 'Open log');
            if (result != undefined) {
                output.show();
            }
        }
    });
    return adapterProcess;
  }
  ```
  
  ```
  // novsc/adapter.ts
  export async function start(
    liblldb: string,
    options: AdapterStartOptions
  ): Promise<cp.ChildProcess> {
    let spawnParams = await getSpawnParams(liblldb, options);
    spawnParams.options.stdio = ['ignore', 'pipe', 'pipe'];
    return cp.spawn(spawnParams.command, spawnParams.args, spawnParams.options);
  }
  
  export async function getSpawnParams(
    liblldb: string,
    options: AdapterStartOptions
  ): Promise<ProcessSpawnParams> {
    let executable = path.join(options.extensionRoot, 'adapter', 'codelldb');
    let portAction = options.connect ? '--connect' : '--port';
    let args = ['--liblldb', liblldb, portAction, options.port.toString()];
    
    if (options.authToken) {
        args = args.concat(['--auth-token', options.authToken]);
    }
    if (options.adapterSettings) {
        args = args.concat(['--settings', JSON.stringify(options.adapterSettings)]);
    }
    
    let env = getAdapterEnv(options.extraEnv);
    env['RUST_TRACEBACK'] = '1';
    if (options.verboseLogging) {
        env['RUST_LOG'] = 'error,codelldb=debug';
    }
  
    // 处理 Apple Silicon 架构
    if (await isRosetta()) {
        args = ['--arm64', executable].concat(args);
        executable = 'arch';
    }
  
    return {
        command: executable,
        args: args,
        options: {
            env: env,
            cwd: options.workDir
        }
    }
  }
  ```
  
  ```
  export class Cargo {
    public async getProgramFromCargoConfig(cargoConfig: CargoConfig): Promise<string> {
        let artifacts: CompilationArtifact[] | Error = [];
  
        // 创建自定义执行任务
        let execution = new CustomExecution(async taskDef => {
            let outputEmitter = new EventEmitter<string>();
            let doneEmitter = new EventEmitter<number>();
            let pty: Pseudoterminal = {
                onDidWrite: outputEmitter.event,
                onDidClose: doneEmitter.event,
                open: async () => {
                    try {
                        let cargoArgs = taskDef.args || [];
                        let pos = cargoArgs.indexOf('--');
                        // 插入 JSON 消息格式参数
                        cargoArgs.splice(pos >= 0 ? pos : cargoArgs.length, 0, '--message-format=json', '--color=always');
  
                        outputEmitter.fire('Running `cargo ' + cargoArgs.join(' ') + '`...\r\n');
                        artifacts = await this.getCargoArtifacts(
                            taskDef.args,
                            taskDef.env,
                            taskDef.cwd,
                            message => outputEmitter.fire(message.replace(/\n/g, '\r\n'))
                        );
                        doneEmitter.fire(0);
                    } catch (err) {
                        artifacts = err;
                        let msg = formatError(err);
                        outputEmitter.fire(msg.replace(/\n/g, '\r\n'));
                        output.appendLine(msg);
                        doneEmitter.fire(1);
                    }
                },
                close: () => { }
            };
            return pty;
        });
  
        // 执行 Cargo 任务
        let task = new Task(cargoConfig, this.workspaceFolder, 'cargo', 'CodeLLDB', execution, cargoConfig.problemMatcher);
        task.presentationOptions.clear = true;
        let taskExecution = await tasks.executeTask(task);
  
        // 等待任务完成
        await new Promise<void>(resolve => {
            tasks.onDidEndTask(e => {
                if (e.execution == taskExecution) {
                    resolve();
                }
            });
        });
  
        if (artifacts instanceof Error)
            throw new ErrorWithCause('Cargo task failed.', { cause: artifacts });
  
        return this.getProgramFromArtifacts(artifacts, cargoConfig.filter);
    }
  }
  ```
  
  ```
  // expand.ts
  export function expandVariablesInObject(obj: any, expander: Expander): any {
    if (typeof obj == 'string' || obj instanceof String)
        return expandVariables(obj, expander);
  
    if (isScalarValue(obj))
        return obj;
  
    if (obj instanceof Array)
        return obj.map(v => expandVariablesInObject(v, expander));
  
    for (let prop of Object.keys(obj))
        obj[prop] = expandVariablesInObject(obj[prop], expander)
    return obj;
  }
  
  // 支持的变量类型：
  // ${env:VAR_NAME} - 环境变量
  // ${cargo:program} - Cargo 程序路径
  // ${dbgconfig:key} - 调试配置变量
  ```
- # 3.  技术点
- ## 3.1.  mpsc：channel
- ### 3.1.1.  基本概念
  
  mpsc 是 **Multiple Producer, Single Consumer** 的缩写，即**多生产者，单消费者**通道。
  
  ```
  use tokio::sync::mpsc;
  
  // 创建一个容量为 100 的通道
  let (sender, receiver) = mpsc::channel::<MessageType>(100);
  ```
- ### 3.1.2.  核心特性
- #### 3.1.2.1.  多生产者 (Multiple Producer)
  
  ```
  let (sender, mut receiver) = mpsc::channel::<String>(10);
  
  // 可以克隆发送端，创建多个生产者
  let sender1 = sender.clone();
  let sender2 = sender.clone();
  let sender3 = sender.clone();
  
  // 多个线程可以同时发送消息
  tokio::spawn(async move {
    sender1.send("来自线程1的消息".to_string()).await.unwrap();
  });
  
  tokio::spawn(async move {
    sender2.send("来自线程2的消息".to_string()).await.unwrap();
  });
  
  tokio::spawn(async move {
    sender3.send("来自线程3的消息".to_string()).await.unwrap();
  });
  ```
- #### 3.1.2.2.  单消费者 (Single Consumer)
  
  ```
  // 只有一个接收端，不能克隆
  while let Some(message) = receiver.recv().await {
    println!("收到消息: {}", message);
  }
  ```
- ### 3.1.3.  在 CodeLLDB 中的具体应用
  
  让我查看一些具体的使用例子：
  
  ```
  use tokio::sync::mpsc;
  
  // 创建一个容量为 100 的通道
  let (sender, receiver) = mpsc::channel::<MessageType>(100);
  ```
  
  ```
  let (sender, mut receiver) = mpsc::channel::<String>(10);
  
  // 可以克隆发送端，创建多个生产者
  let sender1 = sender.clone();
  let sender2 = sender.clone();
  let sender3 = sender.clone();
  
  // 多个线程可以同时发送消息
  tokio::spawn(async move {
    sender1.send("来自线程1的消息".to_string()).await.unwrap();
  });
  
  tokio::spawn(async move {
    sender2.send("来自线程2的消息".to_string()).await.unwrap();
  });
  
  tokio::spawn(async move {
    sender3.send("来自线程3的消息".to_string()).await.unwrap();
  });
  ```
  
  ```
  // 只有一个接收端，不能克隆
  while let Some(message) = receiver.recv().await {
    println!("收到消息: {}", message);
  }
  ```
- #### 3.1.3.1.  LLDB 事件通信
  
  ```
  // debug_event_listener.rs:37
  let (sender, receiver) = mpsc::channel(channel_capacity);
  
  // 生产者：LLDB 事件监听线程 (原生线程)
  thread::spawn(move || {
    let mut event = SBEvent::new();
    loop {
        let received = event_listener.wait_for_event(1, &mut event);
        if received && !state.was_corked {
            // 发送 LLDB 事件到异步世界
            match sender.try_send(event) {
                Ok(_) => {}
                Err(TrySendError::Full(_)) => error!("Could not send event"),
                Err(TrySendError::Closed(_)) => break,
            }
        }
    }
  });
  
  // 消费者：主事件循环 (异步任务)
  tokio::select! {
    Some(event) = debug_events_stream.recv() => {
        shared_session.map(|s| s.handle_debug_event(event)).await;
    }
  }
  ```
- #### 3.1.3.2.  Python 事件通信
  
  ```
  // python.rs:184
  let (sender, receiver) = mpsc::channel(100);
  
  // 生产者：Python 回调函数 (C ABI)
  unsafe extern "C" fn python_event_callback(
    interface: *mut PythonInterface,
    session_id: c_int,
    event: *const c_char,
  ) {
    // 解析 Python 事件并发送
    match serde_json::from_slice::<PythonEvent>(CStr::from_ptr(event).to_bytes()) {
        Ok(event) => (*interface).dispatch_python_event(session_id as u64, event),
        Err(err) => error!("{}", err),
    }
  }
  
  fn dispatch_python_event(&self, session_id: u64, event: PythonEvent) {
    let senders = self.session_event_senders.lock().unwrap();
    if let Some(sender) = senders.get(&session_id) {
        log_errors!(sender.try_send(event));  // 发送到通道
    }
  }
  
  // 消费者：主事件循环
  tokio::select! {
    Some(event) = python_events_stream.recv() => {
        shared_session.map(|s| s.handle_python_event(event)).await;
    }
  }
  ```
- #### 3.1.3.3.  DAP 消息输出通信
  
  ```
  // dap_session.rs:42
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  
  // 多个生产者：调试会话的各种组件
  impl DebugSession {
    fn send_response(&self, request_seq: u32, result: Result<ResponseBody, Error>) {
        let response = /* 构建响应 */;
        // 发送响应到输出通道
        log_errors!(self.dap_session.try_send_response(response));
    }
    
    fn send_event(&self, event_body: EventBody) {
        // 发送事件到输出通道
        log_errors!(self.dap_session.try_send_event(event_body));
    }
  }
  
  // 消费者：DAP 消息分发器
  let dispatcher = async move {
    loop {
        tokio::select! {
            // 处理输出消息
            Some((message_type, response_sender)) = out_receiver.recv() => {
                message_seq += 1;
                let message = ProtocolMessage {
                    seq: message_seq,
                    type_: message_type
                };
                // 发送到客户端
                log_errors!(channel.send(message).await);
            }
        }
    }
  };
  ```
- ### 3.1.4.  其他通道类型对比
- #### 3.1.4.1.   broadcast::channel  (多生产者，多消费者)
  
  ```
  // dap_session.rs:40-41
  let requests_sender = Arc::new(broadcast::channel::<(u32, RequestArguments)>(100).0);
  let events_sender = Arc::new(broadcast::channel::<EventBody>(100).0);
  
  // 用于事件广播，多个订阅者可以接收同一消息
  pub fn subscribe_requests(&self) -> Result<broadcast::Receiver<(u32, RequestArguments)>, Error> {
    match self.requests_sender.upgrade() {
        Some(r) => Ok(r.subscribe()),  // 创建新的接收者
        None => Err("DAP session is gone".into()),
    }
  }
  ```
- #### 3.1.4.2.   oneshot::channel  (单次通信)
  
  ```
  // 用于请求-响应模式
  let (resp_sender, resp_receiver) = oneshot::channel();
  let request = ProtocolMessageType::Request(request_args);
  let out_sender = self.out_sender.clone();
  
  async move {
    out_sender.send((request, Some(resp_sender))).await?;
    let result = resp_receiver.await?;  // 等待单次响应
    // 处理响应
  }
  ```
- ## 3.2.  有界通道
- ### 3.2.1.  有界通道的基本概念
  
  **有界通道**是指具有固定容量限制的通信通道，当通道中的消息数量达到容量上限时，发送操作会被阻塞或失败。
  
  ```
  // 创建不同容量的有界通道
  let (sender, receiver) = mpsc::channel(100);   // 容量为 100
  let (sender, receiver) = mpsc::channel(1000);  // 容量为 1000
  let (sender, receiver) = mpsc::channel(1);     // 容量为 1
  ```
- ### 3.2.2.  容量单位详解
- #### 3.2.2.1.  容量单位是什么？
  
  **容量单位是消息的个数**，不是字节数或其他单位。
  
  ```
  // 示例：容量为 3 的通道
  let (sender, mut receiver) = mpsc::channel::<String>(3);
  
  // 可以发送 3 个消息而不阻塞
  sender.send("消息1".to_string()).await.unwrap();  // 通道: [消息1]
  sender.send("消息2".to_string()).await.unwrap();  // 通道: [消息1, 消息2]  
  sender.send("消息3".to_string()).await.unwrap();  // 通道: [消息1, 消息2, 消息3]
  
  // 第4个消息会阻塞，直到有消息被接收
  // sender.send("消息4".to_string()).await; // 会等待直到通道有空间
  ```
- #### 3.2.2.2.  不同数据类型的容量含义
  
  ```
  // 每个消息占用一个容量单位，无论消息大小
  let (sender1, _) = mpsc::channel::<u8>(100);        // 100个 u8 值
  let (sender2, _) = mpsc::channel::<String>(100);    // 100个 String 对象
  let (sender3, _) = mpsc::channel::<Vec<u8>>(100);   // 100个 Vec<u8> 对象
  
  // 即使 String 很长或 Vec 很大，每个都只占用 1 个容量单位
  sender2.send("很短的字符串".to_string()).await.unwrap();           // 占用 1 个单位
  sender2.send("非常非常长的字符串...".repeat(1000)).await.unwrap(); // 也只占用 1 个单位
  ```
- ### 3.2.3.  CodeLLDB 中的容量配置
  
  让我查看具体的容量配置：
  
  ```
  // 创建不同容量的有界通道
  let (sender, receiver) = mpsc::channel(100);   // 容量为 100
  let (sender, receiver) = mpsc::channel(1000);  // 容量为 1000
  let (sender, receiver) = mpsc::channel(1);     // 容量为 1
  ```
  
  ```
  // 示例：容量为 3 的通道
  let (sender, mut receiver) = mpsc::channel::<String>(3);
  
  // 可以发送 3 个消息而不阻塞
  sender.send("消息1".to_string()).await.unwrap();  // 通道: [消息1]
  sender.send("消息2".to_string()).await.unwrap();  // 通道: [消息1, 消息2]  
  sender.send("消息3".to_string()).await.unwrap();  // 通道: [消息1, 消息2, 消息3]
  
  // 第4个消息会阻塞，直到有消息被接收
  // sender.send("消息4".to_string()).await; // 会等待直到通道有空间
  ```
  
  ```
  // 每个消息占用一个容量单位，无论消息大小
  let (sender1, _) = mpsc::channel::<u8>(100);        // 100个 u8 值
  let (sender2, _) = mpsc::channel::<String>(100);    // 100个 String 对象
  let (sender3, _) = mpsc::channel::<Vec<u8>>(100);   // 100个 Vec<u8> 对象
  
  // 即使 String 很长或 Vec 很大，每个都只占用 1 个容量单位
  sender2.send("很短的字符串".to_string()).await.unwrap();           // 占用 1 个单位
  sender2.send("非常非常长的字符串...".repeat(1000)).await.unwrap(); // 也只占用 1 个单位
  ```
- #### 3.2.3.1.  不同用途的容量配置
  
  ```
  // 1. DAP 输出消息通道 - 容量 1000
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  ```
  
  **用途**: 发送 DAP 响应和事件到客户端
  **容量大的原因**:
- 高频输出 (调试事件、变量更新、控制台输出)
- 避免阻塞调试会话
- 缓冲突发消息
  
  ```
  // 2. Python 事件通道 - 容量 100  
  let (sender, receiver) = mpsc::channel(100);
  ```
  
  **用途**: Python 脚本事件传递
  **中等容量的原因**:
- Python 事件频率中等
- 平衡内存使用和响应性
  
  ```
  // 3. 占位符通道 - 容量 1
  let (_, receiver) = mpsc::channel(1);
  ```
  
  **用途**: 当没有 Python 支持时的占位符
  **容量小的原因**: 不会实际使用，最小容量即可
- ### 3.2.4.  内存使用分析
- #### 3.2.4.1.  通道的内存占用
  
  ```
  // 通道内存 = 容量 × 单个消息大小 + 通道开销
  struct LargeMessage {
    data: Vec<u8>, // 假设 1MB
    metadata: String,
  }
  
  let (sender, receiver) = mpsc::channel::<LargeMessage>(100);
  // 最大内存使用: 100 × 1MB = 100MB + 通道开销
  ```
- #### 3.2.4.2.  CodeLLDB 中的内存考虑
  
  ```
  // DAP 消息类型
  pub enum ProtocolMessageType {
    Request(RequestArguments),    // 通常几KB
    Response(Response),           // 通常几KB  
    Event(EventBody),            // 变化很大，从几字节到几MB
  }
  
  // 容量 1000 的通道
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  // 在最坏情况下可能占用: 1000 × 平均消息大小
  ```
- ### 3.2.5.  容量选择策略
- #### 3.2.5.1.  容量选择考虑因素
  
  ```
  // 1. 低延迟场景 - 小容量
  let (sender, receiver) = mpsc::channel(1);  // 强制同步
  // 优点: 低延迟，及时背压
  // 缺点: 容易阻塞
  
  // 2. 高吞吐场景 - 大容量  
  let (sender, receiver) = mpsc::channel(10000);
  // 优点: 高吞吐，缓冲突发
  // 缺点: 高内存使用，延迟可能增加
  
  // 3. 平衡场景 - 中等容量
  let (sender, receiver) = mpsc::channel(100);
  // 平衡延迟、吞吐和内存使用
  ```
- #### 3.2.5.2.  CodeLLDB 的容量选择逻辑
  
  | 通道用途 | 容量 | 选择原因 |
  | DAP 输出 | 1000 | 高频输出，需要大缓冲 |
  | Python 事件 | 100 | 中频事件，平衡性能和内存 |
  | LLDB 事件 | 可配置 | 根据调试复杂度调整 |
  | 请求过滤 | 100 | 中等请求量 |
- ### 3.2.6.  无界通道对比
  
  ```
  // 无界通道 (如果存在)
  let (sender, receiver) = mpsc::unbounded_channel();
  
  // 优点: 永不阻塞
  // 缺点: 可能导致内存溢出
  ```
  
  Tokio 的 mpsc 默认都是有界的，这是一个重要的设计决策，防止内存泄漏和系统崩溃。
- ### 3.2.7.  总结
- **容量单位**: 消息个数，不是字节数
- **背压控制**: 通过容量限制实现流量控制
- **内存管理**: 容量 × 消息大小 = 最大内存使用
- **性能权衡**: 容量大小影响延迟、吞吐和内存使用
- **错误处理**: try_send vs send 提供不同的背压策略
  
  有界通道是构建健壮异步系统的重要工具，通过合理的容量配置可以在性能和资源使用之间找到平衡。
  
  ```
  // 1. DAP 输出消息通道 - 容量 1000
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  ```
  
  ```
  // 2. Python 事件通道 - 容量 100  
  let (sender, receiver) = mpsc::channel(100);
  ```
  
  ```
  // 3. 占位符通道 - 容量 1
  let (_, receiver) = mpsc::channel(1);
  ```
  
  ```
  // 1. DAP 输出消息通道 - 容量 1000
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  ```
  
  ```
  // 2. Python 事件通道 - 容量 100  
  let (sender, receiver) = mpsc::channel(100);
  ```
  
  ```
  // 3. 占位符通道 - 容量 1
  let (_, receiver) = mpsc::channel(1);
  ```
  
  ```
  // 快速生产者 vs 慢速消费者的问题
  async fn fast_producer(sender: mpsc::Sender<String>) {
    for i in 0..1000000 {
        // 快速产生消息
        sender.send(format!("消息 {}", i)).await.unwrap();
    }
  }
  
  async fn slow_consumer(mut receiver: mpsc::Receiver<String>) {
    while let Some(msg) = receiver.recv().await {
        // 慢速处理 (模拟复杂处理)
        tokio::time::sleep(Duration::from_millis(100)).await;
        println!("处理: {}", msg);
    }
  }
  ```
  
  ```
  // 容量为 10 的通道
  let (sender, mut receiver) = mpsc::channel::<String>(10);
  
  // 当通道满时的不同处理方式
  
  // 方式1: 异步等待 (阻塞直到有空间)
  match sender.send("消息".to_string()).await {
    Ok(_) => println!("发送成功"),
    Err(_) => println!("通道已关闭"),
  }
  
  // 方式2: 非阻塞尝试 (立即返回结果)
  match sender.try_send("消息".to_string()) {
    Ok(_) => println!("发送成功"),
    Err(TrySendError::Full(_)) => {
        println!("通道已满，消息被丢弃");
        // 可以选择丢弃、重试或其他策略
    }
    Err(TrySendError::Closed(_)) => {
        println!("通道已关闭");
    }
  }
  ```
  
  ```
  // debug_event_listener.rs - LLDB 事件处理
  match sender.try_send(event) {
    Ok(_) => {}
    Err(err) => match err {
        TrySendError::Full(_) => {
            // 通道满时记录错误但不阻塞
            error!("Could not send event: {:?}", err);
        }
        TrySendError::Closed(_) => break, // 通道关闭，退出循环
    },
  }
  ```
  
  ```
  // 通道内存 = 容量 × 单个消息大小 + 通道开销
  struct LargeMessage {
    data: Vec<u8>, // 假设 1MB
    metadata: String,
  }
  
  let (sender, receiver) = mpsc::channel::<LargeMessage>(100);
  // 最大内存使用: 100 × 1MB = 100MB + 通道开销
  ```
  
  ```
  // DAP 消息类型
  pub enum ProtocolMessageType {
    Request(RequestArguments),    // 通常几KB
    Response(Response),           // 通常几KB  
    Event(EventBody),            // 变化很大，从几字节到几MB
  }
  
  // 容量 1000 的通道
  let (out_sender, mut out_receiver) = mpsc::channel(1000);
  // 在最坏情况下可能占用: 1000 × 平均消息大小
  ```
  
  ```
  // 1. 低延迟场景 - 小容量
  let (sender, receiver) = mpsc::channel(1);  // 强制同步
  // 优点: 低延迟，及时背压
  // 缺点: 容易阻塞
  
  // 2. 高吞吐场景 - 大容量  
  let (sender, receiver) = mpsc::channel(10000);
  // 优点: 高吞吐，缓冲突发
  // 缺点: 高内存使用，延迟可能增加
  
  // 3. 平衡场景 - 中等容量
  let (sender, receiver) = mpsc::channel(100);
  // 平衡延迟、吞吐和内存使用
  ```
  
  ```
  // 无界通道 (如果存在)
  let (sender, receiver) = mpsc::unbounded_channel();
  
  // 优点: 永不阻塞
  // 缺点: 可能导致内存溢出
  ```
- ## 3.3.  背压控制(Backpressure Control)
- ### 3.3.1.  背压控制的概念
  
  **背压**是指当消费者处理速度跟不上生产者产生速度时，系统采取的流量控制机制。
  
  ```
  // 快速生产者 vs 慢速消费者的问题
  async fn fast_producer(sender: mpsc::Sender<String>) {
    for i in 0..1000000 {
        // 快速产生消息
        sender.send(format!("消息 {}", i)).await.unwrap();
    }
  }
  
  async fn slow_consumer(mut receiver: mpsc::Receiver<String>) {
    while let Some(msg) = receiver.recv().await {
        // 慢速处理 (模拟复杂处理)
        tokio::time::sleep(Duration::from_millis(100)).await;
        println!("处理: {}", msg);
    }
  }
  ```
- ### 3.3.2.  有界通道的背压机制
  
  ```
  // 容量为 10 的通道
  let (sender, mut receiver) = mpsc::channel::<String>(10);
  
  // 当通道满时的不同处理方式
  
  // 方式1: 异步等待 (阻塞直到有空间)
  match sender.send("消息".to_string()).await {
    Ok(_) => println!("发送成功"),
    Err(_) => println!("通道已关闭"),
  }
  
  // 方式2: 非阻塞尝试 (立即返回结果)
  match sender.try_send("消息".to_string()) {
    Ok(_) => println!("发送成功"),
    Err(TrySendError::Full(_)) => {
        println!("通道已满，消息被丢弃");
        // 可以选择丢弃、重试或其他策略
    }
    Err(TrySendError::Closed(_)) => {
        println!("通道已关闭");
    }
  }
  ```
- ### 3.3.3.  CodeLLDB 中的背压处理
  
  ```
  // debug_event_listener.rs - LLDB 事件处理
  match sender.try_send(event) {
    Ok(_) => {}
    Err(err) => match err {
        TrySendError::Full(_) => {
            // 通道满时记录错误但不阻塞
            error!("Could not send event: {:?}", err);
        }
        TrySendError::Closed(_) => break, // 通道关闭，退出循环
    },
  }
  ```
  
  **设计考虑**:
- LLDB 事件线程不能阻塞 (会影响调试器)
- 使用 try_send 避免阻塞
- 通道满时丢弃事件而不是等待
-