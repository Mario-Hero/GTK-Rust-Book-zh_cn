# 内存管理

在编写 gtk-rs 程序时，内存管理可能有点棘手。 让我们来看看为什么会出现这种情况以及如何处理。 

在第一个示例中，我们的窗口只有一个按钮。 每点击一次按钮，一个整数变量  `number` 就会自增一次。

```rust ,no_run,compile_fail
#use gtk::prelude::*;
#use gtk::{self, glib, Application, ApplicationWindow, Button};
#
#const APP_ID: &str = "org.gtk_rs.GObjectMemoryManagement0";
#
// DOES NOT COMPILE!
fn main() -> glib::ExitCode {
    // Create a new application
    let app = Application::builder().application_id(APP_ID).build();

    // Connect to "activate" signal of `app`
    app.connect_activate(build_ui);

    // Run the application
    app.run()
}

fn build_ui(application: &Application) {
    // Create two buttons
    let button_increase = Button::builder()
        .label("Increase")
        .margin_top(12)
        .margin_bottom(12)
        .margin_start(12)
        .margin_end(12)
        .build();

    // A mutable integer
    let mut number = 0;

    // Connect callbacks
    // When a button is clicked, `number` should be changed
    button_increase.connect_clicked(|_| number += 1);

    // Create a window
    let window = ApplicationWindow::builder()
        .application(application)
        .title("My GTK App")
        .child(&button_increase)
        .build();

    // Present the window
    window.present();
}
```

Rust 编译器拒绝编译这个程序，同时还吐出了多条错误信息。 让我们逐一查看。

```console

error[E0373]: closure may outlive the current function, but it borrows `number`, which is owned by the current function
   |
32 |     button_increase.connect_clicked(|_| number += 1);
   |                                     ^^^ ------ `number` is borrowed here
   |                                     |
   |                                     may outlive borrowed value `number`
   |
note: function requires argument type to outlive `'static`
   |
32 |     button_increase.connect_clicked(|_| number += 1);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
help: to force the closure to take ownership of `number` (and any other referenced variables), use the `move` keyword
   |
32 |     button_increase.connect_clicked(move |_| number += 1);
   |  
```

我们的闭包只借用了 `number`. GTK 中的信号处理器要求其引用具有静态( `` `static ``)生命周期，因此我们不能借用一个只在 `build_ui` 函数作用域中存在的变量。 编译器也给出了解决方法。 在闭包前面添加 `move` 关键字，`number `就会被移入闭包。

```rust ,no_run,compile_fail
#use gtk::prelude::*;
#use gtk::{self, glib, Application, ApplicationWindow, Button};
#
#const APP_ID: &str = "org.gtk_rs.GObjectMemoryManagement0";
#
#fn main() -> glib::ExitCode {
#    // Create a new application
#    let app = Application::builder().application_id(APP_ID).build();
#
#    // Connect to "activate" signal of `app`
#    app.connect_activate(build_ui);
#
#    // Run the application
#    app.run()
#}
#
#fn build_ui(application: &Application) {
#    // Create two buttons
#    let button_increase = Button::builder()
#        .label("Increase")
#        .margin_top(12)
#        .margin_bottom(12)
#        .margin_start(12)
#        .margin_end(12)
#        .build();
#
    // DOES NOT COMPILE!
    // A mutable integer
    let mut number = 0;

    // Connect callbacks
    // When a button is clicked, `number` should be changed
    button_increase.connect_clicked(move |_| number += 1);
#
#    // Create a window
#    let window = ApplicationWindow::builder()
#        .application(application)
#        .title("My GTK App")
#        .child(&button_increase)
#        .build();
#
#    // Present the window
#    window.present();
#}
```

但这样仍然会出现以下错误信息：

```console

error[E0594]: cannot assign to `number`, as it is a captured variable in a `Fn` closure
   |
32 |     button_increase.connect_clicked(move |_| number += 1);
   |                                              ^^^^^^^^^^^ cannot assign
```

为了理解该错误信息，我们必须了解 `FnOnce`、`FnMut` 和 `Fn` 这三种闭包(closure) trait 之间的区别。 使用实现 `FnOnce` 特性的闭包的 API 给 API 消费者提供了最大的自由度。 闭包只被调用一次，因此它甚至可以使用自己的状态。 信号处理器可以被多次调用，所以它们不能接受 `FnOnce`。 

