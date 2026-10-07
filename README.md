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
      reset: 1 # 默认输入英文
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
decorations = "none"
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

## 键盘映射
使用 keyd 进行精细化的映射 `apt install keyd`，设置配置文件 `/etc/keyd/default.conf` 如下
```
[ids]
# 任意键盘都生效
*

[main]
# CapsLock → 左 Ctrl
capslock = leftcontrol
# 右 Ctrl → End
rightcontrol = end
# 右 Alt → Home
rightalt = home
# 右 Shift → End
rightshift = end


# 最经典的例子是将 CapsLock 变成一个 导航层（按住 CapsLock 时，HJKL 变成方向键）
# [ids]
# *
#
# [main]
# capslock = layer(nav)
#
# [nav]
# h = left
# j = down
# k = up
# l = right
```
重载 keyd `sudo systemctl restart keyd`


# niri+noctalia on Fedora44
## 字体问题
### 问：edge当前的汉字的“复”，“关”等这些字体怎么窄一些，在noctalia的设置程序中一样的问题
答：不是 Edge 的问题，也不是 noctalia 的问题，而是 fontconfig 在"没有语言标记"时的汉字回退顺序：

系统只有 Noto Sans CJK 这一个 TTC 合集，里面 JP/SC/TC/KR/HK 各是一个 face，JP 是 index 0；
应用做逐字回退时通常不带 lang=zh，于是 fontconfig 在多个 face 同分时按文件顺序取到 JP；
带 lang=zh-cn 时才会选 SC，所以 fc-match 'sans-serif:lang=zh-cn' 和 fc-match ':charset=590d' 结果不一样；
复/关 恰好是日文字形收得明显更紧的两个字，所以你一眼看出来"窄"。带 lang 处理的 Firefox 基本看不出来，Chromium 和 Qt 就一起中招。
证据（字节级比对，不是肉眼判断）：
```
Edge 默认渲染 vs 显式 'Noto Sans CJK JP' : 字节完全一致   ← 确认走的是日文字形
Edge 默认渲染 vs 显式 'Noto Sans CJK SC' : 不同
pango/Qt 路径 修复前 vs JP 参考          : 一致
pango/Qt 路径 修复后 vs SC 参考          : 一致
```
顺带说明为什么不是"字宽"问题：我把所有含这两个字的 face 都量了一遍，`advance 全都是 1000/1000 全宽`，所以纯属字形设计差异，font-feature/缩放那类手段都治不了。
* 怎么改
新建了 99-cjk-prefer-sc.conf（放在用户 ~/.config/fontconfig/conf.d 里，会被 /etc/fonts/fonts.conf 加载，不覆盖系统配置）：
```xml
<alias binding="strong">
  <family>sans-serif</family>
  <prefer><family>Noto Sans CJK SC</family></prefer>
</alias>
```
*回退与取舍
回退：rm ~/.config/fontconfig/conf.d/99-cjk-prefer-sc.conf 即可。
只改了 sans-serif（界面正文用的就是它）。serif/monospace 里的汉字仍是日文字形——实测那两条一旦加上，衬线/等宽的拉丁字母也会被换成 Noto CJK 自带的拉丁字形，所以我按你选的只改 sans。
规则不带语言条件，所以带 lang="ja" 的日文页面以后也会用简体字形。你要是有看日文站的需求，我可以再加一条 lang 含 ja 时优先 JP 的规则排在后面。

## 弹窗问题
### 问：clash在settings中打卡TUN服务模式怎么一直卡着不动，我使用MOD+D启动的clash，clash打卡服务的时候也没有弹出让我输入root密码的弹窗
答：1. 服务根本没装。 TUN 模式需要特权服务 clash-verge-service，而：
```
systemctl is-enabled clash-verge-service  →  not-found（单元不存在）
App 日志: [Service] Service is NotInstalled ... running privileged service action Install
```
rpm 只装了 /usr/bin/clash-verge-service-install 这个安装器（ELF 二进制），系统服务单元要靠它写入。

2. 提权调用没有 polkit 代理可弹窗。 从 GUI 二进制里读出的逻辑是：
```
pkexec --disable-internal-agent ...Install...   ；失败才 ", falling back to sudo"
```
--disable-internal-agent 的含义就是"不要用 pkexec 自带的终端提示，必须有图形 polkit 代理"。而你机器上：
```
polkit / polkit-libs / polkit-pkla-compat    ← 只有框架，没有任何认证代理
（empower.rules 是 systemd 自带的文件，不是 empower 代理）
niri 的 spawn-at-startup 只有 noctalia 和 fcitx5，也不跑 XDG autostart
```
所以提权请求发出去后没人能问密码，PAM 会话瞬间失败：
```
polkit-agent-helper-1: pam_unix(polkit-1:auth): conversation failed
polkit-agent-helper-1: auth could not identify password for [feng]
polkitd: Operator of unix-process:unknown FAILED to authenticate to gain authorization
       for action org.freedesktop.policykit.exec
```
你是 Mod+D 启动的，没有 TTY，sudo 那条回退路也走不通，于是 GUI 就停在 "running privileged service action Install" 死等。
* 怎么改
修好图形提权（以后更新、挂盘、改系统设置都会用到）。仓库里确认可装的是 lxqt-policykit：
```bash
sudo dnf install lxqt-policykit
rpm -ql lxqt-policykit | grep lxqt-policykit-agent        # 确认可执行文件路径，通常是 lxqt-policykit-agent
```
然后在 ~/.config/niri/config.kdl 的 spawn-at-startup 里加一行（路径按上面确认的填）：
/usr/libexec 不在 PATH 里，所以配置里必须写绝对路径。在 ~/.config/niri/config.kdl 的 spawn-at-startup 区域加一行：
```
spawn-at-startup "/usr/libexec/lxqt-policykit-agent"
```
现有那两行是 noctalia 和 fcitx5 -d，加在它们后面即可。之后重新登录 niri 就自动拉起。
重新登录 niri 后，pkexec 才会有弹窗。
