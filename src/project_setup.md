# 项目设置

让我们从安装所有必要的工具开始。 

首先，按照 [GTK 网站](https://www.gtk.org/docs/installations/)上的说明安装 GTK 4。 然后使用 [rustup](https://rustup.rs/) 安装 Rust。

现在，通过执行以下命令，创建一个新项目并移动到新创建的文件夹中：
```
cargo new my-gtk-app
cd my-gtk-app
```

通过运行以下命令来获取电脑上的 GTK 4 版本

```
pkg-config --modversion gtk4
```

运行以下命令将 [gtk4 crate](https://crates.io/crates/gtk4) 添加到 `Cargo.toml` 依赖项。在撰写本文时，最新GTK 4 版本是 `4.12`

```
cargo add gtk4 --rename gtk --features v4_12
```

通过指定此功能，您可以选择使用 GTK 4 次版本中添加的 API。

现在，您可以通过执行以下命令来运行您的应用程序：

```
cargo run
```
