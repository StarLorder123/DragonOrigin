# 1.  VSCode简要介绍
- ## 1.1.  简要介绍
  
  ![](https://cdn.nlark.com/yuque/0/2023/png/2713067/1695532980574-1471cc1e-b413-47a7-9487-163d6e481070.png)
- Visual Studio Code(简称VSCode) 是开源免费的IDE编辑器，原本是微软内部使用的云编辑器(Monaco)。
- VSCode项目的负责人是Erich Gamma，也是曾经Eclipse的架构师，《设计模式》经典书籍的作者。他于2011年加入微软，在瑞士苏黎世组建团队开发基于Web技术的编辑器，也就是后来的Monaco-editor。从定位上来看，VSCode是一个大型的客户端APP项目。涉及的技术包含**Electron、TypeScript、Monaco、XTerm、LSP（Language Server Protocol）、DAP（Debug Adapter Protocol）。**
- VSCode的时间线：
	- 2015.04.29 第一个预览版本
	- 2015.11      开源
	- 2016.4.14   正式发布1.0版本
	- 2018           成为最受欢迎的开发工具
	- 2019.05.02  发布VSCode Remote Development
- ## 1.2.  小结
- 从定位上来看，VSCode是通过代码编辑器+一系列对应的插件组成最终的开发IDE，因此具有一定的门槛。和IDEA系列相比，IDEA更加具有集成性，只需要安装一个软件就可以进行编码，无需关心插件问题。
- # 2.  计算机基础
- ## 2.1.  网络协议升级
- ### 2.1.1.  升级过程描述
- HTTP/1.1 协议提供了一种使用 Upgrade 标头字段的特殊机制，这一机制允许将一个已建立的连接升级成新的、不相容的协议。
- 客户端使用 Upgrade 标头字段请求服务器，以降序优先的顺序切换到其中列出的一个协议。
- 因为 Upgrade 是一个逐跳（Hop-by-hop）标头，它还需要在 Connection 标头字段中列出。这意味着包含 Upgrade 的典型请求类似于：
  
  ```
  GET /index.html HTTP/1.1
  Host: www.example.com
  Connection: upgrade
  Upgrade: example/1, foo/2
  ```
- 根据之前的请求的协议，可能需要其他标头信息，例如：从 HTTP/1.1 升级到 WebSocket 允许配置有关 WebSocket 连接的标头详细信息，以及在连接时提供一定程度的安全性。查看升级到 WebSocket 协议的连接获取更多信息。
- 如果服务器决定升级这次连接，就会返回一个 101 Switching Protocols 响应状态码，和一个要切换到的协议的标头字段 Upgrade。如果服务器没有（或者不能）升级这次连接，它会忽略客户端发送的 Upgrade 标头字段，返回一个常规的响应：例如一个 200 OK).
- 在发送 101 状态码之后，服务器可以使用新协议，并根据需要执行任何额外的特定于协议的握手。实际上，一旦这次升级完成了，连接就变成了双向管道。并且可以通过新协议完成启动升级的请求。
- 至今为止，最经常会需要升级一个 HTTP 连接的场合就是使用 WebSocket，它总是通过升级 HTTP 或 HTTPS 连接来实现。请记住，当你用 WebSocket API 以及其他大部分实现 WebSocket 的库去建立新的连接时，基本上都不用操心升级的过程，因为这些 API 已经实现了这一步。
- ### 2.1.2.  协议升级示例
- 使用java语言实现一个http协议升级到websocket协议的服务器
- ```
  import java.io.*;
  import java.net.*;
  import java.nio.charset.StandardCharsets;
  import java.security.MessageDigest;
  import java.util.Base64;
  
  public class UpgradeServer {
  
      public static void main(String[] args) throws Exception {
          ServerSocket serverSocket = new ServerSocket(8080);
          System.out.println("Server is listening on port 8080...");
  
          while (true) {
              Socket clientSocket = serverSocket.accept();
              new Thread(() -> handleClient(clientSocket)).start();
          }
      }
  
      public static void handleClient(Socket clientSocket) {
          try (BufferedReader reader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
               OutputStream out = clientSocket.getOutputStream()) {
  
              // 1. 读取并解析客户端的 HTTP 请求
              String line;
              StringBuilder request = new StringBuilder();
              while (!(line = reader.readLine()).isEmpty()) {
                  request.append(line).append("\n");
              }
  
              // 打印 HTTP 请求
              System.out.println("Received Request: \n" + request);
  
              // 2. 检查是否是 WebSocket 升级请求
              if (request.toString().contains("Upgrade: websocket")) {
                  // 3. 获取 Sec-WebSocket-Key，并生成 Sec-WebSocket-Accept 响应
                  String secWebSocketKey = getSecWebSocketKey(request.toString());
                  String secWebSocketAccept = generateWebSocketAcceptKey(secWebSocketKey);
  
                  // 4. 响应 HTTP 101 协议升级
                  String response = "HTTP/1.1 101 Switching Protocols\r\n" +
                                    "Upgrade: websocket\r\n" +
                                    "Connection: Upgrade\r\n" +
                                    "Sec-WebSocket-Accept: " + secWebSocketAccept + "\r\n\r\n";
  
                  out.write(response.getBytes(StandardCharsets.UTF_8));
                  out.flush();
  
                  System.out.println("Protocol upgraded to WebSocket!");
  
                  // 5. 开始处理 WebSocket 帧（这里只是简单的例子，没有完整实现）
                  // 实际上你需要根据 WebSocket 帧的格式来读取和写入数据。
                  // 比如通过掩码解码数据帧，处理数据帧等。
  
              } else {
                  // 非 WebSocket 升级请求，响应简单的 HTTP 请求
                  String httpResponse = "HTTP/1.1 200 OK\r\n\r\nHello, HTTP client!";
                  out.write(httpResponse.getBytes(StandardCharsets.UTF_8));
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      // 获取 Sec-WebSocket-Key
      public static String getSecWebSocketKey(String request) {
          for (String line : request.split("\n")) {
              if (line.startsWith("Sec-WebSocket-Key:")) {
                  return line.split(":")[1].trim();
              }
          }
          return null;
      }
  
      // 生成 Sec-WebSocket-Accept
      public static String generateWebSocketAcceptKey(String secWebSocketKey) throws Exception {
          String magicString = "258EAFA5-E914-47DA-95CA-C5AB0DC85B11";
          String acceptKey = secWebSocketKey + magicString;
          MessageDigest md = MessageDigest.getInstance("SHA-1");
          byte[] hash = md.digest(acceptKey.getBytes(StandardCharsets.UTF_8));
          return Base64.getEncoder().encodeToString(hash);
      }
  }
  ```
