# Linting

[clippy] 是一个lint集合，可以帮你发现Rust代码中的常见错误。
clippy很不错，甚至可能可以帮你提升性能——有些涉及到代码模式的lint，可能会导致次优性能。

## 基本使用

[clippy]: https://github.com/rust-lang/rust-clippy

安装后，运行很简单：

```text
cargo clippy
```

性能lint的完整列表可以在[这里]查阅，把Perf以外的组全部取消勾选即可。

[这里]: https://rust-lang.github.io/rust-clippy/master/

让代码运行更快的同时，性能lint建议通常也会让你的代码更简洁，更地道。
因此它们值得遵循，哪怕并不是频繁执行的代码。

## Disallowing Types

在接下来的章节我们会了解到，有时我们应该使用比标准库类型更快的替代。
使用它们时，你很可能在哪里不小心使用了标准库类型。

你可以使用 `clippy` 的 [`disallowed_types`] lint（自Rust 1.55加入），避免这个问题。
比如，你不想使用标准库的散列表（原因在[Hashing]小节），可以将以下内容保存到你的项目中的 `clippy.toml` 文件：

```toml
disallowed-types = ["std::collections::HashMap", "std::collections::HashSet"]
```

[Hashing]: hashing.md
[`disallowed_types`]: https://rust-lang.github.io/rust-clippy/master/index.html#disallowed_types

然后将以下声明加入到你的Rust代码中：

```rust
#![warn(clippy::disallowed_type)]
```

你需要手动加上这一行，因为 `disallow_type` 目前是一个 "nursery"（正在开发）的lint。
