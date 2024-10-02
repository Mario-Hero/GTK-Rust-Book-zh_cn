# Libadwaita

如果您的图形用户界面以某个平台为目标，您就需要遵循该平台的[《人机界面指南 (Human Interface Guidelines)》](https://en.wikipedia.org/wiki/Human_interface_guidelines) （HIG）。 对于 GTK 应用程序而言，该平台可能是 [elementary OS](https://elementary.io/) 或 [GNOME](https://www.gnome.org/)。 在本章中，我们将讨论如何使用  [libadwaita](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/) 遵循 GNOME 的 [HIG](https://developer.gnome.org/hig/). 

Libadwaita 是一个补充 GTK 4 的库：
- 提供更好地遵循 GNOME 的 HIG 的组件
- 提供让应用程序根据可用空间[改变布局](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/main/adaptive-layouts.html)的组件
- 集成了 Adwaita [样式表](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/main/styles-and-appearance.html)
- 允许在运行时使用[命名了的颜色(named colors)](https://gnome.pages.gitlab.gnome.org/libadwaita/doc/1-latest/named-colors.html)重新着色
- 添加了[API](https://world.pages.gitlab.gnome.org/Rust/libadwaita-rs/stable/latest/docs/libadwaita/struct.StyleManager.html) 以支持跨桌面暗黑颜色风格

要使用 Rust 绑定，请执行以下命令，将 [libadwaita crate](https://crates.io/crates/libadwaita) 添加到依赖：

```
cargo add libadwaita --rename adw --features v1_5
```

`gtk4` 和 `libadwaita` crate 的版本需要同步。 只需记住，当你将其中一个更新到最新版本时，也要更新另一个。

库本身的安装与 GTK 类似。 只需按照适合您的发行版的安装说明进行安装即可。

## Linux

Fedora 及其衍生版本：

```
sudo dnf install libadwaita-devel
```

Debian 及其衍生版本：

```
sudo apt install libadwaita-1-dev
```

Arch 及其衍生版本：

```
sudo pacman -S libadwaita
```

## macOS

```
brew install libadwaita
```

## Windows

## 如果使用 gvsbuild

如果你使用 `gvsbuild` 来构建GTK 4:

```
gvsbuild build libadwaita librsvg
```


## 如果使用 MSVC 手动构建:

在 Windows 开始菜单中，搜索  `x64 Native Tools Command Prompt for VS 2019`，这将打开一个配置为使用 MSVC x64 工具的终端。 在此运行以下命令：

```
cd /
git clone --branch libadwaita-1-3 https://gitlab.gnome.org/GNOME/libadwaita.git --depth 1
cd libadwaita
meson setup builddir -Dprefix=C:/gnome -Dintrospection=disabled -Dvapi=false
meson install -C builddir
```

## 解决图标缺失问题

对于[此问题](https://gitlab.gnome.org/GNOME/gtk/-/issues/5303)，GTK < 4.10 可以使用该[解决方法](https://gitlab.gnome.org/GNOME/gtk/-/blob/34b9ec5be2f3a38e1e72c4d96f130a2b14734121/NEWS#L60)。

### gvsbuild

使用命令提示符：

```
xcopy /s /i C:\gtk-build\gtk\x64\release\share\icons\hicolor\scalable\apps C:\gtk-build\gtk\x64\release\share\icons\hicolor\scalable\actions
gtk4-update-icon-cache.exe -t -f C:\gtk-build\gtk\x64\release\share\icons\hicolor
```

### MSVC 手动构建


```
xcopy /s /i C:\gnome\share\icons\hicolor\scalable\apps C:\gnome\share\icons\hicolor\scalable\actions
gtk4-update-icon-cache.exe -t -f C:\gnome\share\icons\hicolor
```
