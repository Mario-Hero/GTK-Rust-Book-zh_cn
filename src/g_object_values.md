# 泛型值

一些与 GObject 相关的函数依赖于其参数或返回参数的泛型。 由于 GObject 的内省（introspection）是通过 C 语言接口工作的，因此这些函数不能依赖于任何强大的 Rust 概念。 在这种情况下，需要使用[`glib::Value`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/value/struct.Value.html) 或 [`glib::Variant`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/variant/struct.Variant.html).



## Value

我们先从 `Value` 开始。从概念上讲，`Value` 类似 Rust 的 `enum` ，定义如下：

```rust, no_run,noplayground
enum Value <T> {
    bool(bool),
    i8(i8),
    i32(i32),
    u32(u32),
    i64(i64),
    u64(u64),
    f32(f32),
    f64(f64),
    // boxed types
    String(Option<String>),
    Object(Option<dyn IsA<glib::Object>>),
}
```

例如，您可以这样使用 `Value` 代表  `i32` 的值。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_values/1/main.rs">listings/g_object_values/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_values/1/main.rs:i32}}
```

还要注意的是，在上述枚举中，`String` 或 `glib::Object` 等 [boxed](https://gnome.pages.gitlab.gnome.org/libsoup/gobject/gobject-Boxed-Types.html) 类型被封装在一个 `Option` 中。 这源于 C 语言，在 C 语言中，每个 boxed 类型都有可能是 `None`（或 C 语言中的 `NULL`）。 您仍然可以按照与上述 `i32` 相同的方式访问它。如果指定了错误的类型，[`get`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/value/struct.Value.html#method.get)  会返回 `Err`，而且如果 `Value` 代表 `None`，也会返回 `Err`。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_values/1/main.rs">listings/g_object_values/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_values/1/main.rs:string}}
```

如果要区分是指定了错误的类型还是 `Value` 值为 `None` ，只需调用 `get::<Option<T>>` 即可。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_values/1/main.rs">listings/g_object_values/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_values/1/main.rs:string_none}}
```

我们将在后面处理属性和信号时使用 `Value`。



## Variant

当数据需要序列化时，例如将数据发送到另一个进程、发送到网络上，或将数据存储到磁盘时，就会用到`Variant`. 虽然 `GVariant` 支持任意复杂的类型，但 Rust 绑定目前仅限于`bool`, `u8`, `i16`, `u16`, `i32`, `u32`, `i64`, `u64`, `f64`, `&str`/`String` 和 [`VariantDict`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/struct.VariantDict.html). 上述类型的容器也可以使用，如 `HashMap`、`Vec`、`Option`、最多 16 个元素的元组(typle)和 `Variant`。 只要 Rust 结构体的成员可以用`Variant`表示，`Variant`甚至可以从 Rust 结构体[派生](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib_macros/derive.Variant.html#)。

在最简单的情况下，将 Rust 类型转换为 `Variant` 或反向转换，与处理 `Value` 的方式非常相似。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_values/2/main.rs">listings/g_object_values/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_values/2/main.rs:i32}}
```

不过，`Variant` 也可以表示 `HashMap` 或 `Vec` 等容器。 下面的代码段展示了如何在 `Vec` 和 `Variant` 之间进行转换。 更多示例请参见[文档](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib/variant/index.html)。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/g_object_values/2/main.rs">listings/g_object_values/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/g_object_values/2/main.rs:vec}}
```

我们将在使用  [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html)  保存设置或通过 [`gio::Action`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Action.html) 激活操作时使用 `Variant`.
