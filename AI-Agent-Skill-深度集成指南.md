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

### 3.1 推荐目录结构

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

### 3.2 Frontmatter 元数据规范

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

### 3.3 模板文件

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
## 关键检查点
- [ ]

## 易遗漏边界
- [ ]

## 异常路径
- [ ]

## 实际踩过的坑
## 问题描述
## 根因
## 解决方案
## 预防措施

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
## 有效等价类
| 类别 | 代表值 | 说明 |
|------|--------|------|
|      |        |      |

## 无效等价类
| 类别 | 代表值 | 预期结果 |
|------|--------|----------|
|      |        |          |

## 边界值
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

## 输入条件
| 条件项 | 说明 | 取值/范围 |
|--------|------|-----------|
|        |      |           |

## 处理逻辑
```
用伪代码或流程图描述处理过程
```

## 输出结果
| 场景 | 预期结果 | 错误码 |
|------|---------|--------|
| 正常 |         |        |
| 异常 |         |        |

## 关联约束
> 此规则与其他规则的依赖或互斥关系，如「规则 A 在规则 B 之前执行」

## 验证方法
## 正向验证
- [ ]

## 反向验证
- [ ]

## 边界验证
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
## 直接原因

## 深层原因

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
## 静态检测（代码审查/工具扫描）
- [ ] 关键字搜索：
- [ ] 代码模式匹配：

## 动态检测（测试手段）
- [ ] 测试方法：
- [ ] 测试工具：

## 监控告警（线上发现）
- [ ] 监控指标：
- [ ] 告警阈值：

## 修复方案
## 临时方案（止血）

## 根治方案

## 预防措施
> 如何在设计和开发阶段就避免此类问题？

## 设计阶段
- [ ]

## 开发阶段
- [ ]

## 测试阶段
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

### 3.4 模板填报示例

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
## 关键检查点
- [ ] 退款金额 = 实付金额（不含已均摊优惠券部分）
- [ ] 退款金额 + 优惠券退回金额 = 原始支付金额
- [ ] 部分退款时金额计算精度（分），不可四舍五入进/舍

## 易遗漏边界
- [ ] 连续点击「申请退款」按钮 → 期望只创建一条退款单
- [ ] 退款审核中，用户取消退款 → 订单状态恢复正常
- [ ] 超过可退款时效（7天）→ 退款入口不可见

## 异常路径
- [ ] 退款时支付渠道接口超时 → 30秒重试，仍失败转人工
- [ ] 退款中订单被风控拦截 → 退款挂起，通知人工处理

## 实际踩过的坑
## 问题描述
压测中发现快速双击「申请退款」可创建两条退款单，均通过「是否已退款」检查。

## 根因
退款入口缺少分布式锁，两次请求同时读到了「未退款」状态。

## 解决方案
退款申请入口加 Redis 分布式锁：`refund:lock:{orderId}`

## 预防措施
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
## 有效等价类
| 类别 | 代表值 | 说明 |
|------|--------|------|
| 中文名 | "张三" | 2 个中文字符 |
| 英文名 | "John" | 4 个英文字符 |
| 中英混合 | "张San" | 中英混合 |
| 含数字 | "user001" | 常见真实场景 |
| 允许的特殊字符 | "user_name-001" | _ 和 - |
| 最短长度 | "ab" | 2 字符 (假设最小 2) |
| 最长长度 | "a" * 20 | 20 字符 (假设最大 20) |

## 无效等价类
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

## 边界值
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

## 输入条件
| 条件项 | 说明 | 取值/范围 |
|--------|------|-----------|
| 实付金额 | 用户实际支付金额 | ≥0.01 元，精确到分 |
| 平台优惠券 | 平台承担的优惠 | 退款时不退回 |
| 商家优惠券 | 商家承担的优惠 | 退款时按比例退回 |
| 退款比例 | 部分退款时的比例 | 0%~100% |

## 处理逻辑
```
if 全部退款:
    退款金额 = 实付金额 - 已均摊的平台优惠券金额
    退回优惠券 = 退回商家优惠券
elif 部分退款:
    退款金额 = 退款商品单价 × 退款数量 - 该商品均摊的平台优惠券金额
    退回优惠券 = 不退回（部分退款不退券）
```

