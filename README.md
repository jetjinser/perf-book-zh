# perf-book-zh

《The Rust Performance Book》的中文翻译

这里是重新开始的翻译，原翻译已经被删除。

## 主要贡献者

[莉特雅姐姐](https://github.com/sinsong)，[恒星姐姐](https://github.com/star-hengxing)，[jinser](https://github.com/jetjinser)

还有[一只中文水平不好的猫猫](https://github.com/poly000)

## 阅读

渲染好的书会放在[这里](https://poly000.github.io/perf-book-zh/)。

## 参与翻译

本书倾向于使用英语原词，除非有社区认可并且很难产生歧义的中文对应词汇。

由于缺少翻译人员，翻译进度可能会十分缓慢。

## Building

The book is built with [`mdbook`](https://github.com/rust-lang/mdBook), which
can be installed with this command:
```
cargo install mdbook
```
To build the book, run this command:
```
mdbook build
```
The generated files are put in the `book/` directory.

## Development

To view the built book, run this command:
```
mdbook serve
```
This will launch a local web server to serve the book. View the built book by
navigating to `localhost:3000` in a web browser. While the web server is
running, the rendered book will automatically update if the book's files
change.

To test the code within the book, run this command:
```
mdbook test
```

## License

Licensed under either of
* Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
