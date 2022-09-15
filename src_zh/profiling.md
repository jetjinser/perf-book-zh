# 性能分析

在优化程序时，你需要判断你的程序哪些部分是“热点”（执行的足够频繁，足以影响运行时间），值得修改。
通过性能分析，你可以很方便地判断。

## Profilers

有很多不同的性能分析工具，它们都有自己的长处与短处。

下面是可以对Rust程序使用的性能分析工具的不完整列表。
- [perf] 是一个通用性能分析工具，使用[HPC](https://en.wikipedia.org/wiki/Hardware_performance_counter)。
  [Hotspot] 和 [Firefox Profiler] 都可以用来查看perf记录的数据。可用于Linux。
- [Instruments] 是Mac的Xcode自带的通用性能分析工具。
- [AMD μProf] 是一个通用性能分析工具。可用于Linux/Windows。
- [flamegraph] 是一个cargo命令，使用perf/DTrace对你的代码进行性能分析，
  并将结果显示为火焰图。可用于Linux和任何支持DTrace的平台（macOS, FreeBSD, NetBSD, Windows也可能支持）
- [Cachegrind] & [Callgrind] 分别提供了全局，函数甚至源码行粒度的指令count，
  并模拟了缓存和分支预测数据。
  它们可用于Linux和其他一些Unix系统。
- [DHAT] is good for finding which parts of the code are causing a lot of
  allocations, and for giving insight into peak memory usage. It can also be
  used to identify hot calls to `memcpy`. It works on Linux and some other
  Unixes. [dhat-rs] is an experimental alternative that is a little less
  powerful and requires minor changes to your Rust program, but works on all
  platforms.
- [heaptrack] and [bytehound] are heap profiling tools. They work on Linux.
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

## Debug Info

To profile a release build effectively you might need to enable source line
debug info. To do this, add the following lines to your `Cargo.toml` file:
```toml
[profile.release]
debug = 1
```
See the [Cargo documentation] for more details about the `debug` setting.

[Cargo documentation]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

Unfortunately, even after doing the above step you won't get detailed profiling
information for standard library code. This is because shipped versions of the
Rust standard library are not built with debug info. To remedy this, you can
build your own version of the compiler and standard library, following [these
instructions], and adding the following lines to the `config.toml` file:
 ```toml
[rust]
debuginfo-level = 1
```
This is a hassle, but may be worth the effort in some cases.

[these instructions]: https://github.com/rust-lang/rust

## Symbol Demangling

Rust uses a mangling scheme to encode function names in compiled code. If a
profiler is unaware of this scheme, its output may contain symbol names
beginning with `_ZN` or `_R`, such as `_ZN3foo3barE` or
`_ZN28_$u7b$$u7b$closure$u7d$$u7d$E` or
`_RMCsno73SFvQKx_1cINtB0_3StrKRe616263_E`

Names like these can be manually demangled using [`rustfilt`].

[`rustfilt`]: https://crates.io/crates/rustfilt