- java+netty实现一个协议升级的服务器
  
  ```
  import io.netty.bootstrap.ServerBootstrap;
  import io.netty.channel.*;
  import io.netty.channel.nio.NioEventLoopGroup;
  import io.netty.channel.socket.SocketChannel;
  import io.netty.channel.socket.nio.NioServerSocketChannel;
  import io.netty.handler.codec.http.*;
  import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
  import io.netty.handler.stream.ChunkedWriteHandler;
  
  public class NettyWebSocketServer {
  
    private final int port;
  
    public NettyWebSocketServer(int port) {
        this.port = port;
    }
  
    public void start() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap bootstrap = new ServerBootstrap()
                    .group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) {
                            ChannelPipeline pipeline = ch.pipeline();
                            pipeline.addLast(new HttpServerCodec());
                            pipeline.addLast(new HttpObjectAggregator(65536));
                            pipeline.addLast(new ChunkedWriteHandler());
                            pipeline.addLast(new WebSocketServerProtocolHandler("/ws"));
                            pipeline.addLast(new WebSocketFrameHandler());
                        }
                    });
  
            Channel ch = bootstrap.bind(port).sync().channel();
            System.out.println("WebSocket server started on port " + port);
            ch.closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
  
    public static void main(String[] args) throws Exception {
        new NettyWebSocketServer(8080).start();
    }
  
    private static class WebSocketFrameHandler extends SimpleChannelInboundHandler<Object> {
        @Override
        protected void channelRead0(ChannelHandlerContext ctx, Object msg) {
            if (msg instanceof FullHttpRequest) {
                // 处理 HTTP 请求升级
                System.out.println("Received HTTP request, upgrading to WebSocket...");
            } else if (msg instanceof WebSocketFrame) {
                // 处理 WebSocket 数据帧
                System.out.println("Received WebSocket frame");
            }
        }
    }
  }
  ```
