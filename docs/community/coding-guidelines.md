---
id: coding-guidelines
title: 编码指南
---

原文链接：[https://developers.libra.org/docs/community/coding-guidelines](https://developers.libra.org/docs/community/coding-guidelines)<br/>
译者：Humyna<br/>
译者注：该文档官方会不定期升级，翻译版本基于2019.9.22网站状态版本。<br/>

文档描述了Libra Core Rust代码库的编码指南。

## 代码格式化

所有的代码格式都是使用了项目的特定配置的 [rustfmt](https://github.com/rust-lang/rustfmt)来实现的。下面是一个运行`rustfmt`并遵守项目约定的简单命令：

```
libra$ cargo fmt
```

## 代码分析

[Clippy](https://github.com/rust-lang/rust-clippy) 用于捕捉常见错误，并作为持续集成的一部分运行。在提交您的代码进行审查之前，您可以使用我们的配置运行clippy：

```
libra$ ./scripts/clippy.sh
```

一般来说，我们遵循 [rust-lang-nursery](https://rust-lang-nursery.github.io/api-guidelines/about.html)的提议。本指南的其余部分提供了有关特定主题的详细指南，以实现代码库的一致性。

## 代码文档

任何共有属性fields，函数和方法应该使用[Rustdoc](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#making-useful-documentation-comments)生成文档。

对于模块、结构、枚举和函数请遵循下面的详细约定。当导航Rustdoc，单行_single line_用作预览。示例请参阅Rustdoc [collections](https://doc.rust-lang.org/std/collections/index.html) 中“结构Structs”和“枚举enums”部分。

```
/// [Single line] One line summary description
///
/// [Longer description] Multiple lines, inline code
/// examples, invariants, purpose, usage, etc.
[Attributes] If attributes exist, add after Rustdoc
```

下面示例：

```
/// Represents (x, y) of a 2-dimensional grid
///
/// A line is defined by 2 instances.
/// A plane is defined by 3 instances.
#[repr(C)]
struct Point {
    x: i32,
    y: i32,
}
```

### 常量和字段

描述此数据的目的和定义。

### 函数和方法

为每个函数记录以下内容:


- 方法执行的操作——“该方法将一笔新交易加到mempool。”使用主动语态和现在时。
- 描述如何和为什么使用这个方法
- 调用方法之前必须满足的任何条件
- 在什么状态条件下函数会`panic!()`或返回`Error`
- 返回值的简要说明
- 其他特殊行为说明


### 根目录的README和其他主要组件

Libra Core每个主要组件都需要一个`README.md`文件。主要组件包括：


- 根目录(如`libra/network`, `libra/language`)
- 系统中最主要的crates(如`vm_runtime`)


这个文件应该包含：


- 组件的概念性文档。
- 组件的外部API文档的链接。
- 项目主许可证的链接。
- 项目主贡献指南的链接。


readmes的模板如下：

```
# Component Name

[Summary line: Start with one sentence about this component.]

## Overview

* Describe the purpose of this component and how the code in
this directory works.
* Describe the interaction of the code in this directory with
the other components.
* Describe the security model and assumptions about the crates
in this directory. Examples of how to describe the security
assumptions will be added in the future.

## Implementation Details

* Describe how the component is modeled. For example, why is the
  code organized the way it is?
* Other relevant implementation details.

## API Documentation

For the external API of this crate refer to [Link to rustdoc API].

[For a top-level directory, link to the most important APIs within.]

## Contributing

Refer to the Libra Project contributing guide [LINK].

## License

Refer to the Libra Project License [LINK].
```

`libra/network/README.md是`README.md的一个不错的例子，它描述了网络crate。

## 编码建议

在下面的小节中，我们为一致的代码库提出了一些最佳实践。我们将研究和识别可以使用clippy实施的实践。这些信息将随着时间的推移而发展和改善。

### 属性

确保使用适当的属性来处理死代码：

```
// For code that is intended for production usage in the future
#[allow(dead_code)]
// For code that is only intended for testing and
// has no intended production use
#[cfg(test)]
```

### 避免Deref多态

不要滥用Deref特性来模拟结构之间的继承，从而重用方法。有关详细信息，请阅读[here](https://github.com/rust-unofficial/patterns/blob/master/anti_patterns/deref.md)。

### 注释

为了一致和simpler grepping，我们建议您使用 `//` 和 `///` 注释，而不是块注释 `/* ... */` 。

### 克隆

如果x是引用计数，请选择r [`Arc::clone(x)`](https://doc.rust-lang.org/std/sync/struct.Arc.html) 而不是 `x.clone()`。[`Arc::clone(x)`](https://doc.rust-lang.org/std/sync/struct.Arc.html) 显式地表示我们正在克隆x。这避免了我们是执行结构体、枚举、其他类型的昂贵克隆，还是只执行廉价的引用副本的困惑。

此外，如果要传递 [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html) 类型，请考虑使用新的类型包装器：

```
#[derive(Clone, Debug)]
pub struct Foo(Arc<FooInner>);
```

### 并发类型

并发类型如 [`CHashMap`](https://docs.rs/crate/chashmap), [`AtomicUsize`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicUsize.html) 等为了支持内部转换方法的并发访问，有一个对自身不可改变的引用，如 `fn foo_mut(&self,...)` 。好的实践（像示例中提到的这些）是避免在外部暴露同步原语(如 `Mutex`, `RwLock`)，并清楚地记录方法语义和不变量。

什么时候使用通道(_channels)_与并发类型(_concurrent types)_？

下面列出的是基于经验的高级建议：


- 通道(Channels)用于所有权转移、类型分离和粗粒度消息。它们非常适合于传输数据的所有权、分发工作单元和异步结果通知。此外，它们有助于打破循环依赖（例如， `struct Foo` 包含一个 `Arc<Bar>` ， `struct Bar` 包含一个 `Arc<Foo>` 会导致复杂的初始化）。
- 并发类型(Concurrent types)（例如 [`CHashMap`](https://docs.rs/crate/chashmap) 或在[`Mutex`](https://doc.rust-lang.org/std/sync/struct.Mutex.html), [`RwLock`](https://doc.rust-lang.org/std/sync/struct.RwLock.html)等上构建内部可变的结构体）更适合于缓存和状态。


### 错误处理

错误处理建议遵循 [Rust book guidance](https://doc.rust-lang.org/book/ch09-00-error-handling.html)。Rust将错误分为两大类：可恢复的和不可恢复的错误。应使用 [Result](https://doc.rust-lang.org/std/result/) 处理可恢复的错误。我们对不可恢复错误的建议如下：

_Panic_

- `panic!()` - Runtime panic!仅当结果状态无法继续处理时才应使用。它不能用于任何可恢复的错误。
- `unwrap()` -Unwrap 只能用于互斥锁（例如`lock().unwrap()`）和测试代码。对于所有其他用例，首选`expect()`。唯一的例外是如果错误消息是自定义生成的，在这种情况下使用 `.unwrap_or_else(|| panic!("error: {}", foo))`
- `expect()` - Expect应在系统预留的不变量时调用。`expect()` 优先于 `unwrap()`，并且在大多数情况下都应该包含失败时的详细错误消息。
- `assert!()` - 这个宏保存在debug/release中，必要时应该用来保护系统的不变量。
- `unreachable!()` - 此宏将在不可达（违反不变量）的代码上死机，可在适当的情况下使用。


### 泛型

泛型允许静态调度的动态行为（类似于 [`trait`](https://doc.rust-lang.org/book/ch10-02-traits.html) 方法）。随着泛型类型参数数量的增加，使用类型/方法的难度也随之增加（例如，考虑此类型所需的特征边界的组合、相关类型上的重复特征边界等）。为了避免这种复杂性，我们通常尽量避免使用大量泛型类型参数。我们发现，使用动态调度将具有大量泛型对象的代码转换为trait对象通常会简化我们的代码。

### Getters/setters

除了测试代码，尽可能将字段可见性设置为private。私有字段允许构造函数强制使用内部不变量。为调用者可能需要的数据实现getters，除非需要可变状态否则避免使用setter。

公共字段最适合C风格的 [`struct`](https://doc.rust-lang.org/book/ch05-00-structs.html) 类型：没有内部不变量的复合被动数据结构。命名建议遵循[指南](https://rust-lang-nursery.github.io/api-guidelines/naming.html#getter-names-follow-rust-convention-c-getter)，示例如下。

```
struct Foo {
    size: usize,
    key_to_value: HashMap<u32, u32>
}
impl Foo {
    /// Return a copy when inexpensive
    fn size(&self) -> usize {
        self.size
    }
    /// Borrow for expensive copies
    fn key_to_value(&self) -> &HashMap<u32, u32> {
        &self.key_to_value
    }
    /// Setter follows set_xxx pattern
    fn set_foo(&mut self, size: usize){
        self.size = size;
    }
    /// For a more complex getter, using get_XXX is acceptable
    /// (similar to HashMap) with well-defined and
    /// commented semantics
    fn get_value(&self, key: u32) -> Option<&u32> {
        self.key_to_value.get(&key)
    }
}
```

### 日志

我们目前使用 [slog](https://docs.rs/slog/) 记录日志。


- [error!](https://docs.rs/slog/2.4.1/slog/macro.error.html) - 错误级别的消息在 [slog](https://docs.rs/slog/) 中具有最高的紧急性。出现意外错误（例如，超过了完成 RPC 的最大重试次数或无法将数据存储到本地存储）。
- [warn!](https://docs.rs/slog/2.4.1/slog/macro.warn.html) -警告级别的消息有助于通知管理员自动处理的问题（例如，重试失败的网络连接或多次接收同一消息等）。
- [info!](https://docs.rs/slog/2.4.1/slog/macro.info.html) - 信息级消息非常适合“一次性”事件（例如一次性启动和关闭时的日志状态）或不经常发生的周期性事件（例如每天更改验证程序集）。
- [debug!](https://docs.rs/slog/2.4.1/slog/macro.debug.html) - 调试级消息可以频繁地发生（即，潜在地每秒大于 1个消息），并且通常不期望在生产中启用。
- [trace!](https://docs.rs/slog/2.4.1/slog/macro.trace.html) -跟踪级日志记录通常仅用于函数进入/退出。


### 测试

_单元测试Unit tests_

理想情况下，所有代码都应该进行单元测试。单元测试文件应该与 `mod.rs` 位于同一目录中，并且它们的文件名应该以 `_test.rs` 结尾。要测试的模块应该用 `#[cfg(test)]`注释测试模块。例如，如果crate有一个数据库模块，则预期的目录结构如下：

```
src/db                        -> directory of db module
src/db/mod.rs                 -> code of db module
src/db/read_test.rs           -> db test 1
src/db/write_test.rs          -> db test 2
src/db/access/mod.rs          -> directory of access submodule
src/db/access/access_test.rs  -> test of access submodule
```

_基于属性的测试Property-based tests_

Libra包含使用 [`proptest` ](https://github.com/AltSysrq/proptest)框架用Rust编写的 [property-based tests](https://blog.jessitron.com/2013/04/25/property-based-testing-what-is-it/) 。基于属性的测试生成随机测试用例，并断言被测代码的不变量（也称为_属性**properties**）。

Libra中测试properties的一些示例：


- 使用序列化程序的随机输入测试每个序列化和反序列化对的正确性。任何一对互逆的函数都可以通过这种方式进行测试。
- 通过VM执行公共交易的结果使用随机生成的场景进行测试，并通过oracle进行验证。


`proptest` 教程可以在[`proptest` ](https://altsysrq.github.io/proptest-book/proptest/getting-started.html)书中找到。

参考资料：


- [What is Property Based Testing?](https://hypothesis.works/articles/what-is-property-based-testing/) (包含与fuzzing的比较)
- [An introduction to property-based testing](https://fsharpforfunandprofit.com/posts/property-based-testing/)
- [Choosing properties for property-based testing](https://fsharpforfunandprofit.com/posts/property-based-testing-2/)


_条件编译测试Conditional compilation of tests_

Libra 的 [conditionally compiles](https://doc.rust-lang.org/stable/reference/conditional-compilation.html) 代码只与测试相关，但不包含测试（unitary或其他）。这方面的例子包括proptest策略、特定特性的实现和派生（例如，偶尔的`Clone`）、helper函数等。由于Cargo[目前不适合激活基准中的特性](https://github.com/rust-lang/cargo/issues/2911)，因此我们依赖两个条件来执行此条件编译：


- 测试标志(test flag)，由与条件测试专用代码位于同一crate中的依赖测试代码激活。
- “测试(testing)”自定义功能，由作为条件测试专用代码的其他crate中的依赖测试代码激活（如下所示）。


因此，推荐您按照以下方式设置专用测试代码。例如，我们将认为您在 `foo_crate`中定义一个测试专用的helper函数 `foo` ：


- 1.在 `foo_crate/Cargo.toml`中定义“testing”标志，并将其设为非默认值：


```
[features]
default = []
testing = []
```

- 2.使用测试标志（用于in-crate调用方）和“testing”自定义功能（用于out-of-crate调用方）为您的测试专用helper foo添加注解


```
#[cfg(any(test, feature = "testing"))]
fn foo() { ... }
```

- 3.将激活“testing”功能的开发依赖项添加到只导入这个测试专用成员的crates中：


```
[dev-dependencies.foo_crate]
path = { "<same as the one in [dependencies]>"}
features = ["testing"]
```

- 4.（可选）使用 `cfg_attr` 使测试专用特性派生成为条件：


```
#[cfg_attr(any(test, feature = "testing"), derive(FooTrait))]
#[derive(Debug, Display, ...)] // inconditional derivations
struct Foo { ... }
```

- 5.（可选）为调用仅包含测试成员的crates的crates设置功能传递性。假设是  `bar_crate` 的情况，它通过其测试helpers调用 `foo_crate` 来使用您的测试专用Foo。下面是如何设置 `bar_crate/Cargo.toml`


```
[features]
default = []
testing = ["foo_crate/testing"]
```

_集成测试的最后一个注意事项_：所有在另一个crate中使用条件测试专用元素的测试都需要通过 `Cargo.toml`中的 `[features]` 部分激活“testing”功能。[集成测试](https://doc.rust-lang.org/rust-by-example/testing/integration_testing.html)既不能依赖于 `test` 标志，也不能有适当的 `Cargo.toml` 去激活特性。因此，在Libra 代码库中，我们建议将依赖于测试专用代码的集成测试提取到它们自己的crate中。您可以在 `language/vm/serializer_tests` 查看这种提取集成测试的示例。

开发人员注意：我们使用特性重新导出（在 `Cargo.toml` 的 `[features]`部分中）的原因是配置文件不足以激活`"testing"`特性标志。详见[cargo-issue #291](https://github.com/rust-lang/cargo/issues/2911) ）。

_模糊测试Fuzzing_

Libra包含用于模糊崩溃代码（如反序列化程序）的保护带，可以通过 [`cargo fuzz`](https://rust-fuzz.github.io/book/cargo-fuzz.html)使用 [`libFuzzer`](https://llvm.org/docs/LibFuzzer.html) 引入。有关更多示例，请参见 `testsuite/libra_fuzzer`目录。
