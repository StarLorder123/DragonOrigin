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
- ## 3.2.  Electron基础
- [[Electron基础]]
- ## 3.3.  Webpack基础
- [[Webpack基础]]
- ## 3.4.  Nodejs事件循环机制
- [[Nodejs事件循环机制]]
- ## 3.5.  Promise
- Promise 对象是 JavaScript 的异步操作解决方案，为异步操作提供统一接口。它起到代理作用（proxy），充当异步操作与回调函数之间的中介，使得异步操作具备同步操作的接口。Promise 可以让异步操作写起来，就像在写同步操作的流程，而不必一层层地嵌套回调函数。
- Promise 的设计思想是，所有异步任务都返回一个 Promise 实例。Promise 实例有一个then方法，用来指定下一步的回调函数。
- ### 3.5.1.  Promise对象的状态
- Promise 对象通过自身的状态，来控制异步操作。Promise 实例具有三种状态。
	- 异步操作未完成（pending）
	- 异步操作成功（fulfilled）
	- 异步操作失败（rejected）
- 上面三种状态里面，fulfilled和rejected合在一起称为resolved（已定型）。
- 这三种的状态的变化途径只有两种。
	- 从“未完成”到“成功”
	- 从“未完成”到“失败”
- 一旦状态发生变化，就凝固了，不会再有新的状态变化。这也是 Promise 这个名字的由来，它的英语意思是“承诺”，一旦承诺成效，就不得再改变了。这也意味着，Promise 实例的状态变化只可能发生一次。
- 因此，Promise 的最终结果只有两种。
	- 异步操作成功，Promise 实例传回一个值（value），状态变为fulfilled。
	- 异步操作失败，Promise 实例抛出一个错误（error），状态变为rejected。
- ### 3.5.2.  Promise的基本使用
  
  ```
  const promise = new Promise((resolve, reject) => {
  let status = true;
  if (status) {
    resolve('操作成功!');
  } else {
    reject('操作失败!');
  }
  });
  
  promise.then(res => {
  console.log('成功结果：' + res);
  }, error => {
  console.log('失败结果：' + error);
  
  });
  ```
- ### 3.5.3.  Promise的进阶使用
- Promise实例具有then方法，then方法是定义在原型对象Promise.prototype上的。它的作用前面说过，第一个回调函数是状态改变fufilled时调用的，第二个回调函数(可选)是状态改变rejected时调用的。
- then方法的基础调用写法，可以写一个回调方法，来执行成功后的回调。then方法返回一个的是一个新的Promise实例，因此我们可以采用链式写法，即then方法后面再调用一个then方法。采用链式的then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。
- catch是用于指定发生错误的回调函数。Promise实例当状态改变为rejected状态或者操作失败抛出异常错误，就会被catch方法捕获。所以在Promise实例中reject方法等同于抛出错误。如果Promise的状态已经变成了resolved，再抛出错误无效。
- finally方法用于指定不管Promis对象最后状态如何，都会执行的操作。该方法是 ES2018 引入标准的。
- Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例。在all方法中可以传递多个Promise对象，当所有的Promise对象状态都返回fufilled，才会返回fulfilled，否则返回rejected。
- Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例。可以传递多个Promise对象作为参数，如果实例有一个实例率先改变状态，那么race的状态就会跟着改变。
- ## 3.6.  Gulp工具
- [[gulp构建工具]]
- ## 3.7.  IIFE(Immediately Invoked Functions Expressions)
- ### 3.7.1.  简要介绍
- JavaScript 中的立即调用函式 (IIFE，Immediately Invoked Function Expression)，指的是一种在定义时立即执行的匿名函式，通常用于创建一个局部作用域，避免全局污染。
- 立即调用函数式的好处：
	- 创建局部作用域：通过使用 IIFE 可以创建一个局部作用域，避免全局变量的污染。以下代码可以看到，在 IIFE 中，有一个局部变量 localVariable。 localVariable 只能在 IIFE 内访问，不能在 IIFE 外访问
	  
	  ```
	  // Global scope
	  var globalVariable = "global variable";
	  
	  (function () {
	  // Local scope inside IIFE
	  var localVariable = "local variable";
	  console.log(localVariable); // local variable
	  })();
	  
	  console.log(localVariable); // ReferenceError: localVariable is not defined
	  console.log(globalVariable); // global variable
	  ```
	- 避免命名冲突：IIFE 可以为变量创建了一个单独的命名空间，避免函式名和变量名的冲突。
	  
	  ```
	  // Global scope
	  var globalVariable = "global variable";
	  
	  (function () {
	  // Local scope inside IIFE
	  var globalVariable = "local variable inside IIFE";
	  console.log(globalVariable); // local variable inside the IIFE
	  })();
	  
	  console.log(globalVariable); // global variable
	  ```
	- 模组化编程：IIFE 可以将代码分为独立的模组，方便了代码的管理和维护。
- ### 3.7.2.  常用场景
- 创建只使用一次的函数，并立即执行它
	- 创建只使用一次的函数比较好理解，在需要调用函数的地方使用IIFE，类似内联的效果：
	  
	  ```
	  (function(){
	  var a = 1, b = 2;
	  console.log(a+b); // 3
	  })();
	  ```
	- 还可以传入参数：
	  
	  ```
	  (function(c){
	  var a = 1, b = 2;
	  console.log(a+b+c); // 6
	  })(3);
	  ```
	- IIFE比较常见的形式是匿名函数，但是也可以是命名的函数：
	  
	  ```
	  (function adder(a, b){
	  console.log(a+b); // 7
	  })(3, 4);
	  ```
	- 在js中应该尽量使用命名函数，因为匿名函数在堆栈跟踪的时候会造成一些不便。
- 创建闭包，保存状态，隔离作用域
	- 隔离作用域比较复杂一点，在ES6以前，JS没有块级作用域，只有函数作用域，作为一种对块级作用域的模拟就只能用function模拟一个作用域，比如如下代码：
	  
	  ```
	  var myBomb = (function(){
	  var bomb = "Atomic Bomb"
	  return {
	   get: function(){
	     return bomb
	   },
	   set: function(val){
	     bomb = val
	   },
	  }
	  })()
	  
	  console.log(myBomb.get()) // Atomic Bomb
	  myBomb.set("h-bomb")
	  console.log(myBomb.get()) // h-bomb
	  console.log(bomb) // ReferenceError: bomb is not defined
	  bomb = "none"
	  console.log(bomb) // none
	  ```
	- 可以看到一个比较奇特的现象，按照常理，一个函数执行完毕，在它内部声明的变量都会被销毁，但是这里变量bomb却可以通过myBomb.get和myBomb.set去读写，但是从外部直接去读和写却不行，这是闭包造成的典型效果。
