# Rpc Generation

Goctl Rpc是`goctl`脚手架下的一个rpc服务代码生成模块，支持proto模板生成和rpc服务代码生成，通过此工具生成代码你只需要关注业务逻辑编写而不用去编写一些重复性的代码。这使得我们把精力重心放在业务上，从而加快了开发效率且降低了代码出错率。

## 特性

* 简单易用
* 快速提升开发效率
* 出错率低
* 支持基于main proto作为相对路径的import
* 支持map、enum类型
* 支持any类型

## 快速开始

### 方式一：快速生成greet服务

  通过命令 `goctl rpc new ${servieName}`生成

  如生成greet rpc服务：

  ```shell script
  goctl rpc new greet
  ```

  执行后代码结构如下:

  ```golang
  └── greet
    ├── etc
    │   └── greet.yaml
    ├── go.mod
    ├── go.sum
    ├── greet
    │   ├── greet.go
    │   ├── greet_mock.go
    │   └── types.go
    ├── greet.go
    ├── greet.proto
    ├── internal
    │   ├── config
    │   │   └── config.go
    │   ├── logic
    │   │   └── pinglogic.go
    │   ├── server
    │   │   └── greetserver.go
    │   └── svc
    │       └── servicecontext.go
    └── pb
        └── greet.pb.go
  ```

rpc一键生成常见问题解决见 <a href="#常见问题解决">常见问题解决</a>

### 方式二：通过指定proto生成rpc服务

* 生成proto模板

  ```shell script
  goctl rpc template -o=user.proto
  ```

  ```golang
  syntax = "proto3";

  package remote;

  message Request {
    // 用户名
    string username = 1;
    // 用户密码
    string password = 2;
  }

  message Response {
    // 用户名称
    string name = 1;
    // 用户性别
    string gender = 2;
  }

  service User {
    // 登录
    rpc Login(Request)returns(Response);
  }
  ```

* 生成rpc服务代码

  ```shell script
  goctl rpc proto -src=user.proto
  ```

  代码tree

  ```Plain Text
  user
      ├── etc
      │   └── user.json
      ├── internal
      │   ├── config
      │   │   └── config.go
      │   ├── handler
      │   │   ├── loginhandler.go
      │   ├── logic
      │   │   └── loginlogic.go
      │   └── svc
      │       └── servicecontext.go
      ├── pb
      │   └── user.pb.go
      ├── shared
      │   ├── mockusermodel.go
      │   ├── types.go
      │   └── usermodel.go
      ├── user.go
      └── user.proto
  ```

## 准备工作

* 安装了go环境
* 安装了protoc&protoc-gen-go，并且已经设置环境变量
* 更多问题请见 <a href="#注意事项">注意事项</a>

## 用法

### rpc服务生成用法

```shell script
goctl rpc proto -h
```

```shell script
NAME:
   goctl rpc proto - generate rpc from proto

USAGE:
   goctl rpc proto [command options] [arguments...]

OPTIONS:
   --src value, -s value         the file path of the proto source file
   --dir value, -d value         the target path of the code,default path is "${pwd}". [option]
   --service value, --srv value  the name of rpc service. [option]
   --idea                        whether the command execution environment is from idea plugin. [option]

```

### 参数说明

* --src 必填，proto数据源，目前暂时支持单个proto文件生成，这里不支持（不建议）外部依赖
* --dir 非必填，默认为proto文件所在目录，生成代码的目标目录
* --service 服务名称，非必填，默认为proto文件所在目录名称，但是，如果proto所在目录为一下结构：

    ```shell script
    user
        ├── cmd
        │   └── rpc
        │       └── user.proto
    ```

    则服务名称亦为user，而非proto所在文件夹名称了，这里推荐使用这种结构，可以方便在同一个服务名下建立不同类型的服务(api、rpc、mq等)，便于代码管理与维护。
  
  > 注意：这里的shared文件夹名称将会是代码中的package名称。

* --idea 非必填，是否为idea插件中执行，保留字段，终端执行可以忽略


### 开发人员需要做什么

关注业务代码编写，将重复性、与业务无关的工作交给goctl，生成好rpc服务代码后，开饭人员仅需要修改

* 服务中的配置文件编写(etc/xx.json、internal/config/config.go)
* 服务中业务逻辑编写(internal/logic/xxlogic.go)
* 服务中资源上下文的编写(internal/svc/servicecontext.go)


### 注意事项

