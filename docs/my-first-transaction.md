---
id: my-first-transaction
title: Libra上的第一笔交易
---

本文档将指导您在 Libra 区块链上执行第一笔交易。 在执行第一笔交易的步骤之前，我们建议您阅读以下文档，以熟悉 Libra 生态系统和 Libra 协议的关键概念：

* [欢迎来到  Libra ](welcome-to-libra.md)
* [ Libra 协议核心概念](libra-protocol.md)

我们提供了一个命令行界面（CLI）客户端来与区块链进行交互。

## 约定假设条件

本文档中的所有命令均假定已经完成如下准备：

* 您运行的是 Linux（基于Red Hat或Debian）或 macOS系统。
* 可以稳定地连接到互联网。
* 您的系统上安装了 `git` 。
* macOS 上已安装了 Homebrew。
* Linux系统上安装了 `yum` 或者 `apt-get` 。

## 提交交易的步骤

在这个例子中，我们将下载必要的 Libra 组件并在两个用户（Alice和Bob）之间执行一个交易操作。
执行以下步骤将交易提交到 Libra 测试网络 testnet上的验证节点：

1. [克隆并构建 Libra Core](#克隆并编译-libra-core).
2. [编译 Libra CLI客户端并连接到testnet](#编译-libra-cli-客户端并连接到testnet).
3. [创建Alice和Bob的帐户](#创建alice和bob的账户).
4. [铸币并添加到Alice和Bob的账户](#添加libra-币到alice和bob账户).
5. [提交交易](#提交交易).

## 克隆并编译 Libra Core

### 克隆 Libra Core库

```bash
git clone https://github.com/libra/libra.git
```

### 安装 Libra Core

要安装 Libra Core，请切换到 `libra` 目录并运行安装脚本来安装相关的依赖项，如下所示：

```
cd libra
./scripts/dev_setup.sh
```

安装脚本执行以下操作：

* 安装rustup ＆mdash; rustup 是 Rust 编程语言的安装程序，Libra Core是基于Rust实现的。
* 安装相应版本的 Rust 工具链（rust-toolchain）.
* 安装CMake &mdash; 用于管理编译过程。
* 安装protoc &mdash; 协议缓存（protocol buffers）编译器。
* 安装Go &mdash; 用于编译协议缓存。

如果您的安装失败，详见[故障排除说明](#安装)

## 编译 Libra Cli 客户端并连接到testnet

为了连接到 Libra testnet 网络的验证器节点，如下图所示运行客户端：

```bash
./scripts/cli/start_cli_testnet.sh
```

这个命令构建和运行客户端使用cargo（Rust的包管理器），并将客户端连接到testnet上的验证器节点。

客户端连接到testnet上的节点后，您将看到以下输出。 要随时退出客户端，请使用`quit`命令。

```
usage: <command> <args>

Use the following commands:

account | a
  Account operations
query | q
  Query operations
transfer | transferb | t | tb
  <sender_account_address>|<sender_account_ref_id> <receiver_account_address>|<receiver_account_ref_id> <number_of_coins> [gas_unit_price (default=0)] [max_gas_amount (default 10000)] Suffix 'b' is for blocking.
  Transfer coins from account to another.
help | h
  Prints this help
quit | q!
  Exit this client


Please, input commands:

libra%
```

如果您在编译运行客户端和连接到testnet时遇到问题，请参阅[故障排除说明](#客户端编译运行).

<blockquote class="block_note">

**注意**: 如果要在系统上本地运行验证器节点，请按照[运行本地验证器](#运行本地验证器节点)中的说明进行操作。 创建帐户，铸币和执行交易的说明与testnet上的节点相同。

</blockquote>

## 创建Alice和Bob的账户

将客户端连接到testnet后，可以运行CLI命令来创建新帐户。这里为两个用户（Alice和Bob）创建帐户。

### 步骤1：检查CLI客户端是否在系统上运行

出现一个 **libra%** 命令行提示符表示您的 Libra  CLI客户端正在运行。 要查看`account`命令的帮助信息，请输入“account”，如下所示：

```plaintext
libra% account
usage: account <arg>

Use the following args for this command:

create | c
  Create an account. Returns reference ID to use in other operations
list | la
  Print all accounts that were created or loaded
recover | r <file path>
  Recover Libra wallet from the file path
write | w <file name>
  Save Libra wallet mnemonic recovery seed to disk
mint | mintb | m | mb <receiver account> <number of coins>
  Mint coins to the account. Suffix 'b' is for blocking
```

### 步骤2：创建Alice账户

请注意，使用CLI创建帐户不会更新区块链，只会创建本地密钥对。

要创建Alice的帐户，请输入以下命令：

`libra% account create`

成功后输出如下：

```plaintext
>> Creating/retrieving next account from wallet
Created/retrieved account #0 address 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8
```

＃0是Alice帐户的索引，十六进制字符串是Alice帐户的地址。 索引只是引用Alice帐户的一种方式。 帐户索引是本地CLI索引，可以在其他CLI命令中使用，以便用户方便地引用他们创建的帐户。 该指数对区块链毫无意义。 只有当通过铸币（挖矿）将任何一笔钱添加到Alice的账户时，才会在区块链上创建Alice的账户，或者通过来自另一个用户的转账将钱转移到Alice的账户。 请注意，您也可以在CLI命令中使用十六进制地址。 帐户索引只是帐户地址的方便显示。

### 步骤3: 创建Bob的账户

要创建Bob的帐户，请重复之前帐户创建命令：

`libra% account create`

成功后输出如下：

```plaintext
>> Creating/retrieving next account from wallet
Created/retrieved account #1 address 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7
```

#1 是Bob帐户的索引，十六进制字符串是Bob帐户的地址。
有关索引的更多详细信息，请参阅[创建Alice的账户](#步骤2-创建Alice账户)

### 步骤4 (可选): 账户列表

要列出您创建的帐户，请输入以下命令：

`libra% account list`

成功后输出如下：

```plaintext
User account index: 0, address: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, sequence number: 0
User account index: 1, address: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, sequence number: 0
```

帐户中的序列号表示从该帐户发送的交易数。 每次从该帐户发送的交易被执行后并存储在区块链中时，它会递增。 要了解更多信息，请参阅 [序列号](reference/glossary.md#sequence-number).

## 添加Libra 币到Alice和Bob账户

在testnet上创建和铸币是通过Faucet完成的。 Faucet是一种与testnet一起运行的服务。 此服务仅用于为testnet创建硬币，并不用运行在[主网](reference/glossary.md#mainnet)上. 它创建的Libra 币并没有真实的价值，假设[创建Alice和Bob的账户](#创建Alice和Bob的账户), 分别使用了索引0和索引1，您可以按照以下步骤将Libra 币添加到两个帐户。

### 步骤1: 添加110个Libra到Alice账户

铸造Libra币，并添加到Alice的帐户，输入以下命令:

`libra% account mint 0 110`

* 0 是Alice账户的索引号。
* 110 是要添加到Alice账户的Libra币数量。

账户成功运行铸币命令，也会在Libra区块链上创建Alice的账户。

成功后输出如下：

```plaintext
>> Minting coins
Mint request submitted
```

请注意，提交请求时，这意味着它已成功添加到内存池（testnet上的验证程序节点）。 但并不一定意味着它将执行完成。 稍后，我们将查询帐户余额以确认铸币是否成功。

如果您的帐户铸币命令未成功提交您的请求，请参阅[故障排除说明](#铸币添加到账户)

### 步骤2: 添加52个Libra币到Bob账户

铸造Libra币，并添加到Bob的帐户，输入以下命令:

`libra% account mint 1 52`

* 1 是Bob账户的索引号。
* 52 是要添加到Bob账户的Libra数量。
* 账户成功运行铸币命令，也会在Libra区块链上创建Bob的账户，在区块链上创建Bob帐户的另一种方法是将钱从Alice的帐户交易到Bob的帐户。

成功后输出如下：

```plaintext
>> Minting coins
Mint request submitted
```

如果您的帐户铸币命令未成功提交您的请求，请参阅[故障排除说明](#铸币添加到账户)

### 步骤3: 余额检查

检查Alice账户的余额，运行如下命令：

`libra% query balance 0`

成功后输出如下：

`Balance is: 110`

检查Bob账户的余额，运行如下命令：

`libra% query balance 1`

成功后输出如下：

`Balance is: 52`

## 提交交易

在我们将Libra从Alice的账户转移到Bob的账户的交易提交之前，我们将查询每个账户的序列号。 这将有助于我们了解交易执行过程中如何更改每个帐户的序列号。

### 查询账户的序列号

```plaintext
libra% query sequence 0
>> Getting current sequence number
Sequence number is: 0
libra% query sequence 1
>> Getting current sequence number
Sequence number is: 0
```

在 `query sequence 0`, 0是Alice的帐户的索引。 Alice和Bob的帐户的序列号都为0，即表示到目前为止尚未执行Alice或Bob的帐户中的任何交易。

### 转账

提交一个从Alice账户转账10个Libra到Bob账户的交易，输入以下命令：

`libra% transfer 0 1 10`

* 0是Alice的帐户索引。
* 1是Bob的帐户索引。
* 10是从Alice的账户交易到Bob账户的Libra数量。

成功后输出如下：

```plaintext
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 0 <fetch_events=true|false>
```

您可以使用命令 `query txn_acc_seq 0 0 true` (按帐户和序列号进行交易)来检索有关您刚提交的交易的信息。 第一个参数是发件人帐户的本地索引，第二个参数是帐户的序列号。要查看此命令的示例输出，请参阅 [输出示例](#通过账号和序号查询交易).

您刚刚将交易提交到testnet上的验证器节点，它添加进验证器的[内存池](reference/glossary.md#mempool) 。这并不一定意味着您的交易已被执行。 理论上，如果系统运行缓慢或过载，则需要一些时间才能看到结果，您可能需要通过查询帐户多次检查。 要查询索引为0的帐户，可以使用命令  `query account_state 0.` 预期输出显示在[输出示例](#事件查询)

要对传输命令出错进行故障排除，请参阅 [故障排除说明](#转账命令).

**块传输命令**: 你可以使用 `transferb` 命令 (如下所示), 而不是 `transfer` 命令。只有将交易提交到区块链上 `transferb` 命令才会提交交易并给客户端反馈响应显示，一个例子如下所示：

`libra% transferb 0 1 10`

参考[交易的生命周期](life-of-a-transaction.md) 了解交易从提交到执行和存储的整个过程。

### 查询交易后序列号

```plaintext
libra% query sequence 0
>> Getting current sequence number
Sequence number is: 1
libra% query sequence 1
>> Getting current sequence number
Sequence number is: 0
```

Alice的帐号（索引0）的序号为1表示到目前为止已经从Alice的帐户发送了一个交易。 Bob的帐户（索引1）的序列号为0表示到目前为止尚未从Bob的帐户发送任何交易。 每次从帐户发送交易时，序列号都会增加1。

### 检查交易后两个账户的余额

要检查两个帐户中的最终余额，请按照 [此页面步骤](#步骤3-余额检查)操作，再次查询余额. 如果您的交易（转账）成功执行，您应该在Alice的账户中看到100个Libra，在Bob的账户中看到62个Libra。

```plaintext
libra% query balance 0
Balance is: 100
libra% query balance 1
Balance is: 62
```

### 恭喜!

您已成功在Libra testnet上执行了您的交易，并将10个Libra从Alice的账户转移到了Bob的账户！

## 故障排除

### 安装

* 更新 Rust:
    * 在libra目录运行 `rustup update` .
* 从libra目录重新运行安装脚本:
    * `./scripts/dev_setup.sh`

### 客户端编译运行

如果您遇到构建失败，请尝试从libra目录中删除cargo.lock文件：

* `rm Cargo.lock`

如果您的客户端没有连接到testnet：

* 检查网络连接。
* 确保您使用的是最新版本的客户端。 拉取最新的Libra Core并重新运行客户端：
    * `./scripts/cli/start_cli_testnet.sh`


### 铸币添加到账户

* 如果您在testnet上连接的验证程序节点不可用，您将收到“服务器不可用”消息，如下所示：

  ```plaintext
  libra% account mint 0 110
  >> Minting coins
  [ERROR] Error minting coins: Server unavailable, please retry and/or check **if** host passed to the client is running
  ```
* 如果您在提交交易后未更新余额，请稍等片刻再次查询余额。 如果区块链上大量交易在提交，那么可能会有延迟。 如果您的余额仍未更新，请再次尝试铸币。

* 要检查帐户是否存在，请查询帐户状态。 对于索引为0的帐户，请输入以下内容：

  `libra% query account_state 0`

### 转账命令

如果testnet验证器节点不可用（而客户端已确定连接上了）或与 testnet 网络的连接超时，您将看到此错误：

```plaintext
libra% transfer 0 1 10
>> Transferring
[ERROR] Failed to perform transaction: Server unavailable, please retry and/or check if host passed to the client is running
```

解决交易故障:

* 检查testnet的连接。
* 查询发件人帐户以确保其确实存在。 对索引为0的帐户使用以下命令：
    * `query account_state 0`
* 使用`quit` 或 `q!`退出, 然后重新运行以下命令以连接到testnet：
    * 从libre目录运行 `./scripts/cli/start_cli_testnet.sh` 

## 查询命令输出示例

### 通过账号和序号查询交易

此示例将使用帐户和序列号查询单个交易的详细信息。

```plaintext
libra% query txn_acc_seq 0 0 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction {
 { raw_txn: RawTransaction {
    sender: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8,
    sequence_number: 0,
    payload: {,
      transaction: peer_to_peer_transaction,
      args: [
        {ADDRESS: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7},
        {U64: 10000000},
      ]
    },
    max_gas_amount: 10000,
    gas_unit_price: 0,
    expiration_time: 1560466424s,
},
 public_key: 55af3fe3f28550a2f1e5ebf073ef193feda44344d94c463b48be202aa0b3255d,
 signature: Signature( R: CompressedEdwardsY: [210, 23, 214, 62, 228, 179, 64, 147, 81, 159, 180, 138, 100, 211, 111, 139, 178, 148, 81, 1, 240, 135, 148, 145, 104, 234, 227, 239, 198, 153, 13, 199], s: Scalar{
  bytes: [203, 76, 105, 49, 64, 130, 162, 81, 22, 237, 159, 26, 80, 181, 111, 94, 84, 6, 152, 126, 181, 192, 62, 103, 130, 94, 246, 174, 139, 214, 3, 15],
} ),
 }
 }
Events:
ContractEvent { access_path: AccessPath { address: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 0, event_data: AccountEvent { account: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, amount: 10000000 } }
ContractEvent { access_path: AccessPath { address: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/received_events_count/" } , index: 0, event_data: AccountEvent { account: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, amount: 10000000 } }
```

请注意，交易金额以 microlibra 单位显示.

### 事件查询

在以下示例中，我们将从索引为0的帐户查询“已发送”事件。我们从此帐户发送了一个交易，所以只有一个交易。 当前状态的证明同样会返回。以便确保没有丢失任何交易的延展可以进行 - 这在查询不返回“有限”事件时完成。

```plaintext
libra% query event 0 sent 0 true 10
>> Getting events by account and event type.
EventWithProof {
  transaction_version: 3,
  event_index: 0,
  event: ContractEvent { access_path: AccessPath { address: e7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 0, event_data: AccountEvent { account: 46efbad798a739c088e0e98dd9d592c27c7eb45ba1f8ccbdfc00bd4d7f2947f3, amount: 10000000 } },
  proof: EventProof { ledger_info_to_transaction_info_proof: AccumulatorProof { siblings: [HashValue(62570ae9a994bcb20c03c055667a4966fa50d0f17867dd5819465072fd2c58ba), HashValue(cce2cf325714511e7d04fa5b48babacd5af943198e6c1ac3bdd39c53c87cb84c)] }, transaction_info: TransactionInfo { signed_transaction_hash: HashValue(69bed01473e0a64140d96e46f594bc4b463e88e244b694e962b7e19fde17f30d), state_root_hash: HashValue(5809605d5eed94c73e57f615190c165b11c5e26873012285cc6b131e0817c430), event_root_hash: HashValue(645df3dee8f53a0d018449392b8e9da814d258da7346cf64cd96824f914e68f9), gas_used: 0 }, transaction_info_to_event_proof: AccumulatorProof { siblings: [HashValue(5d0e2ebf0952f0989cb5b38b2a9b52a09e8d804e893cb99bf9fa2c74ab304bb1)] } }
}
Last event state: Some(
    AccountStateWithProof {
        version: 3,
        blob: Some(
            AccountStateBlob {
             Raw: 0x010000002100000001217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc974400000020000000e7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b00e1f50500000000000000000000000001000000000000000100000000000000
             Decoded: Ok(
                AccountResource {
                    balance: 100000000,
                    sequence_number: 1,
                    authentication_key: 0xe7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b,
                    sent_events_count: 1,
                    received_events_count: 0,
                },
            )
             },
        ),
        proof: AccountStateProof {
            ledger_info_to_transaction_info_proof: AccumulatorProof {
                siblings: [
                    HashValue(62570ae9a994bcb20c03c055667a4966fa50d0f17867dd5819465072fd2c58ba),
                    HashValue(cce2cf325714511e7d04fa5b48babacd5af943198e6c1ac3bdd39c53c87cb84c),
                ],
            },
            transaction_info: TransactionInfo {
                signed_transaction_hash: HashValue(69bed01473e0a64140d96e46f594bc4b463e88e244b694e962b7e19fde17f30d),
                state_root_hash: HashValue(5809605d5eed94c73e57f615190c165b11c5e26873012285cc6b131e0817c430),
                event_root_hash: HashValue(645df3dee8f53a0d018449392b8e9da814d258da7346cf64cd96824f914e68f9),
                gas_used: 0,
            },
            transaction_info_to_account_proof: SparseMerkleProof {
                leaf: Some(
                    (
                        HashValue(c0fbd63b0ae4abfe57c8f24f912f164ba0537741e948a65f00d3fae0f9373981),
                        HashValue(fc45057fd64606c7ca40256b48fbe486660930bfef1a9e941cafcae380c25871),
                    ),
                ),
                siblings: [
                    HashValue(4136803b3ba779bb2c1daae7360f3f839e6fef16ae742590a6698b350a5fc376),
                    HashValue(5350415253455f4d45524b4c455f504c414345484f4c4445525f484153480000),
                    HashValue(a9a6bda22dd6ee78ddd3a42da152b9bd39797b7da738e9d6023f407741810378),
                ],
            },
        },
    },
)
```

### 查询账户状态

在此示例中，我们将查询单个帐户的状态。

```plaintext
libra% query account_state 0
>> Getting latest account state
Latest account state is:
 Account: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8
 State: Some(
    AccountStateBlob {
     Raw: 0x010000002100000001217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc9744000000200000003ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a800e1f50500000000000000000000000001000000000000000100000000000000
     Decoded: Ok(
        AccountResource {
            balance: 100000000,
            sequence_number: 1,
            authentication_key: 0x3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8,
            sent_events_count: 1,
            received_events_count: 0,
        },
    )
     },
)
 Blockchain Version: 3
```

## 运行本地验证器节点

要在本地计算机上启动验证程序节点，并创建本地区块链网络（未连接到Libra testnet），请确保您已经按照[安装Libra Core](#安装-libra-core), 切换到Libra Core库的根目录，然后运行 `libra_swarm` 如下显示:

```bash
$ cd ~/libra
$ cargo run -p libra_swarm -- -s
```

`-p libra_swarm` - 使用cargo运行libra_swarm包，该包启动由一个节点组成的本地区块链。

`-s` 启动本地客户端以连接到本地区块链。

要查看启动节点和连接Libra区块链的其他选项，请运行：

`$ cargo run -p libra_swarm -- -h`

Cargo命令可能需要一些时间才能执行完成。 如果此命令的执行完成且没有错误，则系统上将运行Libra CLI客户端实例和Libra验证器节点。 应该能看到包含 CLI 客你户端菜单和`libra%`  提示符的输出.

## 交易生命周期

执行完第一笔交易后，您可以参考该文件 [交易生命周期](life-of-a-transaction.md):

* 从“底层”说明交易从提交到执行的整个生命周期
* 在Libra生态系统中提交和执行交易时，了解Libra验证器的每个逻辑组件之间的交互。

## 参考

* [来到 Libra 世界](welcome-to-libra.md).
* [Libra 协议核心概念](libra-protocol.md) &mdash; 介绍Libra 协议的核心概念
* [Libra上的第一笔交易](my-first-transaction.md) &mdash; 指导使用Libra CLI客户端在Libra区块链上执行第一笔交易。
* [了解 Move 语言](move-overview.md) &mdash; 介绍新区块链编程语言Move。
* [Libra Core 概要](libra-core-overview.md) &mdash; Libra Core 组件的概念和实现细节。
* [CLI 命令指南](reference/libra-cli.md) &mdash; 列出Libra CLI客户端的命令及其用法。
* [Libra 术语表](reference/glossary.md) &mdash; 提供Libra术语的快速参考。