## 输出结果
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
## 正向验证
- [ ] 全额退款金额 = 实付金额
- [ ] 部分退款金额 = 商品单价 × 数量

## 反向验证
- [ ] 退款金额超过实付金额时拒绝
- [ ] 已全额退款的订单再次退款被拒绝

## 边界验证
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
## 直接原因
接口实现中「检查状态 → 执行业务 → 更新状态」不是原子操作。
并发请求同时通过状态检查，各自独立执行业务逻辑。

## 深层原因
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
## 静态检测（代码审查/工具扫描）
- [ ] 关键字搜索：`update*ByStatus`、`save`（无唯一键约束的）
- [ ] 代码模式匹配：`if (status == xxx)` → `doAction()` → `updateStatus(yyy)` 缺少锁
- [ ] CR 检查清单：所有状态变更接口是否有幂等方案

## 动态检测（测试手段）
- [ ] 接口并发测试：JMeter/Locust 对状态变更接口做 10 并发
- [ ] 快速双击测试：人工快速双击提交按钮（前端+后端双层验证）
- [ ] 网络模拟：Chrome DevTools 限速 3G，验证防重机制

## 监控告警（线上发现）
- [ ] 监控指标：同一 userId+orderId+action 在 1 秒内 >1 次
- [ ] 告警阈值：检测到疑似重复操作，立即告警
- [ ] 业务指标：退款金额 > 实付金额，触发对账告警

## 修复方案
## 临时方案（止血）
- 待修复接口前加 API 网关限流（同一用户同一操作 1秒1次）
- 前端按钮点击后立即 disabled + loading 遮罩

## 根治方案
- 接口入口加 Redis 分布式锁：`lock:{action}:{orderId}`，TTL=30s
- 数据库加唯一约束：`UNIQUE KEY uk_order_refund (order_id)`
- 接口改造为幂等：客户端生成 `idempotent_key`，服务端缓存处理结果

## 预防措施
## 设计阶段
- [ ] 设计评审时识别所有「状态变更」类接口
- [ ] 对这类接口默认要求提供幂等方案（锁/令牌/唯一约束）

## 开发阶段
- [ ] IDE 代码模板：状态变更方法自动生成幂等令牌检查代码
- [ ] 单元测试必须覆盖并发场景（并发测试框架）

## 测试阶段
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

**测试方法模板** (`07-Templates/测试方法模板.md`)：

```markdown
---
type: test-method
method-name: ""       # 方法名称
method-alias: []      # 别名/英文名：Equivalence Partitioning / Boundary Value Analysis ...
method-category: ""   # 黑盒/白盒/灰盒/经验型/模型驱动
method-family: ""     # 所属方法族：等价类族/边界值族/因果图族/正交试验族/状态迁移族/探索性族
difficulty: ""        # 上手难度：入门/适中/进阶/专家
effectiveness: ""     # 有效性评分：★☆☆☆☆ ~ ★★★★★
best-for: []          # 最适用的测试场景
not-for: []           # 不适用/效果差 的场景
requires: []          # 前置条件：需求文档/代码/状态图/领域知识...
deliverables: []      # 本方法产出的制品：测试项/测试用例/测试数据/状态树...
tags: []
related-methods: []   # [[相关/互补的测试方法]]
related-factors: []   # [[常配合的测试因子]]
source: ""
date: ""
status: "draft"
---

# {{title}}

## 一句话概述
> 用一句话描述这个方法是什么，用于解决什么问题。

## 核心思想

## 适用条件
## 什么时候用这个方法？
-

## 什么时候不用这个方法？
-

## 使用此方法需要的输入
| 输入项 | 格式要求 | 获取途径 |
|--------|---------|---------|
|        |         |         |

## 操作步骤

## Step 1:
-

## Step 2:
-

## Step 3:
-

## Step 4: 产出物
-

## 常见误区
| 误区 | 正确做法 |
|------|---------|
|      |         |

## 与因子的配合
> 此方法一般配合哪些类型的因子？如何使用？

## 与经验的配合
> 过往经验中，此方法在哪些模块的哪个场景下应用效果最好？

## 输出示例
> 以此方法输出测试用例的格式样例

## 局限性
-

## 进阶技巧
-

## 相关方法对比
| 方法 | 异同点 | 选择建议 |
|------|--------|---------|
| [[另一个方法]] |  | 当 XXX 时用此法，YYY 时用彼法 |
```

