---
id: network
title: 网络 Network
---

网络组件向验证器的其他组件提供点对点网络服务。

## 概要

网络组件是专门为促进共识和共享内存池协议而设计的。目前，它为这些组件提供了两个主要接口:
* RPC, 用于远程过程调用;
* DirectSend, 用于向单个接收方发送“即发即忘”样式的消息。

网络组件使用：
* [Multiaddr](https://multiformats.io/multiaddr/) ：用于对等寻址的方案。
* TCP ：用于可靠传输。
* [Noise](https://noiseprotocol.org/noise.html) ：用于身份验证和点对点加密。
* [Yamux](https://github.com/hashicorp/yamux/blob/master/spec.md) ：在单个链接上多路复用子流（substreams）。
* 推送类型的 [gossip](https://en.wikipedia.org/wiki/Gossip_protocol) ：用于对节点发现。

每个新的子流（substream）都被分配了一个由发送方和接收方都支持的*协议*。每个RPC和DirectSend类型都对应一个这样的协议。

只有符合条件的成员才允许加入内部验证器网络。它们的标识和公钥信息由共识组件在初始化和更新系统成员时提供。
一个新的验证器还需要一些*种子*节点的网络地址，以帮助它启动连接到网络的链接。种子对等节点首先进行验证器的身份验证，然后与之共享网络状态。

网络中的每个成员都维护一个合法全员视图，并直接连接到需要与之通信的任何验证器。不能直接连接的验证器被假定为属于系统所能容忍的拜占庭式故障的范围。

定期使用活动探测器来确定的验证者的状态，信息不在验证器之间共享; 相反，每个验证器直接监视其对等节点的活动状态。

在完成部分资格成员视图，复杂的故障检测器或网络覆盖之前，此方法应扩展到几百个验证器。

## 实施细节

### 系统架构

                                 +---------------------+---------------------+
                                 |          共识        |       Mempool       |
                                 +---------------------+---------------------+
                                 |            Validator Network              |
                                 +---------------------+---------------------+
                                 |            NetworkProvider                |
    +------------------------------------------------+-----------------+     |
    | Discovery, health, etc     |            RPC    |  DirectSend     |     |
    +--------------+---------------------------------------------------------+
    |                                         Peer Manager                   |
    +------------------------------------------------------------------+-----+

网络组件用 [Actor](https://en.wikipedia.org/wiki/Actor_model) 编程模型中实现 &mdash; 即，它在不同子组件（组件作为独立“任务”运行）之间使用消息传递进行通信。[tokio](https://tokio.rs/) 框架用作任务运行时（task runtime）。网络组件中的不同子组件为:

* **NetworkProvider** &mdash; 向客户端公开网络API。 它将来自上游客户端的请求转发到适当的下游组件，并将传入的RPC和DirectSend请求发送到适当的上游处理程序。
* **Peer Manager** &mdash; 侦听入链接及主动连接网络上的其他对等节点。 它还通知其他组件有关新/丢失的连接事件，并将传入的子流解复用到适当的协议处理程序。
* **连接管理器（Connectivity Manager）** &mdash; 当且仅当它是符合条件的网络成员时，才能确保我们保持与节点的连接。 Connectivity Manager从Discovery组件接收对等方的地址，并向 Peer Manager 发出拨号/断开连接请求。
* **Discovery** &mdash; 使用推送式来发现新的节点并更新现有节点的地址。 在每个*tick*上，它打开一个随机选择的节点的新子流，并将其网络视图发送给该节点。 从入链发现消息中检测到的对网络的任何更改都会通知连接管理器。
* **Health Checker** &mdash; 执行定期活动探测以确保对等/连接的健康状况。如果一系列可配置的探测相继失败，它将重置与对等节点的连接。用户探测当前在可配置的静态超时上失败。
* **Direct Send** &mdash; 允许向/从远程对等节点发送/接收消息。它将入链消息通知上游处理程序。
* **RPC** &mdash; 允许向/从其他对等节点发送/接收RPC。 它通知上游处理程序有关入链RPC的信息。 上游处理程序通过一个通道可以向调用者发送序列化响应。
除了上述子组件之外，网络组件还包括用于执行加密，传输复用，协议协商等的实用程序。

## 模块组织结构？

```
network
├── benches                       # network benchmarks
├── memsocket                     # In-memory transport for tests
├── netcore
│   └── src
│       ├── multiplexing          # substream multiplexing over a transport
│       ├── negotiate             # protocol negotiation
│       └── transport             # composable transport API
├── noise                         # noise framework for authentication and encryption
└── src
    ├── channel                    # mpsc channel wrapped in IntGauge
    ├── connectivity_manager       # component to ensure connectivity to peers
    ├── interface                  # generic network API
    ├── peer_manager               # component to dial/listen for connections
    ├── proto                      # protobuf definitions for network messages
    ├── protocols                  # message protocols
    │   ├── direct_send            # protocol for fire-and-forget style message delivery
    │   ├── discovery              # protocol for peer discovery and gossip
    │   ├── health_checker         # protocol for health probing
    │   └── rpc                    # protocol for remote procedure calls
    ├── sink                       # utilities over message sinks
    └── validator_network          # network API for consensus and mempool
```