- 在nodejs中的net创建的server中绑定一个upgrade事件，这就是在协议升级的时候触发的。
- ### 2.1.3.  协议升级的应用场景
- 通常使用的场景都是HTTP协议升级到WebSocket的场景。一般考虑的因素有如下：
	- WebSocket 是基于 HTTP 的协议升级。WebSocket 是设计为在初始阶段通过 HTTP/1.1 或 HTTP/2 握手来建立连接的协议。这种设计使得 WebSocket 能够利用现有的 HTTP 基础设施进行连接，并在握手完成后切换到全双工通信模式。由于 WebSocket 是基于 HTTP 的，这意味着它可以更容易地通过现有的防火墙、代理和其他网络设备，而这些设备普遍支持 HTTP。这里就是考虑网络设备的兼容性、设备防火墙、代理服务器等等因素。
	- 逐步回退机制。在某些情况下，WebSocket 可能无法通过特定的网络设备或防火墙，这时应用可以有回退机制。例如，应用可以首先尝试升级为 WebSocket，如果失败则可以回退到较为传统的轮询或长轮询的 HTTP 技术。通过 HTTP 开始连接，允许应用具备这种灵活性。
- ## 2.2.  SSH的使用
- 为了讲解的简单，我只讲解 “端口” 的转发。实际上，Unix Socket 也可以用相同的方式转发，请参考 Man Page。[https://man7.org/linux/man-pages/man1/ssh.1.html](https://man7.org/linux/man-pages/man1/ssh.1.html)
- -D 动态转发
  
  ```
  -D [_bind_address :_ ] _port_
  ```
	- 指定一个本地的动态应用级端口转发。
	- 其工作方式是在本地分配一个 Socket 来监听端口，每当连接到这个端口时，连接通过 SSH 隧道被转发到远程主机上，然后远程主机根据协议确定该数据被转发到哪里。此时 SSH 充当 SOCKS 代理服务器。
- -L 本地转发
  
  ```
  -L [bind_address : ] port : host : hostport
  -L [bind_address : ] port : remote_socket
  -L local_socket : host : hostport
  -L local_socket : remote_socket
  ```
	- 指定本地主机上给定的 TCP 端口或 Unix 套接字的连接将被转发到远程的给定主机和端口或 Unix 套接字上。
	- 其工作方式是在本地分配一个 Socket 来监听本地的端口。每当连接到本地端口时，数据通过 SSH 隧道，被转发到远程主机的指定端口。
	- 区分 -D 和 -L：本地转发中，数据被转发到指定端口，而动态转发根据协议内容自动确定数据去向。
- -R 远程转发
  
  ```
  -R [bind_address : ] port : host : hostport
  -R [bind_address : ] port : local_socket
  -R remote_socket : host : hostport
  -R remote_socket : local_socket
  ```
	- 指定将远程（服务器）主机上给定 TCP 端口或 Unix 套接字的连接转发到本地端给定的主机和端口或 Unix 套接字。
	- 其工作方式是远程主机上分配一个 Socket 监听远程的端口。每当连接到远程端口时，数据通过 SSH 隧道，被发送到本地主机的指定端口。
- 区分 -L 和 -R：-L 是别人向本地发数据，数据会被转发到远程的特定端口；-R 是别人往远程发数据，数据会被转发到本地的特定端口。数据流向恰好相反。
- 对于转发而言，还有几个选项比较有用：
  
  ```
  -C: 使用数据压缩。
  -f: 让 SSH 在后台运行，STDIN 会被重定向到 /dev/null。
  -N: 让 SSH 不执行远程命令，即只负责转发端口。
  ```
- 所以一个常见的命令组合就可以是：
  
  ```
  ssh -CNf -D 12345 user@IP
  ```
- ### 2.2.1.  关键方法解析
- forwardOut方法
	- 通过 SSH 隧道将本地机器的请求转发到远程服务器的某个端口，并模拟一个 HTTP 请求发送给远程服务器上的服务。
	- 通常，它用于实现本地端口转发（Local Port Forwarding），也可以作为代理来连接到另一个服务器的地址和端口。
	  
	  ```
	  conn.forwardOut(
	  srcIP,         // 本地发起连接的 IP 地址
	  srcPort,       // 本地发起连接的端口
	  dstIP,         // 目标远程服务器的 IP 地址
	  dstPort,       // 目标远程服务器的端口
	  (err, stream) => {
	   if (err) {
	     console.log('Error forwarding:', err);
	     return;
	   }
	  
	   // stream 对象可以用于读写数据
	   stream.write('Some data to send');
	   stream.on('data', (data) => {
	     console.log('Received data:', data);
	   });
	  }
	  );
	  ```
- srcIP: 本地机器发起连接的 IP 地址，通常是 '127.0.0.1' 或 'localhost'。
- srcPort: 本地发起连接的端口，通常是 0（表示任意端口）。
- dstIP: 目标远程服务器的 IP 地址或主机名，这是你想要连接的目标地址。可以是 'localhost'，也可以是远程服务器的 IP 地址。
- dstPort: 目标服务器的端口号，这是你希望转发到的远程服务器的端口。
- callback: 回调函数，接收两个参数：
	- err: 如果发生错误，将返回错误对象。
	- stream: 一个 stream 对象，可以用于与目标服务器交换数据。
- forwardIn方法
	- 远程端口监听：通过 SSH 连接，远程服务器上的某个指定端口开始监听外部连接。将请求转发到本地：当远程服务器上的这个端口接收到请求时，SSH 隧道会将这些请求转发到本地机器。远程服务代理：可以让远程机器上的服务通过 SSH 隧道连接到本地机器的服务，起到代理作用。
	  
	  ```
	  const { Client } = require('ssh2');
	  
	  const conn = new Client();
	  conn.on('ready', () => {
	  console.log('Client :: ready');
	  
	  // 监听远程服务器上的 8080 端口，并将流量转发到本地
	  conn.forwardIn('0.0.0.0', 8080, (err, port) => {
	   if (err) throw err;
	   console.log(`Listening for connections on remote server at port ${port}`);
	  });
	  }).on('tcp connection', (details, accept, reject) => {
	  console.log('Incoming TCP connection:', details);
	  const stream = accept();
	  stream.write('Hello from local machine!\n');
	  stream.end();
	  }).connect({
	  host: 'remote-server.com',
	  port: 22,
	  username: 'user',
	  password: 'password'
	  });
	  ```
- conn.forwardIn('0.0.0.0', 8080, ...)：
	- 这个方法告诉远程服务器在其 所有网络接口（0.0.0.0） 的 8080 端口上监听连接。
	- 当远程服务器上的 8080 端口接收到请求时，这些请求会通过 SSH 隧道转发到本地。
- tcp connection 事件：
	- 当远程服务器的 8080 端口接收到连接时，ssh2 会触发 tcp connection 事件。这个事件的回调函数中包含请求连接的细节（如来源 IP 和端口）。
	- 在回调中，调用 accept() 接受连接，然后可以像操作本地 stream 一样处理远程的 TCP 流。
- 消息处理：
	- 远程端口接收到的请求被接受后，流量会通过 SSH 隧道传递到本地。在这个例子中，服务器接收到连接后，会向连接返回一条消息 "Hello from local machine!"，然后关闭连接。
- openssh_forwardOutStreamLocal方法
	- 通过 SSH 隧道，将来自本地的 UNIX 套接字流转发到远程服务器的一个指定的 UNIX 套接字文件上。
	- 允许你通过 SSH 来与远程服务器的 UNIX 套接字通信，就像是在本地访问一样。
	  
	  ```
	  const { Client } = require('ssh2');
	  
	  const conn = new Client();
	  conn.on('ready', () => {
	  console.log('Client :: ready');
	  
	  // 转发本地 UNIX 套接字流
	  conn.openssh_forwardOutStreamLocal('/var/run/myservice.sock', (err, stream) => {
	   if (err) throw err;
	  
	   // 当成功连接远程 UNIX 套接字时，可以通过 stream 进行通信
	   stream.on('close', () => {
	     console.log('Stream :: close');
	     conn.end(); // 关闭 SSH 连接
	   }).on('data', (data) => {
	     console.log('DATA: ' + data);
	   });
	  
	   // 向远程服务发送一些数据
	   stream.write('Hello remote service!');
	  });
	  
	  }).connect({
	  host: 'remote-server.com',
	  port: 22,
	  username: 'user',
	  password: 'password'
	  });
	  ```
- conn.openssh_forwardOutStreamLocal('/var/run/myservice.sock', ...)：
	- 该方法尝试通过 SSH 隧道连接远程服务器上的 UNIX 套接字 /var/run/myservice.sock。
	- 当连接成功后，回调函数会返回一个 stream 对象，通过它你可以向远程套接字服务发送数据或接收数据。
- stream.write() 和 stream.on('data')：
	- stream.write() 允许你向远程服务发送数据。
	- stream.on('data') 监听远程服务发送回来的数据，并在控制台打印出来。
- 关闭连接：
	- 当数据传输结束后，调用 stream.on('close') 来处理流关闭事件，并最终关闭 SSH 连接。
- ## 2.3.  Socket
- ### 2.3.1.  sock
- 网络传输，从操作上来看，无非就是，发数据和远端之间互相收发数据。也就是对应着写数据和读数据。
- 这里还有两个问题。
	- 第一个是，接收端和发送端可能不止一个，因此我们需要一些信息做下区分，这个大家肯定很熟悉，可以用IP和端口。IP用来定位是哪台电脑，端口用来定位是这台电脑上的哪个进程。
	- 第二个是，发送端和接收端的传输方式有很多区别，可以是可靠的TCP协议，也可以是不可靠的UDP协议，甚至还需要支持基于icmp协议的ping命令。
	  
	  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728896433685-b7173605-0823-4baa-a5c1-e5a131619adc.png)