**method-category 分类说明：**

| 分类 | 说明 | 代表方法 |
|------|------|---------|
| `黑盒` | 不看代码，仅基于需求和规格 | 等价类、边界值、判定表、因果图、场景法 |
| `白盒` | 看代码逻辑和实现 | 语句覆盖、分支覆盖、路径覆盖、MC/DC |
| `灰盒` | 兼顾需求和代码 | 接口测试、数据库测试 |
| `经验型` | 基于测试人员的经验和直觉 | 错误推测法、探索性测试 |
| `模型驱动` | 基于形式化模型生成测试 | 状态迁移、Model-Based Testing |

**method-family 说明：** 帮助 Agent 在方法之间做关联推荐。例如当用户选择「等价类划分法」时，Agent 自动建议「边界值分析法」（同族互补）。

---

**示例：等价类划分法**

```markdown
---
type: test-method
method-name: "等价类划分法"
method-alias: [Equivalence Partitioning, Equivalence Class Partitioning]
method-category: "黑盒"
method-family: "等价类族"
difficulty: "入门"
effectiveness: "★★★★☆"
best-for: [输入校验, 表单验证, 数据范围测试, 配置项测试]
not-for: [复杂状态流转, 性能测试, 多条件组合]
requires: [需求规格说明书, 接口文档, 字段定义]
deliverables: [等价类表, 测试用例(每个等价类至少一条)]
tags: [黑盒测试, 输入覆盖, 穷举优化, 基础方法]
related-methods:
  - "[[边界值分析法]]"
  - "[[判定表法]]"
related-factors:
  - "[[字符串输入因子]]"
  - "[[数值输入因子]]"
source: "ISTQB Foundation Level Syllabus"
date: "2026-01-01"
status: "published"
---

# 等价类划分法

## 一句话概述
> 将程序的输入域划分为若干子集（等价类），从每个子集中选取少数代表值进行测试，用有限测试覆盖无限输入。

## 核心思想
- 同一等价类中的任意输入，对程序的测试效果等价
- 测试一个代表值 = 测试了整个等价类
- 核心是"以少代多"，大幅减少测试用例数量

## 适用条件
## 什么时候用这个方法？
- 输入是连续或有规律的值范围
- 输入有明确的"有效"和"无效"区分
- 需要减少测试用例数量但又不想牺牲覆盖

## 什么时候不用这个方法？
- 测试目标不是输入覆盖（如性能测试）
- 输入之间高度耦合，不能独立划分等价类
- 需要精确覆盖所有可能输入（如安全测试中的 Fuzzing）

## 使用此方法需要的输入
| 输入项 | 格式要求 | 获取途径 |
|--------|---------|---------|
| 字段定义 | 字段名、类型、约束 | PRD、接口文档 |
| 取值范围 | 有效范围、无效范围 | 需求规格 |
| 业务规则 | 合法性检查逻辑 | 业务规则文档 |

## 操作步骤

## Step 1: 识别输入项
从需求中提取所有需要测试的输入字段。

## Step 2: 划分有效等价类
对每个输入项，按以下维度划分：
- 数值型：正常范围、正/负值、零值
- 字符型：正常字符、空串、超长、特殊字符
- 布尔型：true、false
- 枚举型：每个合法取值

## Step 3: 划分无效等价类
对每个输入项，反着来：
- 超出范围的值
- 不允许的类型
- 空值/null
- 非法格式

## Step 4: 设计测试用例
- 有效用例：一条用例覆盖尽可能多的有效等价类
- 无效用例：一条用例只覆盖一个无效等价类（避免"屏蔽效应"）

## Step 5: 产出物
等价类表 + 测试用例集

## 常见误区
| 误区 | 正确做法 |
|------|---------|
| 把等价类等同于随机值 | 等价类是"有代表性的精选取值"，不是随便挑 |
| 一条用例覆盖多个无效等价类 | 一个无效等价类可能掩盖另一个的错误提示 |
| 只做有效等价类 | 无效等价类往往是 Bug 高发区 |
| 用此方法替代所有其他方法 | 等价类+边界值是最小组合，不能省略边界值 |

## 与因子的配合
等价类划分法直接使用因子库中的因子定义：
- 读取 `02-Factors/` 下的相关因子
- 每个因子的「有效等价类」和「无效等价类」表可以直接作为用例设计的输入
- 因子中的「边界值」表补充边界值分析

## 与经验的配合
- 从 `01-Experiences/` 中查找同模块的经验笔记
- 看「易遗漏边界」部分 → 补充到等价类中
- 看「实际踩过的坑」→ 确认是否有等价类遗漏

## 输出示例
| 用例ID | 输入 | 覆盖等价类 | 预期结果 |
|--------|------|-----------|---------|
| TC-EC-001 | 用户名="张三", 密码="Abc12345", 手机="13800138000" | 所有有效 | 注册成功 |
| TC-EC-002 | 用户名="" | 用户名为空(无效) | 提示"用户名不能为空" |
| TC-EC-003 | 用户名="a"*21 | 用户名超长(无效) | 提示"用户名最长20字符" |
| TC-EC-004 | 手机="12345" | 手机号格式(无效) | 提示"请输入正确手机号" |

## 局限性
- 不处理输入之间的组合关系（需要因果图/判定表/正交试验）
- 不适合时序相关测试（需要状态迁移法）
- 等价类的划分依赖测试人员的经验，不同人划分不同

## 进阶技巧
- 等价类 + 边界值联合使用：先划分等价类，再对每个等价类取边界值
- 结合因子库：因子库中的因子已经维护了等价类，直接引用
- 对安全相关字段（密码、金额）额外增加等价类：SQL 注入、XSS、Unicode

## 相关方法对比
| 方法 | 异同点 | 选择建议 |
|------|--------|---------|
| [[边界值分析法]] | 同属"输入覆盖"，边界值是等价类的补充 | 等价类定义"范围"，边界值确定"临界点"，必须配合使用 |
| [[判定表法]] | 判定表处理输入组合，等价类处理单输入 | 单个输入用等价类，多个输入有组合关系用判定表 |
| [[错误推测法]] | 错误推测靠经验，等价类靠系统性分析 | 等价类做完后，用错误推测补充遗漏 |
```

