---
id: state-machine-replication-paper
title: Libra区块链中的状态机复制
---

<!-- hide the table of contents --><style>.toc-headings {display: none !important; visibility: hidden !important;}</style>

## 摘要

本篇介绍了 LibraBFT共识，这是针对Libra 区块链设计的一个健壮而高效的状态机复制系统。

LibraBFT 基于一种叫HotStuff的最新协议,这个协议利用了数十年来拜占庭容错 (BFT) 方面的科学进步，实现了互联网所需的强大的可扩展性和安全性。 LibraBFT进一步完善了HotStuff协议，引入了明确的活跃度机制，并提供了具体的延迟分析。

LBFT和HotStuff的不同之处在于它的一些新颖功能。LBFT包含了一种新颖的回合同步机制，该机制提供了同步下的有限提交延迟。它引入了一个零块投票，其允许尽管有错误的领导者仍然提交提案。它将参与者的正确行为封装在一个可分离的信任区模块中，允许它在一个安全的硬件飞地中运行，从而减少了参与者的攻击面。
+
+LBFT可以通过在序列中嵌入配置更改命令来重新配置自身。一个新的配置时期可能会改变从验证程序集到协议本身的所有内容。


### 论文下载

[![State Machine Replication in the Libra Blockchain PDF Download](assets/illustrations/state-machine-pdf.png){: .download}](assets/papers/libra-consensus-state-machine-replication-in-the-libra-blockchain.pdf)
