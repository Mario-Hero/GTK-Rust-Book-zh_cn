# 复合模板

在此之前，每当我们构建预定义的部件时，我们都依赖于[构建器模式](https://rust-unofficial.github.io/patterns/patterns/creational/builder.html)。 提醒一下，我们就是这样使用它来构建我们值得信赖的 "Hello World!

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/hello_world/3/main.rs">listings/hello_world/3/main.rs</a>
```rust
{{#rustdoc_include ../listings/hello_world/3/main.rs:all}}
```

<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/hello_world_button.webm" type="video/webm">
  <p>A video which shows that pressing on a button changes its label</p>
 </video>
</div>
直接从代码中创建小部件固然很好，但却很难将逻辑与用户界面分离开来。 这就是为什么大多数工具包都允许使用标记语言来描述用户界面，GTK 也不例外。 例如，下面的 `xml` 文件描述了 “Hello World!” 应用。 

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/resources/window.ui">listings/composite_templates/1/resources/window.ui</a>
```xml
{{#rustdoc_include ../listings/composite_templates/1/resources/window.ui}}
```

最外层的标签必须是  `<interface>` （接口）。然后开始列出要描述的元素。为了定义一个复合模板，我们指定了要创建的自定义控件的名称 `MyGtkAppWindow` 及其派生自的父级 [`gtk::ApplicationWindow`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.ApplicationWindow.html). 这些 `xml` 文件与编程语言无关，这也是为什么这些类使用原始名称的原因。 幸运的是，它们都是这样转换的：`gtk::ApplicationWindow` → `GtkApplicationWindow`. 然后我们就可以为 `ApplicationWindow` 指定属性了（在[这里](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.ApplicationWindow.html#properties)给出）。由于 `ApplicationWindow` 可以包含其他控件，我们使用 `<child>` 标记来添加一个 `gtk::Button`.  我们希望以后能引用该按钮，因此还设置了它的 `id`.

## Resources（资源）

为了将模板文件嵌入应用程序，我们使用了 [`gio::Resource`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Resource.html). 要嵌入的文件同样由 `xml` 文件描述。 我们还为模板文件添加了压缩(`compressed`)和预处理(`preprocess`)属性，以减小资源的最终大小。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/resources/resources.gresource.xml">listings/composite_templates/1/resources/resources.gresource.xml</a>
```xml
{{#rustdoc_include ../listings/composite_templates/1/resources/resources.gresource.xml}}
```

现在，我们必须编译资源并将其链接到应用程序。 
 一种方法是在 cargo [编译脚本](https://doc.rust-lang.org/cargo/reference/build-scripts.html)中执行 [`glib_build_tools::compile_resources`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/glib_build_tools/fn.compile_resources.html). 

首先，我们必须在 `Cargo.toml` 中添加 `glib-build-tools` 作为编译依赖，方法是执行：

```
cargo add glib-build-tools --build
```

然后，我们在软件包根目录下创建一个 `build.rs`，内容如下。 每当我们使用 cargo 触发编译时，它就会对资源进行编译，然后静态链接我们的可执行文件。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/build.rs">listings/build.rs</a>
```rust
fn main() {
    glib_build_tools::compile_resources(
        &["composite_templates/1/resources"],
        "composite_templates/1/resources/resources.gresource.xml",
        "composite_templates_1.gresource",
    );
}
```

最后，我们通过调用宏 [`gio::resources_register_include!`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/macro.resources_register_include.html) 来注册并包含资源。 在您自己的应用程序中，请注意在创建 `gtk::Application` 之前注册资源。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/main.rs">listings/composite_templates/1/main.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/1/main.rs}}
```

在我们的代码中，我们创建了一个继承自 `gtk::ApplicationWindow` 的自定义控件，以使用我们的模板。 
文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/window/mod.rs">listings/composite_templates/1/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/composite_templates/1/window/mod.rs}}
```

在实现结构中，我们添加了派生宏  [`gtk::CompositeTemplate`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4_macros/derive.CompositeTemplate.html). 我们还指定模板信息来自前缀为 `/org/gtk-rs/example` 的资源，其中包含 `window.ui` 文件。

模板的一个非常方便的功能是模板子结构。 使用方法是在模板中添加一个与 id 属性同名的结构成员。 然后，[`TemplateChild`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/subclass/widget/struct.TemplateChild.html) 会存储一个控件的引用，供以后使用。 稍后，当我们要为按钮添加回调时，这将非常有用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/window/imp.rs">listings/composite_templates/1/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/1/window/imp.rs:object}}
```

在 `ObjectSubclass` trait 中，我们确保 `NAME` 与模板中的类相对应，`ParentType` 与模板中的父类相对应。
我们还在 `class_init` 和 `instance_init` 中绑定并初始化模板。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/window/imp.rs">listings/composite_templates/1/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/1/window/imp.rs:subclass}}
```

最后，我们将回调连接到构造函数(`constructed`)中按钮(`button`)的 "clicked" 信号。 由于在 `self` 中存储了引用，所以按钮很容易就能找到。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/1/window/imp.rs">listings/composite_templates/1/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/1/window/imp.rs:object_impl}}
```

## 自定义控件

我们还可以在模板文件中实例化自定义控件。 首先我们定义 `CustomButton`，它继承于 `gtk::Button`. 像往常一样，我们在 `imp.rs` 中定义实现结构。 请注意我们在这里定义的 `NAME`，稍后我们将需要它在模板中进行引用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/2/custom_button/imp.rs">listings/composite_templates/2/custom_button/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/2/custom_button/imp.rs:imp}}
```

我们还在 `mod.rs` 中定义了公共结构。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/2/custom_button/mod.rs">listings/composite_templates/2/custom_button/mod.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/2/custom_button/mod.rs:mod}}
```

由于我们现在要引用 `CustomButton`，因此还必须将模板子代的类型更改为 CustomButton。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/2/window/imp.rs">listings/composite_templates/2/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/2/window/imp.rs:object}}
```

最后，我们可以在复合模板中用 `MyGtkAppCustomButton` 替换 `GtkButton`。 由于自定义按钮是 `gtk::Button` 的直接子类，并且没有做任何修改，因此我们应用程序的行为将保持不变。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/2/resources/window.ui">listings/composite_templates/2/resources/window.ui</a>
```xml
{{#rustdoc_include ../listings/composite_templates/2/resources/window.ui}}
```

## Template Callbacks

我们甚至可以在复合模板中指定信号的处理程序。 这可以通过 `<signal>` 标记来实现，该标记包含信号名称和 Rust 代码中的处理程序。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/3/resources/window.ui">listings/composite_templates/3/resources/window.ui</a>
```xml
{{#rustdoc_include ../listings/composite_templates/3/resources/window.ui}}
```

然后，我们在定义 `handle_button_clicked` 时应用 [`template_callbacks`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4_macros/attr.template_callbacks.html) 宏。 我们可以通过查看要处理的信号的 `connect_*` 方法来确定函数签名。 在我们的例子中就是  [`connect_clicked`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/prelude/trait.ButtonExt.html#tymethod.connect_clicked). 它需要一个 `Fn(&Self)` 类型的函数。 `Self` 指的是我们的按钮。 这意味着 `handle_button_clicked` 有一个 `&CustomButton` 类型的参数。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/3/window/imp.rs">listings/composite_templates/3/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/3/window/imp.rs:template_callbacks}}
```

然后，我们必须使用  [`bind_template_callbacks`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/subclass/widget/trait.CompositeTemplateCallbacksClass.html#tymethod.bind_template_callbacks) 绑定模板回调。 我们还需要删除 `window/imp.rs` 中的 `button.connect_clicked` 回调。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/3/window/imp.rs">listings/composite_templates/3/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/3/window/imp.rs:subclass}}
```

我们还可以访问部件的状态。 比方说，我们要操作存储在 `imp::Window` 中的一个 `number`. 

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/4/window/imp.rs">listings/composite_templates/4/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/4/window/imp.rs:object}}
```

为了访问 widget 的状态，我们必须在信号标记中添加 `swapped="true"`.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/4/resources/window.ui">listings/composite_templates/4/resources/window.ui</a>
```xml
{{#rustdoc_include ../listings/composite_templates/4/resources/window.ui}}
```

现在，我们可以将 `&self` 作为 `handle_button_clicked` 的第一个参数。 这样，我们就可以访问窗口的状态，从而对 `number` 进行操作。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/4/window/imp.rs">listings/composite_templates/4/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/4/window/imp.rs:template_callbacks}}
```

## Registering Types

现在我们使用模板回调，就不再访问模板子模板了。 让我们删除它。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/5/window/imp.rs">listings/composite_templates/5/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/5/window/imp.rs:object}}
```

但是，当我们现在运行它时，GTK 不再将 `MyGtkAppCustomButton` 视为有效的对象类型。 这是怎么回事？

```
Gtk-CRITICAL **: Error building template class 'MyGtkAppWindow' for an instance of
                 type 'MyGtkAppWindow': Invalid object type 'MyGtkAppCustomButton'
```

原来，添加模板子模板不仅可以方便地引用模板中的部件，它还能确保部件类型已注册。 幸运的是，我们自己也可以做到这一点。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/composite_templates/6/window/imp.rs">listings/composite_templates/6/window/imp.rs</a>
```rust
{{#rustdoc_include ../listings/composite_templates/6/window/imp.rs:subclass}}
```

我们调用 `class_init` 中的 `ensure_type ` 方法，瞧：我们的应用程序又能运行了。


## 结论

借助自定义控件，我们可以：
- 将状态和部分状态作为属性保留下来，
- 添加信号并且
- 覆盖行为。

借助复合模板，我们可以：
- 简要地描述复杂的用户界面，
- 轻松访问模板内的控件，
- 并为信号指定处理函数。

这里涉及的应用程序接口非常广泛，因此尤其是在开始学习时，您需要查看相关文档。 UI 文件的基本语法在 [`Builder`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Builder.html#gtkbuilder-ui-definitions) 中解释，而控件的特定语法则在 [`Widget`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Widget.html#gtkwidget-as-gtkbuildable) 中解释。 如果某个控件接受附加元素，通常会在该控件的文档中加以说明。

在下一章中，我们将了解复合模板如何帮助我们创建稍大的应用程序，如 To-Do 应用程序。
