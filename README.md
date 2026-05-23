# Obsidian 部署完全指南

> 最后更新：2026年5月 | 适用版本：Obsidian v1.12+

---

## 目录

1. [macOS 本地完整部署方案](#一macos-本地完整部署方案)
2. [Windows 本地完整部署方案](#二windows-本地完整部署方案)
3. [多设备同步方案](#三多设备同步方案)
4. [笔记发布为网站](#四笔记发布为网站)
5. [Docker 容器化部署（浏览器访问）](#五docker-容器化部署浏览器访问)
6. [安全注意事项](#六安全注意事项)

---

## 一、macOS 本地完整部署方案

> 目标：打造一个高性能、美观、高效的个人知识管理系统。

### 1.1 安装前环境检查

```bash
# 检查 macOS 版本
sw_vers
# 需要 macOS 12.0+

# 检查可用磁盘空间（建议至少 10GB 可用）
df -h

# 如果要用 Git 同步，检查 Git 是否已安装
git --version
# macOS 自带 Git，如果缺失则运行：xcode-select --install
```

### 1.2 安装 Obsidian

**方式一：官网下载（推荐）**

1. 打开 https://obsidian.md/download
2. 下载 macOS 版本的 `.dmg` 文件
3. 双击 `.dmg`，将 Obsidian 拖入 `Applications` 文件夹
4. 首次打开时，如果系统提示「无法验证开发者」：
   - 打开「系统设置」→「隐私与安全性」
   - 点击「仍要打开」按钮
   - 或终端执行：`xattr -d com.apple.quarantine /Applications/Obsidian.app`

**方式二：Homebrew（适合开发者）**

```bash
brew install --cask obsidian
```

### 1.3 创建知识库（Vault）

```
步骤1：启动 Obsidian → 点击「Create new vault」
步骤2：设置仓库名称，如「MyKnowledge」
步骤3：选择存储路径，强烈推荐以下位置：
       ✅ ~/Documents/Obsidian/         # 文档目录
       ✅ ~/Library/Mobile Documents/iCloud~md~obsidian/Documents/  # iCloud 同步
       ✅ ~/OneDrive/Obsidian/           # OneDrive 同步
       ❌ ~/Downloads/                   # 不推荐，容易误删
       ❌ /Applications/                 # 不推荐，无写入权限
```

**推荐的目录规划：**

```
MyKnowledge/
├── 00-Inbox/             # 待处理的暂存笔记
├── 01-Daily/             # 每日日记
├── 02-Projects/          # 项目相关笔记
├── 03-Areas/             # 长期关注的领域
├── 04-Resources/         # 参考资料
├── 05-Archive/           # 归档笔记
├── 06-Templates/         # 模板文件
├── 07-Attachments/       # 附件（图片、PDF等）
└── README.md             # 仓库说明
```

> **说明**：这是经典的 PARA 方法（Tiago Forte）目录结构。也可以按自己的习惯组织，Obsidian 不做强制。

### 1.4 基础设置（必须完成）

#### 1.4.1 语言切换

设置 → 关于 → 语言 → 选择「简体中文」→ 点击按钮重启

#### 1.4.2 编辑器设置

```
设置 → 编辑器：
  - 默认编辑模式：实时预览（Live Preview）
  - 显示行号：开启
  - 缩进：4 个空格
  - 严格换行：关闭（Markdown 中不建议硬换行）
  - 拼写检查：根据个人需要开启/关闭
```

#### 1.4.3 文件与链接

```
设置 → 文件与链接：
  - 新建笔记的存放位置：与当前笔记相同的文件夹
  - 内部链接类型：最短路径（如 [[笔记名]]，而非 [[文件夹/笔记名]]）
  - 使用 [[Wikilinks]]：开启
  - 新建笔记的默认文件夹：00-Inbox
  - 附件的默认存放路径：07-Attachments
    推荐在子文件夹中创建：images/${filename}
    这样每篇笔记的图片都放在 images/笔记名/ 下
```

#### 1.4.4 外观设置

```
设置 → 外观：
  - 基础颜色：根据喜好选「暗色」或「浅色」
  - 字体大小：推荐 16px（macOS 默认字体渲染较好）
  - 缩小行宽：开启（阅读体验更好，约 700px 宽）
  - 窗口框架：推荐「Obsidian 框架」（非原生标题栏）
  - 快速调整字体大小：Ctrl + 鼠标滚轮
```

#### 1.4.5 核心插件启用

```
设置 → 核心插件 → 开启以下插件：

必开（影响基础体验）：
  ☑ 斜杠命令           # 输入 / 呼出命令菜单
  ☑ 快速切换           # Cmd+O 快速打开文件
  ☑ 命令面板           # Cmd+P 执行命令
  ☑ 文件恢复           # 意外退出时恢复未保存内容
  ☑ 页面预览           # 鼠标悬停链接时预览内容

推荐开启（提升工作流）：
  ☑ 每日笔记           # 日记功能
  ☑ 模板               # 笔记模板
  ☑ 标签面板           # 标签管理
  ☑ 大纲               # 文档大纲导航
  ☑ 反向链接           # 查看哪些笔记引用了当前笔记
  ☑ 书签               # 快速访问常用笔记/搜索
  ☑ 白板               # 可视化思维导图
  ☑ 录音               # 内建录音功能
  ☑ 随机笔记           # 回顾旧笔记
  ☑ 字数统计           # 写作统计
```

#### 1.4.6 快捷键自定义

建议自定义以下快捷键（设置 → 快捷键）：

| 功能 | 推荐快捷键 |
|------|-----------|
| 快速切换 | `Cmd + O` |
| 命令面板 | `Cmd + P` |
| 切换编辑/预览 | `Cmd + E` |
| 打开今日日记 | `Cmd + Shift + D` |
| 在当前笔记中搜索 | `Cmd + F` |
| 全局搜索 | `Cmd + Shift + F` |
| 折叠/展开全部 | `Cmd + .` / `Cmd + Shift + .` |
| 插入模板 | `Cmd + Shift + T` |
| 切换左侧边栏 | `Cmd + Shift + L` |
| 切换右侧边栏 | `Cmd + Shift + R` |

### 1.5 主题安装与美化

#### 1.5.1 安装主题

```
步骤1：设置 → 外观 → 主题 → 点击「管理」
步骤2：在社区主题搜索框中搜索以下主题
步骤3：点击「安装并使用」
```

**推荐的 macOS 风格主题：**

| 主题名称 | 特点 | 适合场景 |
|---------|------|---------|
| **Minimal** | 极度简洁，可高度自定义 | 追求极简，通用 |
| **Things** | macOS Things App 风格 | 苹果生态用户 |
| **Cupertino** | macOS/iOS 原生风格 | 苹果系统原生感 |
| **ITS Theme** | 现代感强，夜览优雅 | 长时间写作 |
| **Shimmering Focus** | 极简，少视觉干扰 | 专注写作 |
| **Border** | 带边框设计，结构清晰 | 需要视觉层次 |

#### 1.5.2 中文字体优化

如果显示中文时字体不理想，通过 CSS 片段修复。

```
步骤1：设置 → 外观 → CSS 代码片段 → 打开片段文件夹（文件夹图标）
步骤2：在其中创建文件「chinese-font.css」
步骤3：返回设置 → 外观 → CSS 代码片段 → 刷新并启用
```

`chinese-font.css` 内容：

```css
/* 中文字体优化 */
body {
  --font-text-theme: -apple-system, BlinkMacSystemFont, "PingFang SC",
    "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
  --font-interface-theme: -apple-system, BlinkMacSystemFont, "PingFang SC",
    "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
  --font-monospace-theme: "SF Mono", "Menlo", "JetBrains Mono", "Fira Code",
    monospace;
}

/* 中文字体加大行间距 */
.markdown-source-view, .markdown-reading-view {
  --line-height-normal: 1.8;
}

/* 标题优化 */
.cm-header-1, h1 { font-size: 1.8em !important; }
.cm-header-2, h2 { font-size: 1.5em !important; }
.cm-header-3, h3 { font-size: 1.3em !important; }
```

#### 1.5.3 安装 Style Settings 插件实现深度自定义

```
1. 社区插件 → 搜索「Style Settings」→ 安装并启用
2. 安装支持 Style Settings 的主题（如 Minimal、ITS Theme）
3. 设置 → Style Settings → 按需调整颜色、字体、间距等
```

### 1.6 社区插件精选推荐

#### 1.6.1 效率增强（必装）

| 插件 | 功能 | 说明 |
|------|------|------|
| **Templater** | 高级模板引擎 | 替代内置模板，支持动态变量、日期计算、脚本 |
| **Calendar** | 侧边栏日历 | 点击日期直接打开/创建日记 |
| **QuickAdd** | 快速捕获 | 一键创建笔记、执行宏命令、添加到看板 |
| **Dataview** | 数据查询 | 把仓库当数据库查询，生成动态列表/表格 |
| **Editing Toolbar** | 编辑工具栏 | 类似 Word 的格式工具栏，无需记快捷键 |
| **Recent Files** | 最近文件 | 快速跳转到最近编辑的笔记 |
| **Commander** | 自定义按钮 | 在标题栏、状态栏添加自定义按钮 |

#### 1.6.2 写作与排版

| 插件 | 功能 |
|------|------|
| **Easy Typing** | 优化中文输入体验，输入 Markdown 符号无需切换 |
| **Linter** | 自动规范笔记格式（YAML、标题层级、空行等） |
| **Better Word Count** | 增强字数统计，支持中文字数 |
| **Highlightr** | 更丰富的高亮颜色选择 |
| **Paste URL into selection** | 选中文本后粘贴 URL 自动生成链接 |
| **Note Refactor** | 一键拆分长笔记为多篇，保持链接 |
| **Floating TOC** | 悬浮目录导航 |

#### 1.6.3 可视化与思维

| 插件 | 功能 |
|------|------|
| **Excalidraw** | 手绘风格白板，画流程图、草图 |
| **Kanban** | 看板视图，管理任务与项目 |
| **Mind Map** | 将 Markdown 标题层级渲染为思维导图 |
| **Bases** | Obsidian 官方出品的多维表格（类似 Notion） |
| **Charts** | 在笔记中嵌入图表 |

#### 1.6.4 增强型功能

| 插件 | 功能 |
|------|------|
| **Omnisearch** | 增强全文搜索，支持中文分词、模糊搜索 |
| **Homepage** | 设置启动 Obsidian 时默认显示某篇笔记 |
| **Iconize** | 为文件和文件夹设置自定义图标 |
| **Hover Editor** | 在弹窗中编辑链接指向的笔记，不离开当前页面 |
| **Image Toolkit** | 点击图片放大预览 |
| **Quiet Outline** | 增强型大纲，支持搜索与拖拽排序 |
| **Another Quick Switcher** | 增强版 Cmd+O，支持最近文件、模糊搜索 |

### 1.7 创建模板系统

#### 1.7.1 日记模板

创建 `06-Templates/daily.md`：

```markdown
---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags: [日记]
type: daily
---

# 📅 <% tp.date.now("YYYY年M月D日 dddd") %>

## 🌅 今日目标
- [ ]

## 📝 笔记

## 📋 任务
- [ ]

## 🔗 相关笔记
-
```

#### 1.7.2 通用笔记模板

创建 `06-Templates/note.md`：

```markdown
---
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updated: <% tp.date.now("YYYY-MM-DD HH:mm") %>
tags: []
category:
---

# <% tp.file.title %>

## 摘要

## 正文

## 相关笔记
-

## 参考
-
```

#### 1.7.3 配置 Templater 插件

```
设置 → Templater：
  - Template folder location：06-Templates
  - 打开「Trigger Templater on new file creation」
  - 打开「Automatic jump to cursor」
  快捷键设置：
  - 为「Templater: Insert template」绑定 Cmd+Shift+T
  - 为「Templater: Create new note from template」绑定 Cmd+Alt+T
```

#### 1.7.4 配置 QuickAdd 快速捕获

```
设置 → QuickAdd：
  - 新建一个「Capture」类型的快捷入口
  - 目标文件：00-Inbox/快速笔记.md
  - 格式：- {{value}} （{{time}}）
  - 绑定快捷键：Cmd+Shift+N
  - 效果：随时随地弹出输入框，内容自动追加到快速笔记
```

### 1.8 Dataview 动态看板配置

安装 Dataview 插件后，可以创建强大的动态页面。

**创建主页仪表盘** `README.md`：

```markdown
# 🏠 知识库主页

## 📋 进行中的项目
```dataview
TABLE status, created
FROM "02-Projects"
WHERE status = "active"
SORT created DESC
```

## 📝 最近更新的笔记
```dataview
TABLE file.mtime as "更新时间"
FROM ""
WHERE file.name != "README"
SORT file.mtime DESC
LIMIT 10
```

## 🏷️ 标签统计
```dataview
TABLE length(rows) as "数量"
FROM ""
FLATTEN file.tags as tag
GROUP BY tag
SORT length(rows) DESC
```
```

设置 `README.md` 为主页：安装 Homepage 插件，设置为默认打开。

### 1.9 性能优化（macOS 专属）

#### 1.9.1 插件管理

```bash
# 查看 .obsidian 目录大小（插件和配置都在里面）
du -sh ~/Documents/Obsidian/MyKnowledge/.obsidian/
# 如果超过 200MB，说明插件过多或缓存过大
```

优化原则：
- 禁用至少 30 天未使用的插件
- 避免功能重叠的插件（如同时用多个搜索增强插件）
- 大型插件（Dataview、Excalidraw）按需启用

#### 1.9.2 CSS 性能优化

创建 `performance.css` 片段：

```css
/* 关闭不必要的动画，加快响应速度 */
body {
  --anim-duration-none: 0s;
  --anim-duration-superfast: 50ms;
  --anim-duration-fast: 100ms;
}

/* 减少过渡动画 */
.workspace-tab-header,
.sidebar-toggle-button {
  transition: none !important;
}

/* 大仓库下关闭图谱实时渲染 */
.graph-controls {
  display: none;
}
```

#### 1.9.3 硬件加速

在 macOS 上，Obsidian 基于 Electron，可以通过 flags 优化：

```bash
# 在终端中启动 Obsidian（需要先完全退出 Obsidian）
open -a Obsidian --args --enable-features=UseSkiaRenderer --enable-gpu-rasterization
```

或者在 `.zshrc` 中添加别名：

```bash
alias obsidian='open -a Obsidian --args --enable-features=UseSkiaRenderer --enable-gpu-rasterization --force_high_performance_gpu'
```

#### 1.9.4 附件管理

```bash
# 找出大于 5MB 的文件
find ~/Documents/Obsidian/ -type f -size +5M

# 图片压缩建议使用 ImageOptim (免费 macOS 工具)
brew install --cask imageoptim
```

### 1.10 本地备份方案（macOS）

#### 方案1：Time Machine（自动）

Obsidian 笔记是本地 Markdown 文件，只要文件在 `~/Documents/` 或 `~/` 下，Time Machine 默认就包含它们。

```
系统设置 → 通用 → Time Machine → 添加备份磁盘
```

#### 方案2：定时 Git 自动备份

创建备份脚本 `~/bin/obsidian-backup.sh`：

```bash
#!/bin/bash
VAULT_DIR="$HOME/Documents/Obsidian/MyKnowledge"
cd "$VAULT_DIR" || exit 1

# 检测是否有变更
if [[ -n $(git status --porcelain) ]]; then
    git add -A
    git commit -m "auto backup: $(date '+%Y-%m-%d %H:%M')"
    git push origin main
    echo "[$(date)] Backup completed" >> /tmp/obsidian-backup.log
else
    echo "[$(date)] No changes" >> /tmp/obsidian-backup.log
fi
```

```bash
chmod +x ~/bin/obsidian-backup.sh
```

设置 launchd 定时任务（每30分钟执行）：

```bash
cat > ~/Library/LaunchAgents/com.obsidian.backup.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.obsidian.backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/$USER/bin/obsidian-backup.sh</string>
    </array>
    <key>StartInterval</key>
    <integer>1800</integer>
    <key>RunAtLoad</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/obsidian-backup-launchd.log</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.obsidian.backup.plist
```

#### 方案3：rsync 到外置硬盘

```bash
# 手动备份
rsync -av --delete ~/Documents/Obsidian/ /Volumes/BackupDisk/Obsidian/

# 或者创建别名
alias backup-obsidian='rsync -av --delete ~/Documents/Obsidian/ /Volumes/BackupDisk/Obsidian/ && echo "✅ Obsidian 备份完成"'
```

### 1.11 与 macOS 生态集成

#### 1.11.1 快捷指令（Shortcuts）集成

Obsidian 提供了丰富的 macOS 快捷指令支持：

- 在快捷指令 App 中搜索「Obsidian」查看可用操作
- 常用快捷指令模板：
  - 「今日日记」→ 一键打开今日日记
  - 「快速捕获」→ 弹出窗口，输入内容追加到 Inbox
  - 「剪贴板到笔记」→ 将剪贴板内容保存为新笔记

#### 1.11.2 Alfred / Raycast 集成

**Raycast 集成：**

```bash
# 安装 Raycast Obsidian 插件
# Raycast 商店搜索「Obsidian」→ 安装
# 支持功能：
# - 快速搜索笔记
# - 创建新笔记
# - 打开每日笔记
# - 快速追加内容
```

#### 1.11.3 终端快速访问

在 `.zshrc` 中添加：

```bash
# Obsidian 快速打开仓库
alias ov='open -a Obsidian ~/Documents/Obsidian/MyKnowledge'

# 快速创建笔记
function onote() {
    echo "---\ncreated: $(date '+%Y-%m-%d %H:%M')\ntags: []\n---\n\n# $1\n" > ~/Documents/Obsidian/MyKnowledge/00-Inbox/"$1.md"
    open -a Obsidian ~/Documents/Obsidian/MyKnowledge/00-Inbox/"$1.md"
}

# 使用：onote 会议记录-20260115
```

#### 1.11.4 截图快速粘贴

macOS 截图的默认保存位置可以设置为 Obsidian 附件目录：

```bash
# 将截图直接保存到附件目录
defaults write com.apple.screencapture location ~/Documents/Obsidian/MyKnowledge/07-Attachments/
killall SystemUIServer

# 截图快捷键：
# Cmd+Shift+3     全屏截图
# Cmd+Shift+4     区域截图
# Cmd+Shift+5     录屏
```

### 1.12 macOS 常见问题

**Q：Obsidian 占用内存过大（>2GB）？**
```bash
# 禁用硬件加速（减少内存但可能影响滚动流畅度）
open -a Obsidian --args --disable-gpu
# 或减少打开的标签页数量，关闭不用的分屏
```

**Q：iCloud 同步导致冲突文件？**
```bash
# 关闭 iCloud 的「优化 Mac 存储空间」
# 系统设置 → iCloud → 关闭「优化 Mac 存储空间」
# 确保笔记文件始终在本地可用
```

**Q：Spotlight 搜不到 Obsidian 笔记内容？**
```
Obsidian 笔记是纯 .md 文件，Spotlight 默认支持搜索内容。
如果搜不到：
  系统设置 → Siri 与聚焦 → 聚焦隐私 → 确保 Obsidian 目录不在排除列表中
```

---

## 二、Windows 本地完整部署方案

> 目标：在 Windows 系统上打造高性能的个人知识管理系统。

### 2.1 安装前环境检查

```powershell
# 检查 Windows 版本（需要 Win10 1809+ 或 Win11）
winver

# 检查 PowerShell 版本
$PSVersionTable.PSVersion

# 检查可用磁盘空间
Get-PSDrive C | Select-Object Used,Free

# 如果要用 Git 同步，先安装 Git
# 下载地址：https://git-scm.com/download/win
# 安装后验证：
git --version
```

### 2.2 安装 Obsidian

**方式一：官网安装包（推荐）**

1. 打开 https://obsidian.md/download
2. 下载 Windows 版 `.exe` 安装包
3. 双击运行，选择「**仅为我安装**」（不要选管理员安装，权限会有问题）
4. **安装路径强烈建议改为非 C 盘**：
   ```
   推荐：D:\Apps\Obsidian\
   不推荐：C:\Program Files\Obsidian\  （C盘空间紧张）
   ```
5. 勾选「创建桌面快捷方式」
6. 点击「安装」，等待完成

**方式二：winget（Windows 11）**

```powershell
winget install Obsidian.Obsidian
```

**方式三：便携版（U 盘随身携带）**

1. 官网下载 `.zip` 便携版
2. 解压到任意目录（如 `D:\Obsidian\`）
3. 双击 `Obsidian.exe` 直接使用
4. 所有配置和插件都保存在解压目录内，拷贝整个目录即可迁移

### 2.3 创建知识库（Vault）

```
步骤1：启动 Obsidian → 点击「Create new vault」
步骤2：设置仓库名称，如「MyKnowledge」
步骤3：选择存储路径，强烈推荐：
       ✅ D:\ObsidianVaults\MyKnowledge\   # 非系统盘
       ✅ D:\OneDrive\Obsidian\              # OneDrive 同步
       ✅ D:\坚果云\Obsidian\                # 坚果云同步
       ❌ C:\Users\用户名\Downloads\           # 不推荐
       ❌ C:\Program Files\                   # 无写入权限
```

**推荐的目录规划：**

```
D:\ObsidianVaults\MyKnowledge\
├── 00-Inbox\              # 待处理的暂存笔记
├── 01-Daily\              # 每日日记
├── 02-Projects\           # 项目相关笔记
├── 03-Areas\              # 长期关注的领域
├── 04-Resources\          # 参考资料
├── 05-Archive\            # 归档笔记
├── 06-Templates\          # 模板文件
├── 07-Attachments\        # 附件（图片、PDF等）
│   └── images\            # 图片子目录
└── README.md              # 仓库说明（可设为主页）
```

### 2.4 基础设置（必须完成）

#### 2.4.1 语言切换

设置 → 关于（About）→ 语言 → 选择「简体中文」→ 点击按钮重启

#### 2.4.2 编辑器设置

```
设置 → 编辑器：
  - 默认编辑模式：实时预览（Live Preview）
  - 显示行号：开启
  - 缩进：4 个空格
  - 默认换行符：Windows 换行符（CRLF）→ 如果跨平台同步建议改为 LF
  - 拼写检查：根据需要开启/关闭（中文用户一般关闭）
  - 可折叠缩进：开启
  - 显示缩进参考线：开启
```

**重要提示：** 如果和 macOS/Linux 跨平台同步，强烈建议将默认换行符改为 LF：
```
设置 → 编辑器 → 默认换行符 → LF
```

#### 2.4.3 文件与链接

```
设置 → 文件与链接：
  - 新建笔记的存放位置：与当前笔记相同的文件夹
  - 内部链接类型：最短路径
  - 使用 [[Wikilinks]]：开启
  - 新建笔记的默认文件夹：00-Inbox
  - 附件的默认存放路径：07-Attachments
  - 子文件夹：images/${filename}
  - 检测所有文件扩展名：开启（包括 .png, .pdf, .docx 等）
```

#### 2.4.4 外观设置

```
设置 → 外观：
  - 基础颜色：「暗色」适合夜间使用，「浅色」适合白天（可设置跟随系统）
  - 字体大小：推荐 16-18px（Windows 字体渲染比 macOS 小，需要稍大一些）
  - 缩小行宽：开启
  - 窗口框架：推荐「Obsidian 框架」
  - 快速调整字体大小：Ctrl + 鼠标滚轮
  - 界面缩放：100%（高 DPI 屏幕可能需要调整）
```

**Windows 字体渲染优化：**

如果觉得字体模糊，确保开启 ClearType：
```
设置 → 搜索「调整 ClearType 文本」→ 按照向导优化
```

#### 2.4.5 核心插件启用

同 macOS 的配置（见 1.4.5 节），所有核心插件的功能是一致的。

#### 2.4.6 快捷键自定义

Windows 上的快捷键把 `Cmd` 替换为 `Ctrl`：

| 功能 | 推荐快捷键 |
|------|-----------|
| 快速切换 | `Ctrl + O` |
| 命令面板 | `Ctrl + P` |
| 切换编辑/预览 | `Ctrl + E` |
| 打开今日日记 | `Ctrl + Shift + D` |
| 全局搜索 | `Ctrl + Shift + F` |
| 插入模板 | `Ctrl + Shift + T` |
| 切换左侧边栏 | `Ctrl + Shift + L` |
| 切换右侧边栏 | `Ctrl + Shift + R` |

### 2.5 主题安装与美化

#### 2.5.1 安装主题

流程同 macOS（见 1.5.1 节），推荐的 Windows 适配主题：

| 主题名称 | 特点 | Windows 适配 |
|---------|------|-------------|
| **Minimal** | 极简可自定义 | 优秀，配合 Style Settings |
| **Blue Topaz** | 国内最流行，功能全面 | 极好，中文优化好 |
| **ITS Theme** | 现代感强 | 优秀 |
| **Border** | 带边框设计 | 优秀 |
| **Prism** | 代码高亮优秀 | 极好，适合技术人员 |

#### 2.5.2 Windows 中文字体优化

创建 CSS 片段 `chinese-font.css`：

```css
/* Windows 中文字体优化 */
body {
  --font-text-theme: "Microsoft YaHei UI", "微软雅黑", "Segoe UI",
    -apple-system, BlinkMacSystemFont, sans-serif;
  --font-interface-theme: "Microsoft YaHei UI", "微软雅黑", "Segoe UI",
    -apple-system, BlinkMacSystemFont, sans-serif;
  --font-monospace-theme: "Cascadia Code", "JetBrains Mono", "Fira Code",
    "Consolas", "Sarasa Mono SC", monospace;
}

/* Windows 字体渲染优化 */
body {
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
}

/* 行间距（Windows 字体偏大，需要加大间距） */
.markdown-source-view, .markdown-reading-view {
  --line-height-normal: 1.8;
}
```

**推荐安装的等宽字体：**
```
下载并安装 Sarasa Mono SC（更纱黑体）：
  https://github.com/be5invis/Sarasa-Gothic/releases
  下载 sarasa-mono-sc-*.zip，安装 .ttf 文件

或者 JetBrains Mono：
  https://www.jetbrains.com/lp/mono/
```

### 2.6 社区插件精选推荐

#### 2.6.1 同步推荐

相同的社区插件列表见 1.6 节，这里补充 Windows 专属推荐：

| 插件 | 说明 |
|------|------|
| **坚果云 Nutstore Sync** | 国内用户首选同步方案，零配置，一键单点登录 |
| **Obsidian Git** | Windows 需要安装完整版 Git（不能用 GitHub Desktop 替代） |

### 2.7 创建模板系统

模板的创建与 macOS 完全相同（见 1.7 节），Windows 上的 Templater 配置步骤一致。

唯一差异：文件路径使用反斜杠 `\`：
```
Templater → Template folder location：06-Templates
快捷键：Ctrl+Shift+T
```

### 2.8 Dataview 动态看板配置

配置完全与 macOS 相同（见 1.8 节）。

### 2.9 性能优化（Windows 专属）

#### 2.9.1 插件管理

```
优化原则：
  - 已安装插件数量建议 < 30 个（超过会影响启动速度）
  - 禁用超过 30 天未使用的插件
  - 分场景启用插件组（可以用 Commander 插件的宏命令切换）
```

#### 2.9.2 Windows 性能专属优化

**关闭 Windows Defender 对 Obsidian 目录的实时扫描（可选，提升 I/O 性能）：**

```powershell
# 以管理员身份运行 PowerShell，添加排除项
Add-MpPreference -ExclusionPath "D:\ObsidianVaults"
Add-MpPreference -ExclusionPath "D:\Apps\Obsidian"
```

**SSD 优化：**
```
如果你的笔记目录在 SSD 上，确保 TRIM 已开启：
  PowerShell(管理员)：Optimize-Volume -DriveLetter D -ReTrim -Verbose
```

**关闭不必要的启动项：**
```
Ctrl+Shift+Esc 打开任务管理器 → 启动 → 禁用不需要的程序
```

#### 2.9.3 使用 Obsidian 调试工具检查性能

```powershell
# Ctrl+Shift+I 打开开发者工具
# 切换到 Console 标签页
# 查看启动耗时和插件加载时间
# 对于加载时间超过 500ms 的插件，考虑将其禁用
```

### 2.10 开机自启动配置

#### 方式一：启动文件夹（最简单）

```powershell
# Win+R → 输入 shell:startup → 回车
# 将 Obsidian 快捷方式复制到打开的文件夹中
```

#### 方式二：任务计划程序（更灵活）

```powershell
# 创建开机后延迟 10 秒自动启动 Obsidian 的任务
$Action = New-ScheduledTaskAction -Execute "D:\Apps\Obsidian\Obsidian.exe"
$Trigger = New-ScheduledTaskTrigger -AtLogon
$Settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable

Register-ScheduledTask -TaskName "Obsidian Auto Start" `
    -Action $Action -Trigger $Trigger -Settings $Settings `
    -Description "开机自动启动 Obsidian"
```

#### 方式三：注册表（高级）

```powershell
# 通过注册表添加开机自启
$obsidianPath = "D:\Apps\Obsidian\Obsidian.exe"
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
    -Name "Obsidian" -Value $obsidianPath
```

### 2.11 本地备份方案（Windows）

#### 方案1：文件历史记录

```
设置 → 更新和安全 → 备份 → 使用文件历史记录进行备份
→ 添加驱动器 → 选择外置硬盘
→ 更多选项 → 添加文件夹 → D:\ObsidianVaults
→ 备份频率：每30分钟
→ 保留的版本：永远
```

#### 方案2：定时 Git 备份 + 任务计划

创建备份脚本 `D:\Scripts\obsidian-backup.ps1`：

```powershell
# D:\Scripts\obsidian-backup.ps1
$vaultDir = "D:\ObsidianVaults\MyKnowledge"
Set-Location $vaultDir

$status = git status --porcelain
if ($status) {
    git add -A
    git commit -m "auto backup: $(Get-Date -Format 'yyyy-MM-dd HH:mm')"
    git push origin main
    Add-Content "D:\Scripts\obsidian-backup.log" "[$(Get-Date)] Backup completed"
} else {
    Add-Content "D:\Scripts\obsidian-backup.log" "[$(Get-Date)] No changes"
}
```

```powershell
# 创建任务计划（每30分钟执行一次）
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -WindowStyle Hidden -File D:\Scripts\obsidian-backup.ps1"
$Trigger = New-ScheduledTaskTrigger -Once -At "00:00" -RepetitionInterval (New-TimeSpan -Minutes 30)
$Principal = New-ScheduledTaskPrincipal -UserId "$env:USERNAME" -LogonType Interactive

Register-ScheduledTask -TaskName "Obsidian Git Backup" `
    -Action $Action -Trigger $Trigger -Principal $Principal `
    -Description "每30分钟自动备份 Obsidian 到 Git"
```

#### 方案3：Robocopy 增量备份到外置硬盘

```powershell
# 增量备份脚本
robocopy "D:\ObsidianVaults" "E:\Backup\ObsidianVaults" /MIR /R:3 /W:10 /NP /LOG:D:\Scripts\robocopy-obsidian.log

# 参数说明：
# /MIR - 镜像复制（源目录完全同步到目标，会删除目标中多余文件）
# /R:3 - 失败重试 3 次
# /W:10 - 重试间隔 10 秒
# /NP - 不显示进度百分比
# /LOG - 记录日志
```

### 2.12 与 Windows 生态集成

#### 2.12.1 PowerToys Run 集成

安装 PowerToys（Windows 官方效率工具集）后：

```
PowerToys Run（Alt+Space）可以搜索文件
如果你的 Obsidian 笔记在索引路径中，可以直接搜索 .md 文件名
```

#### 2.12.2 AutoHotkey 快捷操作

创建 `obsidian-hotkey.ahk` 脚本：

```autohotkey
; Ctrl+Shift+O 快速打开 Obsidian
^+o::
    Run "D:\Apps\Obsidian\Obsidian.exe"
    return

; Win+N 快速捕获想法（需要配合 QuickAdd 插件）
#n::
    Run "D:\Apps\Obsidian\Obsidian.exe"
    WinWaitActive "Obsidian"
    Send "^+n"  ; 假设 QuickAdd 绑定了 Ctrl+Shift+N
    return

; 在任何应用中，选中文本按 Win+Shift+O 追加到 Obsidian 日记
#+o::
    Send "^c"
    Sleep 100
    Run "D:\Apps\Obsidian\Obsidian.exe"
    WinWaitActive "Obsidian"
    Send "^+d"  ; 打开今日日记
    Send "^v"
    return
```

设置开机自启：
```
Win+R → shell:startup → 放入 obsidian-hotkey.ahk 的快捷方式
```

#### 2.12.3 终端快速访问

在 PowerShell Profile 中添加：

```powershell
# 编辑 $PROFILE
notepad $PROFILE

# 添加以下内容：
function Open-Obsidian {
    Start-Process "D:\Apps\Obsidian\Obsidian.exe"
}
Set-Alias ov Open-Obsidian

function New-ObsidianNote {
    param([string]$Name)
    $path = "D:\ObsidianVaults\MyKnowledge\00-Inbox\$Name.md"
    "---`ncreated: $(Get-Date -Format 'yyyy-MM-dd HH:mm')`ntags: []`n---`n`n# $Name`n" | Out-File -FilePath $path -Encoding UTF8
    Start-Process "D:\Apps\Obsidian\Obsidian.exe" -ArgumentList $path
}
Set-Alias onote New-ObsidianNote
```

### 2.13 Windows 常见问题

**Q：Obsidian 启动很慢（>15秒）？**
```
解决方案：
1. 减少启用插件数量到 20 个以内
2. 在 Windows Defender 中添加 Obsidian 目录排除
3. 检查 .obsidian 目录大小，如果超过 300MB，清理旧版本插件备份
4. 确保仓库所在磁盘是 SSD 而非 HDD
```

**Q：OneDrive 同步导致文件冲突？**
```
解决：
1. 确保 OneDrive 文件夹设置为「始终保留在此设备上」
2. 关闭 Obsidian 后再关机/重启，让 OneDrive 完成同步
3. 避免两台电脑同时编辑同一笔记
```

**Q：输入中文时标点符号异常？**
```
安装 Easy Typing 插件后可以自动处理中英文之间的空格和标点
同时建议：
  设置 → 编辑器 → 使用 Markdown 格式的链接：关闭
  这样可以避免中文输入状态下输入 ] 和 ) 的混乱
```

**Q：内存占用过高？**
```
Windows 上 Obsidian 通常占用 300-800MB 内存
如果超过 1.5GB：
  1. 检查是否有超大笔记文件（>5000行）
  2. 关闭不用的插件
  3. 减少同时打开的标签页
  4. 关闭图谱视图（占用大量内存）
```

---

## 三、多设备同步方案

### 方案对比总览

| 方案 | 费用 | 难度 | macOS | Windows | iOS | Android | 实时性 |
|------|------|------|-------|---------|-----|---------|--------|
| iCloud | 免费(5GB) | 极低 | ✅ 原生 | ✅ | ✅ | ❌ | 实时 |
| OneDrive | 免费(5GB) | 极低 | ✅ | ✅ 原生 | ✅ | ✅ | 实时 |
| 坚果云 Nutstore Sync | 免费 | 低 | ✅ | ✅ | ✅ | ✅ | 实时 |
| 官方 Obsidian Sync | $5/月 | 低 | ✅ | ✅ | ✅ | ✅ | 实时 |
| Self-hosted LiveSync | 免费(需服务器) | 中高 | ✅ | ✅ | ✅ | ✅ | 实时 |
| Remotely Save + WebDAV | 免费 | 低 | ✅ | ✅ | ✅ | ✅ | 定时 |
| Git 同步 | 免费 | 中 | ✅ | ✅ | 受限 | ✅ | 手动 |

### 方案A：官方 Obsidian Sync

```
适合：不想折腾，追求极致稳定
费用：$5/月（或 $48/年）
优势：零配置、端到端加密、移动端完美支持
```

**步骤：**

1. Obsidian → 设置 → 核心插件 → 启用「Sync」
2. 注册/登录 Obsidian 账号
3. 选择「创建远程仓库」→ 设置密码（端到端加密）
4. 在其他设备上登录同一账号 → 选择同一远程仓库
5. 输入加密密码 → 自动开始同步

### 方案B：Self-hosted LiveSync（自建 CouchDB）

```
适合：极客用户，追求数据完全自主可控
要求：一台有公网 IP 的服务器，需要配置 HTTPS
```

#### 步骤1：部署 CouchDB

```bash
mkdir -p ~/obsidian-sync/{data,config}
cd ~/obsidian-sync
```

**docker-compose.yml：**

```yaml
services:
  couchdb:
    image: apache/couchdb:3.4.2
    container_name: obsidian-couchdb
    restart: unless-stopped
    ports:
      - "5984:5984"
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=your-strong-password-here
    volumes:
      - ./data:/opt/couchdb/data
      - ./config:/opt/couchdb/etc/local.d
```

**config/local.ini：**

```ini
[couchdb]
single_node=true

[chttpd]
require_valid_user = true

[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html

[httpd]
WWW-Authenticate = Basic realm="couchdb"
enable_cors = true

[cors]
origins = app://obsidian.md,capacitor://localhost,http://localhost
credentials = true
headers = accept, authorization, content-type, origin, referer
methods = GET, PUT, POST, HEAD, DELETE
```

```bash
docker compose up -d
```

#### 步骤2：配置 HTTPS 反向代理（必须！）

用 Caddy 最简单：

```bash
sudo tee /etc/caddy/Caddyfile <<EOF
couchdb.yourdomain.com {
    reverse_proxy localhost:5984
}
EOF

sudo systemctl restart caddy
```

#### 步骤3：配置 Obsidian 客户端

1. 安装「Self-hosted LiveSync」插件
2. Setup wizard → 配置远程数据库
3. 填入：`https://couchdb.yourdomain.com`、用户名、密码
4. 设置端到端加密密码 → 复制设置 URI
5. 其他设备通过 URI / QR 码一键配置

### 方案C：Remotely Save + 坚果云 WebDAV（推荐国内用户）

```
适合：国内用户，零成本稳定同步
优势：坚果云增量同步技术成熟，支持所有平台
```

**步骤：**

1. 注册坚果云 → 账户信息 → 安全选项 → 第三方应用管理 → 添加应用
2. 生成应用密码（立即复制，只显示一次）
3. Obsidian 安装「Remotely Save」插件
4. 选择 WebDAV：
   - 地址：`https://dav.jianguoyun.com/dav/你的文件夹名`
   - 用户名：坚果云账号（邮箱）
   - 密码：应用密码
5. 点击「检查」验证 → 设置自动同步间隔 5 分钟
6. 手机端扫描电脑端 QR 码完成配置

### 方案D：Git 同步方案

```
适合：技术人员，需要版本历史
要求：安装 Git，注册 GitHub/Gitee 账号
```

**步骤：**

1. 创建 GitHub 私有仓库，生成 Personal Access Token
2. 在 Obsidian 仓库中 `git init`，创建 `.gitignore`：
   ```
   .obsidian/workspace.json
   .obsidian/workspace-mobile.json
   .trash/
   .DS_Store
   Thumbs.db
   ```
3. 安装「Obsidian Git」插件
4. 设置自动备份间隔、自动拉取等
5. 手机端（Android）用 GitSync App 同步
6. iOS 端用 Working Copy 或 iSH 同步

---

## 四、笔记发布为网站

### 方案A：Obsidian Publish（官方）

```
费用：$8/月（按站点）
步骤：
1. 启用 Publish 核心插件
2. 创建站点，设置站点 ID
3. 选择要发布的笔记
4. 可选：绑定自定义域名
```

### 方案B：Quartz + GitHub Pages（免费推荐）

```
适合：想要免费发布完整知识库到公网
优势：完美支持双向链接、图谱、Callout、标签
```

**步骤1：Fork Quartz 仓库**

访问 https://github.com/jackyzha0/quartz → Use this template

```bash
git clone https://github.com/你的用户名/quartz.git
cd quartz
npm install
```

**步骤2：配置**

编辑 `quartz.config.ts`：
```typescript
pageTitle: "我的数字花园",
baseUrl: "你的用户名.github.io",
```

**步骤3：放入笔记并部署**

```bash
cp -r ~/Documents/Obsidian/MyKnowledge/* content/
npx quartz build --serve  # 本地预览
git add -A && git commit -m "deploy" && git push
# GitHub Actions 会自动部署
```

### 方案C：Perlite（Docker 一键部署）

```bash
docker run -d \
  --name perlite \
  -p 8080:80 \
  -v /path/to/obsidian/vault:/var/www/perlite/content \
  sec77/perlite:latest
```

---

## 五、Docker 容器化部署（浏览器访问）

> 适用场景：在 VPS/NAS 上部署，通过浏览器随时随地访问 Obsidian。

### 5.1 使用 LinuxServer.io 官方镜像

```bash
mkdir -p ~/obsidian-docker && cd ~/obsidian-docker
```

**docker-compose.yml：**

```yaml
version: "3.8"
services:
  obsidian:
    image: lscr.io/linuxserver/obsidian:latest
    container_name: obsidian
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - CUSTOM_USER=admin
      - PASSWORD=your_strong_password
    ports:
      - "3000:3000"
      - "3001:3001"
    volumes:
      - ./config:/config
      - ./vault:/vault
    shm_size: "2gb"
```

```bash
docker compose up -d
# 浏览器访问 http://服务器IP:3000
```

### 5.2 配置 HTTPS

使用 Caddy：

```bash
sudo tee /etc/caddy/Caddyfile <<EOF
obsidian.yourdomain.com {
    reverse_proxy localhost:3000
}
EOF
sudo systemctl restart caddy
```

---

## 六、安全注意事项

1. **端到端加密**：使用 Sync / LiveSync 时务必开启端到端加密，密码不要泄露
2. **HTTPS 必须**：LiveSync 的 CouchDB 必须配置 HTTPS，密码使用 16 位以上随机强密码
3. **Token 安全**：Git Token / API Key 不要提交到仓库中，不要分享给他人
4. **.gitignore 检查**：确保 `.obsidian/workspace.json`、`.trash/`、`.DS_Store`、`Thumbs.db` 不进入版本控制
5. **防火墙**：Docker 部署时只开放必要端口，配合 fail2ban 防护
6. **定期备份**：无论用哪种同步方案，保留额外的本地离线备份（如每周 rsync 到外置硬盘）
7. **敏感信息**：包含密码、API Key 的笔记建议用 `.excalidraw` 或加密方式存储
8. **云盘同步注意**：不要把 Obsidian 仓库直接放在云盘的「在线文件」（如 OneDrive Files On-Demand），确保文件「始终保留在此设备上」

---

## 附录：推荐组合方案速查

### 学生 / 个人用户（零成本）

```
macOS / Windows 本地安装
+ iCloud / OneDrive 免费同步（电脑间）
+ Remotely Save + 坚果云 WebDAV（手机端）
+ Obsidian Git + GitHub 私有仓库（版本备份）
+ Quartz + GitHub Pages（发布网站）
```

### 技术爱好者（完全自主）

```
macOS / Windows 本地安装
+ Self-hosted LiveSync（自建服务器同步）
+ CouchDB 自动备份脚本
+ Quartz + 自有域名 + Cloudflare Pages（发布）
```

### 远程办公 / 随时随地访问

```
VPS 上 Docker 部署 LinuxServer.io Obsidian
+ Nginx/Caddy 反向代理 + HTTPS
+ Self-hosted LiveSync（同步到本地设备）
+ 浏览器 → 随时随地打开使用
```

---

## 七、Obsidian + AI Agent Skill 深度集成

> 核心目标：让 AI Agent（Claude Code / Codex CLI / 自定义 Skill）能够读取、搜索、引用你的 Obsidian 知识库，在自动化测试设计、代码审查、问题排查等场景中，给出基于真实经验积累的精准建议。

### 7.1 整体架构

```
┌──────────────────────────────────────────────────────────────┐
│                   AI Agent (Claude Code)                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐               │
│  │ Test     │  │ Code     │  │ Bug Analysis  │  更多Skills   │
│  │ Design   │  │ Review   │  │ Skill         │               │
│  │ Skill    │  │ Skill    │  │               │               │
│  └────┬─────┘  └────┬─────┘  └───────┬───────┘               │
│       │              │               │                         │
│       └──────────────┼───────────────┘                         │
│                      │                                         │
│              ┌───────▼────────┐                                │
│              │  知识库检索层   │                                │
│              │ grep/Dataview  │                                │
│              │ Smart Connect  │                                │
│              │ MCP REST API   │                                │
│              └───────┬────────┘                                │
└──────────────────────┼────────────────────────────────────────┘
                       │ 读取 / 搜索
┌──────────────────────▼────────────────────────────────────────┐
│                   Obsidian Vault（本地知识库）                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ 测试经验库 │  │ 测试因子库 │  │ 业务规则库 │  │ 缺陷模式库 │      │
│  │experiences│  │ factors  │  │ business │  │ bug-patterns│    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
└──────────────────────────────────────────────────────────────┘
```

**核心原理：** Obsidian 笔记就是本地 Markdown 文件。AI Agent 运行在你的电脑上，可以直接通过文件系统读取 `.md` 文件，也可以借助 grep/ripgrep 搜索、Dataview 结构化查询、Smart Connections 语义搜索等方式，在对话中引用知识库内容。

### 7.2 集成方案对比（三种方案按需选择）

| 方案 | 原理 | 复杂度 | 搜索方式 | 适用场景 |
|------|------|--------|---------|---------|
| **A. 文件系统直读** | Agent 用 Read/Grep 直接读 .md 文件 | 极低 | 关键词匹配 (ripgrep) | 入门、小仓库 |
| **B. MCP + REST API** | Obsidian 插件暴露 HTTP API + 语义搜索 | 中 | 关键词 + 语义搜索 | 大仓库、需智能检索 |
| **C. obsidian-skills** | kepano 开源 Skills，教 AI 正确读写 Obsidian 格式 | 低 | 配合 A/B 使用 | 语法正确性、Canvas/Bases |

**推荐策略：方案A（入门必选）+ 方案C（强烈推荐）→ 方案B（仓库超过500篇笔记时再上）**

### 7.3 知识库结构设计（测试场景）

#### 7.3.1 推荐目录结构

```
Testing-Knowledge-Vault/
├── 00-Inbox/                    # 待整理的经验碎片
├── 01-Experiences/              # 测试经验库（核心）
│   ├── 功能测试/
│   │   ├── 登录模块测试经验.md
│   │   ├── 支付流程测试经验.md
│   │   └── 表单验证测试经验.md
│   ├── 性能测试/
│   ├── 安全测试/
│   └── 兼容性测试/
├── 02-Factors/                  # 测试因子库（核心）
│   ├── 输入因子/                 # 等价类、边界值
│   ├── 环境因子/                 # 系统、网络、硬件
│   ├── 状态因子/                 # 状态机、生命周期
│   └── 组合因子/                 # 正交/因果组合
├── 03-Business-Rules/           # 业务规则库
├── 04-Bug-Patterns/             # 缺陷模式库
├── 05-Test-Methods/             # 测试方法论
├── 06-Cases/                    # 经典测试用例库
├── 07-Templates/                # 模板文件
└── 08-MOCs/                     # 索引/地图（Map of Content）
```

#### 7.3.2 Frontmatter 元数据规范

Agent 搜索知识库主要依赖**文件名**和 **Frontmatter 属性**。规范的元数据是精准检索的关键。

**测试经验笔记 Frontmatter：**

```yaml
---
type: testing-experience
module: "订单退款"         # 被测模块
submodule: "退款审批流"    # 子模块
test-type: "功能测试"      # 功能/性能/安全/兼容性
test-method: "状态迁移"    # 等价类/边界值/因果图/正交
priority: "P0"            # P0/P1/P2/P3
severity: ""              # 缺陷严重度
tags: [退款, 状态机, 幂等]
source: "电商平台v3.2"     # 来源项目
date: "2026-05-23"
status: "reviewed"        # draft/reviewed/published
related-factors: [验证码因子, 金额计算因子]
related-bugs: [BUG-4201, BUG-4205]
---
```

**因子条目 Frontmatter：**

```yaml
---
type: test-factor
category: "输入"           # 输入/环境/状态/组合
subcategory: "字符串"      # 字符串/数值/日期/网络...
factor-name: "用户名输入"
test-methods: [等价类划分, 边界值分析]
related-experiences: [登录模块测试经验]
related-bugs: [BUG-3088]
tags: [输入校验, SQL注入, Unicode]
risk-level: "high"
---
```

#### 7.3.3 模板文件

**测试经验模板** (`07-Templates/测试经验模板.md`)：

```markdown
---
type: testing-experience
module: ""
submodule: ""
test-type: ""
test-method: ""
priority: ""
severity: ""
tags: []
source: ""
date: ""
status: draft
related-factors: []
related-bugs: []
---

# {{title}}

## 场景描述
> 被测功能或场景的简要描述

## 前置条件
-

## 测试要点
### 关键检查点
- [ ]

### 易遗漏边界
- [ ]

### 异常路径
- [ ]

## 实际踩过的坑
### 问题描述
### 根因
### 解决方案
### 预防措施

## 关联因子
## 关联缺陷模式
```

**因子条目模板** (`07-Templates/因子条目模板.md`)：

```markdown
---
type: test-factor
category: ""
subcategory: ""
factor-name: ""
test-methods: []
risk-level: "medium"
tags: []
---

# {{title}}

## 因子定义
-

## 覆盖维度
### 有效等价类
| 类别 | 代表值 | 说明 |
|------|--------|------|
|      |        |      |

### 无效等价类
| 类别 | 代表值 | 预期结果 |
|------|--------|----------|
|      |        |          |

### 边界值
| 边界 | 测试值 | 预期结果 |
|------|--------|----------|
| 上点 |        |          |
| 离点 |        |          |

## 组合策略
## 真实缺陷案例
```

**业务规则模板** (`07-Templates/业务规则模板.md`)：

```markdown
---
type: business-rule
domain: ""            # 业务域：电商/金融/社交/医疗/物流...
module: ""            # 所属模块
rule-id: ""           # 规则编号（如 BR-ORDER-001）
rule-category: ""     # 计算规则/校验规则/流程规则/权限规则/时间规则
priority: ""          # P0(违反即阻断)/P1(影响核心流程)/P2(影响体验)
scope: ""             # 适用范围：全局/模块/场景
trigger-condition: "" # 触发条件简述
tags: []
related-experiences: []   # [[关联的测试经验]]
related-factors: []       # [[关联的测试因子]]
related-bug-patterns: []  # [[关联的缺陷模式]]
source: ""            # 来源：PRD v2.3 / 接口文档 / 行业标准
date: ""
status: "draft"       # draft / verified / deprecated
deprecated-reason: "" # 如已废弃，说明原因
---

# {{title}}

## 规则描述
> 一句话描述这个规则是什么

## 规则详情

### 输入条件
| 条件项 | 说明 | 取值/范围 |
|--------|------|-----------|
|        |      |           |

### 处理逻辑
```
用伪代码或流程图描述处理过程
```

### 输出结果
| 场景 | 预期结果 | 错误码 |
|------|---------|--------|
| 正常 |         |        |
| 异常 |         |        |

## 关联约束
> 此规则与其他规则的依赖或互斥关系，如「规则 A 在规则 B 之前执行」

## 验证方法
### 正向验证
- [ ]

### 反向验证
- [ ]

### 边界验证
- [ ]

## 历史变更
| 日期 | 变更内容 | 变更原因 | 变更人 |
|------|---------|---------|--------|
|      |         |         |        |

## 已知缺陷
- [[BUG-xxxx]] - 简述
```

**规则输入条件举例**：支付金额、用户等级、商品类型、优惠券类型、时间窗口等。

**rule-category 分类说明**：

| 类型 | 说明 | 举例 |
|------|------|------|
| `计算规则` | 涉及金额、积分、扣减等计算 | 满减优惠叠加计算规则 |
| `校验规则` | 输入的合法性校验 | 手机号格式/身份证校验 |
| `流程规则` | 状态流转、审批链 | 退款状态机流转规则 |
| `权限规则` | 角色权限、数据可见性 | 普通用户不能查看管理后台 |
| `时间规则` | 时效性约束 | 未支付订单 30 分钟自动取消 |

**已废弃规则的 Frontmatter 处理**：不要删除笔记，改为 `status: deprecated` 并填写 `deprecated-reason`，保留历史可追溯性。

---

**缺陷模式模板** (`07-Templates/缺陷模式模板.md`)：

```markdown
---
type: bug-pattern
pattern-name: ""      # 模式名称
pattern-id: ""        # 模式编号（如 BUG-PTN-001）
category: ""          # 并发/数据一致性/边界/空值/类型转换/状态机/性能/安全/UI/集成
severity: ""          # P0/P1/P2/P3（此模式通常导致的缺陷严重度）
frequency: ""         # 出现频率：高频/中频/低频/偶发
detection-difficulty: "" # 发现难度：极易/容易/中等/困难/极易遗漏
tags: []
programming-languages: []  # 易出现的语言/框架
tech-stack: []        # 相关技术栈：Redis/MySQL/Kafka/...
related-experiences: []    # [[关联的测试经验]]
related-factors: []        # [[关联的测试因子]]
related-rules: []          # [[关联的业务规则]]
date: ""
status: "draft"
---

# {{title}}

## 一句话描述
> 用一句话描述这个缺陷模式（方便 Agent 搜索命中）

## 典型症状
> 出现这个问题时，用户/测试人员能看到什么现象？
- [ ]

## 根因分析
### 直接原因

### 深层原因

## 触发条件
> 什么条件下会触发此 Bug？尽量列出完整的条件组合

| 条件项 | 说明 |
|--------|------|
| 并发量  |      |
| 数据量  |      |
| 时序    |      |
| 环境    |      |

## 复现步骤
> 最小复现步骤
1.
2.
3.

## 检测方法
### 静态检测（代码审查/工具扫描）
- [ ] 关键字搜索：
- [ ] 代码模式匹配：

### 动态检测（测试手段）
- [ ] 测试方法：
- [ ] 测试工具：

### 监控告警（线上发现）
- [ ] 监控指标：
- [ ] 告警阈值：

## 修复方案
### 临时方案（止血）

### 根治方案

## 预防措施
> 如何在设计和开发阶段就避免此类问题？

### 设计阶段
- [ ]

### 开发阶段
- [ ]

### 测试阶段
- [ ]

## 真实案例
> 此模式在项目中的真实发生记录

| 案例编号 | 项目 | 模块 | 发现阶段 | 影响范围 | 链接 |
|---------|------|------|---------|---------|------|
| CASE-001 |      |      | 线上     |         |      |

## 相关缺陷模式
- [[另一个缺陷模式]] - 关联说明
```

**category 分类详解**：

| category | 典型场景 | 常见根因 |
|----------|---------|---------|
| `并发` | 秒杀超卖、重复退款、库存扣减 | 缺少分布式锁、CAS 失败未重试 |
| `数据一致性` | 订单状态与支付状态不一致 | 分布式事务不完整、缺少补偿 |
| `边界` | 金额 0.01 显示异常、数组越界 | 边界值未覆盖、off-by-one |
| `空值` | NPE、空指针、undefined | 缺少 null guard、Optional 误用 |
| `类型转换` | 字符串数字比较、时区转换 | 隐式转换、精度丢失 |
| `状态机` | 订单跳过中间状态、死循环 | 非法状态输入的转移未定义 |
| `性能` | 慢查询、OOM、连接池耗尽 | 未建索引、N+1、未限流 |
| `安全` | SQL 注入、越权访问、XSS | 输入未校验、权限缺失 |
| `UI` | 页面闪烁、布局错乱、数据不同步 | 渲染时序、缓存未失效、响应式遗漏 |
| `集成` | Webhook 丢失、消息重复消费 | 超时重试未防重、回调未幂等 |

**detection-difficulty 参考**：
- `极易遗漏` — 常规用例覆盖不到，需要白盒或压力测试才能发现
- `困难` — 需要特定环境/数据/并发条件触发
- `适中` — 正常流程测试可以覆盖，但容易被忽略
- `容易` — 基础功能测试即可发现

---

#### 7.3.4 模板填报示例

以下是每种模板的一个完整填报示例，帮助理解各字段含义和填写深度。

**示例：测试经验**

```markdown
---
type: testing-experience
module: "订单退款"
submodule: "退款审批流"
test-type: "功能测试"
test-method: "状态迁移法"
priority: "P0"
severity: "S1"
tags: [退款, 状态机, 审批流, 幂等]
source: "电商平台 v3.2 回归测试"
date: "2026-03-15"
status: reviewed
related-factors:
  - "[[金额计算因子]]"
  - "[[时间窗口因子]]"
related-bugs:
  - "[[BUG-4201 并发退款重复到账]]"
---

# 订单退款审批流测试经验

## 场景描述
> 用户在订单已支付状态发起退款申请，经审核后执行退款。

## 前置条件
- 订单状态 = 已支付
- 支付方式为微信/支付宝/银行卡
- 退款金额 ≤ 实付金额

## 测试要点
### 关键检查点
- [ ] 退款金额 = 实付金额（不含已均摊优惠券部分）
- [ ] 退款金额 + 优惠券退回金额 = 原始支付金额
- [ ] 部分退款时金额计算精度（分），不可四舍五入进/舍

### 易遗漏边界
- [ ] 连续点击「申请退款」按钮 → 期望只创建一条退款单
- [ ] 退款审核中，用户取消退款 → 订单状态恢复正常
- [ ] 超过可退款时效（7天）→ 退款入口不可见

### 异常路径
- [ ] 退款时支付渠道接口超时 → 30秒重试，仍失败转人工
- [ ] 退款中订单被风控拦截 → 退款挂起，通知人工处理

## 实际踩过的坑
### 问题描述
压测中发现快速双击「申请退款」可创建两条退款单，均通过「是否已退款」检查。

### 根因
退款入口缺少分布式锁，两次请求同时读到了「未退款」状态。

### 解决方案
退款申请入口加 Redis 分布式锁：`refund:lock:{orderId}`

### 预防措施
- 设计评审时识别所有「状态变更」类接口，默认要求加锁
- 自动化用例覆盖并发场景（JMeter 10并发）
```

**示例：因子条目**

```markdown
---
type: test-factor
category: "输入因子"
subcategory: "字符串"
factor-name: "用户名输入"
test-methods: [等价类划分, 边界值分析, 错误推测法]
related-experiences:
  - "[[注册模块测试经验]]"
  - "[[登录模块测试经验]]"
related-bugs:
  - "[[BUG-3088 用户名 SQL 注入]]"
  - "[[BUG-3101 空用户名绕过登录]]"
tags: [输入校验, SQL注入, Unicode, 前端校验]
risk-level: "high"
date: "2026-01-10"
---

# 用户名输入因子

## 因子定义
- 用户注册/登录时输入的用户名字段

## 覆盖维度
### 有效等价类
| 类别 | 代表值 | 说明 |
|------|--------|------|
| 中文名 | "张三" | 2 个中文字符 |
| 英文名 | "John" | 4 个英文字符 |
| 中英混合 | "张San" | 中英混合 |
| 含数字 | "user001" | 常见真实场景 |
| 允许的特殊字符 | "user_name-001" | _ 和 - |
| 最短长度 | "ab" | 2 字符 (假设最小 2) |
| 最长长度 | "a" * 20 | 20 字符 (假设最大 20) |

### 无效等价类
| 类别 | 代表值 | 预期结果 |
|------|--------|----------|
| 空字符串 | "" | 提示「用户名不能为空」 |
| 超长 | "a" * 21 | 提示「用户名最长 20 字符」 |
| 含非法特殊字符 | "user@name" | 提示「仅支持字母数字下划线」 |
| SQL 注入 | "'; DROP TABLE users; --" | 提示格式错误，不执行 SQL |
| XSS 注入 | "<script>alert(1)</script>" | HTML 转义后展示 |
| 纯空格 | "   " | 去空格后为空，提示不能为空 |
| Emoji | "张😀三" | 根据产品需求决定是否允许 |
| Unicode 零宽字符 | "user\u200Bname" | 过滤零宽字符 |

### 边界值
| 边界 | 测试值 | 预期结果 |
|------|--------|----------|
| 最小值 -1 | "a" (1 字符) | 提示「用户名最少 2 字符」 |
| 最小值 | "ab" (2 字符) | 通过 |
| 最大值 | "12345678901234567890" (20 字符) | 通过 |
| 最大值 +1 | "123456789012345678901" (21 字符) | 提示「用户名最长 20 字符」 |

## 组合策略
- 与「密码输入因子」正交组合 (用户名有效/无效 x 密码有效/无效)
- 与「验证码因子」因果组合（用户名错误时不需要验证码正确）

## 真实缺陷案例
- BUG-3088: 用户名输入 `' OR '1'='1` 登录接口未参数化查询，导致 SQL 注入
- BUG-3101: 输入纯空格用户名，trim 后为空，校验逻辑判断 null != "" 通过
```

**示例：业务规则**

```markdown
---
type: business-rule
domain: "电商"
module: "订单退款"
rule-id: "BR-ORDER-REFUND-001"
rule-category: "计算规则"
priority: "P0"
scope: "模块"
trigger-condition: "用户对已支付订单发起退款申请"
tags: [退款, 金额计算, 优惠券, 精度]
related-experiences:
  - "[[订单退款审批流测试经验]]"
related-factors:
  - "[[金额计算因子]]"
related-bug-patterns:
  - "[[浮点数精度丢失模式]]"
source: "PRD v3.2 第4.3节 + 运营规则文档"
date: "2026-02-20"
status: verified
---

# 退款金额计算规则

## 规则描述
> 退款金额 = 实付金额 − 已均摊的不可退回优惠金额，精确到分，采用向下取整策略（宁可少退不可多退）。

## 规则详情

### 输入条件
| 条件项 | 说明 | 取值/范围 |
|--------|------|-----------|
| 实付金额 | 用户实际支付金额 | ≥0.01 元，精确到分 |
| 平台优惠券 | 平台承担的优惠 | 退款时不退回 |
| 商家优惠券 | 商家承担的优惠 | 退款时按比例退回 |
| 退款比例 | 部分退款时的比例 | 0%~100% |

### 处理逻辑
```
if 全部退款:
    退款金额 = 实付金额 - 已均摊的平台优惠券金额
    退回优惠券 = 退回商家优惠券
elif 部分退款:
    退款金额 = 退款商品单价 × 退款数量 - 该商品均摊的平台优惠券金额
    退回优惠券 = 不退回（部分退款不退券）
```

### 输出结果
| 场景 | 预期结果 | 错误码 |
|------|---------|--------|
| 全额退款，无优惠券 | 退款金额 = 实付金额 | - |
| 全额退款，有平台券 | 退款金额 = 实付金额 - 平台券均摊 | - |
| 全额退款，有商家券 | 退款金额 = 实付金额 + 商家券退回 | - |
| 部分退款 | 退款金额 = 退款商品实付 - 均摊券 | - |
| 退款金额>实付金额 | 拒绝退款请求 | ERR_REFUND_EXCEED |

## 关联约束
- 满足 BR-ORDER-STATUS-001 订单在"已支付"状态
- 满足 BR-ORDER-TIME-001 退款申请在交易完成后 7 天内
- 此规则在 BR-ORDER-STATUS-002（退款状态流转规则）之前执行

## 验证方法
### 正向验证
- [ ] 全额退款金额 = 实付金额
- [ ] 部分退款金额 = 商品单价 × 数量

### 反向验证
- [ ] 退款金额超过实付金额时拒绝
- [ ] 已全额退款的订单再次退款被拒绝

### 边界验证
- [ ] 实付 0.01 元 → 退款 0.01 元
- [ ] 实付 0.01 元 + 平台券 0.01 元 → 实际退款 0 元
- [ ] 实付 0.03 元，退款 1/3 → 退款 0.01（向下取整）

## 历史变更
| 日期 | 变更内容 | 变更原因 | 变更人 |
|------|---------|---------|--------|
| 2026-02-20 | 初始版本 | PRD v3.2 | 测试团队 |
| 2026-04-01 | 向下取整策略变更 | 财务审计发现多退 | 测试团队 |

## 已知缺陷
- [[BUG-4401 退款金额精度多退 1 分]] - 四舍五入改为向下取整
- [[BUG-4410 部分退款时平台券均摊计算错误]] - 未按件均摊
```

**示例：缺陷模式**

```markdown
---
type: bug-pattern
pattern-name: "并发请求缺乏幂等守护"
pattern-id: "BUG-PTN-CONC-001"
category: "并发"
severity: "P0"
frequency: "高频"
detection-difficulty: "极易遗漏"
tags: [并发, 幂等, 分布式锁, 重复扣款, 超卖]
programming-languages: [Java, Go, Python]
tech-stack: [Redis, MySQL, Kafka]
related-experiences:
  - "[[订单退款审批流测试经验]]"
  - "[[支付模块测试经验]]"
related-factors:
  - "[[并发状态因子]]"
related-rules:
  - "[[退款金额计算规则]]"
date: "2026-03-15"
status: published
---

# 并发请求缺乏幂等守护

## 一句话描述
> 一个可变更状态的接口，在并发请求时缺少分布式锁或幂等令牌，导致同一操作被执行多次。

## 典型症状
- [ ] 点击一次按钮，扣费两次
- [ ] 退款金额重复到账
- [ ] 库存变成负数（超卖）
- [ ] 同一订单创建了多条退款记录
- [ ] 接口日志中同一参数出现多次，时间戳间隔 <100ms

## 根因分析
### 直接原因
接口实现中「检查状态 → 执行业务 → 更新状态」不是原子操作。
并发请求同时通过状态检查，各自独立执行业务逻辑。

### 深层原因
1. 设计阶段未识别该接口为「状态变更敏感」接口
2. 开发只考虑单线程场景，忽略并发
3. 幂等性设计只在接口文档中提及，代码未实现

## 触发条件
| 条件项 | 说明 |
|--------|------|
| 请求间隔 | 两次请求间隔 < 业务处理耗时 |
| 请求参数 | 两次请求参数完全相同 |
| 并发量 | ≥2 个并发即可触发 |
| 网络条件 | 弱网/高延迟环境更容易触发（用户重复点击） |
| 前端防护 | 按钮无防重、loading 遮罩晚于请求发出 |

## 复现步骤
1. 准备一个已支付订单
2. 使用 JMeter 设置 5 线程并发，调用退款接口（参数相同）
3. 查看数据库，退款记录 > 1 条
4. 用户账户余额增加了默认金额 × 并发次数

## 检测方法
### 静态检测（代码审查/工具扫描）
- [ ] 关键字搜索：`update*ByStatus`、`save`（无唯一键约束的）
- [ ] 代码模式匹配：`if (status == xxx)` → `doAction()` → `updateStatus(yyy)` 缺少锁
- [ ] CR 检查清单：所有状态变更接口是否有幂等方案

### 动态检测（测试手段）
- [ ] 接口并发测试：JMeter/Locust 对状态变更接口做 10 并发
- [ ] 快速双击测试：人工快速双击提交按钮（前端+后端双层验证）
- [ ] 网络模拟：Chrome DevTools 限速 3G，验证防重机制

### 监控告警（线上发现）
- [ ] 监控指标：同一 userId+orderId+action 在 1 秒内 >1 次
- [ ] 告警阈值：检测到疑似重复操作，立即告警
- [ ] 业务指标：退款金额 > 实付金额，触发对账告警

## 修复方案
### 临时方案（止血）
- 待修复接口前加 API 网关限流（同一用户同一操作 1秒1次）
- 前端按钮点击后立即 disabled + loading 遮罩

### 根治方案
- 接口入口加 Redis 分布式锁：`lock:{action}:{orderId}`，TTL=30s
- 数据库加唯一约束：`UNIQUE KEY uk_order_refund (order_id)`
- 接口改造为幂等：客户端生成 `idempotent_key`，服务端缓存处理结果

## 预防措施
### 设计阶段
- [ ] 设计评审时识别所有「状态变更」类接口
- [ ] 对这类接口默认要求提供幂等方案（锁/令牌/唯一约束）

### 开发阶段
- [ ] IDE 代码模板：状态变更方法自动生成幂等令牌检查代码
- [ ] 单元测试必须覆盖并发场景（并发测试框架）

### 测试阶段
- [ ] 自动化回归用例中包含「状态变更接口并发」专项
- [ ] 每次发布前执行「幂等性检查清单」

## 真实案例
| 案例编号 | 项目 | 模块 | 发现阶段 | 影响范围 | 链接 |
|---------|------|------|---------|---------|------|
| CASE-001 | 电商平台 | 退款 | 压测 | 资金损失 1200 元 | [[BUG-4201]] |
| CASE-002 | 电商平台 | 下单 | 线上 | 超卖 500 单 | [[BUG-3502]] |
| CASE-003 | 支付系统 | 提现 | 线上 | 重复打款 3 笔 | [[BUG-5103]] |

## 相关缺陷模式
- [[状态机缺省处理模式]] - 非法状态转移未定义默认分支
- [[事务边界过大模式]] - 锁持有时间过长导致死锁/超时
```

---

### 7.4 方案A：文件系统直读（入门必选）

**原理：** Agent 原生就支持 Read/Grep 工具，直接读取 Obsidian vault 中的 `.md` 文件。零额外配置。

#### Step 1：在项目 CLAUDE.md 中声明知识库路径

```markdown
# CLAUDE.md（项目根目录）

## Obsidian Testing Knowledge Base

测试知识库路径：`~/Documents/Obsidian/Testing-Knowledge-Vault/`

关键目录：
- 01-Experiences/  测试经验库（按模块记录历史经验、踩过的坑）
- 02-Factors/      测试因子库（等价类、边界值、正交因子）
- 03-Business-Rules/ 业务规则库
- 04-Bug-Patterns/ 缺陷模式库
- 08-MOCs/         总索引页

进行测试设计、用例编写、Bug分析时，请先搜索该知识库：
1. 用 grep 搜索关键模块名、方法名
2. 读取匹配的笔记，提取测试要点、踩坑经验、因子组合
3. 结合知识库内容给出建议，并标注信息来源
```

#### Step 2：Agent 自动搜索流程

当你在 Claude Code 中要求设计测试用例时，Agent 会自动执行：

```
用户：为「订单退款」功能设计测试用例

Agent 自动执行（基于 CLAUDE.md 中的指示）：
1. grep -ril "退款|refund" ~/Documents/Obsidian/Testing-Knowledge-Vault/
2. 命中：
   - 01-Experiences/支付流程测试经验.md
   - 03-Business-Rules/电商业务规则.md
   - 04-Bug-Patterns/数据一致性问题模式.md
3. 读取这些笔记，提取关键信息
4. 融合知识库内容，给出标记了来源的测试建议
```

**优点：** 零配置、直接可用、任意 Agent 都支持
**缺点：** 大 vault 时搜索慢，依赖关键词精确匹配

---

### 7.5 方案C：kepano/obsidian-skills（强烈推荐）

> GitHub: https://github.com/kepano/obsidian-skills

Obsidian CEO kepano 开源的 Agent Skills，教会 AI 正确读写 Obsidian 专属格式。**全套包含 4 个 Skill**，下面逐一说明它们在你的测试知识库场景中如何落地。

**安装（推荐全局安装）：**

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/kepano/obsidian-skills.git /tmp/obsidian-skills
cp -r /tmp/obsidian-skills/skills/* ~/.claude/skills/
rm -rf /tmp/obsidian-skills
```

或在 Claude Code 中：`/plugin install obsidian@obsidian-skills`

验证：运行 `/skills` 应看到 `obsidian-markdown`、`obsidian-bases`、`json-canvas`、`obsidian-cli` 四个 Skill。

---

#### 7.5.1 obsidian-markdown — 教 Agent 说 Obsidian 的"方言"

**这个 Skill 解决什么问题？**

普通的 AI 只会写标准 Markdown。让它在 Obsidian 里读/写笔记时，它会用 `[文本](链接)` 而不是 `[[wikilinks]]`，不会用 Callout、不会正确嵌入图片、不会写 Frontmatter。`obsidian-markdown` 教会 Agent 这些 Obsidian 专属语法。

**在你的测试场景中怎么用？举例说明。**

##### 例子1：Agent 帮你沉淀一条测试经验

没有 `obsidian-markdown` 时，Agent 可能会写出这样的坏笔记：

```markdown
# 订单退款测试经验

相关经验见: [并发竞态模式](并发竞态模式.md)
图片: ![架构图](images/refund-flow.png)

注意: 这里有个重要的问题
退款金额在并发场景下会重复计算
```

装了 `obsidian-markdown` 后，Agent 会正确写出：

```markdown
---
type: testing-experience
module: "订单退款"
test-type: "功能测试"
test-method: "状态迁移"
priority: "P0"
tags:
  - 退款
  - 并发
  - 状态机
related-factors:
  - "[[金额计算因子]]"
  - "[[并发状态因子]]"
related-bugs:
  - "[[BUG-4201 并发退款重复到账]]"
status: draft
date: 2026-05-23
---

# 订单退款测试经验

## 场景描述
> 用户在订单已支付状态下发起退款，涉及金额计算、状态流转、第三方支付回调。

## 前置条件
- 订单处于「已支付」状态
- 支付方式支持原路退回
- 用户账户状态正常

## 关键检查点
- [ ] 退款金额 = 实付金额（不含已使用优惠券部分）
- [ ] 部分退款时金额精度（分级别）
- [ ] 退款状态机：申请 → 审核 → 退款中 → 已完成 / 已拒绝

## 易遗漏边界
- [ ] 并发发起两次退款请求 → 期望只退一次

> [!bug] 踩坑：并发退款导致重复到账
> 2025年3月，压测时发现用户快速点击两次「退款」按钮，
> 两次请求同时进入 `refund()` 方法，均通过「未退款」状态检查，
> 导致金额重复退回。根因是缺少分布式锁。
>
> **解决方案**：在退款入口加 Redis 分布式锁 `refund:lock:{orderId}`

## 关联因子
![[金额计算因子#边界值]]

## 缺陷模式
参见 [[并发竞态模式]]
```

关键差异：正确的 `[[wikilinks]]`、`> [!bug]` Callout、规范 Frontmatter、`![[embed]]` 语法。

##### 例子2：Agent 批量更新知识库内的交叉引用

```
用户：帮我把所有 P0 的测试经验和对应的因子做双向链接

Agent（借助 obsidian-markdown）：
1. Grep 找到所有 priority: "P0" 的经验笔记
2. 读取每篇的 related-factors 字段
3. 在对应的因子笔记中，追加 related-experiences 反向链接
4. 使用正确的 [[wikilinks]] 语法，Obsidian 能自动跟踪重命名
```

##### 例子3：Agent 用 Callout 结构化你的笔记

Agent 在帮你整理已有笔记时，会自动识别内容类型并使用对应 Callout：

| 内容类型 | 使用的 Callout | 示例 |
|---------|---------------|------|
| 重要提醒 | `> [!important]` | 关键检查点 |
| 踩坑记录 | `> [!bug]` | 历史缺陷案例 |
| 注意事项 | `> [!warning]` | 易遗漏边界 |
| 测试技巧 | `> [!tip]` | 实用的测试方法 |
| 参考信息 | `> [!info]` | 相关的业务规则 |
| 待办事项 | `> [!todo]` | 待补充的测试场景 |

---

#### 7.5.2 obsidian-bases — 把你的笔记变成可筛选、可排序的数据视图

**这个 Skill 解决什么问题？**

你的测试知识库存了几百条经验、因子、Bug 模式后，靠文件树和搜索很难快速了解全貌。`obsidian-bases` 让 Agent 帮你创建 `.base` 文件——类似数据库视图，自动聚合所有符合条件的笔记，支持筛选、排序、分组、公式计算。

**在你的测试场景中怎么用？举例说明。**

##### 例子1：创建测试经验仪表盘

让 Agent 帮你创建一个 `.base` 文件 `08-MOCs/测试经验仪表盘.base`：

```
用户：帮我创建一个 Bases 视图，展示所有测试经验，
      按优先级分颜色显示，按模块分组，统计每个模块的经验数量

Agent（借助 obsidian-bases）会生成：
```

```yaml
filters:
  and:
    - 'type == "testing-experience"'   # 只显示测试经验类型的笔记
    - 'file.ext == "md"'

formulas:
  priority_label: 'if(priority == "P0", "🔴 P0", if(priority == "P1", "🟡 P1", if(priority == "P2", "🟢 P2", "⚪ P3")))'
  days_since_update: '(now() - file.mtime).days.round(0)'
  is_stale: 'if(days_since_update > 90, true, false)'

properties:
  module:
    displayName: "被测模块"
  formula.priority_label:
    displayName: "优先级"
  formula.days_since_update:
    displayName: "距上次更新(天)"
  formula.is_stale:
    displayName: "是否过期"

views:
  - type: table
    name: "全部测试经验"
    order:
      - file.name
      - module
      - formula.priority_label
      - test-type
      - status
      - formula.days_since_update
    groupBy:
      property: module
    summaries:
      test-type: Unique
```

Agent 生成后，你在 Obsidian 中打开 `测试经验仪表盘.base`，就能看到一个可交互的表格：
- 按模块分组显示所有测试经验
- P0 标记为红色、P1 黄色、P2 绿色
- 超过 90 天未更新的自动标记为"过期"
- 底部统计每个模块的经验数量

##### 例子2：创建因子覆盖率矩阵

```
用户：帮我建一个因子覆盖率 Base，列出所有因子对应的测试方法，
      标记哪些因子还没有关联经验

Agent 生成：
```

```yaml
filters:
  and:
    - 'type == "test-factor"'
    - 'file.ext == "md"'

formulas:
  has_experience: 'if(related-experiences && length(related-experiences) > 0, "✅ 已关联", "❌ 未关联")'
  experience_count: 'if(related-experiences, length(related-experiences), 0)'
  method_list: 'test-methods.join(", ")'

properties:
  category:
    displayName: "因子分类"
  subcategory:
    displayName: "子分类"
  formula.method_list:
    displayName: "适用测试方法"
  formula.has_experience:
    displayName: "关联状态"
  risk-level:
    displayName: "风险等级"

views:
  - type: table
    name: "因子覆盖率矩阵"
    order:
      - file.name
      - category
      - subcategory
      - formula.method_list
      - formula.has_experience
      - risk-level
    groupBy:
      property: category
    filters:
      and:
        - 'formula.experience_count == 0'   # 只看未关联经验的因子

  - type: table
    name: "完整因子列表"
    order:
      - file.name
      - category
      - formula.has_experience
```

##### 例子3：创建 Bug 模式频率热力图

```
用户：帮我创建一个 Base，统计缺陷模式的出现频率，
      按风险等级排序，标记最近发现的模式

Agent 生成包含以下公式的 Base：
- is_recent：判断该模式最近 30 天内是否被触发
- severity_score：根据 severity 字段计算风险分数
- related_cases：统计关联的测试用例数量
```

**核心价值：** Agent 不只是帮你"写"笔记，而是帮你"组织"和"可视化"笔记。几百条经验不再散落各处，通过 Base 视图一目了然。

---

#### 7.5.3 json-canvas — 让 Agent 画可视化的测试策略图

**这个 Skill 解决什么问题？**

Markdown 文档适合描述细节，但很难直观展示关系和流程。`json-canvas` 教会 Agent 创建和编辑 `.canvas` 文件——Obsidian 的无限画布，可以在上面放置笔记节点、图片、网页链接，用连线表达关系。

**在你的测试场景中怎么用？举例说明。**

##### 例子1：自动生成测试策略脑图

```
用户：帮我画一个「订单系统」的测试策略画布，包含所有模块和测试重点

Agent（借助 json-canvas）执行：
1. Grep 搜索订单相关的测试经验
2. 提取每个模块的测试重点
3. 生成 .canvas 文件，节点和连线清晰呈现
```

Agent 生成的 `订单系统测试策略.canvas` 结构：

```json
{
  "nodes": [
    {
      "id": "order-root-00001",
      "type": "text",
      "x": 0, "y": 0,
      "width": 350, "height": 80,
      "text": "# 订单系统测试策略\n基于知识库经验生成",
      "color": "6"
    },
    {
      "id": "order-module-0001",
      "type": "text",
      "x": -300, "y": 150,
      "width": 280, "height": 140,
      "text": "## 下单模块\n\n### 重点\n- 库存扣减原子性 [[库存并发经验]]\n- 优惠券计算精度 [[金额计算因子]]\n\n### 风险\n- P0: 超卖问题",
      "color": "1"
    },
    {
      "id": "order-module-0002",
      "type": "text",
      "x": 100, "y": 150,
      "width": 280, "height": 140,
      "text": "## 支付模块\n\n### 重点\n- 支付回调幂等性 [[幂等处理经验]]\n- 超时关单机制\n\n### 风险\n- P0: 重复扣款",
      "color": "1"
    },
    {
      "id": "order-module-0003",
      "type": "text",
      "x": 500, "y": 150,
      "width": 280, "height": 140,
      "text": "## 退款模块\n\n### 重点\n- 退款状态机 [[退款经验]]\n- 并发锁机制 [[并发竞态模式]]\n\n### 风险\n- P0: 重复退款",
      "color": "2"
    },
    {
      "id": "factor-node-0001",
      "type": "file",
      "x": -150, "y": 400,
      "width": 350, "height": 200,
      "file": "02-Factors/输入因子/金额计算因子.md"
    },
    {
      "id": "pattern-node-0001",
      "type": "file",
      "x": 300, "y": 400,
      "width": 350, "height": 200,
      "file": "04-Bug-Patterns/并发竞态模式.md"
    }
  ],
  "edges": [
    {
      "id": "edge-001",
      "fromNode": "order-root-00001",
      "fromSide": "bottom",
      "toNode": "order-module-0001",
      "toSide": "top",
      "label": "下单"
    },
    {
      "id": "edge-002",
      "fromNode": "order-root-00001",
      "fromSide": "bottom",
      "toNode": "order-module-0002",
      "toSide": "top",
      "label": "支付"
    },
    {
      "id": "edge-003",
      "fromNode": "order-root-00001",
      "fromSide": "bottom",
      "toNode": "order-module-0003",
      "toSide": "top",
      "label": "退款"
    },
    {
      "id": "edge-004",
      "fromNode": "order-module-0001",
      "fromSide": "bottom",
      "toNode": "factor-node-0001",
      "toSide": "top",
      "label": "引用因子"
    },
    {
      "id": "edge-005",
      "fromNode": "order-module-0003",
      "fromSide": "bottom",
      "toNode": "pattern-node-0001",
      "toSide": "top",
      "label": "关联缺陷模式"
    }
  ]
}
```

在 Obsidian 中打开这个 `.canvas` 文件后，你会看到一张可视化的测试策略图：
- 根节点「订单系统测试策略」居中
- 三个模块节点按风险等级着色（红色=高风险）
- 因子节点和缺陷模式节点通过连线关联到对应模块
- 点击任意文件节点可以直接打开对应笔记

##### 例子2：Bug 影响范围分析图

```
用户：帮我画一个 Bug 影响分析画布，分析「支付超时」这个 Bug 的影响范围

Agent 执行：
1. 搜索知识库中与「支付超时」相关的经验、因子、缺陷模式
2. 创建画布，中心是 Bug 节点，向外辐射受影响模块
3. 每个影响节点标注严重程度和关联的测试用例
```

最终生成的画布效果：
```
                  ┌─────────────────┐
                  │  订单状态不一致   │
                  │  严重度: P0      │
                  └────────┬────────┘
                           │
  ┌─────────────────┐     │     ┌─────────────────┐
  │  库存回滚失败    │ ←───┼───→ │  用户资金冻结    │
  │  严重度: P1      │     │     │  严重度: P0      │
  └─────────────────┘     │     └─────────────────┘
                           │
                  ┌────────┴────────┐
                  │  支付超时 Bug    │
                  │  (中心节点)      │
                  └────────┬────────┘
                           │
                  ┌────────┴────────┐
                  │  三方对账异常    │
                  │  严重度: P2      │
                  └─────────────────┘
```

##### 例子3：测试覆盖度热力图

```
用户：用 Canvas 画一个功能模块 x 测试类型的覆盖矩阵，
      红色=未覆盖，黄色=部分覆盖，绿色=完整覆盖

Agent 读取所有经验的 module 和 test-type 字段，
生成覆盖矩阵画布（10个模块 x 5种测试类型 = 50个节点的矩阵）
```

---

#### 7.5.4 obsidian-cli — 命令行直接操控 Obsidian

**这个 Skill 解决什么问题？**

前面的 Skill 都是在文件层面操作 `.md` / `.base` / `.canvas` 文件。`obsidian-cli` 更进一步——通过 Obsidian 的 CLI 接口，在 Obsidian 正在运行时直接操控它：搜索、创建笔记、读取内容、设置属性、操作任务等。

> 前置条件：需要 Obsidian 正在运行，且已开启 CLI 功能（设置 → 核心插件 → CLI）

**在你的测试场景中怎么用？举例说明。**

##### 例子1：测试执行中快速追加发现的问题到知识库

```
场景：你正在做「红包发放」功能的测试，发现了一个边界问题。
你不想切出 Obsidian，直接在 Claude Code 中：

用户：记到今天的测试日记里：
      「发现红包金额 0.01 元时，前端显示为 0 元，后端存储正常，
        是前端 toFixed(2) 没有处理小数点后两位都为零的情况」

Agent（借助 obsidian-cli）执行：
  obsidian vault="Testing-Knowledge" daily:append \
    content="- [ ] [BUG] 红包金额 0.01 元前端显示异常 \n  - 根因: toFixed(2) 返回 \\\"0.00\\\" \n  - 关联因子: [[金额计算因子]] \n  - 发现时间: $(date '+%Y-%m-%d %H:%M')"

结果：直接追加到今天的日记文件中，无需切换窗口。
```

##### 例子2：Agent 搜索知识库时借用 Obsidian 的索引

```
用户：哪些经验笔记提到了「分布式事务」但没有标记 P0？

Agent（借助 obsidian-cli）：
  obsidian vault="Testing-Knowledge" search query="分布式事务"

  → 返回 7 篇匹配笔记

  obsidian property:get name="priority" file="分布式事务经验1"
  → P1
  obsidian property:get name="priority" file="分布式事务经验2"
  → P2
  obsidian property:get name="priority" file="分布式事务经验3"
  → (none)

Agent 输出：
  「分布式事务」相关经验共 7 篇，其中：
  - P0: 3 篇
  - P1: 1 篇
  - P2: 1 篇
  - 未标优先级: 2 篇（建议补充：[[分布式事务经验3]]、[[消息队列事务经验]]）
```

##### 例子3：测试结束后批量归档

```
用户：帮我检查知识库中所有 status=draft 且超过 30 天未修改的经验，
      如果是已经验证过的，帮我标记为 reviewed

Agent（借助 obsidian-cli）：
  1. obsidian vault="Testing-Knowledge" search query="status: draft"
  2. 检查每篇的 mtime
  3. 列出清单，让用户确认
  4. obsidian property:set name="status" value="reviewed" file="XXX"
```

---

#### 7.5.5 四合一实战：完整工作流演示

以下展示一个完整场景，四个 Skill 如何协同工作：

```
用户：帮我做一个「用户注册」功能的完整测试设计，沉淀到知识库

Agent 协同使用 4 个 Skill：

Step 1（obsidian-cli）：搜索知识库
  obsidian search query="注册|register" → 找到 3 篇经验、2 个因子

Step 2（obsidian-cli）：读取搜索结果
  obsidian read file="注册模块测试经验"
  obsidian read file="字符串输入因子"

Step 3（test-design Skill）：分析需求 + 知识库内容 → 生成测试用例

Step 4（obsidian-markdown）：将测试经验保存为规范的 .md 文件
  使用 [[wikilinks]] 链接因子和缺陷模式
  使用 > [!important] 标注关键检查点
  使用规范 Frontmatter

Step 5（obsidian-bases）：更新经验仪表盘
  新的经验自动出现在 Base 视图中

Step 6（json-canvas）：更新测试策略画布
  在「用户系统」画布中添加「注册」节点和连线

最终产出：
  ✅ 01-Experiences/功能测试/用户注册测试经验.md （遵循 obsidian-markdown 规范）
  ✅ 08-MOCs/测试经验仪表盘.base 中自动可见新条目
  ✅ 用户系统测试策略.canvas 中新增了注册模块节点
  ✅ 与已有因子 [[字符串输入因子]] [[验证码因子]] 建立双向链接
```

---

#### 7.5.6 Skill 组合使用速查

| 场景 | 用到的 Skill | Agent 实际做了什么 |
|------|-------------|-------------------|
| 沉淀一条测试经验 | `obsidian-markdown` | 写规范的 .md，用 `[[wikilinks]]` + Callout + Frontmatter |
| 创建经验数据看板 | `obsidian-bases` | 写 `.base` YAML，筛选+分组+公式+汇总 |
| 画测试策略脑图 | `json-canvas` | 生成 `.canvas` JSON，节点+连线+颜色+布局 |
| 快速搜索+追加日记 | `obsidian-cli` | `obsidian search` + `obsidian daily:append` |
| 批量更新元数据 | `obsidian-cli` | `obsidian property:set` 批量修改 Frontmatter |
| 补充双向链接 | `obsidian-markdown` | 在因子笔记中补充 `[[经验笔记]]` 反向链接 |
| 生成测试覆盖矩阵 | `obsidian-bases` | Base 的交叉筛选 + 公式判定未覆盖项 |
| Bug 影响分析图 | `json-canvas` | Bug 中心节点 → 辐射影响节点 → 连线标注 |

---

### 7.6 实战：创建「Test Design Skill」

这是最关键的部分——让 Agent 在做测试设计时自动搜索知识库并引用。

#### 7.6.1 SKILL.md 完整内容

创建 `~/.claude/skills/test-design/SKILL.md`：

```markdown
---
name: test-design
description: >
  软件测试设计助手。结合 Obsidian 测试知识库（经验库、因子库、缺陷模式库）
  生成全面的测试建议和测试用例。Use when user asks about 测试设计, 测试用例,
  测试策略, 用例评审, test design, test cases, test strategy.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash(grep:*,rg:*)
---

# Test Design Skill

你是资深软件测试架构师。你的任务是结合 **Obsidian 测试知识库** 给出全面精准的测试设计建议。

## 知识库路径

默认路径：`~/Documents/Obsidian/Testing-Knowledge-Vault/`
如果项目 CLAUDE.md 中指定了其他路径，以 CLAUDE.md 为准。

## 工作流程

### Phase 1：信息收集
1. 明确被测对象（模块、功能、接口）
2. 提取搜索关键词（中文 + 英文）
3. 识别涉及的测试类型

### Phase 2：知识库搜索（必须执行，不可跳过）

```
搜索1 → 经验库（历史踩坑、检查点）
  rg -il "关键词" ~/path/01-Experiences/

搜索2 → 因子库（等价类、边界值、正交因子）
  rg -il "关键词" ~/path/02-Factors/

搜索3 → 业务规则
  rg -il "关键词" ~/path/03-Business-Rules/

搜索4 → 缺陷模式
  rg -il "关键词" ~/path/04-Bug-Patterns/

搜索5 → 经典用例
  rg -il "关键词" ~/path/06-Cases/
```

**策略：**
- 先用精确关键词，无结果则扩展搜索范围
- 至少选 2-3 篇最相关笔记深入阅读
- 如果知识库路径不存在或搜索无结果，告知用户并降级使用通用方法论

### Phase 3：知识提取
从搜索到的笔记中提取：
- 「关键检查点」「易遗漏边界」「异常路径」（经验库）
- 「等价类」「边界值」「组合策略」（因子库）
- 「业务约束」（业务规则库）
- 「典型缺陷」（缺陷模式库）

### Phase 4：生成输出

## 输出格式

### 1. 知识库命中情况
| 知识库 | 命中笔记 | 关联度 |
|--------|---------|--------|
| 经验库 | [[笔记名]] | 高/中/低 |

### 2. 测试策略建议
- 测试重点（基于经验库历史踩坑）
- 测试优先级
- 推荐测试方法

### 3. 测试因子分析
| 因子 | 来源 | 等价类 | 边界值 | 异常值 |
|------|------|--------|--------|--------|

### 4. 测试用例
| 用例ID | 场景 | 前置条件 | 步骤 | 预期结果 | 知识来源 |
|--------|------|---------|------|---------|---------|

### 5. 易遗漏检查点 ⚠️
- 基于经验库「实际踩过的坑」章节

### 6. 风险评估
- 基于缺陷模式库的关联分析

## 质量要求
- 每个基于知识库的建议必须标注来源笔记名
- 明确区分「来自知识库的经验」和「来自通用方法论」
- 知识库没有的信息不编造，使用通用方法论补充并注明
- 输出格式便于后续导入 Obsidian
```

#### 7.6.2 安装 Skill

```bash
mkdir -p ~/.claude/skills/test-design
# 将上面的 SKILL.md 内容写入
cat > ~/.claude/skills/test-design/SKILL.md <<'SKILLEOF'
# ... 上面的完整 SKILL.md 内容 ...
SKILLEOF
```

#### 7.6.3 使用演示

**场景：为登录功能设计测试用例**

```
用户：/test-design 为手机号验证码登录功能设计测试用例

Agent 执行：
Phase 2:
  → rg -il "登录|login|验证码|SMS" .../01-Experiences/
  命中：登录模块测试经验.md, 短信服务测试经验.md
  → rg -il "手机号|验证码" .../02-Factors/
  命中：验证码因子.md
  → rg -il "并发|超时" .../04-Bug-Patterns/
  命中：并发竞态模式.md, 接口超时处理模式.md

输出摘要：
  知识库命中：
  | 经验库 | 登录模块测试经验.md | 高 |
  | 经验库 | 短信服务测试经验.md | 高 |
  | 因子库 | 验证码因子.md | 高 |
  | 缺陷库 | 并发竞态模式.md | 中 |

  易遗漏检查点 ⚠️：
  - 图形验证码绕过（来源：登录模块测试经验.md 实际踩过的坑 #3）
  - 多设备同时登录 token 处理（来源：并发竞态模式.md）
  - 验证码有效期边界 59s/60s/61s（来源：验证码因子.md 边界值表）
```

### 7.7 进阶：辅助 Skills

#### 7.7.1 test-capture（沉淀经验 Skill）

```markdown
---
name: test-capture
description: >
  将测试中发现的 Bug、经验快速沉淀到 Obsidian 知识库。
  Use when user says 沉淀经验, 记录bug, 补充知识库, capture lesson.
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Test Capture Skill

将测试中发现的有价值内容快速沉淀到 Obsidian 测试知识库。

## 工作流程
1. 询问用户要沉淀的内容类型（经验/因子/缺陷模式/业务规则）
2. 读取对应的模板文件 `07-Templates/`
3. 基于用户的描述填充模板，生成完整的 .md 文件
4. 保存到对应的知识库目录
5. 更新 MOC 索引页，添加新条目链接
6. 如果涉及多个知识库条目，建立 [[双向链接]]
```

#### 7.7.2 test-kb-health（知识库健康检查 Skill）

```markdown
---
name: test-kb-health
description: >
  检查测试知识库健康状况：未审阅经验、孤立笔记、过期内容、MOC完整性。
  Use when user says 知识库检查, KB health check.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash(grep:*,rg:*)
---

# Knowledge Base Health Check

## 检查项
1. 扫描 status=draft 的笔记 → 提醒审阅
2. 检查 [[broken links]] → 提醒修复
3. 列出 30 天未修改的 WIP 笔记 → 提醒归档
4. 检查 MOC 索引是否覆盖所有目录中的笔记
5. 统计各类知识数量，输出概览报告
```

### 7.8 方案B：MCP + Smart Connections（大仓库进阶）

> 当你的 vault 超过 500 篇笔记，关键词搜索不够精准时，升级到此方案。

#### Install Obsidian 插件

在 Obsidian 社区插件市场搜索并安装：

1. **Local REST API** — 为 Obsidian 提供 HTTP API（端口 27124）
2. **Smart Connections** — 基于 embedding 的语义搜索
3. **MCP Tools** — 将 Obsidian 作为 MCP Server 暴露

#### 验证 API 可用

```bash
# 获取 vault 信息
curl -X GET "http://127.0.0.1:27124/vault/" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 语义搜索笔记
curl -X GET "http://127.0.0.1:27124/smart-connections/search?query=退款并发安全" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

#### 注册为 MCP Server

```bash
claude mcp add obsidian-knowledge \
  --command "npx" --args "-y" "mcp-obsidian" \
  --env OBSIDIAN_API_KEY="你的Key" \
  --env OBSIDIAN_API_URL="http://127.0.0.1:27124"
```

### 7.9 一键部署脚本

```bash
#!/bin/bash
# Obsidian Testing Knowledge Base + AI Agent Skills 一键部署
VAULT=~/Documents/Obsidian/Testing-Knowledge-Vault

# 1. 创建知识库目录
mkdir -p "$VAULT"/{00-Inbox,"01-Experiences"/{功能测试,性能测试,安全测试,兼容性测试},"02-Factors"/{输入因子,环境因子,状态因子,组合因子},03-Business-Rules,04-Bug-Patterns,05-Test-Methods,06-Cases,07-Templates,08-MOCs}

# 2. 安装 obsidian-skills
mkdir -p ~/.claude/skills
git clone https://github.com/kepano/obsidian-skills.git /tmp/obsidian-skills 2>/dev/null
cp -r /tmp/obsidian-skills/skills/* ~/.claude/skills/ 2>/dev/null
rm -rf /tmp/obsidian-skills

# 3. 创建 test-design skill 目录
mkdir -p ~/.claude/skills/test-design
# 请手动将 7.6.1 节的 SKILL.md 内容复制进去

# 4. 创建模板（测试经验、因子、业务规则、缺陷模式）
# 将 7.3.3 节中的四个模板内容分别写入以下文件：
# "$VAULT/07-Templates/测试经验模板.md"
# "$VAULT/07-Templates/因子条目模板.md"
# "$VAULT/07-Templates/业务规则模板.md"
# "$VAULT/07-Templates/缺陷模式模板.md"
# 同时在 03-Business-Rules/ 和 04-Bug-Patterns/ 中放入示例条目（参见 7.3.4 节）

# 5. Git 管理
cd "$VAULT"
git init 2>/dev/null
echo ".obsidian/workspace.json
.trash/
.DS_Store
Thumbs.db" > .gitignore
git add -A && git commit -m "初始化测试知识库" 2>/dev/null

echo "✅ 测试知识库已创建：$VAULT"
echo "✅ obsidian-skills 已安装"
echo "📝 下一步：将 test-design SKILL.md 复制到 ~/.claude/skills/test-design/"
echo "📝 然后在 Obsidian 中打开 $VAULT"
```

### 7.10 关键注意事项

1. **知识库预热**：首次使用每个目录至少手动填充 3-5 篇笔记，空知识库对 Agent 无帮助
2. **粒度控制**：每篇经验笔记聚焦一个具体场景，不要写成「大而全」文档
3. **持续积累**：每次发现新的测试盲区或踩坑，立即用 `/test-capture` 沉淀
4. **元数据规范**：严格遵循模板的 Frontmatter 字段，这是 Agent 检索的关键
5. **双向链接**：多用 `[[链接]]` 关联相关知识，Agent 会顺着链接发现更多内容
6. **多语言关键词**：笔记中同时包含中文和英文关键词，提高搜索命中率
7. **隐私安全**：知识库是纯本地 Markdown，Agent 本地运行，数据不上传云端
8. **Git 备份**：知识库是你的核心资产，用 Git 管理并定期 push 到远程仓库

### 7.11 扩展：其他领域的知识库架构

| 领域 | Vault 示例 | 核心目录 | 配套 Skill |
|------|-----------|---------|------------|
| 代码审查 | Code-Review-Vault | 代码异味库/安全漏洞库/重构模式库 | `/code-review` |
| 技术方案 | Tech-Design-Vault | 架构模式库/技术选型库/踩坑记录 | `/tech-design` |
| 运维排障 | Ops-Vault | 告警模式库/故障案例库/恢复流程库 | `/ops-debug` |
| 需求分析 | Req-Vault | 需求模式库/干系人库/验收标准库 | `/req-analysis` |
| 项目管理 | PM-Vault | 风险库/干系人模式库/复盘模板 | `/pm-review` |

**统一原则：** 每个 Vault 独立，Skill 可跨 Vault 搜索；保持统一的 Frontmatter 规范；全部用 Git 管理。