- 作为独立模块存在，防止命名冲突，命名空间注入
	- 可以使用以下代码为ns这个命名空间注入变量和方法：
	  
	  ```
	  var ns = ns || {};
	  
	  (function (ns){
	  ns.name = 'Tom';
	  ns.greet = function(){
	   console.log('hello!');
	  }
	  })(ns);
	  console.log(ns); // { name: 'Tom', greet: [Function] }
	  ```
	- 还可以扩展到更多的用途：
	  
	  ```
	  (function (ns, undefined){
	  var salary = 5000; // 私有属性
	  ns.name = 'Tom'; // 公有属性
	  ns.greet = function(){ // 公有方法
	   console.log('hello!');
	  }
	  
	  ns.externalEcho = function(msg){
	   console.log('external echo: ' + msg);
	   insideEcho(msg);
	  }
	  
	  function insideEcho(msg){ // 私有方法
	   console.log('inside echo: ' + msg);
	  }
	  })(window.ns = window.ns || {});
	  
	  console.log(ns.name); // Tom
	  ns.greet(); // hello
	  ns.age = 25;
	  console.log(ns.age); // 25
	  console.log(ns.salary); // undefined
	  ns.externalEcho('JavaScript'); // external echo: JavaScript/inside echo: JavaScript
	  insideEcho('JavaScript'); // Uncaught ReferenceError: insideEcho is not defined
	  ns.insideEcho('JavaScript'); // Uncaught TypeError: ns.insideEcho is not a function
	  ```
	- 命名空间注入是IIFE作为命名空间的装饰器和扩展器的一个变体，使其更具有通用性。作用是可以在一个IIFE(这里可以把它理解成一个函数包装器)内部为一个特定的命名空间注入变量/属性和方法，并且在内部使用this指向该命名空间。
