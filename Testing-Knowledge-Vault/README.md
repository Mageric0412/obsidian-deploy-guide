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
├── 01-Experiences/        # 测试经验库（核心）
│   ├── 功能测试/           # 按测试类型组织
│   ├── 性能测试/
│   ├── 安全测试/
│   └── 兼容性测试/
├── 02-Factors/            # 测试因子库（核心）
│   ├── 输入因子/           # 等价类、边界值
│   ├── 环境因子/           # 系统、网络、硬件
│   ├── 状态因子/           # 状态机、生命周期
│   └── 组合因子/           # 正交/因果组合
├── 03-Business-Rules/     # 业务规则库
├── 04-Bug-Patterns/       # 缺陷模式库
├── 05-Test-Methods/       # 测试方法论
├── 06-Cases/              # 经典测试用例库
├── 07-Templates/          # 笔记模板（5个模板文件）
└── 08-MOCs/               # 索引/地图（Map of Content）
```

**各目录说明：**

| 目录 | 存储内容 | 笔记 `type` | 搜索时机 |
|------|---------|------------|---------|
| `01-Experiences/` | 测试中积的经验教训 | `testing-experience` | 每次测试设计必查 |
| `02-Factors/` | 测试因子、等价类、边界值 | `test-factor` | 设计测试用例时查 |
| `03-Business-Rules/` | 业务规则、计算公式、约束 | `business-rule` | 分析需求时查 |
| `04-Bug-Patterns/` | 可复用的缺陷模式 | `bug-pattern` | 设计检查点时查 |
| `05-Test-Methods/` | 系统化测试方法指南 | `test-method` | 选择测试策略时查 |
| `06-Cases/` | 可复用的经典测试用例 | - | 参考已有用例 |
| `07-Templates/` | 创建新笔记的模板 | - | 新建笔记时使用 |
| `08-MOCs/` | 索引、导航页 | `moc` | 浏览知识库全貌 |

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

### 核心原则
- 每篇笔记只聚焦一个具体场景
- Frontmatter 字段填写完整，尤其是 `type`、`tags`、`status`
- 使用 `[[wikilinks]]` 建立笔记间的交叉引用
- Frontmatter 变更通过 `related-*` 字段同步双向链接

### 各类型 Frontmatter 规范

**测试经验** (`type: testing-experience`)：

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

**测试因子** (`type: test-factor`)：

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

**业务规则** (`type: business-rule`)：

```yaml
---
type: business-rule
domain: "电商"              # 电商/金融/社交/医疗/物流
module: "订单退款"
rule-id: "BR-ORDER-REFUND-001"
rule-category: "计算规则"   # 计算规则/校验规则/流程规则/权限规则/时间规则
priority: "P0"             # P0(违反即阻断)/P1(影响核心流程)/P2(影响体验)
scope: "模块"              # 全局/模块/场景
trigger-condition: "用户对已支付订单发起退款申请"
tags: [退款, 金额计算, 优惠券, 精度]
related-experiences: ["[[订单退款审批流测试经验]]"]
related-factors: ["[[金额计算因子]]"]
related-bug-patterns: ["[[浮点数精度丢失模式]]"]
source: "PRD v3.2 第4.3节"
date: "2026-02-20"
status: verified           # draft/verified/deprecated
deprecated-reason: ""      # 废弃时填写原因
---
```

**缺陷模式** (`type: bug-pattern`)：

```yaml
---
type: bug-pattern
pattern-name: "并发请求缺乏幂等守护"
pattern-id: "BUG-PTN-CONC-001"
category: "并发"            # 并发/数据一致性/边界/空值/类型转换/状态机/性能/安全/UI/集成
severity: "P0"             # 此模式通常导致的缺陷严重度
frequency: "高频"           # 高频/中频/低频/偶发
detection-difficulty: "极易遗漏"  # 极易/容易/中等/困难/极易遗漏
tags: [并发, 幂等, 分布式锁, 重复扣款, 超卖]
programming-languages: [Java, Go, Python]
tech-stack: [Redis, MySQL, Kafka]
related-experiences: ["[[订单退款审批流测试经验]]"]
related-factors: ["[[并发状态因子]]"]
related-rules: ["[[退款金额计算规则]]"]
date: "2026-03-15"
status: published
---
```

**测试方法** (`type: test-method`)：

```yaml
---
type: test-method
method-name: "等价类划分法"
method-alias: [Equivalence Partitioning]
method-category: "黑盒"    # 黑盒/白盒/灰盒/经验型/模型驱动
method-family: "等价类族"  # 等价类族/边界值族/因果图族/正交试验族/状态迁移族/探索性族
difficulty: "入门"         # 入门/适中/进阶/专家
effectiveness: "★★★★☆"
best-for: [输入校验, 表单验证, 数据范围测试]
not-for: [复杂状态流转, 性能测试, 多条件组合]
requires: [需求规格说明书, 接口文档]
deliverables: [等价类表, 测试用例]
tags: [黑盒测试, 输入覆盖, 基础方法]
related-methods: ["[[边界值分析法]]", "[[判定表法]]"]
related-factors: ["[[用户名输入因子]]"]
source: "ISTQB Foundation Level Syllabus"
date: "2026-01-01"
status: "published"
---
```

### 分类参考速查

**rule-category（业务规则分类）：**

| 类型 | 说明 | 举例 |
|------|------|------|
| `计算规则` | 涉及金额、积分、扣减等计算 | 满减优惠叠加计算规则 |
| `校验规则` | 输入的合法性校验 | 手机号格式/身份证校验 |
| `流程规则` | 状态流转、审批链 | 退款状态机流转规则 |
| `权限规则` | 角色权限、数据可见性 | 普通用户不能查看管理后台 |
| `时间规则` | 时效性约束 | 未支付订单 30 分钟自动取消 |

**bug-patterns category（缺陷模式分类）：**

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

**method-category（测试方法分类）：**

| 分类 | 说明 | 代表方法 |
|------|------|---------|
| `黑盒` | 不看代码，仅基于需求和规格 | 等价类、边界值、判定表、因果图、场景法 |
| `白盒` | 看代码逻辑和实现 | 语句覆盖、分支覆盖、路径覆盖、MC/DC |
| `灰盒` | 兼顾需求和代码 | 接口测试、数据库测试 |
| `经验型` | 基于测试人员的经验和直觉 | 错误推测法、探索性测试 |
| `模型驱动` | 基于形式化模型生成测试 | 状态迁移、Model-Based Testing |

### 已废弃内容的处理
- 不要删除笔记，改为 `status: deprecated` 并填写 `deprecated-reason`
- 保留历史可追溯性，方便 Agent 了解规则演变过程

## 与 AI Agent 配合使用

部署完成后，在 Claude Code 中输入：

```
/test-design 为订单退款功能设计测试用例
```

Agent 将自动：
1. 搜索知识库中的相关经验和因子
2. 提取业务规则和缺陷模式
3. 生成带知识来源标注的测试建议

→ 完整的 AI Agent 集成方案（三种架构、Skill 开发、部署脚本）请参见 **[AI Agent Skill 深度集成指南](../AI-Agent-Skill-深度集成指南.md)**
