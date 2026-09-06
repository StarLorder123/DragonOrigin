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
-