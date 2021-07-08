---
slug: gRPC-go-python
title: 通过gRPC实现跨语言通信
author: Stitch-Zhang
author_title: An iddle programer
author_url: https://github.com/Stitch-Zhang
author_image_url: https://avatars1.githubusercontent.com/u/61350804?s=460&v=4
tags: [gRPC,Go,Python,ProtoBuffer]
---

在项目中，有时候会使用利用多种语言相互配合来完成某项任务,其必离不开跨语言之间的通信，通常为远程过程调用(`RPC`)或HTTP接口(`Restful API`)

本文中将介绍如何使用[gRPC](https://www.gRPC.io)实现`Go`与 `Python`之间的通信

<!--truncate-->

### 过程调用

过程调用可分为两类

- 本地过程调用(LPC，Local Procedure Call)是由`Windows NT内核`提供的内部`进程间通信`方式

- 远程过程调用(Remote Procedure Call) 首次出现于 `Unix`平台但后续发展迅速，几乎登陆所有操作系统。RPC是一种服务器-客户端（Client/Server）模式，经典实现是一个通过发送请求-接受回应进行信息交互的系统

## gRPC

[gRPC](https://www.gRPC.io) 是一个由Google开源的，基于`HTTP2.0`协议及`Protocol Buffer`的高性能跨平台统一RPC框架，能够实现跨平台跨语言之间的通信

![GRPC](https://www.grpc.io/img/landing-2.svg)
### 底层数据传输格式

- 【默认】[Protocol Buffers](https://developers.google.cn/protocol-buffers/docs/overview) 一种接口定义语言（Interface Definition Language)
- 【可选】JSON

#### 浅析Protocol Buffers

在 `Protocol Buffers` 中，一个最小的数据信息发送单位为：`消息（message）`。

`消息`结构类似于`结构体`，即一个或多个属性的结合体

### 使用流程🍜

- 编写`proto`文件

- 使用`protoc`将`proto`文件编译成指定语言的对应的`源`文件

- 程序引用

:::info
通常一个proto文件编译输出为两个源文件

如编写一个名为`aloha`的`proto`文件，指定`Go`为输出语言

- `aloha.pb.go` 负责对数据序列化/反序列化，获取/设置属性值``

- `aloha_grpc.pb.go` gRPC相关服务实现

:::

### 支持如下语言

- [Go](https://www.gRPC.io/docs/languages/go/)
- [Python](https://www.gRPC.io/docs/languages/python/)
- [C#](https://www.gRPC.io/docs/languages/csharp/)
- [C++](https://www.gRPC.io/docs/languages/cpp/)
- [Dart](https://www.gRPC.io/docs/languages/dart/)
- [Java](https://www.gRPC.io/docs/languages/java/)
- [Kotlin](https://www.gRPC.io/docs/languages/kotlin/)
- [Node](https://www.gRPC.io/docs/languages/node/)
- [Objective-C](https://www.gRPC.io/docs/languages/objective-c/)
- [PHP](https://www.gRPC.io/docs/languages/python/)
- [Ruby](https://www.gRPC.io/docs/languages/ruby/)

## 安装gRPC

### 本地环境信息

- Go：`1.16.3`
- Python：`3.7.6`

目录结构：

```shell
gRPC_demo:
├─go        Go相关代码
│  ├─ proto proto编译后的文件
│  ├─ server 服务端
│  └─ client 客户端
├─proto     公共proto文件     
└─py        Python相关代码
```

### Go准备gRPC相关环境

- 下载[protoc](https://github.com/protocolbuffers/protobuf/releases)执行文件，放置于环境变量目录中

- 安装Go的protoc插件

 ```go
    go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
    go install google.golang.org/gRPC/cmd/protoc-gen-go-gRPC@v1.1
 ```

:::tip
由于网络环境问题，`go install` 可能会显示超时，请参考[配置go module代理](https://goproxy.cn)

若安装后执行`protoc-gen-go`显示未找到，请确保`go根目录/bin`目录位于环境变量中
:::

### Python准备gRPC环境

#### 虚拟环境

> 在Python中，默认的包（模块）管理工具`pip`安装新的包，会将新安装的包的源文件放置于`python根目录/Lib/site-packages`下，以便程序使用，但这样通常会导致
 > - 若多个项目使用某包的指定不同版本，则会造成冲突
 > - 无法通过`pip freeze`有效的输出某个项目的依赖包集合信息

在Python 3.3版本时，内置了虚拟环境工具[venv](https://docs.python.org/zh-cn/3/library/venv.html)

#### 新建虚拟环境

```shell
cd py
python -m venv gRPC_demo
```

此时目录结构为为

```shell
gRPC_demo:
├─go
├─proto
└─py
    └─gRPC_demo
        ├─Include
        ├─Lib
        └─Scripts     环境激活脚本
        └─pyvenv.cfg  虚拟环境的配置信息
```

#### 当前窗口激活虚拟环境

终端类型|命令
--|:--:|
cmd|`activate.bat`
bash|`source activate`

请在 `Scripts`目录下执行

```shell
$~Scripts: source activate
```

此时命令行前应新增标识符 `(gRPC_demo)`

#### 使用pip安装gRPC

```shell
安装gRPC包
python -m pip install gRPCio
安装protocol buffer编译器 protoc
python -m pip install gRPCio-tools
```

## 示例

### 需求说明

我们将定义一个名为`Expirement`的类型(对象)，其有三个属性(成员)

- `name`  | 字符串
- `id`    | 数字
- `skill` | 字符串

实现通过一个`Name`属性来查询其对应的完整信息

### 定义proto文件

在`proto`目录中新建一个名为`expirement`的proto文件

`expirement.proto`

```go title="gRPC_demo/proto/expirement.proto"
// 设置protocol buffer版本
syntax = "proto3";

// 设置Go中包名
option go_package = "gRPC_demo/go";

// 定义一个名为Expirement的消息
// 其表示一个完整的Expirement类型
// 属性/成员 序号从1开始
message Expirement{
    string name = 1;  //属性序号：1
    int32 id = 2;     //属性序号：2
    string skill = 3; //属性序号：3
}

// 请求输入参数的消息
message NameRequest{
    string name = 1;
}

// 定义一个名为Manage的RPC服务，其返回的消息为Expirement
service Manage{
    //rpc的方法名为:PickOne
    rpc PickOne (NameRequest) returns (Expirement) {}
}
```

### 编译proto文件

请确保当前目录位于 `proto`

#### Go

```shell
protoc --go_out=../go/proto --go_opt=paths=source_relative --go-grpc_out=../go/proto --go-grpc_opt=paths=source_relative expirement.proto
```

参数解释：

- --go_out 输出的`protocol buffer`序列化/反序列化源文件路径
- --go_opt=paths=source_relative `go_out`路径输出选项为相对路径
- --go-grpc_out `gRPC`相关服务实现的文件路径，通常和 序列化/反序列化源文件放置同一目录
- --go-grpc_opt `grpc_opt`路径输出选项为相对路径
- `<proto文件>`

##### 输出的文件

- `expirement.pb.go`  负责定义消息的序列化/发布序列化、属性的值获取/设置
- `expirement_grpc.pb.go` 实现RPC服务

#### Python

```shell
python -m grpc_tools.protoc -I . --python_out=../py --grpc_python_out=../py expirement.proto
```

参数解释:

- --m grpc_tools.protoc 调用`grpc_tools.protoc`功能，依赖于上述中[使用pip安装gRPC](#使用pip安装grpc)
- -I . `proto`文件的输入目录
- --python_out 输出的`protocol buffer`序列化/反序列化源文件路径
- --grpc_python_out `gRPC`相关服务实现的文件路径
- `<proto文件>`

##### 输出的文件

- `expirement_pb2.py`  负责定义消息的序列化/发布序列化、属性的值获取/设置
- `expirement_pb2_grpc.py` 实现RPC服务

此时目录结构为为

```shell
gRPC_demo:
├─go        
│  ├─ proto
│  │     ├─ expirement_grpc.pb.go
│  │     └─ expirement.pb.go 
│  ├─ server 
│  └─ client 
├─proto
│    └─ expirement.proto     proto文件     
└─py 
    ├─ expirement_pb2_grpc.py
    └─ expirement_pb2.py     
```

### 启用Go Module

在`Go 1.13` 后官方推荐使用Go Module 作为包管理。若需要导入本地包，启用Go Module将更加方便。且不再局限于你的项目文件的存放位置，在此之前即`Go PATH`时，Go项目文件必须存放于`$GOPATH/src/`

请位于 `gRPC_demo/go` 目录下执行

```shell
go mod init grpc_demo
```

## 实现

### Go为服务端，Python为客户端

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs
  defaultValue="Go"
  values={[
    {label: 'Go', value: 'Go'},
    {label: 'Python', value: 'Python'},
  ]}>
<TabItem value="Go">

```go title="gRPC_demo/go/server/server.go"
    package main

    import (
        "context"
        "errors"
        pb "grpc_demo/proto" //编译文proto后的源程序
        "log"
        "net"

        "google.golang.org/grpc"
        "google.golang.org/grpc/codes"
    )

    //监听端口
    var port = ":6000"
    // expirements即为我们的数据源
    var expirements []*pb.Expirement
    
    //所有的Expirement名称
    var names = []string{"Cyber", "Shrink", "Doubledip", "Howcome", "Squawk", "Truxx", "Gigi", "Stitch"}
    //所有的Expirement对应的skill
    var skills = []string{"breaking", "short-flying", "filting anying", "making why", "advanced language skill", "seting up heavy machine", "touching ur heart", "making annoyed noise", "anything expect for water"}

    // 实例化RPC服务，先创建一个结构体，然后实现RPC对应的PickOne方法
    type server struct {
        pb.UnimplementedManageServer
    }

    // 实现'PickOne' RPC服务
    // 其输入的参数即为我们创建的 'NameRequest'消息
    // 输出为 'Expirement' 消息
    func (s *server) PickOne(ctx context.Context, in *pb.NameRequest) (*pb.Expirement, error) {
        // 通过遍历数据来实现查找名称相符合的Expirement
        for _, v := range expirements {
            if in.Name == v.Name {
                log.Println("Recived query:", in.GetName())
                return v, nil
            }
        }
        return nil, grpc.Errorf(codes.NotFound, "expirement not found")
    }

    func main() {
        initExpirements()
        // 获取一个本地端口监听器
        listener, err := net.Listen("tcp", "127.0.0.1"+port)
        if err != nil {
            panic(err)
        }
        // 新建RPC服务器
        s := grpc.NewServer()
        // 注册RPC服务
        pb.RegisterManageServer(s, &server{})
        log.Println("gRPC listening at", listener.Addr())
        // 开启服务于监听器上
        if err := s.Serve(listener); err != nil {
            log.Panic(err)
        }
    }

    // 加载所有Expirement数据
    func initExpirements() {
        for i := 0; i < 8; i++ {
            expirements = append(expirements, &pb.Expirement{Name: names[i], Id: int32(i), Skill: skills[i]})
        }
        log.Println("expirements loaded")
    }
```

</TabItem>
<TabItem value="Python">

```py title="gRPC_demo/py/client.py"
import expirement_pb2
import expirement_pb2_grpc
import grpc

# gRPC服务器地址
gRPC_HOST = '127.0.0.1:6000'

def main():
    # 监理费安全的链接
    with grpc.insecure_channel(target=gRPC_HOST) as channel:
        # 创建请求消息
        req = expirement_pb2.NameRequest(name="Stitch")
        # 调用`Manage`RPC方法，生成客户端
        cli = expirement_pb2_grpc.ManageStub(channel)
        # 发送请求
        resp = cli.PickOne(req)
    print(f'Expirement:\n---Name:{resp.name} \n---ID:{resp.id} \n---Skill:{resp.skill}')

if __name__ == '__main__':
    main()
```

</TabItem>

</Tabs>

#### 运行测试

<Tabs
  defaultValue="Go"
  values={[
    {label: 'Go', value: 'Go'},
    {label: 'Python', value: 'Python'},
  ]}>
<TabItem value="Go">

```go
go run gRPC_demo/go/server/server.go

//输出
2021/07/09 00:47:45 expirements loaded
2021/07/09 00:47:45 gRPC listening at 127.0.0.1:6000
```

</TabItem>
<TabItem value="Python">

```py
python gRPC_demo/py/client.py

# 输出
Expirement:
---Name:Stitch
---ID:7
---Skill:making annoyed noise
```


</TabItem>

</Tabs>


### Python为服务端，Go为客户端

<Tabs
  defaultValue="Go"
  values={[
    {label: 'Go', value: 'Go'},
    {label: 'Python', value: 'Python'},
  ]}>
<TabItem value="Go">

```go title="gRPC_demo/go/client/client.go"
    package main

    import (
        "context"
        "fmt"
        pb "grpc_demo/proto"

        "google.golang.org/grpc"
    )

    // RPC服务器地址
    var gRPCHost = "127.0.0.1:6000"

    func main() {
        
        //建立非安全的连接，并且进行堵塞 
        conn, err := grpc.Dial(gRPCHost, grpc.WithInsecure(), grpc.WithBlock())
        if err != nil {
            panic(err)
        }

        // 关闭程序前关闭连接
        defer conn.Close()
        
        // 使用新建的连接创建客户端
        client := pb.NewManageClient(conn)
        // 创建name为"Stitch"的请求消息
        req := &pb.NameRequest{Name: "Stitch"}
        // 调用远程'PickOne'的RPC服务，进行查询
        // 传入我们刚刚的消息其返回一个'Expirement'消息，正如我们在proto文件中定义一样
        expirement, err := client.PickOne(context.Background(), req)
        if err != nil {
            panic(err)
        }
        //输出查询结果
        fmt.Printf("Expirement:\n---Name:%s \n---ID:%d \n---Skill:%s",
            expirement.GetName(), expirement.GetId(), expirement.GetSkill())
    }

```

</TabItem>
<TabItem value="Python">

```py title="gRPC_demo/py/server.py"
import expirement_pb2
import expirement_pb2_grpc
import grpc
from concurrent import futures

expirements = []
gRPC_HOST = "127.0.0.1:6000"

# Expirement的name和skill 映射
names = ["Cyber", "Shrink", "Doubledip", "Howcome", "Squawk", "Truxx", "Gigi", "Stitch"]
skills = [
    "breaking",
    "short-flying",
    "filting anying",
    "making why",
    "advanced language skill",
    "seting up heavy machine",
    "touching ur heart",
    "making annoyed noise",
    "anything expect for water",
]

# 定义一个类，并使其实现PickOne PRC服务
# 该类必须继承 PRC的服务类，ManageServicer
class service(expirement_pb2_grpc.ManageServicer):
    # 实例化'PickOnew' RPC服务
    def PickOne(self, request, context):
        if request.name not in names:
            return "not found"
        idx = names.index(request.name)
        return expirements[idx]

# 初始化实验品
def init_Expirements():
    id = 0
    for name, skill in zip(names, skills):
        expirements.append(expirement_pb2.Expirement(name=name, skill=skill, id=id))
        id += 1
    print("expirements loaded")


def main():
    # 初始化Expirement
    init_Expirements()
    # 获取服务器类，并设定并发数限制
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 将服务添加至服务器中
    expirement_pb2_grpc.add_ManageServicer_to_server(service(), server)
    # 设置非安全的监听地址及端口
    server.add_insecure_port(gRPC_HOST)
    # 启动服务
    server.start()
    print("server started at", gRPC_HOST)
    server.wait_for_termination()


if __name__ == "__main__":
    main()

```

</TabItem>

</Tabs>

#### 运行测试

<Tabs
  defaultValue="Python"
  values={[
    {label: 'Go', value: 'Go'},
    {label: 'Python', value: 'Python'},
  ]}>
<TabItem value="Go">

```go
go run gRPC_demo/go/client/client.go

//输出
Expirement:
---Name:Stitch
---ID:7
---Skill:making annoyed noise
```

</TabItem>
<TabItem value="Python">

```py
python gRPC_demo/py/server.py

# 输出
expirements loaded
server started at 127.0.0.1:6000
```

</TabItem>
</Tabs>

## 结语

至此我们已经能够实现两个语言互通了，分别使用它们来充当服务端/客户端，实现通过一个`name`属性来查询其对应的`skill`和`id`

### 参考链接

> https://www.grpc.io/docs/languages/go/basics

> https://github.com/grpc/grpc-go/tree/master/examples/helloworld

> https://developers.google.cn/protocol-buffers/docs/overview#simple