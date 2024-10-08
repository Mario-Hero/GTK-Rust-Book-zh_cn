# 动作(Actions)

到目前为止，我们已经学会了许多将控件粘在一起的方法。 我们可以通过通道发送消息、发射信号、共享引用计数状态和绑定属性。 现在，我们将通过学习动作(Actions)来完成我们的设置。 

动作是绑定到某个 GObject 的功能。 让我们来看看最简单的情况，即在没有参数的情况下激活一个动作。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/1/main.rs">listings/actions/1/main.rs</a>

```rust,no_run
{{#rustdoc_include ../listings/actions/1/main.rs:build_ui}}
```

首先，我们创建了一个名为 "close" 的新  [`gio::ActionEntry`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.ActionEntry.html) ，它不需要任何参数。 我们还连接了一个回调，用于在激活动作时关闭窗口。 最后，我们通过 [`add_action_entries`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/prelude/trait.ActionMapExtManual.html#method.add_action_entries)将操作条目添加到窗口中。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/1/main.rs">listings/actions/1/main.rs</a>

```rust,no_run
{{#rustdoc_include ../listings/actions/1/main.rs:main}}
```

使用动作的最常见原因之一是快捷键，因此我们在此添加了一个。 通过  [`set_accels_for_action`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/prelude/trait.GtkApplicationExt.html#tymethod.set_accels_for_action)，可以为某个动作分配一个或多个快捷键。 有关 [`accelerator_parse`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/functions/fn.accelerator_parse.html)的语法，请查阅文档。

在我们继续讨论动作的其他方面之前，让我们先来了解一下这里的一些奇特之处。 "win.close" 中的 "win" 是动作组。 但 GTK 如何知道 "win" 是我们窗口的动作组呢？ 答案是，在窗口和应用程序中添加操作非常普遍，因此已经有两个预定义的组可用：

- "app" 用于应用程序的全局动作，
- "win" 用于与应用程序窗口相关的动作。

我们可以通过  [`insert_action_group`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/prelude/trait.WidgetExt.html#method.insert_action_group) 方法为任何控件添加动作组。 让我们将动作添加到动作组 "custom-group"，然后将该组添加到我们的窗口。该动作项(action entry)不再是针对我们的窗口，"activate（激活）" 回调的第一个参数类型是 `SimpleActionGroup`，而不是 `ApplicationWindow`。 这意味着我们必须将窗口克隆到闭包中。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/2/main.rs">listings/actions/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/actions/2/main.rs:build_ui}}
```

如果我们将快捷键绑定到 "custom-group.close"，它就会像以前一样工作。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/2/main.rs">listings/actions/2/main.rs</a>

```rust
{{#rustdoc_include ../listings/actions/2/main.rs:accel}}
```

此外，如果我们有多个相同窗口的实例，我们会希望在激活 "win.close" 时只关闭当前聚焦的窗口。 事实上，"win.close" 将被派发到当前聚焦的窗口。 不过，这也意味着我们实际上为每个窗口实例定义了一个动作。 如果我们想使用一个全局动作，可以在**应用程序**上调用 `add_action_entries`.

> 添加 "win.close" 作为一个简单的示例非常有用。 不过，今后我们将使用预定义的  ["window.close"](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Window.html#actions) 操作，其作用完全相同。

## 参数和状态

与大多数函数一样，动作可以接受一个参数。 不过，与大多数函数不同的是，它也可以是有状态的。 让我们看看它是如何工作的。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/3/main.rs">listings/actions/3/main.rs</a>

```rust
{{#rustdoc_include ../listings/actions/3/main.rs:build_ui}}
```

在这里，我们创建了一个 "win.count" 动作，每次激活时都会按给定参数增加状态。 它还负责用当前状态更新标签。 每次点击按钮都会激活动作，同时将 "1" 作为参数传递。 我们的应用程序就是这样运行的：

<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/actions_counter.webm" type="video/webm">
  <p>A video which shows that pressing on one button also changes the label below</p>
 </video>
</div>


## 可执行动作的(Actionable)

将动作连接到按钮的 "clicked"（点击）信号是一个典型的使用案例，这就是为什么所有按钮都实现了 [`Actionable`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.Actionable.html) 接口。 这样，就可以通过设置 "action-name" 属性来指定动作。 如果动作接受参数，则可通过 "action-target" 属性进行设置。 有了 [`ButtonBuilder`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/builders/struct.ButtonBuilder.html), 我们可以通过调用其方法来设置一切。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/4/main.rs">listings/actions/4/main.rs</a>

```rust
{{#rustdoc_include ../listings/actions/4/main.rs:button_builder}}
```

还可以通过界面生成器轻松访问可执行动作的部件。 像往常一样，我们通过一个复合模板来创建窗口。 然后，我们可以在模板中设置 "动作名称(action-name)"和 "动作目标(action-target)"属性。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/5/resources/window.ui">listings/actions/5/resources/window.ui</a>

```xml
{{#rustdoc_include ../listings/actions/5/resources/window.ui}}
```

我们将在 `Window::setup_actions` 方法中连接操作并将其添加到窗口。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/5/window/mod.rs">listings/actions/5/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/5/window/mod.rs:impl_window}}
```

最后，`setup_actions` 将在 `constructed` 函数内调用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/5/window/imp.rs">listings/actions/5/window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/actions/5/window/imp.rs:object_impl}}
```

该应用程序的行为与我们之前的示例相同，但它将使我们在下一部分添加菜单时更加简单。


## 菜单

如果要创建[菜单](https://developer.gnome.org/hig/patterns/controls/menus.html)，就必须使用动作，而且要使用界面生成器。 通常情况下，菜单条目中的操作符合以下三种描述之一：

- 无参数和无状态，
- 或无参数和布尔状态，
- 或字符串参数和字符串状态。

让我们修改我们的小程序来演示这些情况。 首先，我们扩展 `setup_actions`。 对于不带参数或状态的动作，我们可以使用预定义的 "window.close" 动作。 因此，我们无需在此处添加任何内容。

通过动作 "button-frame"，我们可以操作按钮的 "has-frame" 属性。 这里的惯例是，不带参数并且为布尔状态的操作应该像切换操作一样。 这意味着调用者可以期待布尔状态在激活动作后切换。 幸运的是，这正是带有布尔属性的 [`gio::PropertyAction`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.PropertyAction.html) 的默认行为。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/6/window/mod.rs">listings/actions/6/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/6/window/mod.rs:action_button_frame}}
```

> 当您需要一个操作 GObject 属性的动作时，`PropertyAction` 就会派上用场。 属性将作为动作的状态。 如上所述，如果属性是布尔型，则动作没有参数，并在激活时切换属性。 在所有其他情况下，操作都有一个与属性类型相同的参数。 激活动作时，属性会被设置为与动作参数相同的值。

最后，我们添加了 "win.orientation"，一个带有字符串参数和字符串状态的动作。 该操作可用于改变 `gtk_box` 的朝向。 这里的惯例是状态(state)应设置为给定的参数。 我们并不需要动作状态来实现方向切换，但它在使菜单显示当前朝向时非常有用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/6/window/mod.rs">listings/actions/6/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/6/window/mod.rs:action_orientation}}
```

尽管 [`gio::Menu`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Menu.html) 也可以通过绑定创建，但最方便的方法还是使用界面生成器。 我们可以在模板前添加菜单。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/6/resources/window.ui">listings/actions/6/resources/window.ui</a>

```diff
 <?xml version="1.0" encoding="UTF-8"?>
 <interface>
+  <menu id="main-menu">
+    <item>
+      <attribute name="label" translatable="yes">_Close window</attribute>
+      <attribute name="action">window.close</attribute>
+    </item>
+    <item>
+      <attribute name="label" translatable="yes">_Toggle button frame</attribute>
+      <attribute name="action">win.button-frame</attribute>
+    </item>
+    <section>
+      <attribute name="label" translatable="yes">Orientation</attribute>
+      <item>
+        <attribute name="label" translatable="yes">_Horizontal</attribute>
+        <attribute name="action">win.orientation</attribute>
+        <attribute name="target">Horizontal</attribute>
+      </item>
+      <item>
+        <attribute name="label" translatable="yes">_Vertical</attribute>
+        <attribute name="action">win.orientation</attribute>
+        <attribute name="target">Vertical</attribute>
+      </item>
+    </section>
+  </menu>
   <template class="MyGtkAppWindow" parent="GtkApplicationWindow">
     <property name="title">My GTK App</property>
+    <property name="width-request">360</property>
+    <child type="titlebar">
+      <object class="GtkHeaderBar">
+        <child type ="end">
+          <object class="GtkMenuButton">
+            <property name="icon-name">open-menu-symbolic</property>
+            <property name="menu-model">main-menu</property>
+          </object>
+        </child>
+      </object>
+    </child>
     <child>
       <object class="GtkBox" id="gtk_box">
         <property name="orientation">vertical</property>
```

由于我们通过 [menu-model](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.MenuButton.html#menu-model) 属性将菜单连接到了  [`gtk::MenuButton`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.MenuButton.html) ，因此菜单(`Menu`)应为  [`gtk::PopoverMenu`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.PopoverMenu.html). `PopoverMenu` 的[文档](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.PopoverMenu.html)还为界面生成器解释了其 xml 语法。

还要注意我们是如何指定目标的：

```xml
<attribute name="target">Horizontal</attribute>
```

字符串是目标的默认类型，因此我们无需指定类型。 对于其他类型的目标，则需要手动指定正确的 [GVariant 格式字符串](https://docs.gtk.org/glib/gvariant-format-strings.html)。 例如，一个值为 "5 "的 `i32` 变量对应的格式如下：

```xml
<attribute name="target" type="i">5</attribute>
```

这就是该应用程序的实际效果：


<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/actions_menu.webm" type="video/webm">
  <p>A video which now also shows the menu</p>
 </video>
</div>

>我们将菜单按钮(`MenuButton`)的属性 "icon-name" 设置为 "open-menu-symbolic"，从而更改了菜单按钮的图标。 您可以在[图标库](https://flathub.org/apps/org.gnome.design.IconLibrary)中找到更多图标。 这些图标可以嵌入  [`gio::Resource`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Resource.html) ，然后在合成模板（或其他地方）中引用。

## 设置(Settings)

菜单项很好地显示了有状态动作的状态，但在应用程序关闭后，对该状态的所有更改都会丢失。 像往常一样，我们使用 [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html) 解决这个问题。 首先，我们创建一个 schema，其中包含与之前创建的有状态操作相对应的设置。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/org.gtk_rs.Actions7.gschema.xml">listings/actions/7/org.gtk_rs.Actions7.gschema.xml</a>

```xml
{{#rustdoc_include ../listings/actions/7/org.gtk_rs.Actions7.gschema.xml}}
```

同样，我们按照[设置一章]()中描述来安装 schema. 然后将设置添加到 `imp::Window`. 由于 [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html) 没有实现 `Default`，我们将其封装在 [`std::cell::OnceCell`](https://doc.rust-lang.org/std/cell/struct.OnceCell.html) 中。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/imp.rs">listings/actions/7/window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/imp.rs:imp_struct}}
```

现在，我们创建一些函数，使设置更容易访问。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:settings}}
```

通过设置条目创建有状态的动作非常常见，因此`设置(Settings)`提供了一种方法来实现这一目的。 我们使用 [ `create_action`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/prelude/trait.SettingsExt.html#tymethod.create_action) 方法创建动作，然后将其添加到窗口的动作组中。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:settings_create_actions}}
```

由于来自 `create_action` 的动作遵循上述约定，我们可以尽量减少进一步的改动。 每次激活时，"win.button-frame" 动作都会切换其状态，而 "win.orientation" 动作的状态则遵循给定的参数。

不过，我们仍需指定动作激活时应发生的情况。 对于有状态的操作，我们不用为它的"激活"信号添加回调，而是将设置绑定到我们要操作的属性上。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:bind_settings}}
```

最后，我们要确保 `bind_settings` 在构造(`constructed`)内部被调用。

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/imp.rs">listings/actions/7/window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/imp.rs:object_impl}}
```

动作的功能非常强大，我们在此只是浅尝辄止。 如果您想了解更多，[GNOME 开发者文档](https://developer.gnome.org/documentation/tutorials/actions.html)是一个很好的开始。
