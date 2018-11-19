# Node.js 提供 gRPC 服务
## grpc
1. gRPC是一个高性能、开源的、通用的、面向移动端的RPC框架，传输协议基于HTTP/2
2. 支持双向流、流控、头部压缩、单TCP连接上的请求多路复用等特性。
3. 接口层面，gRPC默认使用 Protocol Buffers （简称 protobuf）做为其接口定义语言(IDL)来描述其服务接口和负载消息的格式。
## 与 Node.js 集成
protobuf 做为IDL的特点是语义良好、数据类型定义完备，当前语言版本分为 proto2 和 proto3 。

protobuf 的接口定义大致分为几个部分：

1. IDL版本 proto2/3
2. 包名字
3. 服务定义 和 方法定义
4. 消息定义：请求消息和响应消息
## 必备条件
node：这需要node安装，版本4.0或以上。如果您nodejs在Debian上使用可执行文件，则应安装该nodejs-legacy软件包。

注意：如果您node通过软件包管理器安装并且版本仍然不足4.0，请尝试从nodejs.org直接安装它。
## 安装
### 安装gRPC NPM包

```
npm install grpc
```

## Demo 
### 中间接口定义(hello.proto)

```
syntax = "proto3";

option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
option java_outer_classname = "HelloWorldProto";
option objc_class_prefix = "HLW";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
  string msg = 2;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```
### 公用配置(config.js)

```
$IF = {
    "grpcServer": '127.0.0.1:50051'
}
```

### 服务端(greeter_server.js)

```
var PROTO_PATH = './arctern.proto';

var grpc = require('grpc');
var protoLoader = require('@grpc/proto-loader');
var packageDefinition = protoLoader.loadSync(
  PROTO_PATH,
  {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true
  });
var hello_proto = grpc.loadPackageDefinition(packageDefinition).arctern;

/**
 * Implements the SayHello RPC method.
 */
function sayHello (call, callback) {
  // console.log('new message', call.request);
  callback(null, { message: 'success' });
}

/**
 * Starts an RPC server that receives requests for the Greeter service at the
 * sample server port
 */
(function() {
  var server = new grpc.Server();
  server.addService(hello_proto.Arctern.service, { HandleArcternRequest: sayHello, SayHello: sayHello });
  server.bind($IF.grpcServer, grpc.ServerCredentials.createInsecure());
  server.start();
})();
```
### 客户端(greeter_client.js)

```
var PROTO_PATH = './arctern.proto';

var grpc = require('grpc');
var protoLoader = require('@grpc/proto-loader');
var packageDefinition = protoLoader.loadSync(
  PROTO_PATH,
  {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true
  });
var hello_proto = grpc.loadPackageDefinition(packageDefinition).arctern;

function main () {
  var client = new hello_proto.Arctern($IF.grpcServer,
    grpc.credentials.createInsecure());
  var user = process.argv.length >= 3 ? process.argv[2] : 'world';
  client.sayHello({name: user, msg: arg1}, function(err, response) {
    console.log('Greeting:', response.message);
  });
}
```

#### 运行这个服务


```
$ node greeter_server.js
```

#### 在另一个终端，运行客户端


```
$ node greeter_client.js
```

