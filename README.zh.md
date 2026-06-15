# build-forward

<p align="center">
  <b>Agent 的新想法防火墙 — 先分类，不推倒。</b>
</p>

<p align="center">
  <a href="https://github.com/dmlin7777777/build-forward/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="许可: MIT"></a>
  <a href="SKILL.md"><img src="https://img.shields.io/badge/skill-v1.1.0-brightgreen" alt="版本 1.1.0"></a>
  <a href="#优化记录"><img src="https://img.shields.io/badge/评分-82%2F100-orange" alt="质量评分 82/100"></a>
</p>

<p align="center">
  <a href="#加载前后对比">效果演示</a> ·
  <a href="#快速开始">快速开始</a> ·
  <a href="#工作原理">工作原理</a> ·
  <a href="SKILL.md">完整协议</a> ·
  <a href="test-prompts.json">测试场景</a>
</p>

---

## 这是什么？

**AI Agent 在新想法出现时的默认行为是推倒重写。`build-forward` 阻止这件事。**

它是一个五步决策协议，以 Skill 形式加载。当你在开发中途说"我突然想到……"时，Agent 不再推平你的已有代码，而是暂停、分类、评估影响范围，并在动手前征求你的意见。控制权始终在你手里，已有代码保持完好。

---

## 加载前后对比

**没有 `build-forward`**（大多数 Agent 的默认行为）：

```
用户："我突然想到……能不能加个实时协作功能？"
Agent："好主意！让我把 CRUD 接口重写成 WebSocket……"
       [开始重构认证层、改数据库 schema、破坏现有 API]

用户："等等，我只是想先把基础博客做完……"
Agent：[已经把 3 个文件改得面目全非]
```

**有 `build-forward`**（加载后的 Agent 行为）：

```
用户："我突然想到……能不能加个实时协作功能？"
Agent："[build-forward] 分类：C 类（扩展功能，新场景）
       建议进 inbox 并 24 小时冷却。实时协作涉及 WebSocket、
       OT 算法、DB schema 变更——这是单向门改动。
       继续当前博客 CRUD 开发？"

用户："对，先把博客做完。"
Agent：[不受干扰，继续博客 CRUD]
```

> 就是一个 Skill 的区别。详见 [`test-prompts.json`](test-prompts.json) 中的 8 个可复现验证场景。

---

## 为什么需要 build-forward？

每个 AI 辅助开发者都踩过同一个坑：正在做功能 A，脑子里蹦出功能 B 的想法，Agent 为了"给 B 腾地方"重写了半个代码库，结果全崩了。这不是工具问题——是**纪律**问题。Agent 没有纪律。

`build-forward` 给它加了一套协议：

| 痛点 | build-forward 怎么做 |
|------|---------------------|
| Agent 不思考直接动手改代码 | **先分类**——改一行代码之前，每个想法先标 A/B/C |
| Agent 为"腾地方"推平已有功能 | **消费者审计**——先数清楚有几个调用方再决定建不建 |
| Agent 不区分改动大小，一视同仁 | **评估破坏性**——单向门改动必须暂停等确认 |
| Agent "为未来"过度抽象 | **数清楚再抽象**——0 个消费者 = 不建；1 个 = 内联 |
| Agent 反复写同样的模式 | **重复报警**——≥3 次触发 stop-and-ask |

---

## 快速开始

**1. 安装**

```bash
# OpenClaw / WorkBuddy
openclaw skills install build-forward

# 或直接 clone
git clone https://github.com/dmlin7777777/build-forward.git ~/.workbuddy/skills/build-forward
```

**2. 正常开发。** 当你说以下关键词时，Skill 自动触发：

| 触发词（中文） | 触发词（English） |
|---------------|-------------------|
| "我突然想到…" | "I just thought of…" |
| "要不要顺便…" | "can we also…" |
| "能不能改成…" | "what if we change…" |
| "加个功能…" | "let's also add…" |

**3. Agent 会暂停、分类、建议**——而非重写一切。

**4. 验证是否生效：** 开发中途说"要不要顺便加个支付系统？" Agent 应输出 C 类分类 + 24 小时冷却建议。

---

## 工作原理

