# 编译时间

虽然这本书主要关于提升Rust程序的性能，这一节是关于减少Rust程序的编译时间，
因为这也是大多人关注的重要话题。

## Linking

编译时间，其实有很大一部分是链接时间。尤其是小更改以后重新构建程序时。
我们可以选择比默认的更快的链接器。

这里推荐[lld]，它支持ELF，PE/COFF，Mach-O，wasm等等。

[lld]: https://lld.llvm.org/

通过命令行指定使用 lld，你可以在你的构建命令前加上

```bash
RUSTFLAGS="-C link-arg=-fuse-ld=lld"
```

通过[config.toml]指定使用 lld（应用于一个或者多个项目)，加入这些行：

```toml
[build]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

[config.toml]: https://doc.rust-lang.org/cargo/reference/config.html

lld并未完全支持让Rust使用，但大多情况下可以。有个[Github Issue]追踪lld的完整支持。

另外你也可以选择[mold]，目前只支持ELF。指定使用它和lld一样，把`lld`换成`mold`就可以了。

[mold]: https://github.com/rui314/mold

mold通常比lld更快。它也更新，有时无法工作。

[GitHub Issue]: https://github.com/rust-lang/rust/issues/39915#issuecomment-618726211

## 增量编译

Rust编译器支持[增量编译]，避免在重编译的时候做重复的工作。它可以大大提升编译速度，
但有时会让生成的可执行程序运行的慢一些。因此，它只默认为调试构建启用。
如果你也想为发布构建启用，把这些加到`Cargo.toml`：

```toml
[profile.release]
incremental = true
```

`incremental`设置，以及为不同配置启用特定设置详见[Cargo文档]。

[增量编译]: https://blog.rust-lang.org/2016/09/08/incremental.html
[Cargo文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#incremental

## 可视化

Cargo有个功能，可以可视化你的程序的编译过程。在Rust 1.60以上使用该命令构建：

```text
cargo build --timings
```

或者（1.59以下）：

```text
cargo +nightly build -Ztimings
```

On completion it will print the name of an HTML file. Open that file in a web
browser. It contains a [Gantt chart] that shows the dependencies between the
various crates in your program. This shows how much parallelism there is in
your crate graph, which can indicate if any large crates that serialize
compilation should be broken up. See [the documentation][timings] for more
details on how to read the graphs.

[Gantt chart]: https://en.wikipedia.org/wiki/Gantt_chart
[timings]: https://doc.rust-lang.org/nightly/cargo/reference/timings.html

## LLVM IR

The Rust compiler uses [LLVM] for its back-end. LLVM's execution can be a large
part of compile times, especially when the Rust compiler's front end generates
a lot of [IR] which takes LLVM a long time to optimize.

[LLVM]: https://llvm.org/
[IR]: https://en.wikipedia.org/wiki/Intermediate_representation

These problems can be diagnosed with [`cargo llvm-lines`], which shows which
Rust functions cause the most LLVM IR to be generated. Generic functions are
often the most important ones, because they can be instantiated dozens or even
hundreds of times in large programs.

[`cargo llvm-lines`]: https://github.com/dtolnay/cargo-llvm-lines/

If a generic function causes IR bloat, there are several ways to fix it. The
simplest is to just make the function smaller.
[**Example 1**](https://github.com/rust-lang/rust/pull/72166/commits/5a0ac0552e05c079f252482cfcdaab3c4b39d614),
[**Example 2**](https://github.com/rust-lang/rust/pull/91246/commits/f3bda74d363a060ade5e5caeb654ba59bfed51a4).

Another way is to move the non-generic parts of the function into a separate,
non-generic function, which will only be instantiated once. Whether or not this
is possible will depend on the details of the generic function. The non-generic
function can often be written as an inner function within the generic function,
to minimize its exposure to the rest of the code.
[**Example**](https://github.com/rust-lang/rust/pull/72013/commits/68b75033ad78d88872450a81745cacfc11e58178).

Sometimes common utility functions like [`Option::map`] and [`Result::map_err`]
are instantiated many times. Replacing them with equivalent `match` expressions
can help compile times.

[`Option::map`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[`Result::map_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err

The effects of these sorts of changes on compile times will usually be small,
though occasionally they can be large.
[**Example**](https://github.com/servo/servo/issues/26585).
