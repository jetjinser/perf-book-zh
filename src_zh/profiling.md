# 性能分析

在优化程序时，你需要判断你的程序哪些部分是“热点”（执行的足够频繁，足以影响运行时间），值得修改。
通过性能分析，你可以很方便地判断。

## Profilers

有很多不同的性能分析工具，它们都有自己的长处与短处。

下面是可以对Rust程序使用的性能分析工具的不完整列表。
- [perf] 通用性能分析工具，使用[HPC](https://en.wikipedia.org/wiki/Hardware_performance_counter)。
  [Hotspot] 和 [Firefox Profiler] 都可以用来查看perf记录的数据。支持Linux。
- [Instruments] Mac的Xcode自带的通用性能分析工具。
- [AMD μProf] 通用性能分析工具。支持Linux和Windows。
- [flamegraph] cargo子命令，使用perf/DTrace对你的代码进行性能分析，
  并将结果显示为火焰图。可用于Linux和任何支持DTrace的平台（macOS, FreeBSD, NetBSD, Windows也可能支持）
- [Cachegrind] & [Callgrind] 分别提供了全局，函数甚至源码行粒度的指令count，
  并模拟了缓存和分支预测数据。
  它们支持Linux和其他一些Unix系统。
- [DHAT] 可以帮你分析代码的哪一部分导致了大量内存分配，并且可以查看内存使用峰值。
  你也可以用它验证 `memcpy` 是否被频繁调用。它支持Linux和一些其他Unix系统。
  [dhat-rs]是一个实验性替代，功能稍微没那么强大，而且需要对你的Rust程序进行一些小修改。
  不过，它支持任何平台。
  platforms.
- [heaptrack] 和 [bytehound] 是堆性能分析工具，支持Linux。
- [`counts`] supports ad hoc profiling, which combines the use of `eprintln!`
  statement with frequency-based post-processing, which is good for getting
  domain-specific insights into parts of your code. It works on all platforms.
- [Coz] performs *causal profiling* to measure optimization potential, and has
  Rust support via [coz-rs]. It works on Linux. 

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page
[Hotspot]: https://github.com/KDAB/hotspot
[Instruments]: https://developer.apple.com/forums/tags/instruments
[Firefox Profiler]: https://profiler.firefox.com/
[AMD μProf]: https://developer.amd.com/amd-uprof/
[flamegraph]: https://github.com/flamegraph-rs/flamegraph
[Cachegrind]: https://www.valgrind.org/docs/manual/cg-manual.html
[Callgrind]: https://www.valgrind.org/docs/manual/cl-manual.html
[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[dhat-rs]: https://github.com/nnethercote/dhat-rs/
[heaptrack]: https://github.com/KDE/heaptrack
[bytehound]: https://github.com/koute/bytehound
[`counts`]: https://github.com/nnethercote/counts/
[Coz]: https://github.com/plasma-umass/coz
[coz-rs]: https://github.com/plasma-umass/coz/tree/master/rust

## 调试信息

为了对发布构建进行性能分析，你可能需要启用源码行调试信息。
在 `Cargo.toml` 加入：

```toml
[profile.release]
debug = 1
```

`debug` 设置详见[Cargo 文档]。

[Cargo 文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

然而，只做以上步骤，你无法得到标准库的详细性能分析信息。
这是因为 Rust 标准库发布不携带调试信息。你可以用[这里的方法]自己编译一份标准库与编译器，并将如下行添加到 `config.toml`：

 ```toml
[rust]
debuginfo-level = 1
```
这有些麻烦，但有时值得这么做。

[这里的方法]: https://github.com/rust-lang/rust

## 符号Demangling

Rust在编码函数名到编译的代码时使用一种mangling机制。如果性能分析工具不支持这种机制，它的输出可能含有这种以 `_ZN` 或者 `_R`开始的符号名：
`_ZN28_$u7b$$u7b$closure$u7d$$u7d$E`，或者 `_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`。

这种符号名可以用[`rustfilt`]手动demangle。

[`rustfilt`]: https://crates.io/crates/rustfilt
