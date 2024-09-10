# 基于 Rust 和 GTK 4 的 GUI 开发
*作者：Julian Hofer，社区贡献； 翻译：陈竞阁*

*本翻译基于 [gtk4-rs/book at master(github.com)](https://github.com/gtk-rs/gtk4-rs/tree/master/book) 2024年9月9日版本*

GTK 4 是用 C 语言编写的、流行的、跨平台部件工具箱的最新版本。 多亏了GObject-Introspection技术，GTK 的 API 可以轻松地被各种编程语言所使用。 API 甚至可以描述其参数的所有权！

在不牺牲速度的前提下管理所有权是 Rust 的最大优势之一，这使其成为开发GTK 应用程序的极佳选择。 

有了这种组合，您就不必再担心在项目中期遇到瓶颈了。 此外，使用 Rust，您还可以获得如下好处：

 - 线程安全
 - 内存安全
 - 合理的依赖管理
 - 优秀的第三方库

[`gtk-rs`](https://gtk-rs.org/) 项目提供了许多 GTK 相关库的绑定，我们将在本书中使用这些库。




## 这本书是写给谁的

本书假定您了解 Rust 语言。

如果现在还不是这种情况，那么阅读 [The Rust Programming Language](https://doc.rust-lang.org/stable/book/) 是让您了解 Rust的一种愉快方式。 如果你有使用其他低级语言（如 C 或 C++）的经验，则可能会发现阅读 [A half hour to learn Rust](https://fasterthanli.me/articles/a-half-hour-to-learn-rust) 也能为您提供足够的信息。

幸运的是，这一点再加上开发图形应用程序的愿望就是从本书中受益所必需的全部了。



## 如何使用这本书

一般来说，这本书假定你是按顺序从头读到尾。但是，如果您将其用作某个主题的参考， 您可能会发现直接跳入会很有用。

本书分为两种章节：概念章节和项目章节。 

在概念章节中，您将了解 GTK 开发的某个方面。 

在项目章节中，我们将一起构建小程序，应用您目前为止所学的知识。

本书致力于通过实际示例来解释基本的 GTK 概念。 但是，如果一个概念可以用一个不太实际的例子更好地传达，我们大多数时候都会走这条路。 如果您对包含的有用示例感兴趣，我们建议您参阅 `gtk4-rs` 的 [仓库](https://github.com/gtk-rs/gtk4-rs/tree/master/examples)。

书中的每个有效代码片段都是列表的一部分。 与示例一样，这些列表可以在 `gtk4-rs` 的[仓库](https://github.com/gtk-rs/gtk4-rs/tree/master/book/listings)中找到。



## License

The book itself is licensed under the [Creative Commons Attribution 4.0 International license](https://creativecommons.org/licenses/by/4.0/).
The only exception are the code snippets which are licensed under the [MIT license](https://github.com/gtk-rs/gtk4-rs/blob/master/README.md).
This translation of this book is licensed under the [Creative Commons Attribution 4.0 International license](https://creativecommons.org/licenses/by/4.0/).
