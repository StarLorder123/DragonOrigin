# 1.  RPC通信协议
- RPC（Remote Procedure Call）远程过程调用协议是一个用于建立适当框架的协议。从本质上讲，它使一台机器上的程序能够调用另一台机器上的子程序，而不会意识到它是远程的。
- RPC 是一种软件通信协议，一个程序可以用来向位于网络上另一台计算机的程序请求服务，而不必了解网络的细节。RPC 被用来像本地系统一样调用远程系统上的其他进程。过程调用有时也被称为函数调用或子程序调用。
- 过程是什么？ 过程就是业务处理、计算任务，更直白的说，就是程序。
- RPC 就是想调用本地方法一样调用远程的过程。与本地过程调用一样，RPC 也是一种同步操作，需要挂起请求程序，直到返回远程过程的结果。但是，使用共享相同地址空间的轻量级进程或线程可以同时执行多个 RPC。
- ## 1.1.  RPC工作流程
- 在 RPC 的工作期间，将执行以下步骤：
  
  ![](https://cdn.nlark.com/yuque/0/2024/png/2713067/1724858207553-473d25b9-6aa0-4536-bb17-4035aebbe0af.png)
- RPC 的工作过程如下：
	- 服务消费方（client）调用以本地调用方式调用服务；
	- client stub 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；这就是所谓的 marshalling。
	- client stub 找到服务地址，并将消息发送到服务端；这个过程可能是面向连接或无连接的。
	- server stub 收到消息后进行解码；也叫 unmarshalling。
	- server stub 根据解码结果调用本地的服务；
	- 本地服务执行并将结果返回给 server stub ；
	- server stub 将返回结果打包成消息并发送至消费方；
	- client stub 接收到消息，并进行解码；
	- 服务消费方得到最终结果。并且返回值被设置在本地进程的堆栈中。
- # 2.  Protobuf
- protocol buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。
- 可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。
- 可自定义数据的结构，使用各种语言进行编写和读取结构数据。也可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。
- ## 2.1.  proto解析
  
  ```
  syntax = "proto3";      // 必须加上，否则默认使用proto2
  
  message SearchRequest {
    string query = 1;     // 字段编号为1的字符串类型
    int32 page_number = 2;
    int32 result_per_page = 3;
    enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
    }
    Corpus corpus = 4;
  }
  
  message SearchResponse {
  ...
  }
  ```
- 注意：每一个字段都有一个字段编号，用于二进制格式标识字段。1-15编号需要1个byte编码，16-2047编号需要两个字节。所以高频使用字段放在1~15编号中
- singular是默认字段规则，具有0/1个；repeated重复0~任意次
- reserved预留字段；或enum中的保留值
- import导入其他文件的定义，注意：导入proto2是可行的，但enum不能直接使用
- 支持嵌套类型
- Any需要import "google/protobuf/any.proto"，可以在没有指定proto定义情况下作为一个嵌套类型使用
- oneof：针对多个可选字段且只有一个字段会被设置时使用，可以节省内存
- map：关联映射如map<string, Project> projects = 3;
- package：可选添加，防止不同消息类型有命名冲突
- 可以在不破坏现有代码的情况下扩展proto，只需要满足以下条件：
	- 不改变任何现有字段号
	- 如果添加新字段，则在新代码中注意旧消息的默认值；旧代码中会视为未知字段并包含在序列化输出中（3.5及更晚版本）
	- 如果删除某个字段，确保该字段不在新proto中使用
	- int32、uint32 int64、uint64、bool是兼容的，可以直接转换而不会破坏兼容性。（注意：64位数字用32位读取会产生截断）
	- sint32、sint64是兼容的但不与其他int类型兼容
	- string和bytes是兼容的，只要bytes是utf-8编码
	- 嵌套消息和bytes是兼容的，只要bytes包含该消息的一个编码过的版本
	- fixed32、sfixed32是兼容的，fixed64、sfixed64是兼容的
	- 对于string、bytes、message，optional与repeated兼容
	- enum与int32、uint32 int64、uint64兼容（注意如果值不相兼容则会被截断），但客户端反序列化后可能需要不同的处理方式。
	- 给oneof添加一个单独字段是安全且二进制兼容的，添加多个字段则需要小心只能有一个字段被设置。给任何现有的oneof添加任何字段都需要谨慎，这未必安全。
- # 3.  RPC协议的实现
- ## 3.1.  Nodejs-RPC
- ### 3.1.1.  JSON-RPC
- rpc实现的方式可以有两种，其中一种就是JSON格式的
- 客户端
  
  ```
  const jayson = require('jayson');
  
  const client = jayson.client.http('http://localhost:3000');
  
  client.request('myFunction', ['World'], function(err, error, response) {
  if (!err) {
    console.log(response);
  } else {
    console.error(err);
  }
  });
  ```
- 服务端
  
  ```
  const jayson = require('jayson');
  
  const server = new jayson.Server({
  myFunction: function(args, callback) {
    callback(null, `Hello, ${args[0]}`);
  }
  });
  
  server.http().listen(3000);
  ```
- ### 3.1.2.  gRPC
- 首先需要定义一个proto的内容
  
  ```
  syntax = "proto3";
  
  service MyService {
  rpc MyFunction (MyRequest) returns (MyResponse);
  }
  
  message MyRequest {
  string name = 1;
  }
  
  message MyResponse {
  string message = 1;
  }
  ```
- 客户端
  
  ```
  const grpc = require('@grpc/grpc-js');
  const protoLoader = require('@grpc/proto-loader');
  const path = require('path');
  const packageDefinition = protoLoader.loadSync(path.join(__dirname, 'message.proto'), {});
  const protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
  
  const client = new protoDescriptor.MyService('localhost:50051', grpc.credentials.createInsecure());
  
  client.myFunction({ name: 'World' }, (error, response) => {
    if (!error) {
        console.log(response.message);
    } else {
        console.error(error);
    }
  });
  ```
- 服务端
  
  ```
  const grpc = require('@grpc/grpc-js');
  const protoLoader = require('@grpc/proto-loader');
  const path = require('path');
  const packageDefinition = protoLoader.loadSync(path.join(__dirname, 'message.proto'), {});
  const protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
  
  const myService = protoDescriptor.MyService;
  
  function myFunction(call, callback) {
  callback(null, { message: `Hello, ${call.request.name}` });
  }
  
  function main() {
  const server = new grpc.Server();
  server.addService(myService.service, { myFunction: myFunction });
  server.bindAsync('127.0.0.1:50051', grpc.ServerCredentials.createInsecure(), () => {
    server.start()
  });
  }
  
  main();
  ```
-