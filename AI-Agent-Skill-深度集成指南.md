# Obsidian + AI Agent Skill 深度集成指南

> 最后更新：2026年5月 | 基于 [Obsidian 部署完全指南](./README.md)

本文档从 [Obsidian 部署完全指南](./README.md) 中独立出来，聚焦于 **AI Agent Skill 与 Obsidian 知识库的深度集成**，涵盖三种集成方案的完整实施步骤、知识库结构设计、模板示例、Skill 开发与部署脚本。

---

## 目录

1. [整体架构](#1-整体架构)
2. [集成方案对比](#2-集成方案对比三种方案按需选择)
3. [知识库结构设计（测试场景）](#3-知识库结构设计测试场景)
4. [方案A：文件系统直读（入门必选）](#4-方案a文件系统直读入门必选)
5. [方案C：kepanoobsidian-skills（强烈推荐）](#5-方案ckepanoobsidian-skills强烈推荐)
6. [实战：创建「Test Design Skill」](#6-实战创建test-design-skill)
7. [进阶：辅助 Skills](#7-进阶辅助-skills)
8. [方案B：MCP + Smart Connections（大仓库进阶）](#8-方案bmcp--smart-connections大仓库进阶)
9. [一键部署脚本](#9-一键部署脚本)
10. [关键注意事项](#10-关键注意事项)
11. [扩展：其他领域的知识库架构](#11-扩展其他领域的知识库架构)

---

> 核心目标：让 AI Agent（Claude Code / Codex CLI / 自定义 Skill）能够读取、搜索、引用你的 Obsidian 知识库，在自动化测试设计、代码审查、问题排查等场景中，给出基于真实经验积累的精准建议。

## 1. 整体架构

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

## 2. 集成方案对比（三种方案按需选择）

| 方案 | 原理 | 复杂度 | 搜索方式 | 适用场景 |
|------|------|--------|---------|---------|
| **A. 文件系统直读** | Agent 用 Read/Grep 直接读 .md 文件 | 极低 | 关键词匹配 (ripgrep) | 入门、小仓库 |
| **B. MCP + REST API** | Obsidian 插件暴露 HTTP API + 语义搜索 | 中 | 关键词 + 语义搜索 | 大仓库、需智能检索 |
| **C. obsidian-skills** | kepano 开源 Skills，教 AI 正确读写 Obsidian 格式 | 低 | 配合 A/B 使用 | 语法正确性、Canvas/Bases |

**推荐策略：方案A（入门必选）+ 方案C（强烈推荐）→ 方案B（仓库超过500篇笔记时再上）**

## 3. 知识库结构设计（测试场景）

> 完整的知识库结构设计、Frontmatter 元数据规范、5 种模板内容、填报示例已直接内置在仓库的 `Testing-Knowledge-Vault/` 目录中。
>
> → 详见 **[Testing-Knowledge-Vault/README.md](./Testing-Knowledge-Vault/README.md)**

### 3.1 目录结构速览

```
Testing-Knowledge-Vault/
├── 00-Inbox/                    # 待整理的经验碎片
├── 01-Experiences/              # 测试经验库
├── 02-Factors/                  # 测试因子库
├── 03-Business-Rules/           # 业务规则库
├── 04-Bug-Patterns/             # 缺陷模式库
├── 05-Test-Methods/             # 测试方法论
├── 06-Cases/                    # 经典测试用例库
├── 07-Templates/                # 笔记模板（5个）
└── 08-MOCs/                     # 索引/地图
```

### 3.2 元数据速查

Agent 搜索主要依赖 **Frontmatter 属性**。核心字段：

| 笔记类型 | `type` 值 | 核心检索字段 |
|---------|----------|-------------|
| 测试经验 | `testing-experience` | `module`, `test-type`, `tags`, `related-factors` |
| 测试因子 | `test-factor` | `category`, `factor-name`, `tags` |
| 业务规则 | `business-rule` | `domain`, `rule-id`, `rule-category`, `tags` |
| 缺陷模式 | `bug-pattern` | `category`, `pattern-name`, `tags` |
| 测试方法 | `test-method` | `method-category`, `method-family`, `tags` |

> 完整 Frontmatter 规范及 YAML 示例见 [Testing-Knowledge-Vault/README.md](./Testing-Knowledge-Vault/README.md)

---

## 4. 方案A：文件系统直读（入门必选）

**原理：** Agent 原生就支持 Read/Grep 工具，直接读取 Obsidian vault 中的 `.md` 文件。零额外配置。

### Step 1：在项目 CLAUDE.md 中声明知识库路径

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

### Step 2：Agent 自动搜索流程

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

## 5. 方案C：kepano/obsidian-skills（强烈推荐）

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

### 5.1 obsidian-markdown — 教 Agent 说 Obsidian 的"方言"

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

### 5.2 obsidian-bases — 把你的笔记变成可筛选、可排序的数据视图

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

### 5.3 json-canvas — 让 Agent 画可视化的测试策略图

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

### 5.4 obsidian-cli — 命令行直接操控 Obsidian

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

### 5.5 四合一实战：完整工作流演示

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

### 5.6 Skill 组合使用速查

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

## 6. 实战：创建「Test Design Skill」

这是最关键的部分——让 Agent 在做测试设计时自动搜索知识库并引用。

### 6.1 SKILL.md 完整内容

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

## Phase 1：信息收集
1. 明确被测对象（模块、功能、接口）
2. 提取搜索关键词（中文 + 英文）
3. 识别涉及的测试类型

## Phase 2：知识库搜索（必须执行，不可跳过）

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

## Phase 3：知识提取
从搜索到的笔记中提取：
- 「关键检查点」「易遗漏边界」「异常路径」（经验库）
- 「等价类」「边界值」「组合策略」（因子库）
- 「业务约束」（业务规则库）
- 「典型缺陷」（缺陷模式库）

## Phase 4：生成输出

## 输出格式

## 1. 知识库命中情况
| 知识库 | 命中笔记 | 关联度 |
|--------|---------|--------|
| 经验库 | [[笔记名]] | 高/中/低 |

## 2. 测试策略建议
- 测试重点（基于经验库历史踩坑）
- 测试优先级
- 推荐测试方法

## 3. 测试因子分析
| 因子 | 来源 | 等价类 | 边界值 | 异常值 |
|------|------|--------|--------|--------|

## 4. 测试用例
| 用例ID | 场景 | 前置条件 | 步骤 | 预期结果 | 知识来源 |
|--------|------|---------|------|---------|---------|

## 5. 易遗漏检查点 ⚠️
- 基于经验库「实际踩过的坑」章节

## 6. 风险评估
- 基于缺陷模式库的关联分析

## 质量要求
- 每个基于知识库的建议必须标注来源笔记名
- 明确区分「来自知识库的经验」和「来自通用方法论」
- 知识库没有的信息不编造，使用通用方法论补充并注明
- 输出格式便于后续导入 Obsidian
```

### 6.2 安装 Skill

```bash
mkdir -p ~/.claude/skills/test-design
# 将上面的 SKILL.md 内容写入
cat > ~/.claude/skills/test-design/SKILL.md <<'SKILLEOF'
# ... 上面的完整 SKILL.md 内容 ...
SKILLEOF
```

### 6.3 使用演示

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

## 7. 进阶：辅助 Skills

### 7.1 test-capture（沉淀经验 Skill）

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

### 7.2 test-kb-health（知识库健康检查 Skill）

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

## 8. 方案B：MCP + Smart Connections（大仓库进阶）

> 当你的 vault 超过 500 篇笔记，关键词搜索不够精准时，升级到此方案。

### Install Obsidian 插件

在 Obsidian 社区插件市场搜索并安装：

1. **Local REST API** — 为 Obsidian 提供 HTTP API（端口 27124）
2. **Smart Connections** — 基于 embedding 的语义搜索
3. **MCP Tools** — 将 Obsidian 作为 MCP Server 暴露

### 验证 API 可用

```bash
# 获取 vault 信息
curl -X GET "http://127.0.0.1:27124/vault/" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 语义搜索笔记
curl -X GET "http://127.0.0.1:27124/smart-connections/search?query=退款并发安全" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### 注册为 MCP Server

```bash
claude mcp add obsidian-knowledge \
  --command "npx" --args "-y" "mcp-obsidian" \
  --env OBSIDIAN_API_KEY="你的Key" \
  --env OBSIDIAN_API_URL="http://127.0.0.1:27124"
```

## 9. 一键部署脚本

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

## 10. 关键注意事项

1. **知识库预热**：首次使用每个目录至少手动填充 3-5 篇笔记，空知识库对 Agent 无帮助
2. **粒度控制**：每篇经验笔记聚焦一个具体场景，不要写成「大而全」文档
3. **持续积累**：每次发现新的测试盲区或踩坑，立即用 `/test-capture` 沉淀
4. **元数据规范**：严格遵循模板的 Frontmatter 字段，这是 Agent 检索的关键
5. **双向链接**：多用 `[[链接]]` 关联相关知识，Agent 会顺着链接发现更多内容
6. **多语言关键词**：笔记中同时包含中文和英文关键词，提高搜索命中率
7. **隐私安全**：知识库是纯本地 Markdown，Agent 本地运行，数据不上传云端
8. **Git 备份**：知识库是你的核心资产，用 Git 管理并定期 push 到远程仓库

## 11. 扩展：其他领域的知识库架构

| 领域 | Vault 示例 | 核心目录 | 配套 Skill |
|------|-----------|---------|------------|
| 代码审查 | Code-Review-Vault | 代码异味库/安全漏洞库/重构模式库 | `/code-review` |
| 技术方案 | Tech-Design-Vault | 架构模式库/技术选型库/踩坑记录 | `/tech-design` |
| 运维排障 | Ops-Vault | 告警模式库/故障案例库/恢复流程库 | `/ops-debug` |
| 需求分析 | Req-Vault | 需求模式库/干系人库/验收标准库 | `/req-analysis` |
| 项目管理 | PM-Vault | 风险库/干系人模式库/复盘模板 | `/pm-review` |

**统一原则：** 每个 Vault 独立，Skill 可跨 Vault 搜索；保持统一的 Frontmatter 规范；全部用 Git 管理。
