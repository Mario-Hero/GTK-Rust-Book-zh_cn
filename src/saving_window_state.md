# 保存窗口状态

很多时候，我们希望窗口状态在会话之间持续存在。 如果用户调整窗口大小或将其最大化，他们可能会希望下次打开应用程序时，窗口仍保持同样的状态。 GTK 并没有提供这种功能，但幸运的是，手动实现它并不难。 我们基本上需要两个整数(高度 `height` 和 宽度 `width`) 和一个布尔值（`is_maximized`）来持久化。 我们已经知道如何通过使用  [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html) 来做到这一点。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/saving_window_state/1/org.gtk_rs.SavingWindowState1.gschema.xml">listings/saving_window_state/1/org.gtk_rs.SavingWindowState1.gschema.xml</a>

```xml
{{#rustdoc_include ../listings/saving_window_state/1/org.gtk_rs.SavingWindowState1.gschema.xml}}
```

由于我们不关心中间状态，因此只在构建窗口时加载窗口状态，并在关闭窗口时保存窗口状态。 这可以通过创建自定义窗口来实现。 首先，我们创建一个窗口，并添加用于访问设置和窗口状态的便捷方法。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/saving_window_state/1/custom_window/mod.rs">listings/saving_window_state/1/custom_window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/saving_window_state/1/custom_window/mod.rs:mod}}
```

> 我们通过将 "application" 属性传递给 [`glib::Object::new`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/object/struct.Object.html#method.new) 来设置该属性。 您甚至可以用这种方法设置多个属性。 在创建新的 GObject 时，这比手动调用 setter 方法要好得多。

实现的结构包含 `settings`. 您可以看到，我们将 `settings` 嵌入到  [`std::cell::OnceCell`](https://doc.rust-lang.org/std/cell/struct.OnceCell.html) 中。 当你知道只需初始化一次值时，这是 `RefCell<Option<T>>` 的一个很好的替代方法。 我们还重写了 `constructed` 和 `close_request` 方法，通过它们来加载或保存窗口状态。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/saving_window_state/1/custom_window/imp.rs">listings/saving_window_state/1/custom_window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/saving_window_state/1/custom_window/imp.rs:imp}}
```

就是这样！ 现在，我们的窗口可以在应用程序会话之间保持状态。