**示例：状态迁移法**

```markdown
---
type: test-method
method-name: "状态迁移法"
method-alias: [State Transition Testing, Finite State Machine Testing]
method-category: "模型驱动"
method-family: "状态迁移族"
difficulty: "进阶"
effectiveness: "★★★★★"
best-for: [订单流程, 审批流, 用户生命周期, 支付流程, 退款流程, 工单系统]
not-for: [无状态服务, 纯计算类接口, 简单CRUD]
requires: [状态机图, 状态转移矩阵, 业务规则文档]
deliverables: [状态转移树/表, N-switch覆盖用例, 非法转移用例]
tags: [状态机, 流转规则, 模型驱动, 全路径覆盖]
related-methods:
  - "[[场景法]]"
  - "[[判定表法]]"
related-factors:
  - "[[并发状态因子]]"
source: "ISTQB Advanced Level - Test Analyst"
date: "2026-02-01"
status: "published"
---

# 状态迁移法

## 一句话概述
> 将系统建模为有限状态机，测试所有合法的状态转移路径，以及所有非法的状态转移被正确拒绝。

## 核心思想
- 系统的行为取决于其当前状态 + 触发事件
- 测试要覆盖「状态 × 事件」的有效组合
- 非法转移必须被拒绝并给出正确的错误处理

## 适用条件
## 什么时候用这个方法？
- 被测对象有明确的状态和状态转移规则
- 状态之间有先后顺序依赖
- 同一事件在不同状态下行为不同

## 什么时候不用这个方法？
- 系统无状态概念
- 状态过多（>100个），状态迁移图过于复杂
- 状态转移没有明确的业务规则定义

## 使用此方法需要的输入
| 输入项 | 格式要求 | 获取途径 |
|--------|---------|---------|
| 状态机图 | 所有状态 + 合法转移路径 | 设计文档、状态机图 |
| 状态转移规则 | 事件 → 当前状态 → 目标状态 → 约束条件 | 业务规则文档 |
| 非法转移定义 | 哪些转移不允许 | 业务规则 + 测试经验 |

## 操作步骤

## Step 1: 识别所有状态
从需求中提取所有合法的系统状态。

## Step 2: 构建状态转移矩阵
| 当前状态 \ 事件 | 事件A | 事件B | 事件C |
|----------------|-------|-------|-------|
| 状态1 | 状态2 | 非法 | 状态3 |
| 状态2 | 状态1 | 状态3 | 非法 |
| 状态3 | 非法 | 非法 | 状态1 |

## Step 3: 设计 0-switch 用例
覆盖所有合法的单步转移（从状态A经事件X到状态B）。

## Step 4: 设计 1-switch 用例
覆盖所有合法的两步转移（A→B→C），这是大部分 Bug 的来源。

## Step 5: 设计非法转移用例
尝试所有非法转移，验证被正确拒绝。
Bug 高发区：非法转移时有异常但未回滚状态。

## Step 6: 产出物
状态转移树 + N-switch 用例集 + 非法转移用例

## 常见误区
| 误区 | 正确做法 |
|------|---------|
| 只覆盖合法转移 | 非法转移是 Bug 高发区，必须覆盖 |
| 只测 0-switch | 1-switch 覆盖两步转移，发现的 Bug 量通常是 0-switch 的 2~3 倍 |
| 忽略并发状态冲突 | 两个事件同时到达时应如何处理 |
| 假设状态转移原子化 | 转移过程中任何步骤失败，状态应回滚 |
| 终点状态没有退出路径 | 有些业务需要从"终态"退回到"活跃态"（如撤销删除） |

## 与因子的配合
- 状态迁移法优先配合 `02-Factors/状态因子/` 中的状态定义因子
- 每个因子中「状态转移矩阵」可以直接导入状态迁移法
- 并发场景使用 `02-Factors/组合因子/` 中的并发状态因子

## 与经验的配合
- 从 `01-Experiences/` 中查找同模块的经验
- 「实际踩过的坑」中标记的非法转移未处理、状态回滚失败 → 作为重点测试
- 「易遗漏边界」中的并发状态冲突 → 增加并发状态覆盖

## 输出示例
| 用例ID | 起始状态 + 事件 | 目标状态 | 预期结果 |
|--------|---------------|---------|---------|
| TC-ST-001 | 待支付 + 支付成功 | 已支付 | 订单状态更新为已支付 |
| TC-ST-002 | 已支付 + 发货 | 已发货 | 状态更新，物流单号生成 |
| TC-ST-003 | 已发货 + 签收 | 已完成 | 订单完成，触发评价 |
| TC-ST-004 | 待支付 + 超时 | 已取消 | 订单取消，库存回滚 |
| TC-ST-005 | 已支付 + 支付成功(重复) | 已支付(不变) | 幂等提示"已支付" |
| TC-ST-006 | 已完成 + 退款 | 已完成(不变) | 拒绝 + "已完成不可退款" |

## 局限性
- 状态过多时矩阵爆炸（需要基于风险的抽样策略）
- N-switch > 2 时用例数指数增长（一般做到 1-switch 即可）
- 需要准确的状态机图，需求文档经常缺失

## 进阶技巧
- 用 Obsidian Canvas 或 Excalidraw 画状态机图，可直接嵌入笔记
- 结合业务规则库中的状态流转规则，半自动生成状态转移矩阵
- 非法转移用例设计时结合错误推测法，补充非典型攻击路径

## 相关方法对比
| 方法 | 异同点 | 选择建议 |
|------|--------|---------|
| [[场景法]] | 场景法按业务流程设计，状态法按状态转移设计 | 有状态但有多个参与者选场景法；单对象多状态选状态法 |
| [[因果图法]] | 因果图测输入组合，状态图测状态转移 | 输入之间有依赖用因果图；状态之间有流转用状态图 |
| [[判定表法]] | 都用于处理条件组合 | 条件无时序用判定表；条件有时序用状态迁移 |
```

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
