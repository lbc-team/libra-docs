---
id: storage
title: 存储模块 Storage
---

存储模块为Libra区块链上的所有数据集以及Libra Core内部使用的必要数据提供可靠和高效的持久存储。

## 概要

存储模块旨在实现下面两个主要目的：

1. 保留区块链数据，特别是验证方通过共识协议达成一致的交易及其输出。
2. 为任何查询提供一个带有Merkle证明的响应，该查询要求部分区块链数据。如果客户端获得了正确的root hash，则可以轻松地验证响应的完整性。

Libra 区块链可以被视为Merkle树，包含以下组件：

![data](https://developers.libra.org/docs/assets/data.png)

### 历史账本

账本（Ledger）历史由Merkle累加器表示。 每次将交易 `T` 添加到区块链时, 一个包含交易 `T` 的 *TransactionInfo* 结构，执行 `T` 后状态Merkle树的root hash和生成交易Merkle树的root hash 通过 `T` 附加到累加器。

### 账本状态

每个版本的账本状态由稀疏Merkle树表示，该树具有所有帐户的状态。 密钥是地址的256位散列值，它们的对应值是序列化为二进制blob的整个帐户的状态。 虽然大小为 `2^256` 的树是一个难以处理的表示方式，但完全由空节点组成的子树将替换为占位符值，而由一个叶子组成的子树将替换为单个节点。

然每个 *TransactionInfo* 结构指向不同的状态树，但新树可以重用前一树的未更改部分，从而形成持久数据结构。

### 事件

每个交易发出一个事件列表，这些事件构成一个Merkle累加器。与状态Merkle树类似，交易的事件累加器的root hash记录在相应的 *TransactionInfo* 模块。


### 账本信息和签名

*LedgerInfo* 结构在某个版本和其他元数据处具有账本历史累计器的根散列，它是对该版本之前的账本历史的绑定承诺。验证器每次就一组交易及其执行结果达成一致时，都要签署相应的 *LedgerInfo* 结构。对于存储的每个 *LedgerInfo* 结构，还存储来自验证器的一组关于该结构的签名，以便客户端在获得每个验证器的公钥后可以验证该结构。

## 实现细节

存储模块使用 [RocksDB](https://rocksdb.org/) 作为其物理存储引擎。 由于存储模块需要存储多种类型的数据，而RocksDB中的键值对是字节数组，因此RocksDB上有一个包装器来处理键和值的序列化。 此包装器强制执行数据库内外的所有数据都是根据预定义的模式构建的。

实现主要功能的核心模块称为 *LibraDB*. 虽然我们使用单个RocksDB实例来存储整个数据集，但相关数据被分组到逻辑存储中 &mdash; 例如，账本存储，状态存储和交易存储等。

对于表示账本状态的稀疏Merkle树，我们通过使用具有16个子节点的分支节点来优化磁盘布局，这些子节点表示4级子树，而扩展节点表示没有分支的路径。 但是，在计算root hash和证明时，我们仍然会模拟二叉树。 此修改结果证明，比以太坊的Merkle Patricia树生成的证明更短。

## 模块组织结构？
```
    storage
          └── accumulator      # Implementation of Merkle accumulator.
          └── libradb          # Implementation of LibraDB.
          └── schemadb         # Schematized wrapper on top of RocksDB.
          └── scratchpad       # In-memory representation of Libra core data structures used by execution.
          └── sparse_merkle    # Implementation of sparse Merkle tree.
          └── state_view       # An abstraction layer representing a snapshot of state where the Move VM reads data.
          └── storage_client   # A Rust wrapper on top of GRPC clients.
          └── storage_proto    # All interfaces provided by the storage module.
          └── storage_service  # Storage module as a GRPC service.
```
