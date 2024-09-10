# 信号

GObject 信号是一个为特定事件注册回调的系统。 例如，如果我们按下一个按钮，"clicked（点击）"信号就会发出。 然后，该信号将负责执行所有已注册的回调函数。

`gtk-rs` 提供了注册回调的便捷方法。 在我们的 "Hello World" 示例中，我们将 "clicked" 信号[连接](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/prelude/trait.ButtonExt.html#tymethod.connect_clicked)到一个闭包，一旦它被调用，就会将按钮的标签设置为 "Hello World"。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/hello_world/3/main.rs">listings/hello_world/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/hello_world/3/main.rs:callback}}
```

如果我们愿意，可以使用通用的 [`connect_closure`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/object/trait.ObjectExt.html#tymethod.connect_closure) 方法和 [`glib::closure_local!`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/macro.closure_local.html) 宏来连接它。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_signals/1/main.rs">listings/g_object_signals/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_signals/1/main.rs:callback}}
```

`connect_closure` 的优势在于它还能与自定义信号一起使用。

> 如果您需要在闭包中克隆引用计数对象，则不必将其封装在另一个`clone!` 宏中。
> `closure_local!` 接受与创建强/弱引用相同的语法，并具有[监视](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/macro.closure.html#object-watching)功能，一旦监视对象被删除，闭包就会自动断开连接。 

## 向自定义 GObject 添加信号

让我们看看如何创建自己的信号。 现在，让我们可以来扩展 `CustomButton`。 首先，我们覆盖 `ObjectImpl` 中的 `signals` 方法。 [`std::sync::OnceLock`](https://doc.rust-lang.org/std/sync/struct.OnceLock.html) 可以确保 `SIGNALS` 只被初始化一次。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_signals/2/custom_button/imp.rs">listings/g_object_signals/2/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_signals/2/custom_button/imp.rs:object_impl}}
```

`signals` 方法负责定义一组信号。 在本例中，我们只创建了一个名为 "max-number-reached" 的信号。 在命名信号时，我们确保使用[短横线命名法(kebab-case)](https://en.wikipedia.org/wiki/Letter_case#Kebab_case)进行命名。 当信号发出时，它会发送一个 `i32` 值。

我们希望在`number` 达到 `MAX_NUMBER` 时发出信号。 在发送信号的同时，我们还将发送 `number` 当前的值。 之后，我们将`number` 重置为 0.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_signals/2/custom_button/imp.rs">listings/g_object_signals/2/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_signals/2/custom_button/imp.rs:button_impl}}
```

如果我们现在按下按钮，其标签上的数字就会增加，直到达到 `MAX_NUMBER`. 然后它会发出 "max-number-reached" 信号，我们可以很好地连接到该信号。 现在，每当我们收到 "max-number-reached" 信号时，相应的数字就会打印到[标准输出](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout))中。

Filename: <a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_signals/2/main.rs">listings/g_object_signals/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_signals/2/main.rs:signal_handling}}
```

现在，您已经知道如何连接各种信号以及如何创建自己的信号。 如果您想通知 GObject 的消费者发生了某一事件，自定义信号尤其有用。
