# 控件

控件(Widget)是组成 GTK 应用程序的组件。 GTK 提供了许多控件，如果这些控件不合适，您甚至可以自定义。 例如，控件包含了显示控件、按钮、容器和窗口。 一种控件可能能够包含其他控件，它可能会用于展示信息，并且可能会对交互做出反应。

[Widget Gallery](https://docs.gtk.org/gtk4/visual_index.html) 有助于找出适合你的需求的控件。 假设我们想向应用程序添加一个按钮。 我们这里有很多选择，但让我们以最简单的一个——`Button`。

<div style="text-align:center"><img src="img/widgets_button.png" /></div>

GTK 是一个面向对象的框架，因此所有控件都是继承树的一部分，顶部是 `GObject`. 继承树如下所示：

```console
GObject
╰── Widget
    ╰── Button
```

[GTK 文档](https://docs.gtk.org/gtk4/class.Button.html#implements)还告诉我们，`Button`实现了 `GtkAccessible`, `GtkActionable`, `GtkBuildable`, `GtkConstraintTarget` 接口。

现在让我们将其与 `gtk-rs` 中的 `Button` 结构进行比较。 [gtk-rs 文档](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Button.html#implements)告诉我们它实现了哪些 trait。 我们发现这些 trait 在 GTK 文档中要么有相应的基类，要么有接口。 在 “Hello World” 程序中，我们希望对按钮单击做出反应。 这种行为是按钮特有的，因此我们希望在 `ButtonExt` trait 中找到合适的方法。 而事实上，`ButtonExt` 包含了 [`connect_clicked`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/prelude/trait.ButtonExt.html#tymethod.connect_clicked) 方法。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/hello_world/3/main.rs">listings/hello_world/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/hello_world/3/main.rs:button}}
```
