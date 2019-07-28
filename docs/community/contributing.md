---
id: contributing
title: 贡献指南
---

我们的目标是让Libra项目变得简单和透明。

<blockquote class="block_note">
Libra Core项目目前还是一个正在快速发展的早期原型。 在对项目做出任何实质性贡献之前，请务必在论坛中进行讨论，以确保其符合项目路线图。
</blockquote>

## 对Libra Core的贡献

要对Libra Core有所贡献，请确保您拥有最新版本的代码库。使用所有必要的依赖项来设置Libra Core，包括linting、测试和构建文档，可以运行以下命令:

```bash
$ git clone https://github.com/libra/libra.git
$ cd libra
$ cargo install
$ cargo test
```

## 编码指南

有关如何为Libra Core代码库做出贡献的详细指导，请参阅 [编码指南](coding-guidelines.md).

## 文档

所有开发人员文档都发布在Libra开发者网站上。 开发人员站点是开源的，构建网站的源代码在[这个库](https://github.com/libra/website/). 文档网站使用 [Docusaurus](https://docusaurus.io/) 构建.

如果您熟悉Markdown，您可以开始贡献!

## Pull Requests

在开发的初始阶段，我们计划只审计和审查pull request。随着代码的稳定，我们将能够接受来自社区的pull request。

提交你的pull request:

1. Fork `libra` ，然后从 `master` 创建你的分支。
2. 如果您添加了代码，它需要经过测试，请添加单元测试。
3. 如果您对api做了更改，请更新相关文档，并构建和测试开发者网站。
4. 验证并确保测试套件通过。
5. 确保代码同时通过linters。
6. 如果您还没有完成贡献者许可协议(CLA)，请完成它。
7. 提交您的pull request。

## 贡献者许可协议

如果你的pull request被Libra的任何项目接受，你需要签一个CLA。在Libra的任何开源项目中，你只需要签署一次。代表自己可以签署 [个人CLA](https://github.com/libra/libra/blob/master/contributing/individual-cla.pdf). 如果你是代表你的雇主贡献，请让他们签署[公司 CLA](https://github.com/libra/libra/blob/master/contributing/corporate-cla.pdf).

## 代码准则
请参阅 [代码准则](../policies/code-of-conduct.md), 它描述社区内交互的准则。

## Issues

Libra 使用 [GitHub issues](https://github.com/libra/libra/issues) 来跟踪bugs，请提供必要的信息和说明以重现您的问题。安全相关的bug应使用我们的[security procedures](../policies/security.md) 来报告。