- 为了支持这些功能，我们需要定义一个数据结构去支持这些功能。
- 这个数据结构，叫sock。
- 首先，可以在sock里加入IP和端口字段。这样解决了第一个问题
- 第二个问题，我们会发现这些协议虽然各不相同，但还是有一些功能相似的地方，比如收发数据时的一些逻辑完全可以复用。按面向对象编程的思想，我们可以将不同的协议当成是不同的对象类（或结构体），将公共的部分提取出来，通过"继承"的方式，复用功能。
- ### 2.3.2.  基于sock实现网络传输功能
- 于是，我们将功能重新划分下，定义了一些数据结构。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728896642227-b2256a65-8e5d-44ae-9b91-2dc3634e941d.png)
- sock是最基础的结构，维护一些任何协议都有可能会用到的收发数据缓冲区。
- inet_sock特指用了网络传输功能的sock，在sock的基础上还加入了TTL，端口，IP地址这些跟网络传输相关的字段信息。
- inet_connection_sock 是指面向连接的sock，在inet_sock的基础上加入面向连接的协议里相关字段，比如accept队列，数据包分片大小，握手失败重试次数等。虽然我们现在提到面向连接的协议就是指TCP，但设计上linux需要支持扩展其他面向连接的新协议，
- tcp_sock 就是正儿八经的tcp协议专用的sock结构了，在inet_connection_sock基础上还加入了tcp特有的滑动窗口、拥塞避免等功能。同样udp协议也会有一个专用的数据结构，叫udp_sock。
- ### 2.3.3.  提供socket层
- 可以想象得到，这里面的代码肯定非常复杂，同时还操作了网卡硬件，需要比较高的操作系统权限，再考虑到性能和安全，于是决定将它放在操作系统内核里。我们将这部分功能抽象成一个个简单的接口。以后别人只需要调用这些接口，就可以驱动我们写好的这一大堆复杂的数据结构去发送数据。
- 既然跟远端服务端进程收发数据可以抽象为“读和写”，操作文件也可以抽象为"读和写"，正好有句话叫，"linux里一切皆是文件"，那我们索性，将内核的sock封装成文件就好了。创建sock的同时也创建一个文件，文件有个句柄fd，说白了就是个文件系统里的身份证号码，通过它可以唯一确定是哪个sock。
  
  这个文件句柄fd其实就是 sock_fd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) 里的sock_fd。
