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

With the action "button-frame", we manipulate the "has-frame" property of `button`.
Here, the convention is that actions with no parameter and boolean state should behave like toggle actions.
This means that the caller can expect the boolean state to toggle after activating the action. Luckily for us, that is the default behavior for [`gio::PropertyAction`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.PropertyAction.html) with a boolean property.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/6/window/mod.rs">listings/actions/6/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/6/window/mod.rs:action_button_frame}}
```

> A `PropertyAction` is useful when you need an action that manipulates the property of a GObject.
> The property then acts as the state of the action.
> As mentioned above, if the property is a boolean the action has no parameter and toggles the property on activation.
> In all other cases, the action has a parameter of the same type as the property.
> When activating the action, the property gets set to the same value as the parameter of the action.


Finally, we add "win.orientation", an action with string parameter and string state.
This action can be used to change the orientation of `gtk_box`.
Here the convention is that the state should be set to the given parameter.
We don't need the action state to implement orientation switching, however it is useful for making the menu display the current orientation.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/6/window/mod.rs">listings/actions/6/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/6/window/mod.rs:action_orientation}}
```

Even though [`gio::Menu`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Menu.html) can also be created with the bindings, the most convenient way is to use the interface builder for that.
We do that by adding the menu in front of the template.

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

Since we connect the menu to the [`gtk::MenuButton`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.MenuButton.html) via the [menu-model](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.MenuButton.html#menu-model) property, the `Menu` is expected to be a [`gtk::PopoverMenu`](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.PopoverMenu.html).
The [documentation](https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/struct.PopoverMenu.html) for `PopoverMenu` also explains its `xml` syntax for the interface builder.

Also note how we specified the target:

```xml
<attribute name="target">Horizontal</attribute>
```

String is the default type of the target which is why we did not have to specify a type.
With targets of other types you need to manually specify the correct [GVariant format string](https://docs.gtk.org/glib/gvariant-format-strings.html).
For example, an `i32` variable with value "5" would correspond to this:

```xml
<attribute name="target" type="i">5</attribute>
```

This is how the app looks in action:


<div style="text-align:center">
 <video autoplay muted loop>
  <source src="vid/actions_menu.webm" type="video/webm">
  <p>A video which now also shows the menu</p>
 </video>
</div>

>We changed the icon of the `MenuButton` by setting its property "icon-name" to "open-menu-symbolic".
>You can find more icons with the [Icon Library](https://flathub.org/apps/org.gnome.design.IconLibrary).
>They can be embedded with [`gio::Resource`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Resource.html) and then be referenced within the composite templates (or other places).

## Settings

The menu entries nicely display the state of our stateful actions, but after the app is closed, all changes to that state are lost.
As usual, we solve this problem with [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html).
First we create a schema with settings corresponding to the stateful actions we created before.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/org.gtk_rs.Actions7.gschema.xml">listings/actions/7/org.gtk_rs.Actions7.gschema.xml</a>

```xml
{{#rustdoc_include ../listings/actions/7/org.gtk_rs.Actions7.gschema.xml}}
```

Again, we install the schema as described in the settings [chapter](./settings.html).
Then we add the settings to `imp::Window`.
Since [`gio::Settings`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/struct.Settings.html) does not implement `Default`, we wrap it in a [`std::cell::OnceCell`](https://doc.rust-lang.org/std/cell/struct.OnceCell.html).

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/imp.rs">listings/actions/7/window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/imp.rs:imp_struct}}
```

Now we create functions to make it easier to access settings.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:settings}}
```


Creating stateful actions from setting entries is so common that `Settings` provides a method for that exact purpose.
We create actions with the[ `create_action`](https://gtk-rs.org/gtk-rs-core/stable/latest/docs/gio/prelude/trait.SettingsExt.html#tymethod.create_action) method and then add them to the action group of our window.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:settings_create_actions}}
```

Since actions from `create_action` follow the aforementioned conventions, we can keep further changes to a minimum.
The action "win.button-frame" toggles its state with each activation and the state of the "win.orientation" action follows the given parameter.

We still have to specify what should happen when the actions are activated though.
For the stateful actions, instead of adding callbacks to their "activate" signals, we bind the settings to properties we want to manipulate.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/mod.rs">listings/actions/7/window/mod.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/mod.rs:bind_settings}}
```

Finally, we make sure that `bind_settings` is called within `constructed`.

文件名：<a class=file-link href="https://github.com/gtk-rs/gtk4-rs/blob/master/book/listings/actions/7/window/imp.rs">listings/actions/7/window/imp.rs</a>

```rust
{{#rustdoc_include ../listings/actions/7/window/imp.rs:object_impl}}
```

Actions are extremely powerful, and we are only scratching the surface here.
If you want to learn more about them, the [GNOME developer documentation](https://developer.gnome.org/documentation/tutorials/actions.html) is a good place to start.
