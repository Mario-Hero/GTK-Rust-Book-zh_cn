# Linux

首先您必须安装 rustup。 您可以在 [rustup.rs](https://rustup.rs/) 上找到最新的安装教程。

然后安装 GTK 4 和 build essentials。 为此，请执行属于您正在使用的发行版的命令。

Fedora 及其衍生版本：

```
sudo dnf install gtk4-devel gcc
```

Debian 及其衍生版本：

```
sudo apt install libgtk-4-dev build-essential
```

Arch 及其衍生版本：

```
sudo pacman -S gtk4 base-devel
```
