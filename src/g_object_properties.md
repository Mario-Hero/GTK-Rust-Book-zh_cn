# 属性

属性(Properties)为访问 GObject 的状态提供了一个公共 API.

让我们通过 `Switch` 控件来看看如何实现这一点。 它的一个属性叫做  [active](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Switch.html#active). 根据 GTK 文档，它可以被读取和写入。 因此，`gtk-rs` 提供了相应的  [`is_active`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Switch.html#method.is_active) 和 [`set_active`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Switch.html#method.set_active) 方法。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/1/main.rs">listings/g_object_properties/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/1/main.rs:switch}}
```

属性不仅可以通过 getter 和 setter 访问，还可以相互绑定。 让我们看看两个 `Switch` 实例是如何绑定的。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/2/main.rs">listings/g_object_properties/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/2/main.rs:switches}}
```

在本例中，我们希望将 `switch_1` 的 "active" 属性与 `switch_2` 的 "active" 属性绑定。 我们还希望绑定是双向的，因此通过调用 [`bidirectional`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/object/struct.BindingBuilder.html#method.bidirectional) 方法来指定。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/2/main.rs">listings/g_object_properties/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/2/main.rs:bind_active}}
```

现在，当我们点击两个开关中的一个时，另一个也会被切换。

<div style="text-align:center">
 <video autoplay muted loop>
    <source src="vid/g_object_properties_switches.webm">
    <p>A video which shows that toggling one button also toggles the other one </p>
 </video>
</div>

## 给自定义 GObject 添加属性

我们还可以为自定义 GObject 添加属性。 我们可以通过将 `CustomButton` 的 `number` 绑定到一个属性来演示这一点。 大部分工作由 [`glib::Properties`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/derive.Properties.html) 派生宏(derive macro)完成。 我们告诉它封装类型是 `super::CustomButton`。 我们还给 `number` 添加了注释，这样宏就知道它应该创建一个可读可写的属性 "number"。 宏还会生成本章稍后将使用的[封装方法](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/derive.Properties.html#generated-wrapper-methods)。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/3/custom_button/imp.rs">listings/g_object_properties/3/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/3/custom_button/imp.rs:custom_button}}
```

[`glib::derived_properties`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/attr.derived_properties.html) 宏为每个使用 `Property` 宏生成属性的 GObject 生成了模板。 在 `constructed` 中，我们通过绑定 "label "属性来使用新属性 "number"。`bind_property` 会自行将 "number "的整数值转换为 "label "的字符串。 现在，我们不必再在 "clicked" 回调函数中调整标签了。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/3/custom_button/imp.rs">listings/g_object_properties/3/custom_button/imp.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/3/custom_button/imp.rs:object_impl}}
```

我们还必须调整 `clicked` 方法。 以前我们直接修改 `number`，现在我们可以使用生成的封装方法 `number` 和 `set_number`。 这样就会发出 "notify "信号，这对于绑定正常工作是必要的。

```rust
{{#rustdoc_include ../listings/g_object_properties/3/custom_button/imp.rs:button_impl}}
```

让我们通过创建两个自定义按钮来看看能做些什么。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/3/main.rs">listings/g_object_properties/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/3/main.rs:buttons}}
```

我们已经看到，绑定的属性不一定必须是同一类型。 利用 [`transform_to`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/object/struct.BindingBuilder.html#method.transform_to) 和 [`transform_from`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/object/struct.BindingBuilder.html#method.transform_from), 我们可以确保 `button_2` 显示的数字总是比 `button_1` 显示的数字大 1.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/3/main.rs">listings/g_object_properties/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/3/main.rs:bind_numbers}}
```
现在，如果我们点击其中一个按钮，另一个按钮的 "number" 和 "label" 属性也会随之改变。

<div style="text-align:center">
 <video autoplay muted loop>
    <source src="vid/g_object_properties_buttons.webm">
    <p>A video which shows that pressing on one button also changes the number on the other one</p>
 </video>
</div>

属性的另一个优点是，当属性发生变化时，可以将回调连接到事件。 例如：

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_properties/3/main.rs">listings/g_object_properties/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_properties/3/main.rs:connect_notify}}
```

现在，每当 "number" 属性发生变化时，闭包就会被执行，并将 "number" 的当前值打印到标准输出中。 

将属性引入您的自定义 GObject 将会非常有用，如果您想：

- 绑定（不同）GObject 的状态
- 在属性值发生变化时通知消费者

请注意，每次值发生变化时发送信号都会产生（计算）代价。 如果您只想暴露内部状态，添加 getter 和 setter 方法是更好的选择。
