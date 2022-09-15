# 基准测试

基准测试通常用于比较功能相同的程序的性能。有时要比较不同的程序，比如，火狐、Chrome、Safari。
有时则要比较一个程序的不同版本。这可以验证“这个修改会让它更快吗？”。

基准测试是一个复杂的话题，彻底的覆盖超出了本书的范围，但这里有一些基本知识。

首先，你需要根据工作负载来衡量。理想情况下，你会有各种工作负载来表示你的程序的实际使用情况。
使用真实世界的输入的工作负载是最好的，但微基准测试（[microbenchmarks]）和压力测试（[stress tests]）也可以适度地发挥作用。

[microbenchmarks]: https://stackoverflow.com/questions/2842695/what-is-microbenchmarking
[stress tests]: https://en.wikipedia.org/wiki/Stress_testing_(software)

其次，你需要一种方法来运行工作负载，这也将决定使用的指标。
Rust的内置基准测试（[benchmark tests]）是一个简单的起点，
但它们使用的是不稳定的特性（unstable features），而且只适用于 Nightly Rust。
[`bencher`] crate 与其类似，但它适用于稳定的 Rust。[Criterion] 是一个更复杂的选择。
自定义基准线束也是可能的。例如，[rustc-perf] 是用于对Rust编译器进行基准测试的线束。

[benchmark tests]: https://doc.rust-lang.org/1.16.0/book/benchmark-tests.html
[`bencher`]: https://crates.io/crates/bencher
[Criterion]: https://github.com/bheisler/criterion.rs
[rustc-perf]: https://github.com/rust-lang/rustc-perf/

当涉及到指标时，有许多选择，正确的选择将取决于被测程序的性质。
例如，对批处理程序有意义的指标可能对交互式程序没有意义。
挂钟时间（Wall-time） 在许多情况下是一个明显的选择，因为它与用户的感觉相吻合。
然而，它可能会受到高突变（high variance）的影响。特别是，内存布局的微小变化会导致显著但短暂的性能波动。
因此，其他突变程度较低的指标（如周期或指令计数）可能是一个合理的选择。

总结多个工作负载的测量结果也是一个挑战，有多种方法可以做到这一点，没有一种方法是明显最好的。

做好基准测试是很难的。说到这里，不要过分强调有一个完美的基准测试设置，特别是当你开始优化一个程序时。
一个平庸的设置要比没有设置好得多。对你正在测量的东西保持开放的心态，随着时间的推移，你可以在了解你的程序的性能特征时对基准测试进行改进。
