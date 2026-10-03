# Linux_distribution
linux安装相关

# niri+dms
## 输入法
**输入法框架 + Rime 引擎 + 小鹤双拼方案。**

Ubuntu 26.04 这类较新桌面，**优先推荐 Fcitx5 + Rime**；如果你不想改系统默认输入法框架，也可以用 **IBus + Rime**。

---

### 方案一：Fcitx5 + Rime（推荐）

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
```

`double_pinyin_flypy` 就是小鹤双拼。

#### 3. 重启 Fcitx5

```bash
fcitx5 -r
```

然后在 Fcitx5 配置里添加 **Rime / 中州韵** 输入法。  
如果候选框不显示，Ubuntu 26.04 的 Wayland 环境建议安装并启用 **Input Method Panel** GNOME 扩展。

---

如果你说的 “dms” 不是指 Linux 桌面，而是某个具体系统或软件，比如 Deepin、DMS 终端、某个国产系统，可以把完整名称或截图发我，我再按那个环境给你写对应命令。
