---
id: contributing
title: 贡献指南
---

我们的目标是让Libra项目变得简单和透明。

<blockquote class="block_note">
Libra Core项目目前是一个早期的原型，正在快速发展。 在对项目做出任何实质性贡献之前，请务必在论坛中进行讨论，以确保其符合项目路线图。
</blockquote>

## 对Libra Core的贡献

要对Libra Core有所贡献，请确保您拥有最新版本的代码库。要使用所有必要的依赖项来设置Libra Core，包括包、测试和构建文档，可以运行以下命令:
```
$ git clone https://github.com/libra/libra.git
$ cd libra
$ cargo install
$ cargo test
```

## 编码指南

有关如何为Libra Core代码库做出贡献的详细指导，请参阅 [编码指南](编码指南.md).

## 文档

所有开发人员文档都发布在Libra开发人员站点上。 开发人员站点是开源的，构建站点的代码位于此 [repository](https://github.com/libra/website/). 开发人员使用 [Docusaurus](https://docusaurus.io/) 构建.

如果您熟悉Markdown，您可以开始贡献!

## Pull请求

在开发的初始阶段，我们计划只审计和审查pull请求。随着代码基的稳定，我们将能够接受来自社区的pull请求。

提交你的pull请求:

1. Fork `libra` 从 `master` 创建你的分支。
2. 如果您已经添加了应该测试的代码，请添加单元测试。
3. 如果您对api做了更改，请更新相关文档，并构建和测试开发人员站点。
4. 验证并确保测试套件通过。
5. 确保代码同时通过上述两项。
6. 如果您还没有完成贡献者许可协议(CLA)，请完成它。
7. 提交您的pull请求。

## 贡献者许可协议

如果你的pull请求被Libra的任何项目接受，你需要签一个CLA。在Libra的任何开源项目中，你只需要这样做一次。代表自己个人可以签署 [Individual CLA](https://github.com/libra/libra/blob/master/contributing/individual-cla.pdf). 如果你是代表你的雇主贡献，请让他们签署 [Corporate CLA](https://github.com/libra/libra/blob/master/contributing/corporate-cla.pdf).

## 行为守则
请参阅 [行为守则](../policies/行为守则.md), 它描述了社区内交互的准则。

## 问题

Libra 使用 [GitHub issues](https://github.com/libra/libra/issues) 请提供必要的信息和说明以重现您的问题。 应使用我们的 [security procedures](../policies/security.md) 报告与安全相关的bug。.
