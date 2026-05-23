# Testing Knowledge Vault

基于 Obsidian 的软件测试知识库，为 AI Agent (Claude Code) 提供可搜索的测试经验和知识。

## 快速开始

### 1. 在 Obsidian 中打开此 Vault

1. 下载本仓库到本地
2. 打开 Obsidian → 点击「Open folder as vault」
3. 选择 `Testing-Knowledge-Vault/` 目录

### 2. 安装推荐插件

- **Templater** — 模板引擎（可选，方便基于模板创建笔记）
- **Dataview** — 结构化查询（可选，在 Obsidian 内做数据查询）

### 3. 安装 AI Agent Skills

参考主文档 [README.md](../README.md) 第 7 节的部署脚本，安装以下 Skills：

- `obsidian-markdown` — 教 Agent 写 Obsidian 专属格式
- `obsidian-bases` — 创建 Base 视图
- `json-canvas` — 创建 Canvas 画布
- `obsidian-cli` — 命令行交互
- `test-design` — 测试设计 Skill（本仓库包含）

**Skills 安装路径：**

本仓库的 `skills/` 目录包含自定义的测试相关 Skills：

```
skills/
└── test-design/
    └── SKILL.md         # 测试设计 Skill
```

将这些 Skills 复制到 Claude Code 的 skills 目录：

```bash
cp -r skills/* ~/.claude/skills/
```

或直接在 Claude Code 中安装 kepano 的 obsidian-skills：

```bash
git clone https://github.com/kepano/obsidian-skills.git /tmp/obsidian-skills
cp -r /tmp/obsidian-skills/skills/* ~/.claude/skills/
rm -rf /tmp/obsidian-skills
```

### 4. 创建你的第一篇笔记

1. 在 Obsidian 中按 `Cmd+N`（Mac）或 `Ctrl+N`（Win）新建笔记
2. 插入模板：`Cmd+Shift+T` → 选择对应模板
3. 基于模板填写内容

**建议：** 首次使用每个目录至少手动填充 3-5 篇笔记，空知识库对 AI Agent 无帮助。

## 目录结构

```
Testing-Knowledge-Vault/
├── 00-Inbox/              # 待整理的经验碎片
├── 01-Experiences/        # 测试经验库
│   ├── 功能测试/
│   ├── 性能测试/
│   ├── 安全测试/
│   └── 兼容性测试/
├── 02-Factors/            # 测试因子库
│   ├── 输入因子/
│   ├── 环境因子/
│   ├── 状态因子/
│   └── 组合因子/
├── 03-Business-Rules/     # 业务规则库
├── 04-Bug-Patterns/       # 缺陷模式库
├── 05-Test-Methods/       # 测试方法论
├── 06-Cases/              # 经典测试用例库
├── 07-Templates/          # 笔记模板
└── 08-MOCs/               # 索引/地图（Map of Content）
```

## 已包含的内容

### 模板（5个）
- `07-Templates/测试经验模板.md`
- `07-Templates/因子条目模板.md`
- `07-Templates/业务规则模板.md`
- `07-Templates/缺陷模式模板.md`
- `07-Templates/测试方法模板.md`

### 填报示例（5个）
- `01-Experiences/功能测试/订单退款审批流测试经验.md`
- `02-Factors/输入因子/用户名输入因子.md`
- `03-Business-Rules/退款金额计算规则.md`
- `04-Bug-Patterns/并发请求缺乏幂等守护.md`
- `05-Test-Methods/等价类划分法.md`
- `05-Test-Methods/状态迁移法.md`

### 索引
- `08-MOCs/知识库总索引.md` — 全库导航

## 元数据规范

Agent 搜索知识库主要依赖**文件名**和 **Frontmatter 属性**。规范的元数据是精准检索的关键。

核心原则：
- 每篇笔记只聚焦一个具体场景
- Frontmatter 字段填写完整，尤其是 `type`、`tags`、`status`
- 使用 `[[wikilinks]]` 建立笔记间的交叉引用
- 前端（Frontmatter）变更通过 `related-*` 字段同步双向链接

## 与 AI Agent 配合使用

部署完成后，在 Claude Code 中输入：

```
/test-design 为订单退款功能设计测试用例
```

Agent 将自动：
1. 搜索知识库中的相关经验和因子
2. 提取业务规则和缺陷模式
3. 生成带知识来源标注的测试建议

详细说明参见主文档 [README.md](../README.md) 第 7 节。
