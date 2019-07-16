---
id: vm
title: 虚拟机
---

MoveVM 执行以 Move字节码中表示的交易。有两个主要的包:核心VM（core VM）和VM运行环境（VM runtime）。VM核心包含VM的底层数据类型—主要是文件格式和上层的抽象表示。还定义了gas测算逻辑抽象。

## 概要

MoveVM是一个带有静态类型系统的堆栈计算机。MoveVM通过混合文件格式、验证 (参考 [bytcode verifier README](https://github.com/libra/libra/blob/master/language/bytecode_verifier/README.md))MoveVM是一个带有静态类型系统的堆栈计算机。MoveVM通过混合文件格式、验证(供参考[bytcode verifier README])和运行时约束来认可的Move语言的规范。文件格式的结构允许定义模块、类型(资源和无限制类型)和函数。代码通过字节码指令来表示，这些指令可能引用外部函数和类型。文件格式还强制使用语言的某些不变量，如隐藏类型和私有字段。从文件格式定义可以清楚地看出，模块为函数和类型定义了范围/名称空间。类型是不透明的，因为所有字段都是私有的，而且类型不带任何函数或方法。

## 实现细节

MoveVM核心包提供文件格式的定义以及与文件格式相关的所有实用程序：
* 一个简单的Rust抽象文件格式 (`libra/language/vm/src/file_format.rs`) 和字节码。这些Rust结构广泛用于代码库中。
* 文件格式的序列化和反序列化。这些定义了代码的链上二进制表示。
* 一些漂亮的输出展示功能。
* 文件格式的基本架构。
* gas 成本/综合基础设施。

`CompiledModule` 和 `CompiledScript` 在 `libra/language/vm/src/file_format.rs` 中定义，分别是Move *Module* 和 *Transaction Script* 的顶级结构, 这些结构提供了文件格式的简单抽象。此外，还定义了一组 [*Views*](https://github.com/libra/libra/blob/master/language/vm/src/views.rs) 来方便地导航和检查 `CompiledModule` 和 `CompiledScript`。

## 文件夹结构

```
.
├── cost_synthesis  # Infrastructure for gas cost synthesis
├── src             # VM core files
├── tests           # Proptests
├── vm_genesis      # Helpers to generate a genesis block, the initial state of the blockchain
└── vm_runtime      # Interpreter and runtime data types (see README in that folder)
```