```mermaid
graph TD
    START[💡 新想法到达] --> L1

    subgraph "铁律一 — 分类"
        L1{什么类型？}
        L1 -->|"A — 堵漏（当前路径断裂）"| L2
        L1 -->|"B — 优化（能用但不顺）"| ASK1[问用户：现在做还是进 inbox？]
        L1 -->|"C — 扩展（全新场景）"| INBOX[📥 进 inbox + 24h 冷却]
    end

    ASK1 -->|"现在做"| L2
    ASK1 -->|"进 inbox"| INBOX
    INBOX --> END[🏁 结束 — 冷却后重新评估]

    subgraph "铁律二 — 评估破坏性"
        L2{是单向门？}
        L2 -->|"是 — DB schema、API、全局状态"| CHECK1((🔴 CHECKPOINT))
        L2 -->|"否 — UI、新字段、新路由"| L3
    end

    CHECK1 -->|"输出影响矩阵 → 等用户确认"| L3

    subgraph "铁律三 — 消费者审计"
        L3{几个调用方？}
        L3 -->|"0"| SKIP[❌ 不建]
        L3 -->|"1"| INLINE[最简实现 — 不抽象]
        L3 -->|"2"| EXTRACT[可提取 — 不泛化]
        L3 -->|"≥3"| ABSTRACT[可考虑抽象层]
    end

    subgraph "铁律四 — 选接入模式"
        INLINE --> L4
        EXTRACT --> L4
        ABSTRACT --> L4
        L4[选破坏性最低的路径]
        L4 --> WRAP[1. Wrap]
        WRAP --> EXTEND[2. Extend]
        EXTEND --> BRANCH[3. Branch]
        BRANCH --> REPLACE[4. Replace — 最后手段]
    end

    subgraph "铁律五 — 重复报警"
        REPLACE --> L5{≥3 次重复？}
        L5 -->|"是"| CHECK2((🔴 CHECKPOINT))
        L5 -->|"否"| ALLCLEAR
    end

    CHECK2 -->|"问用户：现在统一还是进 inbox"| ALLCLEAR
    ALLCLEAR[✅ 最终确认门] --> CODE[开始写代码]
    SKIP --> END
```

---

## 五条铁律概要

### 铁律一 — 先分类，不动手

| 类型 | 标准 | 默认处置 |
|------|------|---------|
| **A — 堵漏** | 当前主路径断裂，必须修 | 立即处理 → 铁律二 |
| **B — 优化** | 体验更好，但当前能用 | 问用户：现在做还是进 inbox？ |
| **C — 扩展** | 全新功能/场景 | 进 inbox + 24h 冷却 |

24 小时冷却是有意设计：大多数功能冲动要么消散，要么沉淀为真实需求。

### 铁律二 — 评估破坏性

| 门类型 | 例子 | 处置 |
|--------|------|------|
| **双向门**（可逆） | UI 微调、新字段、新函数、新路由 | 继续 |
| **单向门**（难反悔） | DB schema、公开 API、文件删除、全局状态、核心依赖升级 | 🔴 **CHECKPOINT** — 输出影响矩阵，等用户确认 |

### 铁律三 — 消费者审计

动手前先数调用方。

| 消费者数 | 处置 |
|---------|------|
| **0** | 不建 |
| **1** | 最简内联 — 不抽象 |
| **2** | 可提取 — 不泛化 |
| **≥3** | 可考虑抽象层 |

从源头杜绝"为未来而建"。

### 铁律四 — 选接入模式

按破坏性从低到高：

**Wrap** → **Extend** → **Branch** → **Replace**（最后手段）

### 铁律五 — 重复报警

同一逻辑 ≥3 次重复 → 🔴 **CHECKPOINT**。问用户：现在统一，还是进 inbox？

> 完整协议（含边界场景和自纠正规则）见 [`SKILL.md`](SKILL.md)。

---

## 安全声明

**本 Skill 是决策协议，不是执行引擎。** 它不会删除文件、修改数据库、执行 shell 命令、自动 commit、发送网络请求、或替你决定单向门操作。最大破坏半径：略微降低开发节奏——这是设计意图，不是 bug。

---

## Skill 生态

`build-forward` 填补了"要做什么？"和"怎么安全地做？"之间的空白。

| Skill | 角色 | 时机 |
|-------|------|------|
| `brainstorming` / `grill-me` | 需求澄清 | 写代码之前 |
| **`build-forward`** | **新想法防火墙** | **开发中途新想法到达时** |
| `ideas-inbox` | B/C 类归档 + 冷却跟踪 | 分类完成后 |
| `vibecoding-workflow` | 执行纪律 | 接入路径确定后 |
| `incremental-implementation` | 分步稳妥实现 | 已知需求，如何安全执行 |

完整链路：**brainstorming → build-forward → ideas-inbox → vibecoding-workflow / incremental-implementation**

---

## 优化记录

| 日期 | 版本 | 评分变化 | 方法 |
|------|------|---------|------|
| 2026-06-15 | v1.1.0 | 66 → 82 | 鲁班八步打磨 |
| 2026-05-31 | v1.0.0 | 79.6 → 85.0 | Darwin Skill 九维优化 |

---

## 参与贡献

1. **发现边界场景？** 提 Issue 附上场景描述
2. **有测试用例？** 加到 [`test-prompts.json`](test-prompts.json)
3. **想改进协议？** 先读 [`SKILL.md`](SKILL.md)，再提 PR

---

## 许可证

MIT © [dmlin7777777](https://github.com/dmlin7777777)
