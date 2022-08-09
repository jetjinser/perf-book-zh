# 编译配置

> by @[sinsong](https://github.com/sinsong)

好的构建配置可以提升 Rust 程序的性能，不需要修改程序代码。

## Release 构建

Rust 性能建议中最重要的一点很简单，但也 [容易被忽视]： 当你需要更好的性能的时候，确保你使用 release 构建，而不是 debug 构建。
通常给 Cargo 指定 `--release` 标志就可以做到。

[容易被忽视]: extern/47764.md

release 构建运行的通常比 debug 构建快 *很多*。
比 debug 构建快 10-100 倍很正常。

默认的是debug构建——
如果你运行 `cargo build`，`cargo run`，`rustc`，并且不带其他选项，就会生成 debug 构建。
debug 构建对调试很有用，但是并不优化。

看看 `cargo build` 运行后输出的最后一行：

```text
Finished dev [unoptimized + debuginfo] target(s) in 29.80s
```

`[unoptimized + debuginfo]` 表示生成的是 debug 构建。
编译后的代码会放在 `target/debug/` 目录中。
`cargo run` 会运行 debug 构建的程序。

release 构建相较于 debug 构建，会有更多优化。
也会忽略一些检查，例如调试断言(debug assertions)，以及整数溢出检查。
可以用 `cargo build --release`，`cargo run --release`，`rustc -O` 生成。
（另外，`rustc` 有许多其他优化选项，例如 `-C opt-level`。）
因为额外的优化，这通常会比 debug 构建花费更长的时间。

看看 `cargo build --release` 运行后输出的最后一行：

```text
Finished release [optimized] target(s) in 1m 01s
```

`[optimized]` 表示生成的是 release 构建。
编译好的代码会放在 `target/release/` 目录中。
`cargo run --release` 会运行 release 构建。

如果想要进一步了解 debug 构建（使用 `dev` profile）和 release 构建（使用 `release` profile）之间的区别，可以参考 [Cargo profile文档]。

[Cargo profile文档]: https://doc.rust-lang.org/cargo/reference/profiles.html

## 链接时优化

链接时优化 (Link-time optimization, LTO) 是一种适用于整个程序的优化技术，
以增加构建时间为代价，可以提高 10%-20% 或更多的运行时性能，
对于单个 Rust 程序，通常用编译时间换取运行性能是值得的。

启用 LTO 最简单的方法是，向 `Cargo.toml` 中添加下列行，然后进行 release 构建。

```toml
[profile.release]
lto = true
```

这样会启用 "重量级"(fat) LTO，会优化依赖图中的所有 crate。

另外，在 `Cargo.toml` 中使用 `lto = "thin"` 则会启用 "轻量级"(thin) LTO——一种不那么激进的 LTO 形式，通常与 重量级 LTO 一样有效，但不会过多增加构建时间。

你可以通过 [Cargo LTO文档] 深入了解 `lto` 设置，以及如何对不同 profile 启用特定设置。

[Cargo LTO文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#lto

## Codegen Units

Rust 编译器将 crate 拆分为多个 [代码生成单元] 来并行化（同时加速）编译。
然而，这会导致它错过一些可能的优化。
如果你想要以更长的编译时间为代价，提升运行时性能，你可以将单元数设置为 1：

```toml
[profile.release]
codegen-units = 1
```

[**示例**](https://likebike.com/posts/How_To_Write_Fast_Rust_Code.html#emit-asm).

[代码生成单元]: https://doc.rust-lang.org/rustc/codegen-options/index.html#codegen-units

请注意，代码生成单元的数量是启发式的，以至于更小的数量可能导致实际产生的程序变慢。

## 使用 CPU 特定的指令

如果你并不在意你的二进制程序代码在更老（或其他类型的）处理器上的兼容性，你可以告诉编译器生成指定的 [特定 CPU 架构] 上的，最新（并且可能最快）的指令。

[特定 CPU 架构]: https://doc.rust-lang.org/1.41.1/rustc/codegen-options/index.html#target-cpu

例如，如果你向 rustc 传递 `-C target-cpu=native`，他会为你当前 CPU 使用最合适的指令：

```bash
RUSTFLAGS="-C target-cpu=native" cargo build --release
```

这可能产生很大的影响，特别是当编译器发现了你代码中进行矢量化的机会。

截止 2022 年 7 月，在 M1 Macs 上使用 `-C target-cpu=native` 会有，没有检测到所有 CPU 特性的 [问题]。
你可以使用 `-C target-cpu=apple-m1` 作为替代。

[问题]: https://github.com/rust-lang/rust/issues/93889

如果你不确定 `-C target-cpu=native` 是否工作最佳，可以比较 `rustc --print cfg` 和 `rustc --print cfg -C target-cpu=native` 的输出，来检查后者是否正确检测 CPU 特性。
如果没有，你可以使用 `-C target-feature` 来指定特定特性。

## `panic!` 时 abort

如果你不需要捕获或展开 panic，你可以告诉编译器在 panic 时简单的 abort。
这可以减少二进制文件体积，略微增加性能：

```toml
[profile.release]
panic = "abort"
```

## Profile-guided Optimization

Profile-guided optimization (PGO)是一种编译模型——
编译你的程序，用采样数据运行并收集性能分析数据，然后基于这些数据引导改程序的下次编译。
[**Example**](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html).

这是一种较高级的技术，需要花一些精力设置，但有时值得这样做。
详见[rustc PGO 文档]。

[rustc PGO 文档]: https://doc.rust-lang.org/rustc/profile-guided-optimization.html
