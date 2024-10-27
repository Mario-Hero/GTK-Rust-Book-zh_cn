# 设置(Settings)

我们现在已经学会了多种处理状态的方法。 但是，每次我们关闭应用程序时，它都消失了。 让我们学习如何通过在 [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html) 中存储 [`Switch`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Switch.html) 的状态来使用它。

一开始，我们必须创建一个`GSchema` xml 文件，以描述我们的应用程序计划在设置中存储的数据类型。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/1/org.gtk_rs.Settings1.gschema.xml">listings/settings/1/org.gtk_rs.Settings1.gschema.xml</a>

```xml
{{#rustdoc_include ../listings/settings/1/org.gtk_rs.Settings1.gschema.xml}}
```
让我们一步一步来。 这个`id`与我们在创建应用程序时使用的应用程序 ID 相同。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/1/main.rs">listings/settings/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/settings/1/main.rs:application}}
```
`path` 必须以正斜杠字符 （'/'） 开头和结尾，并且不能包含两个连续的斜杠字符。 创建  `path` 时，我们建议使用`id`，将 '.' 替换为 '/'，并在开头和结尾添加 '/'。

我们只想存储一个`name`为 "is-switch-enabled" 的键。 这是一个布尔值，因此其类型为 "b"（其他选项请参见  [GVariant Format Strings](https://docs.gtk.org/glib/gvariant-format-strings.html)）。 我们还将其默认值设为 `false`（完整语法请参见  [GVariant Text Format](https://docs.gtk.org/glib/gvariant-text-format.html)）。 最后，我们添加一个摘要。

现在，我们需要复制并编译 schema.

> 在 Linux 或 macOS 机器上执行以下命令即可安装 schema：
> ```bash
> mkdir -p $HOME/.local/share/glib-2.0/schemas
> cp org.gtk_rs.Settings1.gschema.xml $HOME/.local/share/glib-2.0/schemas/
> glib-compile-schemas $HOME/.local/share/glib-2.0/schemas/
> ```
> 
> 或者在 Windows 上运行：
> ```powershell
> mkdir C:/ProgramData/glib-2.0/schemas/
> cp org.gtk_rs.Settings1.gschema.xml C:/ProgramData/glib-2.0/schemas/
> glib-compile-schemas C:/ProgramData/glib-2.0/schemas/
> ```

我们通过指定应用程序 ID 来初始化 `Settings` 对象。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/1/main.rs">listings/settings/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/settings/1/main.rs:settings}}
```

然后我们获取设置密钥，并在创建 `Switch` 时使用它。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/1/main.rs">listings/settings/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/settings/1/main.rs:switch}}
```

最后，我们保证，只要点击开关，开关状态就会保存在设置中。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/1/main.rs">listings/settings/1/main.rs</a>

```rust
{{#rustdoc_include ../listings/settings/1/main.rs:connect_state_set}}
```

<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/settings_1.webm" type="video/webm">
  <p>A video which shows that the app can now store the app state</p>
 </video>
</div>

现在，即使关闭应用程序，`Switch` 也会保留其状态。 但我们可以做得更好。 开关有一个属性 "active"，而  `Settings` 允许我们将属性与特定设置绑定。 因此，我们就来做这件事。

我们可以删除初始化 `Switch` 之前的 [`boolean`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/prelude/trait.SettingsExt.html#tymethod.boolean) 调用以及 [`connect_state_set`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Switch.html#method.connect_state_set) 调用。 然后，我们通过指定键、对象和属性名称将设置绑定到属性。 

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/settings/2/main.rs">listings/settings/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/settings/2/main.rs:settings_bind}}
```

只要有一个属性与一个设置很好地对应，你可能就会想把它绑定到设置上。 在其他情况下，通过 getter 和 setter 方法与设置交互往往是正确的选择。