* `google.golang.org/grpc`需要降级到v1.26.0,且protoc-gen-go版本不能高于v1.3.2（see [https://github.com/grpc/grpc-go/issues/3347](https://github.com/grpc/grpc-go/issues/3347)）即
  
  ```shell script
  replace google.golang.org/grpc => google.golang.org/grpc v1.26.0
  ```

* proto不支持暂多文件同时生成
* proto不支持外部依赖包引入，message不支持inline
* 目前main文件、shared文件、handler文件会被强制覆盖，而和开发人员手动需要编写的则不会覆盖生成，这一类在代码头部均有

```shell script
    // Code generated by goctl. DO NOT EDIT!
    // Source: xxx.proto
```

的标识，请注意不要将也写业务性代码写在里面。

## any和import支持
* 支持any类型声明
* 支持import其他proto文件

  any类型固定import为`google/protobuf/any.proto`,且从${GOPATH}/src中查找，proto的import支持main proto的相对路径的import，且与proto文件对应的pb.go文件必须在proto目录中能被找到。不支持工程外的其他proto文件import。

> ⚠️注意： 不支持proto嵌套import，即：被import的proto文件不支持import。

### import书写格式
import书写格式
```golang
// @{package_of_pb} 
import {proto_omport}
```
@{package_of_pb}：pb文件的真实import目录。
{proto_omport}：proto import


如：demo中的

```golang
// @greet/base
import "base/base.proto";
```

工程目录结构如下
```
greet
│   ├── base
│   │   ├── base.pb.go
│   │   └── base.proto
│   ├── demo.proto
│   ├── go.mod
│   └── go.sum
```

demo 
```golang
syntax = "proto3";
import "google/protobuf/any.proto";
// @greet/base
import "base/base.proto";
package stream;


enum Gender{
  UNKNOWN = 0;
  MAN = 1;
  WOMAN = 2;
}

message StreamResp{
  string name = 2;
  Gender gender = 3;
  google.protobuf.Any details = 5;
  base.StreamReq req = 6;
}
service StreamGreeter {
  rpc greet(base.StreamReq) returns (StreamResp);
}
```



## 常见问题解决(go mod工程)

* 错误一:

  ```golang
  pb/xx.pb.go:220:7: undefined: grpc.ClientConnInterface
  pb/xx.pb.go:224:11: undefined: grpc.SupportPackageIsVersion6
  pb/xx.pb.go:234:5: undefined: grpc.ClientConnInterface
  pb/xx.pb.go:237:24: undefined: grpc.ClientConnInterface
  ```

  解决方法：请将`protoc-gen-go`版本降至v1.3.2及一下

* 错误二:

  ```golang

  # go.etcd.io/etcd/clientv3/balancer/picker
  ../../../go/pkg/mod/go.etcd.io/etcd@v0.0.0-20200402134248-51bdeb39e698/clientv3/balancer/picker/err.go:25:9: cannot use &errPicker literal (type *errPicker) as type Picker in return argument:*errPicker does not implement Picker (wrong type for Pick method)
    have Pick(context.Context, balancer.PickInfo) (balancer.SubConn, func(balancer.DoneInfo), error)
    want Pick(balancer.PickInfo) (balancer.PickResult, error)
    ../../../go/pkg/mod/go.etcd.io/etcd@v0.0.0-20200402134248-51bdeb39e698/clientv3/balancer/picker/roundrobin_balanced.go:33:9: cannot use &rrBalanced literal (type *rrBalanced) as type Picker in return argument:
    *rrBalanced does not implement Picker (wrong type for Pick method)
		have Pick(context.Context, balancer.PickInfo) (balancer.SubConn, func(balancer.DoneInfo), error)
    want Pick(balancer.PickInfo) (balancer.PickResult, error)
    #github.com/tal-tech/go-zero/zrpc/internal/balancer/p2c
    ../../../go/pkg/mod/github.com/tal-tech/go-zero@v1.0.12/zrpc/internal/balancer/p2c/p2c.go:41:32: not enough arguments in call to base.NewBalancerBuilder
	have (string, *p2cPickerBuilder)
  want (string, base.PickerBuilder, base.Config)
  ../../../go/pkg/mod/github.com/tal-tech/go-zero@v1.0.12/zrpc/internal/balancer/p2c/p2c.go:58:9: cannot use &p2cPicker literal (type *p2cPicker) as type balancer.Picker in return argument:
	*p2cPicker does not implement balancer.Picker (wrong type for Pick method)
		have Pick(context.Context, balancer.PickInfo) (balancer.SubConn, func(balancer.DoneInfo), error)
		want Pick(balancer.PickInfo) (balancer.PickResult, error)
  ```

  解决方法：
  
    ```golang
    replace google.golang.org/grpc => google.golang.org/grpc v1.26.0
    ```