限制性更强的 `FnMut` trait 不允许闭包消耗其状态，但它们仍可以对其进行修改。 信号处理器也不允许这样做，因为它们可以从自身内部调用。 这将导致多个可变引用，而借用检查器根本不喜欢这种情况。 

这样就只剩下 `Fn`. 状态可以不可变地借用，但我们如何修改 `number` 呢？ 我们需要一种具有内部可变性的数据类型，比如 [`std::cell::Cell`](https://doc.rust-lang.org/std/cell/struct.Cell.html).

> `Cell` 只适用于实现了 [`Copy`](https://doc.rust-lang.org/core/marker/trait.Copy.html) trait 的对象。对于其他对象，则应使用[`RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html). 
> 有关内部可变性的更多信息，请参见《Rust Atomics and Locks》一书中的[这一部分](https://marabos.nl/atomics/basics.html#interior-mutability)。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/2/main.rs">listings/g_object_memory_management/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/1/main.rs:build_ui}}
```

现在编译结果符合预期。 

让我们举一个稍微复杂一点的例子：两个按钮都修改了同一个数字`number`。 为此，我们需要一种方法，让两个闭包都拥有同一个值的所有权？

这正是 [`std::rc::Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html) 类型的作用。 

`Rc` 会计算通过 [`Clone::clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html#tymethod.clone) 创建和通过 [`Drop::drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop) 释放的强引用的数量，只有当这个数量降为零时，才会释放这个值。 

如果我们想修改 [`Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html)的内容，可以再次使用 [`Cell`](https://doc.rust-lang.org/std/cell/struct.Cell.html) 类型。


文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/2/main.rs">listings/g_object_memory_management/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/2/main.rs:callback}}
```

不过，用临时变量（如 number_copy）来填充作用域并不是一件好事。 

我们可以使用 [`glib::clone!`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/macro.clone.html) 宏来改善这种情况。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/3/main.rs">listings/g_object_memory_management/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/3/main.rs:callback}}
```

就像 `Rc<Cell<T>>`一样，GObject 也是引用计数和可变的。 因此，我们可以像传递`number`一样将按钮传递给闭包。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/4/main.rs">listings/g_object_memory_management/4/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/4/main.rs:callback}}
```

如果我们现在点击其中一个按钮，另一个按钮的标签就会改变。 

但是，哎呀！ 我们是不是忘了引用计数系统的一个恼人之处？是的：[循环引用](https://doc.rust-lang.org/book/ch15-06-reference-cycles.html).。`button_increase` 持有 `button_decrease` 的强引用，反之亦然。 强引用可以防止引用的值被释放。 如果这个链条导致一个循环，那么这个循环中的所有值都不会被释放。 使用弱引用可以打破这种循环，因为弱引用不会使其值存活，而是提供了一种方法，可以在该值仍然存活时检索强引用。 由于我们希望应用程序释放不需要的内存，因此我们应该在按钮上使用弱引用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/5/main.rs">listings/g_object_memory_management/5/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/5/main.rs:callback}}
```

现在循环引用被破坏了。 

每次点击按钮时，`glib::clone` 都会尝试升级弱引用。 例如，如果我们现在点击了一个按钮，而另一个按钮已不存在，回调将被跳过。 

默认情况下，它会立即从闭包返回，返回值为 `()` 。 如果闭包期望不同的返回值，可以指定 `@default-return`。 

请注意，我们在第二个闭包中移动了 `number`。 如果我们在两个闭包中都移动了弱引用，那么没有任何东西能让 `number` 继续存活，闭包也不会被调用。 考虑到这一点，`button_increase` 和 `button_decrease` 也会在 `build_ui` 的作用域结束时被丢弃。 那么谁来保持按钮的存活呢？

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/5/main.rs">listings/g_object_memory_management/5/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/5/main.rs:box_append}}
```

当我们将按钮附加到 `gtk_box` 时，`gtk_box` 会保留对按钮的强引用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_memory_management/5/main.rs">listings/g_object_memory_management/5/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_memory_management/5/main.rs:window_child}}
```

当我们将 `gtk_box` 设置为 `window` 的子窗口时，`window` 会保持对它的强引用。 

直到我们关闭窗口，它都会保持 `gtk_box` 以及按钮的存活。 

由于我们的应用程序只有一个窗口，关闭窗口也就意味着退出应用程序。 

只要尽可能使用弱引用，您就会发现完全可以避免应用程序中的循环引用。 在没有循环引用的情况下，您可以依靠 GTK 来正确管理您传递给它的 GObject 的内存。
