# 子类化

GObject 在很大程度上依赖于继承。因此，如果我们想创建一个自定义的 GObject，通过子类化来实现是很合理的。 让我们用一个自定义按钮来替换 "Hello World!" 应用程序中的按钮，看看它是如何工作的。 首先，我们需要创建一个实现结构体来保存状态并重写虚方法。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_subclassing/1/custom_button/imp.rs">listings/g_object_subclassing/1/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_subclassing/1/custom_button/imp.rs}}
```
有关子类化的说明请参见 `ObjectSubclass`.
- `NAME` 应由 crate-name 和 object-name 组成，以避免名称冲突。这里应使用[大驼峰命名法](https://en.wikipedia.org/wiki/Camel_case)。
- `Type` 指的是之后将创建的实际 GObject。
- `ParentType` 是我们继承的 GObject。

之后，我们就可以选择重写我们祖先的虚方法。 由于我们现在只想拥有一个普通按钮，所以我们什么也不重写。 不过，我们仍然需要添加空的 `impl`。 接下来，我们将描述我们自定义的 GObject 的公共接口。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_subclassing/1/custom_button/mod.rs">listings/g_object_subclassing/1/custom_button/mod.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_subclassing/1/custom_button/mod.rs:mod}}
```

[`glib::wrapper!`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/macro.wrapper.html)实现了与 `ParentType` 相同的 trait。 理论上，这意味着 `ParentType` 也是我们唯一需要指定的。 不幸的是，还没有人找到好的方法来做到这一点。 这就是为什么到目前为止，在 Rust 中对 GObjects 进行子类化需要提及 `GObject` 和 `GInitiallyUnowned` 以外的所有祖先和接口。 对于 `gtk::Button`，我们可以在 GTK4 的相应 [文档页](https://docs.gtk.org/gtk4/class.Button.html#hierarchy) 中查找其祖先和接口。

完成这些步骤后，我们就可以用自定义按钮( `CustomButton`)替换 `gtk::Button` 了。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_subclassing/1/main.rs">listings/g_object_subclassing/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_subclassing/1/main.rs}}
```

> 用两个结构体来描述对象是 C 语言中定义 GObject 的一种特殊方式。`imp::CustomButton` 处理 GObject 的状态和重写的虚方法。 `CustomButton` 从已实现的 trait 和添加的方法中确定要暴露的方法。

## 添加功能

我们可以用 `CustomButton` 代替 `gtk::Button`。 这很酷，但在实际应用中不太有吸引力。 尽管收益为零，但毕竟提供了不少代码模板。

所以，让我们把它变得更有趣一些吧！`gtk::Button` 并不会保存太多状态，但我们可以让 `CustomButton` 保存一个数字。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_subclassing/2/custom_button/imp.rs">listings/g_object_subclassing/2/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_subclassing/2/custom_button/imp.rs}}
```
我们在 `ObjectImpl` 中重写了 `constructed`，这样按钮的标签就会初始化为`number`.  我们还重写了 `ButtonImpl` 中的 `clicked`，这样每次点击都会使`number`自增并更新标签。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_subclassing/2/main.rs">listings/g_object_subclassing/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_subclassing/2/main.rs:activate}}
```

在 `build_ui` 中，我们不再调用 `connect_clicked`，仅此而已。 重新构建后，应用程序出现了标签为 "0" 的自定义按钮。 每次点击按钮，标签显示的数字都会增加 1。

<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/g_object_subclassing.webm" type="video/webm">
  <p>A video showing that pressing on a button increases the number</p>
 </video>
</div>

那么，我们什么时候需要从 GObject 继承呢？
- 我们想使用某个控件，但要添加状态和重写虚函数。
- 我们想将 Rust 对象传递给函数，但该函数需要一个 GObject。
- 我们想向对象添加属性或信号。