- ### 3.7.3.  资料总结
- [https://nullcc.github.io/2017/05/08/%E7%90%86%E8%A7%A3JavaScript%E7%9A%84%E7%AB%8B%E5%8D%B3%E8%B0%83%E7%94%A8%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F(IIFE)/](https://nullcc.github.io/2017/05/08/%E7%90%86%E8%A7%A3JavaScript%E7%9A%84%E7%AB%8B%E5%8D%B3%E8%B0%83%E7%94%A8%E5%87%BD%E6%95%B0%E8%A1%A8%E8%BE%BE%E5%BC%8F(IIFE)/)
- ## 3.8.  闭 包
- ### 3.8.1.  什么是闭包？
- 闭包就是能够读取其他函数内部变量的函数。
- 由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。
- 所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。
- ### 3.8.2.  闭包的用途
- 闭包可以用在许多地方。它的最大用处有两个，一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。
  
  ```
  function f1(){
  
  　　var n=999;
  
  　　nAdd=function(){n+=1}
  
  　　function f2(){
  　　　　alert(n);
  　　}
  
  　　return f2;
  
  }
  
  var result=f1();
  
  result(); // 999
  
  nAdd();
  
  result(); // 1000
  ```
- 在这段代码中，result实际上就是闭包f2函数。它一共运行了两次，第一次的值是999，第二次的值是1000。这证明了，函数f1中的局部变量n一直保存在内存中，并没有在f1调用后被自动清除。
- 为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。
- 这段代码中另一个值得注意的地方，就是"nAdd=function(){n+=1}"这一行，首先在nAdd前面没有使用var关键字，因此nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。
- 注意点：
	- 1）由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
	- 2）闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。
- ### 3.8.3.  闭包在VSCode中的表现
- 在VSCode中，最典型的闭包表现就是创建shared Process这一块。
  
  ```
  private setupSharedProcess(machineId: string, sqmId: string): { sharedProcessReady: Promise<MessagePortClient>; sharedProcessClient: Promise<MessagePortClient> } {
  const sharedProcess = this._register(this.mainInstantiationService.createInstance(SharedProcess, machineId, sqmId));
  
  const sharedProcessClient = (async () => {
  	this.logService.trace('Main->SharedProcess#connect');
  
  	const port = await sharedProcess.connect();
  
  	this.logService.trace('Main->SharedProcess#connect: connection established');
  
  	return new MessagePortClient(port, 'main');
  })();
  
  const sharedProcessReady = (async () => {
  	await sharedProcess.whenReady();
  
  	return sharedProcessClient;
  })();
  
  return { sharedProcessReady, sharedProcessClient };
  }
  ```
- SharedProcess是一个内部的变量，通过sharedProcessReady和sharedProcessClient这两个方法传递出去，被调用。
- ### 3.8.4.  资料总结
- [https://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html](https://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
- ## 3.9.  Proxy代理
- javascript中的proxy代理和java中的反射机制很像。
- [[JavaScript Proxy解析]]
- # 4.  VSCode中的基本概念
- ## 4.1.  Disposable
- Disposable 是一个非常重要和基础的概念的，它贯穿了整个 vscode 项目中，90% 的对象都是继承 Disposable，还有大量的实现 IDisposable 接口的对象。 Disposable 本身并没有做太多事情: 它是一个抽象类，提供了两个方法 _register （protected） 和 dispose (public)， 可以通过 dispose 方法把 _register 注册的 listener (IDispsable 对象) 给全部销毁。其核心的工作就是将继承Disposable的对象管理起来，再其销毁的时候销毁掉这个对象以及其依赖的对象，提高内存的利用率。
- 为了保证插件的高效运行，VS Code使用了Dispose模式，大部分插件API都实现了IDisposable接口，生成的对象则会拥有一个dispose函数属性。
  
  ```
  interface IDisposable {
  dispose(): void;
  }
  ```
- Dispose模式主要用来资源管理，资源比如内存被对象占用，则会通过调用方法来释放，这些方法通常被命名为‘close’，‘dispose’，‘free’，‘release’。一个著名的例子便是C#，C#通过Dipose Pattern来释放不受CLR(Common Language Runtime)管理的非托管资源。
- Javascript的内存分配是通过GC(garbage collector)进行管理，大部分情况下它都是自动执行且对用户不可见的。然而这种自动化的管理方式却存在一个潜在的问题，就是Javascript开发者会错误的认为他们不需要再关心内存管理了，从而再无意间书写一些不利于内存回收的代码。
- 所以，最清楚被分配的内存在未来是否需要使用的还是开发者，但是每次使用完一个对象后就手动的将其销毁，这样的做法即不高效，也不可靠。正因为此，VS Code使用了Dispose Pattern来管理对象销毁。当扩展功能执行时，Extension Host会在正确的时机调用dispose方法，销毁Code生成的对象，减少内存使用。比如说，方法‘setStatusBarMessage(value: string)’返回一个‘Disposable’对象，当调用dispose方法的时候会移除掉信息对象。
- ```
  // 第一个重载参数是单个disposable类型
  function dispose<T extends IDisposable>(disposable: T): T;
  // 第二个重载参数是多个disposable类型传参数，参数可能为undefined。
  function dispose<T extends IDisposable>(...disposables: Array<T | undefined>): T[];
  // 第三个重载参数是一个disposable类型的数组。
  function dispose<T extends IDisposable>(disposables: T[]): T[];
  // 第三个重载参数为两种，第一个是disposable类型或disposable数组类型，剩余的为disposable类型。
  function dispose<T extends IDisposable>(first: T | T[], ...rest: T[]): T | T[] | undefined {
  // 如果第一个参数是数组，则依次调用传参数的dispose方法
  if (Array.isArray(first)) {
    first.forEach(d => d && d.dispose());
    // 返回空的数组
    return [];
  } else if (rest.length === 0) {
    // 如果没有没有剩余参数
    if (first) {
      // 如果存在first
      // 调用第一个dispose
      first.dispose();
      // 返回first
      return first;
    }
  - return undefined;
  } else {
    // first不是数组，且rest长度不为0
    dispose(first);
    dispose(rest);
  - // 返回空数组
    return [];
  }
  }
  - // implement IDisposable 的Disposable 抽象类
  abstract class Disposable implements IDisposable {
  - // Disposable类的静态对象，用于返回一个包含空的dispose方法的IDisposable对象。dispose被执行了，则表示该对象不再需要了。
  // 部分基础API使用了该对象，用于标志资源释放。
  static None = Object.freeze<IDisposable>({ dispose() { } });
  - // protected属性toDispose返回protected对象_toDispose, 该对象初始值是一个空的数组。
  protected _toDispose: IDisposable[] = [];
  // 返回IDisposable数组。
  protected get toDispose(): IDisposable[] { return this._toDispose; }
  - // 设置状态标志，表示该对象是否有被销毁。
  private _lifecycle_disposable_isDisposed = false;
  - // 暴露公共方法dispose，执行完后将_lifecycle_disposable_isDisposed状态标志设为true，同时调用lifecycle内的dispose方法处理_toDispose数组，并重新赋值空数组。
  public dispose(): void {
    this._lifecycle_disposable_isDisposed = true;
    this._toDispose = dispose(this._toDispose);
  }
  - // 内部方法注册实例，若_lifecycle_disposable_isDisposed为true，则表明该方法已经被dispose过，则不能再使用，需dispose掉，否则，推入_toDispose数组。
  protected _register<T extends IDisposable>(t: T): T {
    // 判断这个对象有没有被dispose过
    if (this._lifecycle_disposable_isDisposed) {
      console.warn('Registering disposable on object that has already been disposed.');
      t.dispose();
    } else {
      this._toDispose.push(t);
    }
  - return t;
  }
  }
  ```
- ## 4.2.  Event和Emitter
- 在vscode中事件模块是一个比较基础，而且比较核心的一块内容，可以说是vscode应用程序的一块基石。
- ### 4.2.1.  Event事件
- Event 接口规定了一个函数，当调用了这个函数，就表示监听了这个函数所对应的事件流。
	- listener 参数是事件派发时将会被调用的回调函数，参数 e 为单个事件，换句话说， listener 就是事件的消费者
	- thisArgs 参数是回调函数中 this 所指向的对象
	- disposables
	  
	  ```
	  export interface Event<T> {
	  (listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[] | DisposableStore): IDisposable;
	  }
	  ```
- 返回的 IDisposable 对象用于解除这个监听的（通过调用它的 dispose 方法）。
- 另外一种解除监听的方式就是 disposable 了，Event 函数在执行的过程中会将 IDisposable 插入 disposables，方便调用方决定在什么时候解除监听。
- 在VSCode源码的实现中，实现了一个Event的库。里面包含了很多其他的有关Event事件的方法。
- ```
  // 主要定义了一些接口协议，以及相关方法
  // 使用 namespace 的方式将相关内容包裹起来
  export namespace Event {
  	// 来看看里面比较关键的一些方法
  
  	// 给定一个事件，返回另一个仅触发一次的事件
    export function once<T>(event: Event<T>): Event<T> {}
  
    // 给定一连串的事件处理功能（过滤器，映射等），每个事件和每个侦听器都将调用每个函数
    // 对事件链进行快照可以使每个事件每个事件仅被调用一次
    // 以此衍生了 map、forEach、filter、any 等方法此处省略
  	export function snapshot<T>(event: Event<T>): Event<T> {}
  
  	// 给事件增加防抖
  	export function debounce<T>(event: Event<T>, merge: (last: T | undefined, event: T) => T, delay?: number, leading?: boolean, leakWarningThreshold?: number): Event<T>;
  
  	// 触发一次的事件，同时包括触发时间
  	export function stopwatch<T>(event: Event<T>): Event<number> {}
  
  	// 仅在 event 元素更改时才触发的事件
  	export function latch<T>(event: Event<T>): Event<T> {}
  
  	// 缓冲提供的事件，直到出现第一个 listener，这时立即触发所有事件，然后从头开始传输事件
  	export function buffer<T>(event: Event<T>, nextTick = false, _buffer: T[] = []): Event<T> {}
  
    // 可链式处理的事件，支持以下方法
  	export interface IChainableEvent<T> {
  		event: Event<T>;
  		map<O>(fn: (i: T) => O): IChainableEvent<O>;
  		forEach(fn: (i: T) => void): IChainableEvent<T>;
  		filter(fn: (e: T) => boolean): IChainableEvent<T>;
  		filter<R>(fn: (e: T | R) => e is R): IChainableEvent<R>;
  		reduce<R>(merge: (last: R | undefined, event: T) => R, initial?: R): IChainableEvent<R>;
  		latch(): IChainableEvent<T>;
  		debounce(merge: (last: T | undefined, event: T) => T, delay?: number, leading?: boolean, leakWarningThreshold?: number): IChainableEvent<T>;
  		debounce<R>(merge: (last: R | undefined, event: T) => R, delay?: number, leading?: boolean, leakWarningThreshold?: number): IChainableEvent<R>;
  		on(listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[] | DisposableStore): IDisposable;
  		once(listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[]): IDisposable;
  	}
  	class ChainableEvent<T> implements IChainableEvent<T> {}
  
    // 将事件转为可链式处理的事件
  	export function chain<T>(event: Event<T>): IChainableEvent<T> {}
  
    // 来自 DOM 事件的事件
  	export function fromDOMEventEmitter<T>(emitter: DOMEventEmitter, eventName: string, map: (...args: any[]) => T = id => id): Event<T> {}
  
    // 来自 Promise 的事件
  	export function fromPromise<T = any>(promise: Promise<T>): Event<undefined> {}
  }
  ```
- Event中主要是一些对事件的处理和某种类型事件的生成。其中，除了常见的once和 DOM 事件等兼容，还提供了比较丰富的事件能力：
	- 防抖动
	- 可链式调用
	- 缓存
	- Promise 转事件等等
- ### 4.2.2.  Emitter事件发射器
- Emitter 类型暴露了两个重要方法：
	- fire，从这个方法的函数签名就能看出它就是用来派发一个事件的，该方法的主要逻辑就是将 this._listeners 当中的保存的 listener 全部调用一遍（省略了部分分支逻辑和性能监控相关代码）
	  
	  ```
	  fire(event: T): void {
	  if (this._listeners) {
	  for (let listener of this._listeners) {
	  	this._deliveryQueue.push([listener, event]);
	  }
	  
	  while (this._deliveryQueue.size > 0) {
	  	const [listener, event] = this._deliveryQueue.shift()!;
	  	try {
	  		if (typeof listener === 'function') {
	  			listener.call(undefined, event);
	  		} else {
	  			listener[0].call(listener[1], event);
	  		}
	  	} catch (e) {
	  		onUnexpectedError(e);
	  	}
	  }
	  }
	  }
	  ```
	- get event()，这个方法会在 Emitter 中创建一个 Event，其主要逻辑就是将 listener 添加到 this._listeners 当中
	  
	  ```
	  const remove = this._listeners.push(!thisArgs ? listener : [listener, thisArgs]);
	  ```
- Emitter 类型还提供了一些特殊的回调接口：
  
  ```
  export interface EmitterOptions {
  onFirstListenerAdd?: Function;
  onFirstListenerDidAdd?: Function;
  onListenerDidAdd?: Function;
  onLastListenerRemove?: Function;
  }
  ```
- 这使得 Emitter 在注册消费者的时候执行一些额外的逻辑。
- 事件的使用方式主要包括：
	- 注册事件发射器
	- 对外提供定义的事件
	- 在特定时机向订阅者触发事件
- 在Emitter的实现过程中，还实现了其他的一些场景，比如防抖、once等等。具体的解析可以参考：[https://github.com/wzhudev/blog/issues/40](https://github.com/wzhudev/blog/issues/40)
- ### 4.2.3.  简单的Event-Emitter使用
- 事件的定义:（生产一个事件）
  
  ```
  export class EditorService extends Disposable implements EditorServiceImpl {
  declare readonly _serviceBrand: undefined;
  //#region events
  private readonly _onDidActiveEditorChange = this._register(new Emitter<void>());
  readonly onDidActiveEditorChange = this._onDidActiveEditorChange.event;
  private readonly _onDidVisibleEditorsChange = this._register(new Emitter<void>());
  readonly onDidVisibleEditorsChange = this._onDidVisibleEditorsChange.event;
  private readonly _onDidEditorsChange = this._register(new Emitter<IEditorsChangeEvent>());
  readonly onDidEditorsChange = this._onDidEditorsChange.event;
  private readonly _onDidCloseEditor = this._register(new Emitter<IEditorCloseEvent>());
  readonly onDidCloseEditor = this._onDidCloseEditor.event;
  //#endregion
  }
  ```
- 事件的消费：（注册监听函数）
  
  ```
  import { Event } from 'vs/base/common/event';
  class MainThreadDocumentAndEditorStateComputer {
  constructor(
  @IEditorService private readonly _editorService: IEditorService,
  ) {
  
  this._editorService.onDidActiveEditorChange(_ => this._updateState(), this, this._toDispose);
  Event.filter(this._paneCompositeService.onDidPaneCompositeOpen, event => event.viewContainerLocation === ViewContainerLocation.Panel)(_ => this._activeEditorOrder = ActiveEditorOrder.Panel, undefined, this._toDispose);
  Event.filter(this._paneCompositeService.onDidPaneCompositeClose, event => event.viewContainerLocation === ViewContainerLocation.Panel)(_ => this._activeEditorOrder = ActiveEditorOrder.Editor, undefined, this._toDispose);
  this._editorService.onDidVisibleEditorsChange(_ => this._activeEditorOrder = ActiveEditorOrder.Editor, undefined, this._toDispose);
  }
  }
  ```
- ### 4.2.4.  观察者模式（订阅-发布）
	- Observer：抽象观察者，是观察者的抽象类，它定义了一个更新接口，使得在得到主题更改通知时更新自己。
	- ConcrereObserver：具体观察者，实现抽象观察者定义的更新接口，以便在得到主题更改通知时更新自身的状态。
- 【例】微信公众号
	- 在使用微信公众号时，大家都会有这样的体验，当你关注的公众号中有新内容更新的话，它就会推送给关注公众号的微信用户端。我们使用观察者模式来模拟这样的场景，微信用户就是观察者，微信公众号是被观察者，有多个的微信用户关注了程序猿这个公众号。
- ```
  public interface Observer {
      void update(String message);
  }
  
  public class WeixinUser implements Observer {
      // 微信用户名
      private String name;
  
      public WeixinUser(String name) {
          this.name = name;
      }
      @Override
      public void update(String message) {
          System.out.println(name + "-" + message);
      }
  }
  
  public interface Subject {
      //增加订阅者
      public void attach(Observer observer);
  
      //删除订阅者
      public void detach(Observer observer);
      
      //通知订阅者更新消息
      public void notify(String message);
  }
  
  public class SubscriptionSubject implements Subject {
      //储存订阅公众号的微信用户
      private List<Observer> weixinUserlist = new ArrayList<Observer>();
  
      @Override
      public void attach(Observer observer) {
          weixinUserlist.add(observer);
      }
  
      @Override
      public void detach(Observer observer) {
          weixinUserlist.remove(observer);
      }
  
      @Override
      public void notify(String message) {
          for (Observer observer : weixinUserlist) {
              observer.update(message);
          }
      }
  }
  
  public class Client {
      public static void main(String[] args) {
          SubscriptionSubject mSubscriptionSubject=new SubscriptionSubject();
          //创建微信用户
          WeixinUser user1=new WeixinUser("孙悟空");
          WeixinUser user2=new WeixinUser("猪悟能");
          WeixinUser user3=new WeixinUser("沙悟净");
          //订阅公众号
          mSubscriptionSubject.attach(user1);
          mSubscriptionSubject.attach(user2);
          mSubscriptionSubject.attach(user3);
          //公众号更新发出消息给订阅的微信用户
          mSubscriptionSubject.notify("传智黑马的专栏更新了");
      }
  }
  ```
- **优点：**
	- 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系。
	- 被观察者发送通知，所有注册的观察者都会收到信息【可以实现广播机制】
- **缺点：**
	- 如果观察者非常多的话，那么所有的观察者收到被观察者发送的通知会耗时
	- 如果被观察者有循环依赖的话，那么被观察者发送通知会使观察者循环调用，会导致系统崩溃
- **使用场景**
	- 对象间存在一对多关系，一个对象的状态发生改变会影响其他对象。
	- 当一个抽象模型有两个方面，其中一个方面依赖于另一方面时。
- ### 4.2.5.  小结
- 事件是什么，事件本身没有任何功能逻辑，事件的功能逻辑都在监听函数的处理中，事件提供的是一种**注册事件标识符、外界能注册某个事件发生时的回调（监听函数）、外界能够触发事件的能力、内部在事件发生时能执行监听函数、移出监听函数销毁事件等清理能力**，VSCode 将事件的清理做成了自动化的方式。
- 类的具名化调用
	- 通过new Emitter<IXXXEvent>()生产一个包含类型的事件，提供的get event()获得可以注册回调函数 listener 的事件接口，并且将这个 event 挂载在功能类XXXService的属性onXXXEvent上
	- 其它类通过依赖注入和上述功能类产生依赖关系，调用上述XXXService.onXXXEvent(() => //listener )注册回调函数
- 类的自动销毁：通过依赖的标准化注册流程，**一个类销毁时，自动分析依赖销毁依赖里相关的内容，做到了自动化的链式销毁。**
	- 事件定义方的类销毁时：this._register(new Emitter<IXXXEvent>())将生产的事件_register 到自己的类上，收集了一个依赖，在自己的类 dispose 时调用 _register 里注册的事件 Emitter 的 dispose 方法，做到了自己销毁时，自己生产的事件也被销毁
	- 事件监听方的类销毁时：监听方 xxxService 的类销毁 -> 调用 xxxService.dispose() -> 找到 xxxService 上 _register 的事件监听器的返回内容 -> 执行事件监听器返回内容 SafeDisposable.dispose -> 抹除了原始事件监听列表里对 xxxService 对该事件注册
- ## 4.3.  通信机制
- Electron框架是一个典型的多进程多线程的框架，免不了需要去设计进程间的通信。
- 接下来介绍一下VSCode中的一些关于通信方面的概念
- 通信机制会有如下内容需要设计：
	- 协议设计-Protocol
	- 通信频道-Channel
	- 连接-Connection
	- 服务端-IPCServer
	- 客户端-IPCClient
- ### 4.3.1.  通信概念释义
  
  | **概念名词** | **概念解释** | **作用** |
  | **Protocol** | 通信协议 | 通信的基础协议规范 |
  | **Channel** | 客户端频道 | 端与端之间进行信息传输的通道，类似于电台频道 |
  | **ServerChannel** | 服务端频道 | 端与端之间进行信息传输的通道，类似于电台频道 |
  | **ChannelClient** | 频道的客户端 | 客户端频道的管理 |
  | **ChannelServer** | 频道的服务端 | 服务端频道的管理 |
  | **Connection** | 连接 | 端与端之间的连接对应关系 |
  | **IPCClient** | IPC 客户端 | 负责连接的建立以及 Channel 的注册和获取 |
  | **IPCServer** | IPC 服务端 | 负责连接的建立以及 Channel 的注册和获取 |
- 协议（Protocol）
	- 两个端之间进行消息通信的约定，比如我们是通过语言还是手语比划进行通信，需要通过协议进行约定。
	- 在 VS Code 中，约定了最基础的协议范围包括发送和接收消息两个方法：
		- 发送：send
		- 接收：onMessage
- 频道（Channel）
	- 狭义定义上，频道又叫信道，信道是信号在通信系统重传输的通道，是信号从发射端传输到接收端所经过的传输煤质。
	- 在 VS Code 中，频道是一组可供其他端进行调用的服务集合。一个标准的频道有两个功能：
		- 点播：call
		- 收听：listen
	- 在频道中，Server是专门处理消息的类，Client是专门发送消息的类。
- 客户端&服务端（Server&Client）
	- 客户端 & 服务端是频道的承载主体。一般客户端是指发起连接的一端，服务端是被连接的一端。
	- 在 VS Code 中，服务端提供一系列服务的频道；渲染进程是客户端，调用服务端频道中的服务或者收听服务端消息。不管是服务端还是客户端，都需要具备发送和接受消息的能力，才能实现正常的通信。
- 连接（Connection）
	- 客户端 & 服务端之间进行通信依赖的连接。
	- 在 VS Code 中一个连接其实是一对客户端与服务端的对应关系。
- 上述接口和类的定义对应文件为：src/vs/base/parts/ipc/common/ipc.ts
- ### 4.3.2.  实现示例
  
  ![](https://cdn.nlark.com/yuque/0/2024/jpeg/2713067/1708998583271-d0834f61-54a8-43c4-9d1d-f72522c04aca.jpeg)
- 请求实现（基于 IMessagePassingProtocol 协议）
  
  ```
  class QueueProtocol implements IMessagePassingProtocol {
  private buffering = true;
  private buffers: VSBuffer[] = [];
  
  private readonly _onMessage = new Emitter<VSBuffer>({
    onDidAddFirstListener: () => {
      for (const buffer of this.buffers) {
        this._onMessage.fire(buffer);
      }
  
      this.buffers = [];
      this.buffering = false;
    },
    onDidRemoveLastListener: () => {
      this.buffering = true;
    }
  });
  
  readonly onMessage = this._onMessage.event;
  other!: QueueProtocol;
  
  send(buffer: VSBuffer): void {
    this.other.receive(buffer);
  }
  
  protected receive(buffer: VSBuffer): void {
    if (this.buffering) {
      this.buffers.push(buffer);
    } else {
      this._onMessage.fire(buffer);
    }
  }
  }
  ```
- 客户端（基于IPCClient）
  
  ```
  class TestIPCClient extends IPCClient<string> {
  private readonly _onDidDisconnect = new Emitter<void>();
  readonly onDidDisconnect = this._onDidDisconnect.event;
  
  constructor(protocol: IMessagePassingProtocol, id: string) {
  super(protocol, id);
  }
  
  override dispose(): void {
  this._onDidDisconnect.fire();
  super.dispose();
  }
  }
  ```
- 服务端（基于IPCServer）
  
  ```
  class TestIPCServer extends IPCServer<string> {
  private readonly onDidClientConnect: Emitter<ClientConnectionEvent>;
  
  constructor() {
  const onDidClientConnect = new Emitter<ClientConnectionEvent>();
  super(onDidClientConnect.event);
  this.onDidClientConnect = onDidClientConnect;
  }
  
  // 创建一个客户端 & 服务端的连接
  createConnection(id: string): IPCClient<string> {
  const [pc, ps] = createProtocolPair();
    const pc = new QueueProtocol();
    const ps = new QueueProtocol();
    pc.other = ps;
    ps.other = pc;
  const client = new TestIPCClient(pc, id);
  
  this.onDidClientConnect.fire({
  	protocol: ps,
  	onDidClientDisconnect: client.onDidDisconnect
  });
  
  return client;
  }
  }
  ```
- 服务端频道及其对应的服务
  
  ```
  // 服务接口
  interface ITestService {
  marco(): Promise<string>;
  onPong: Event<string>;
  }
  
  // 服务
  class TestService implements ITestService {
  private readonly _onPong = new Emitter<string>();
  readonly onPong = this._onPong.event;
  
  marco(): Promise<string> {
  return Promise.resolve('polo');
  }
  
  ping(msg: string): void {
  this._onPong.fire(msg);
  }
  }
  
  // 服务频道
  class TestChannel implements IServerChannel {
  constructor(private service: ITestService) { }
  
  call(_: unknown, command: string, arg: any, cancellationToken: CancellationToken): Promise<any> {
  switch (command) {
  	case 'marco': return this.service.marco();
  	default: return Promise.reject(new Error('not implemented'));
  }
  }
  
  listen(_: unknown, event: string, arg?: any): Event<any> {
  switch (event) {
  	case 'onPong': return this.service.onPong;
  	default: throw new Error('not implemented');
  }
  }
  }
  ```
- 客户端频道服务
  
  ```
  class TestChannelClient implements ITestService {
  get onPong(): Event<string> {
  return this.channel.listen('onPong');
  }
  
  constructor(private channel: IChannel) { }
  
  marco(): Promise<string> {
  return this.channel.call('marco');
  }
  }
  ```
- 实现一对多IPC通信
  
  ```
  // 创建服务
  const service = new TestService();
  // 创建服务端
  const server = new TestIPCServer();
  // 创建服务端频道
  const channel = new TestChannel(service);
  // 服务端注册服务
  server.registerChannel('channel', channel);
  
  // 创建客户端-1
  const client1 = server.createConnection('client1');
  // 创建客户端-1对应的频道服务
  const ipcService1 = new TestChannelClient(client1.getChannel('channel'));
  // 创建客户端-2
  const client2 = server.createConnection('client2');
  // 创建客户端-2对应的频道服务
  const ipcService2 = new TestChannelClient(client2.getChannel('channel'));
  ```
- 客户端监听服务端
  
  ```
  ipcService1.onPong(() => console.log('Receive ping message on service1'));
  ipcService2.onPong(() => console.log('Receive ping message on service2'));
  ```
- 服务端通知：
  
  ```
  service.ping('hello world');
  ```
- ### 4.3.3.  资料总结
- [https://developer.aliyun.com/article/1191616](https://developer.aliyun.com/article/1191616)
- ## 4.4.  同步屏障
- barrier 类通常指的是 TypeScript 或 JavaScript 中的 Barrier 类型，它用于控制异步操作的执行顺序。Barrier 类型是 vscode 扩展 API 中的一部分，用于确保异步操作在特定的顺序下执行，或者在所有操作都完成之后才执行某些操作。
- Barrier 类型的主要作用包括：
	- 同步异步操作：Barrier 可以用来同步多个异步操作，确保它们按照特定的顺序执行。这在需要等待多个异步操作完成后才能继续执行下一步的场景中非常有用。
	- 等待所有操作完成：使用 Barrier，你可以等待一组异步操作全部完成后再执行某些代码。这可以通过 Barrier.wait() 方法实现，该方法会阻塞直到所有注册的异步操作都完成。
	- 避免竞态条件：在多线程或多任务环境中，Barrier 可以帮助避免竞态条件，确保资源在被访问之前已经准备好。
	- 简化异步代码：通过使用 Barrier，可以简化异步代码的复杂性，使得代码更加清晰和易于管理。
- 在 VSCode 扩展开发中，Barrier 类型通常用于以下场景：
	- 当扩展需要等待用户完成某些操作（如选择文件、输入文本等）后再继续执行。
	- 当扩展需要在多个异步操作完成后更新 UI 或执行其他逻辑。
	  
	  ```
	  export class Barrier {
	  private _isOpen: boolean;
	  private _promise: Promise<boolean>;
	  private _completePromise!: (v: boolean) => void;
	  
	  constructor() {
	  this._isOpen = false;
	  this._promise = new Promise<boolean>((c, e) => {
	  this._completePromise = c;
	  });
	  }
	  
	  isOpen(): boolean {
	  return this._isOpen;
	  }
	  
	  open(): void {
	  this._isOpen = true;
	  this._completePromise(true);
	  }
	  
	  wait(): Promise<boolean> {
	  return this._promise;
	  }
	  }
	  ```
- # 5.  VSCode的关键机制和关键部分解析
- ## 5.1.  IoC和DI
- ### 5.1.1.  简要介绍
- **控制反转（Inversion of Control）**是一种是面向对象编程中的一种设计原则，用来减低计算机代码之间的耦合度。其基本思想是：借助于“第三方”实现具有依赖关系的对象之间的解耦。
  
  ![](https://cdn.nlark.com/yuque/0/2023/webp/2713067/1695540765789-f62a4f13-ca6a-4c62-b1b7-2810c517cb84.webp)![](https://cdn.nlark.com/yuque/0/2023/webp/2713067/1695540773800-54593eba-df0e-49aa-8fde-3b3ad9ca417b.webp)
- 由于引进了中间位置的“第三方”，也就是IOC容器，使得A、B、C、D这4个对象没有了耦合关系，齿轮之间的传动全部依靠“第三方”了，全部对象的控制权全部上缴给“第三方”IOC容器，所以，IOC容器成了整个系统的关键核心。
- **依赖注入（Dependency Injection）**就是将实例变量传入到一个对象中。**控制反转**是一种思想，**依赖注入**是一种设计模式。IoC框架使用依赖注入作为实现控制反转的方式，但是控制反转还有其他的实现方式。
- ### 5.1.2.  VSCode依赖注入简要介绍
- VSCode 的代码是围绕着各式各样的 service 组织起来的，这些 service 基本都定义在 platform 那一层，通过构造函数注入器（constructor injection）注入到 client 中。
- 一个 service 需要两部分定义：1. service 接口定义 2. service 标志符。service 标志符是一个装饰器（Decorator是ES7的提案）并且必须和 service 接口同名。
- 在VSCode的编码实现中，我们会发现实现一个服务所需要的通用步骤：
	- 定义服务接口：首先需要定义服务接口，该接口定义了服务的 API，即服务能够提供哪些功能。接口通常放在 vs/platform 文件夹下的一个子文件夹中，比如 vs/platform/telemetry/common/telemetry。
	- 定义与服务器同名的Identifier，比如export const IProductService = createDecorator<IProductService>('productService');。
	- 注册服务：其次需要在应用程序的入口处，即 vs/code/electron-main/main.ts 中注册服务，并将其添加到容器中（以Identifier为key，实例或者SyncDescriptor实例作为value）。
- VS Code 中依赖注入的实现主要在 vs/platform/instantiation/common 文件夹下，如下：
	- descriptors.ts           服务实例包装类
	- extensions.ts            通用服务注册、获取服务
	- graph.ts                   基于有向图的依赖分析
	- instantiation.ts         服务实例(创建 Decorator、存储服务的依赖)
	- instantiationService.ts    容器
	- serviceCollection.ts         服务集合
- VSCode的依赖注入大致实现的原理就是：
	- 第一次会将需要注入的服务形成一个有向图
	- 第二次会遍历这个有向图然后生成对应的实例注入。使用图的数据结构可以清晰地看到依赖关系链路，并且能够快速识别出循环依赖问题。
- ### 5.1.3.  基本用法
- 首先需要定义一个类并在其构造函数中声明依赖的服务
  
  ```
  class MyClass {
  constructor(
  @IAuthService private readonly authService: IAuthService,
  @IStorageService private readonly storageService: IStorageService,
  ) {
  }
  }
  ```
- 构造函数中的 @IAuthService 和 @IStorageService 是两个 Decorator（装饰器），装饰器在 JavaScript 中还属于一项提案，在 TypeScript 中是一项实验性特性。它可以被附加到类声明、方法、访问符以及参数上，在这段代码中他们被附加到了 MyClass 的构造函数参数 authService 和 storageService 上，也就是参数装饰器。参数装饰器会在运行时被函数调用，并传入三个参数：
- 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象
- 成员的名字
- 参数在函数参数列表中的索引
- 服务的 Decorator 和接口定义一般如下
  
  ```
  // 创建装饰器
  export const IAuthService = createDecorator<IAuthService>('AuthService');
  // 接口
  export interface IAuthService {
  readonly id: string;
  readonly nickName: string;
  readonly firstName: string;
  readonly lastName: string;
  requestService: IRequestService;
  }
  ```
- 服务接口需要有具体实现，同时也允许依赖其他服务
- ```
  class AuthServiceImpl implements IAuthService {
    constructor(
  		@IRequestService public readonly requestService IRequestService,		
    ){
    }
  
    public async getUserInfo() {
  		const { id, nickName, firstName } = await getUserInfo();
  		this.id = id;
  		this.nickName = nickName;
  		this.firstName = firstName;
  		//...
  	} 
  }
  ```
- 还需要一个服务集，用于保存一组服务，并用其来创建一个容器
  
  ```
  // 服务集
  export class ServiceCollection {
  
  private _entries = new Map<ServiceIdentifier<any>, any>();
  
  constructor(...entries: [ServiceIdentifier<any>, any][]) {
  for (let [id, service] of entries) {
  	this.set(id, service);
  }
  }
  
  set<T>(id: ServiceIdentifier<T>, instanceOrDescriptor: T | SyncDescriptor<T>): T | SyncDescriptor<T> {
  const result = this._entries.get(id);
  this._entries.set(id, instanceOrDescriptor);
  return result;
  }
  
  forEach(callback: (id: ServiceIdentifier<any>, instanceOrDescriptor: any) => any): void {
  this._entries.forEach((value, key) => callback(key, value));
  }
  
  has(id: ServiceIdentifier<any>): boolean {
  return this._entries.has(id);
  }
  
  get<T>(id: ServiceIdentifier<T>): T | SyncDescriptor<T> {
  return this._entries.get(id);
  }
  }
  ```
- 前文说到对象由容器自动实例化，实际上在 VSCode 中一些服务没有其他依赖（例如日志服务），仅被其他服务所依赖，所以可以手动实例化并注册到容器中。而这个例子中 AuthServiceImpl 还依赖 IRequestService，需要用 SyncDescriptor 封装一下保存在服务集中
  
  ```
  const services = new ServiceCollection(); // 创建一个服务集
  const logService = new LogService(); // 直接实例化一个服务
  
  services.set(ILogService, logService);
  services.set(IAuthService, new SyncDescriptor(AuthServiceImpl)); // 第一个参数即服务的装饰器
  ```
- SyncDescriptor 是一个用于包装需要被容器实例化容器的描述符对象，它保存了对象的构造器和静态参数（需要被直接传递给构造函数）
  
  ```
  export class SyncDescriptor<T> {
  
  readonly ctor: any;
  readonly staticArguments: any[];
  readonly supportsDelayedInstantiation: boolean;
  
  constructor(ctor: new (...args: any[]) => T, staticArguments: any[] = [], supportsDelayedInstantiation: boolean = false) {
  this.ctor = ctor; // 服务的构造器
  this.staticArguments = staticArguments; // 静态参数
  this.supportsDelayedInstantiation = supportsDelayedInstantiation; // 是否支持延迟实例化
  }
  }
  ```
- 到这里我们可以创建容器并把服务注册到容器中了，VSCode 中容器是 InstantiationService
  
  ```
  const instantiationService = new InstantiationService(services, true);
  ```
- InstantiationService 是依赖注入的核心，当服务被注册到容器后，我们需要先手动实例化程序入口，在 VSCode 中即是 CodeApplication，容器（instantiationService）保存着这些对象的依赖关系，所以 CodeApplication 也需要借助容器来实例化。
  
  ```
  // 这里第二和第三个参数是 CodeApplication 构造器的静态参数，需要手动传递进去
  instantiationService.createInstance(CodeApplication, mainIpcServer, instanceEnvironment).startup();
  ```
- 同时也可以手动获取服务实例，需要调用 instantiationService.invokeFunction 方法，传入一个回调函数，其参数是一个访问器，当通过访问器获取指定服务时，容器会自动去分析它所依赖的服务并自动实例化后返回。
  
  ```
  instantiationService.invokeFunction(accessor => {
  const logService = accessor.get(ILogService);
  const authService = accessor.get(IAuthService);
  });
  ```
- instantiationService 包含一个成员方法 createChild ，可以创建一个子容器，为了更好地划分依赖关系，子容器可以访问父容器中的服务实例，反之父容器则无法访问子容器的实例，当子容器中不存在所需要的服务实例时会调用 instantiationService._parent 获取父容器的引用并逐层往上查找依赖。
- 以上是 VSCode 中实现依赖注入的基本用法，相比传统 Spring 等框架来说简单了不少，没有那么多种注入方式，不需要将依赖关系写到单独某个文件中，同时也提供了手动获取依赖及实例化的机制。
- **总体来说，VSCode除了可以运用这种方式来实现Service的自动注入以外，还可以通过这种方式来自由地、灵活地加载在不同环境下的Service服务。**
- ### 5.1.4.  实现原理
- 一开始调用 createDecorator 函数定义了一个服务的装饰器，用于在构造函数中声明依赖关系以方便注入依赖。createDecorator 的主要作用是返回一个装饰器
- ```
  export function createDecorator<T>(serviceId: string): { (...args: any[]): void; type: T; } {
  
    // 已经保存过的服务会直接返回其装饰器
  	if (_util.serviceIds.has(serviceId)) {
  		return _util.serviceIds.get(serviceId)!;
  	}
  
    // 声明装饰器
  	const id = <any>function (target: Function, key: string, index: number): any {
  		if (arguments.length !== 3) {
  			throw new Error('@IServiceName-decorator can only be used to decorate a parameter');
  		}
      // 将服务作为依赖保存在为目标类的属性中
  		storeServiceDependency(id, target, index, false);
  	};
  
  	id.toString = () => serviceId;
  
  	_util.serviceIds.set(serviceId, id);
  	return id;
  }
  ```
- 同时调用 storeServiceDependency 函数将传入的服务 ID (唯一的字符串)及索引保存在所装饰类的一个成员 **$di$dependencies** 数组中
  
  ```
  function storeServiceDependency(id: Function, target: Function, index: number, optional: boolean): void {
  if (target[_util.DI_TARGET] === target) {
  target[_util.DI_DEPENDENCIES].push({ id, index, optional });
  } else {
  target[_util.DI_DEPENDENCIES] = [{ id, index, optional }];
  target[_util.DI_TARGET] = target;
  }
  }
  ```
- 其中 _util.DI_DEPENDENCIES 和 _util.DI_TARGET 分别是两个 magic string
  
  ```
  export const DI_TARGET = '$di$target';
  export const DI_DEPENDENCIES = '$di$dependencies';
  ```
- 对于第一个例子中 MyClass，其构造函数中的两个装饰器在编译时会被自动执行，并将依赖的服务记录到 $di$dependencies
  
  ```
  MyClass['$di$dependencies'] = [
  { id: 'AuthService', index: 0, optional: false },
  { id: 'StorageService', index: 1, optional: false }
  ];
  
  MyClass['$di$target'] = MyClass;
  ```
- 对于被装饰器装饰过的入参，会直接通过IoC容器被注入到对应的类中去。
- ### 5.1.5.  资料总结
- [https://zhuanlan.zhihu.com/p/60228431](https://zhuanlan.zhihu.com/p/60228431)
- ## 5.2.  AMD机制
- ### 5.2.1.  AMD和CMD的异同点
- AMD规范采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。这里介绍用require.js实现AMD规范的模块化：用require.config()指定引用路径等，用definde()定义模块，用require()加载模块。
- 首先我们需要引入require.js文件和一个入口文件main.js。main.js中配置require.config()并规定项目中用到的基础模块。
  
  ```
  /** 网页中引入require.js及main.js **/
  <script src="js/require.js" data-main="js/main"></script>
  
  /** main.js 入口文件/主模块 **/
  // 首先用config()指定各模块路径和引用名
  require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
    "underscore": "underscore.min",
  }
  });
  // 执行基本操作
  require(["jquery","underscore"],function($,_){
  // some code here
  });
  ```
- 引用模块的时候，我们将模块名放在[]中作为reqiure()的第一参数；如果我们定义的模块本身也依赖其他模块,那就需要将它们放在[]中作为define()的第一参数。
- ```
  // 定义math.js模块
  define(function () {
      var basicNum = 0;
      var add = function (x, y) {
          return x + y;
      };
      return {
          add: add,
          basicNum :basicNum
      };
  });
  
  // 定义一个依赖underscore.js的模块
  define(['underscore'],function(_){
    var classify = function(list){
      _.countBy(list,function(num){
        return num > 30 ? 'old' : 'young';
      })
    };
    return {
      classify :classify
    };
  })
  
  // 引用模块，将模块放在[]内
  require(['jquery', 'math'],function($, math){
    var sum = math.add(10,20);
    $("#sum").html(sum);
  });
  ```
- CMD是另一种js模块化方案，它与AMD很类似，不同点在于：AMD推崇依赖前置、提前执行，CMD推崇依赖就近、延迟执行。此规范其实是在sea.js推广过程中产生的。
  
  ```
  /** AMD写法 **/
  define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) { 
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    } 
  });
  
  /** CMD写法 **/
  define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
  });
  
  /** sea.js **/
  // 定义模块 math.js
  define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
  });
  
  // 加载模块
  seajs.use(['math.js'], function(math){
    var sum = math.add(1+2);
  });
  ```
- ### 5.2.2.  vscode-loader
- 在 vscode 的加载过程中，有这样一行代码值得注意，它位于 main.js 文件中，而这是 Electron App 运行的入口文件：
  
  ```
  require('./bootstrap-amd').load('vs/code/electron-main/main', () => {
    // ...
  });
  ```
- vs/code/election-main/main 是 vscode 应用的主入口，这句代码的意思是加载主入口文件并执行。bootstrap-amd 文件暴露的 load 方法则又调用了 vs/loader 文件暴露的 loader 方法：
  
  ```
  const loader = require('./vs/loader')
  
  exports.load = function (entrypoint, onLoad, onError) {
    // ...
    loader([entrypoint], onLoad, onError);
  };
  ```
- loader.js中，被默认执行的是以下的代码：
  
  ```
  export function init(): void {
        if (typeof global.require !== 'undefined' || typeof require !== 'undefined') {
            // 将原本 node.js 的 require 函数保存在这个局部变量中
            const _nodeRequire = (global.require || require);
            if (typeof _nodeRequire === 'function' && typeof _nodeRequire.resolve === 'function') {
                // 然后在 RequireFunc 上挂在原来的 require 函数
                const nodeRequire = ensureRecordedNodeRequire(moduleManager.getRecorder(), _nodeRequire);
                global.nodeRequire = nodeRequire;
                (<any>RequireFunc).nodeRequire = nodeRequire;
                (<any>RequireFunc).__$__nodeRequire = nodeRequire;
            }
        }
  
        // 在 node.js 环境中（非 Electron 渲染进程环境）
        if (env.isNode && !env.isElectronRenderer) {
            module.exports = RequireFunc; // vscode-loader.js 文件导出 RequireFunc
            require = <any>RequireFunc; // patch 全局 require 函数
        } else {
            if (!env.isElectronRenderer) {
                global.define = DefineFunc; // patch 全局 define 函数
            }
            global.require = RequireFunc; // patch 全局 require 函数
        }
    }
  
    if (typeof global.define !== 'function' || !global.define.amd) {
        moduleManager = new ModuleManager(env, createScriptLoader(env), DefineFunc, RequireFunc, Utilities.getHighPerformanceTimestamp());
  
        // The global variable require can configure the loader
        if (typeof global.require !== 'undefined' && typeof global.require !== 'function') {
            RequireFunc.config(global.require);
        }
  
        // This define is for the local closure defined in node in the case that the loader is concatenated
        define = function () {
            return DefineFunc.apply(null, arguments);
        };
        define.amd = DefineFunc.amd;
  
        if (typeof doNotInitLoader === 'undefined') {
            init();
        }
    }
  ```
	- 创建了一个 ScriptLoader，即脚本加载器，脚本加载器因代码运行环境而异，有 NodeScriptLoaderWorkScriptLoader 和 BrowserScriptLoader 三种，负责加载 js 文件
	- 定义了 DefineFunc 和 RequireFunc，即模块化系统中的 define 函数和 require 函数，分别用于定义和加载一个模块
	- 创建了一个 ModuleManager，即模块管理器，它用于按照正确的顺序来解析 js 文件并执行
	- 覆盖了全局作用域中的 define 以及 require 变量（node 原生 require 变量的值被保存在 _nodeRequire 变量中）
- **总体来说，vscode-loader使用了IIFE，实现了对不同环境下的js代码的模块依赖加载。**
- ### 5.2.3.  资料总结
- [https://zhuanlan.zhihu.com/p/367639259](https://zhuanlan.zhihu.com/p/367639259) （vscode-loader）
- vscode-loader源代码仓库：[https://github.com/microsoft/vscode-loader](https://github.com/microsoft/vscode-loader)