- 将句柄暴露给用户，之后用户就可以像操作文件句柄那样去操作这个sock句柄。在用户空间里操作这个句柄，文件系统就会将操作指向内核sock结构。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728896949760-dac6002a-bd1b-496f-bc9c-59754f85f978.png)
- 有了sock_fd句柄之后，我们就需要提供一些接口方法，让用户更方便的实现特定的网络编程功能。这些接口，我们列了一下，发现需要有send()，recv()，bind(), listen()，connect()这些。到这里，我们的内核网络传输功能就算设计完成了。
- socket其实就是个代码库 or 接口层，它介于内核和应用程序之间，提供了一些高度封装过的接口，让我们去使用内核网络传输功能。
- 平时写的应用程序里代码里虽然用了socket实现了收发数据包的功能，但其实真正执行网络通信功能的，不是应用程序，而是linux内核。相当于应用程序通过socket提供的接口，将网络传输的这部分工作外包给了linux内核。
- 这听起来像不像我们最熟悉的前后端分离的服务架构，虽然这么说不太严谨，但看上去linux就像是被分成了应用程序和内核两个服务。内核就像是后端，暴露了好多个api接口，其中一类就是socket的send()和recv()这些方法。应用程序就像是前端，负责调用内核提供的接口来实现想要的功能。
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728897008070-28f7d58c-4c7c-4b1a-9ea6-f484c263ce69.png)
- 在操作系统内核空间里，实现网络传输功能的结构是sock，基于不同的协议和应用场景，会被泛化为各种类型的xx_sock，它们结合硬件，共同实现了网络传输功能。为了将这部分功能暴露给用户空间的应用程序使用，于是引入了socket层，同时将sock嵌入到文件系统的框架里，sock就变成了一个特殊的文件，用户就可以在用户空间使用文件句柄，也就是socket_fd来操作内核sock的网络传输能力。
- 这个socket_fd是一个int类型的数字。现在回去看socket的中文翻译，套接字，将它理解为一套用于连接的数字。
- ### 2.3.4.  socket实现网络通信
- 整个阶段分为两部分，分别是建立连接和数据传输。
- 建立连接
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728897309086-c1e96574-4aa1-4079-ac4b-0264bf6fefd5.png)
	- 在客户端，代码执行socket提供的connect(sockfd, "ip:port")方法时，会通过sockfd句柄找到对应的文件，再根据文件里的信息指向内核的sock结构。通过这个sock结构主动发起三次握手。
	- 在服务端握手次数还没达到"三次"的连接，叫半连接，完成好三次握手的连接，叫全连接。它们分别会用半连接队列和全连接队列来存放，这两个队列会在你执行listen()方法的时候创建好。当服务端执行accept()方法时，就会从全连接队列里拿出一条全连接。
	- 至此，连接就算准备好了，之后，就可以开始传输数据。
	  
	  虽然都叫队列，但半连接队列其实是个hash表，而全连接队列其实是个链表。
	  
	  那么问题来了，为什么半连接队列要设计成哈希表而全连接队列是个链表？这个在我在我之前写的《没有accept，能建立TCP连接吗？》 已经提到过，不再重复。
