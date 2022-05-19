# gRPC Java Quick start

最初にJavaRPCアプリを実行しよう！！  
このガイドでは、簡単な実例を使用してJavaでgRPCを使い始めることができます。  
[公式ドキュメント](https://grpc.io/docs/languages/java/quickstart/)
***

## 前提条件

* JDK バージョン7以降インストール済み

## サンプルコードを取得する

サンプルコードは[ここ](https://github.com/grpc/grpc-java)のリポジトリにあります。

1. リポジトリをzipファイルで[ダウンロード](https://github.com/grpc/grpc-java/archive/v1.45.1.zip)する。もしくは、リポジトリをクローンする。

```commands
> git clone -b v1.45.1 --depth 1 https://github.com/grpc/grpc-java
```

2. examplesフォルダに移動する。

```
> cd grpc-java\examples
```
---


## exampleコードを動かす
**※以下の作業はexamplesフォルダから行う。**

1. クライアントコードとサーバーコードをコンパイルする
```
> .\gradlew installDist
```
2. サーバーを動かす
```
> .\build\install\examples\bin\hello-world-server
```
下記の表示が出ればOK
>INFO: Server started, listening on 50051

※タイムスタンプは省略

3. 他のターミナルから、クライアントコードを動かす
```
> .\build\install\examples\bin\hello-world-client
```
下記の表示が出ればOK
>INFO: Will try to greet world ...  
>INFO: Greeting: Hello world

※タイムスタンプは省略

**おめでとう！これでクライアントサーバーアプリをgRPCで動かせたよ！！**

---
## gPRCシステムを更新する
このセクションでは、サーバーに関数を追加して、システムを更新します。gRPCシステムは、[Protocol Buffer](https://developers.google.com/protocol-buffers)を使用して定義されます。`.proto`ファイルについての詳細は[Basic tutorial](https://grpc.io/docs/languages/java/basics/)をご覧ください。今あなたは`SayHello()`メソッドがサーバーとクライアントの両方にあることが確認できます。そのメソッドは、クライアントから`HelloRequest`パラメータを渡されます。そして、サーバーから`HelloReply`が返ってきます。その関数は以下のように定義されます。

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

`src/main/proto/helloworld.proto`を開いて、`SayHello()`関数と同じレスポンスとリクエストで、新しく`sayHelloAgain()`関数を追加してください。

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

セーブするのを忘れずに！

---

## アプリケーションを更新する

サンプルプログラムをビルドしたときに、`GreeterGrpc.java`は再生成されます。これにより、リクエストの入力とレスポンスの取得やシリアライズも再生成されます。  
ここで、サンプルプログラムに新しい関数を追加したため、新しい関数を実装して呼び出す必要があります。

---

## サーバーを更新する

同じフォルダの中で、`src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java`を開いてください。次の関数を追加します。

``` protobuf
private class GreeterImpl extends GreeterGrpc.GreeterImplBase {

  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }

  @Override
  public void sayHelloAgain(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello again " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
}
```

---

## クライアントを更新する

同じフォルダの中で、`src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java`を開いてください。次の関数を追加します。

``` protobuf
public void greet(String name) {
  logger.info("Will try to greet " + name + " ...");
  HelloRequest request = HelloRequest.newBuilder().setName(name).build();
  HelloReply response;
  try {
    response = blockingStub.sayHello(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
  try {
    response = blockingStub.sayHelloAgain(request);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
    return;
  }
  logger.info("Greeting: " + response.getMessage());
}
```

---

## アップデートしたアプリケーションを実行する

先ほどのようにクライアントとサーバーを実行してください。**下のコマンドを`example`フォルダから行うのを忘れずに。**

1. クライアントコードとサーバーコードをコンパイルする
```
> .\gradlew installDist
```
2. サーバーを動かす
```
> .\build\install\examples\bin\hello-world-server
```
下記の表示が出ればOK
>INFO: Server started, listening on 50051

※タイムスタンプは省略

3. 他のターミナルから、クライアントコードを動かす
```
> .\build\install\examples\bin\hello-world-client
```
下記の表示が出ればOK
>INFO: Will try to greet world ...  
>INFO: Greeting: Hello world

※タイムスタンプは省略

---

## 次のステップ
- どのようにgRPCが動いているのか[イントロダクション](https://grpc.io/docs/what-is-grpc/introduction/)と[コアの概念](https://grpc.io/docs/what-is-grpc/core-concepts/)から学ぶ。
- [ベーシックチュートリアル](https://grpc.io/docs/languages/java/basics/)を行う。
- [APIリファレンス](https://grpc.io/docs/languages/java/api)を調べる


