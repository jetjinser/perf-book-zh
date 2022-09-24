# 性能分析

> by @[jinser](https://github.com/jetjinser) and @[poly000](https://github.com/poly000)

在优化程序时，你需要判断你的程序哪些部分是“热点”（执行得足够频繁，足以影响运行时间），值得修改。
通过性能分析，你可以很方便地判断。

## 性能分析器（Profilers）

有许多不同的性能分析器，每一个都有其优势和劣势。
下面是一个不完整的清单，其中包括已经成功用于 Rust 程序的性能分析器。

- [perf] 是一个通用的性能分析器，它使用 hardware performance counters ([HPC](https://en.wikipedia.org/wiki/Hardware_performance_counter))。
  [Hotspot] 和 [Firefox Profiler] 是查看 perf 记录的数据的好工具。它在 Linux 上工作。
- [Instruments] 是一个通用的性能分析器，在 macOS 的 Xcode 中自带。
- [AMD μProf] 是一个通用的性能分析器，在 Windows 和 Linux 上工作。
- [flamegraph] 是一个 Cargo 命令，它使用 perf/DTrace 对你的代码进行分析，然后以火焰图的形式显示结果。
  它适用于 Linux 和所有支持 DTrace 的平台（macOS、FreeBSD、NetBSD，可能还有 Windows）。
- [Cachegrind] 和 [Callgrind] 提供了全局的、每个函数的和每行源码的指令计数（instruction counts）
  以及模拟的缓存（simulated cache）和分支预测数据（branch prediction data）。
  它们可以在 Linux 和其他一些 Unix 系统上运行。
- [DHAT] 有利于找到代码中哪些部分导致了大量的分配，并能深入了解内存使用的峰值。它还可以用来识别对 memcpy 的热调用（hot calls）。
  [dhat-rs] 是一个实验性的替代品，功能稍差，需要对你的 Rust 程序做一些小的改动，但在所有平台上都能使用。
- [heaptrack] 和 [bytehound] 是堆分析工具。它们在 Linux 上工作
- [`counts`] 支持 ad-hoc 分析，它结合使用 `eprintln!` 语句和基于频率的后处理，这对获得你的代码部分的领域特定（domain-specific）的亮点很有好处。它适用于所有平台。
- [Coz] 执行 *causal profiling* 以衡量优化潜力，并通过 [coz-rs] 支持 Rust。它在 Linux 上工作。

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

## 调试信息（Debug Info）

为了有效地分析 release 的构建产物，你可能需要启用源码行的调试信息。
要做到这一点，请在你的 Cargo.toml 文件中添加以下几行。

```toml
[profile.release]
debug = 1
```
`debug` 设置详见[Cargo 文档]。

[Cargo 文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

然而，即使做了上述步骤，你也无法得到标准库的详细性能分析信息。
这是因为 Rust 标准库发布版本不携带调试信息。你可以用[这里的方法]自己编译一份标准库与编译器，并将如下行添加到 `config.toml`：

 ```toml
[rust]
debuginfo-level = 1
```

这有些麻烦，但有时值得这么做。

[这里的方法]: https://github.com/rust-lang/rust

## 符号Demangling

Rust 使用一种[mangle机制](https://rust-lang.github.io/rfcs/2603-rust-symbol-name-mangling-v0.html)，对生成的代码中的符号名进行编码。
如果性能分析器不支持这种机制，它的输出可能包含以 `_ZN` 或 `_R` 开头的符号名称，
例如 `_ZN3foo3barE` 或 `_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` 或 `_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`

这种符号名可以用[`rustfilt`]手动demangle。

[`rustfilt`]: https://crates.io/crates/rustfilt