- 数据传输
	- 为了实现发送和接收数据的功能，sock结构体里带了一个发送缓冲区和一个接收缓冲区，说是缓冲区，但其实就是个链表，上面挂着一个个准备要发送或接收的数据。
	- 当应用执行send()方法发送数据时，同样也会通过sock_fd句柄找到对应的文件，根据文件指向的sock结构，找到这个sock结构里带的发送缓冲区，将数据会放到发送缓冲区，然后结束流程，内核看心情决定什么时候将这份数据发送出去。
	- 接收数据流程也类似，当数据送到linux内核后，数据不是立马给到应用程序的，而是先放在接收缓冲区中，数据静静躺着，卑微的等待应用程序什么时候执行recv()方法来拿一下。
	  
	  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1728897433251-7c7711ac-9c27-4db7-9784-27fc76191f72.png)
	- 当你的应用进程执行recv()方法尝试获取（阻塞场景下）接收缓冲区的数据时。
		- 如果有数据，那正好，取走就好了。这点没啥疑问。
		- 但如果没数据，就会将自己的进程信息注册到这个sock用的等待队列里，然后进程休眠。如果这时候有数据从远端发过来了，数据进入到接收缓冲区时，内核就会取出sock的等待队列里的进程，唤醒进程来取数据。
	- 有时候，你会看到多个进程通过fork的方式，listen了同一个socket_fd。在内核，它们都是同一个sock，多个进程执行listen()之后，都嗷嗷等待连接进来，所以都会将自身的进程信息注册到这个socket_fd对应的内核sock的等待队列中。如果这时真来了一个连接，是该唤醒等待队列里的哪个进程来接收连接呢？这个问题的答案比较有趣。
	- 在linux 2.6以前，会唤醒等待队列里的所有进程。但最后其实只有一个进程会处理这个连接请求，其他进程又重新进入休眠，这些被唤醒了又无事可做最后只能重新回去休眠的进程会消耗一定的资源。就好像你在广东的街头，想问路，叫一声靓仔，几十个人同时回头，但你其实只需要其中一个靓仔告诉你路该怎么走。你这种一不小心惊动这群靓仔的场景，在计算机领域中，就叫惊群效应。
	- 在linux 2.6之后，只会唤醒等待队列里的其中一个进程。是的，socket监听的惊群效应问题被修复了。
- ### 2.3.5.  参考资料
  
  [https://www.51cto.com/article/742745.html](https://www.51cto.com/article/742745.html)
-
- # 3.  编程语言基础
- VSCode源代码是使用的typescript写的。typescript和javascript有着非常密切的关系，因此需要前期有一些基本的语言的基础和习惯。
- 比较重要的基础包括：TypeScript、Electron（基础框架）、Webpack（打包）、Nodejs事件循环机制、Promise、Gulp工具（自动化任务构建工具）等。
- 其余还包括浏览器中的DOM、HTML等相关知识、计算机网络、操作系统、Linux、Pty（伪终端）等知识没有单独列出来介绍，需要的时候再捎带一些。
- ## 3.1.  TypeScript基础
- [[TypeScript基础知识]]
-