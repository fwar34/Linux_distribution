# Linux_distribution
linux安装相关

# niri+dms
## 输入法
**输入法框架 + Rime 引擎 + 小鹤双拼方案。**

Ubuntu 26.04 这类较新桌面，**优先推荐 Fcitx5 + Rime**；如果你不想改系统默认输入法框架，也可以用 **IBus + Rime**。

---

### Fcitx5 + Rime（推荐）

#### 1. 安装

```bash
sudo apt update
sudo apt install fcitx5 fcitx5-rime librime-data-double-pinyin
```

#### 2. 配置 Rime 启用小鹤双拼

```bash
mkdir -p ~/.local/share/fcitx5/rime/
```

创建或编辑：

```bash
nano ~/.local/share/fcitx5/rime/default.custom.yaml
```

写入：

```yaml
patch:
  schema_list:
    - schema: double_pinyin_flypy
  switches:
    - name: ascii_mode
      reset: 0
      states: [ "中文", "西文" ]
    - name: full_shape
      states: [ "半角", "全角" ]
    - name: simplification
      reset: 1  # 这里的 1 代表默认开启简体中文
      states: [ "汉简", "漢繁" ]
    - name: ascii_punct
      states: [ "。,", ".," ]
```

`double_pinyin_flypy` 就是小鹤双拼。

#### 3. 重启 Fcitx5

```bash
fcitx5 -r
```

然后在 Fcitx5 配置里添加 **Rime / 中州韵** 输入法。  
如果候选框不显示，Ubuntu 26.04 的 Wayland 环境建议安装并启用 **Input Method Panel** GNOME 扩展。

#### 4. 在 niri 中 设置 fcitx5 环境变量
在 `~/.config/niri/config.kdl` 中添加如下环境变量，开机启动 fcitx5
```bash
// fcitx5 环境变量设置
environment {
  QT_IM_MODULE "fcitx"
  XMODIFIERS "@im=fcitx"
  INPUT_METHOD "fcitx"
}
// 开机启动 fcitx5
spawn-at-startup "fcitx5" "-d"
```

## 终端设置
### Alacritty
#### 配置文件 `.config/alacritty/alacritty.toml`
```toml
[general]
live_config_reload = true

[terminal.shell]
program = "/usr/bin/bash"

[window]
padding = { x = 12, y = 12 }
opacity = 0.95
decorations = "Full"
dynamic_title = true

[scrolling]
history = 10000
multiplier = 3

[font]
size = 13.0
builtin_box_drawing = true

[cursor]
style = { shape = "Beam", blinking = "Off" }

[selection]
save_to_clipboard = true # wayland 安装 wl-clipboard

[colors.primary]
background = "#0a0a0f"
foreground = "#c0caf5"

[colors.normal]
black   = "#1a1b26"
red     = "#f7768e"
green   = "#9ece6a"
yellow  = "#e0af68"
blue    = "#7aa2f7"
magenta = "#bb9af7"
cyan    = "#7dcfff"
```
* 复制 / 粘贴方式
在 Alacritty 里用鼠标选中文本后，会自动进入系统剪贴板。
复制到其它程序：在 Alacritty 里用鼠标选中文字，然后到目标程序按 Ctrl+V。
快捷键复制：Ctrl+Shift+C
快捷键粘贴：Ctrl+Shift+V
如果 Ctrl+Shift+C 不生效，多半是 wl-clipboard 没装，或者 Alacritty 没有重启。
white   = "#a9b1d6"
```
