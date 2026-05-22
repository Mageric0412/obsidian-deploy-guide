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
