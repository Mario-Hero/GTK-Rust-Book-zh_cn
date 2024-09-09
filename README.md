# GTK-Rust-Book-zh_cn

本仓库为 The GTK-Rust Book 的中文翻译，正在施工中。现已翻译到“安装”部分。

*本翻译基于 [gtk4-rs/book at master(github.com)](https://github.com/gtk-rs/gtk4-rs/tree/master/book) 2024年9月9日版本*。

This repo is the Chinese translation of The GTK Rust Book and is currently under construction. It has now been translated to the "Installation" section.

*This translation is based on [gtk4-rs/book at master(github.com)](https://github.com/gtk-rs/gtk4-rs/tree/master/book) version dated September 9th, 2024*

- [Stable](https://gtk-rs.org/gtk4-rs/stable/latest/book)
- [Development](https://gtk-rs.org/gtk4-rs/git/book)

## Build instructions

1. Install mdbook

```bash
cargo install mdbook
```

2. Building

To view the book, execute:

```bash
mdbook serve
```

## Listings

Go to the listings folder

```bash
cd listings
```

To check if the examples build, execute:

```bash
cargo build
```

To run a specific listing, execute:

```bash
cargo run --bin listing_name
```

## License

The book itself is licensed under the [Creative Commons Attribution 3.0 Unported license](https://creativecommons.org/licenses/by/3.0/).
One exception are the code snippets which are licensed under the [MIT license](https://mit-license.org/).
