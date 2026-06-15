# 三思

<p align="center">
  <b>三思而后行 — Agent 的新想法防火墙。</b>
</p>

<p align="center">
  <a href="https://github.com/dmlin7777777/think-thrice/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue" alt="许可: MIT"></a>
  <a href="SKILL.md"><img src="https://img.shields.io/badge/skill-v1.2.0-brightgreen" alt="版本 1.2.0"></a>
  <a href="#优化记录"><img src="https://img.shields.io/badge/过尺-81%2F100-orange" alt="鲁班过尺 81/100"></a>
  <a href="https://skills.sh/dmlin7777777/think-thrice"><img src="https://skills.sh/b/dmlin7777777/think-thrice" alt="skills.sh installs"></a>
</p>

<p align="center">
  <a href="#加载前后对比">效果演示</a> ·
  <a href="#快速开始">快速开始</a> ·
  <a href="#五条铁律">五条铁律</a> ·
  <a href="SKILL.md">完整协议</a> ·
  <a href="test-prompts.json">测试场景</a>
</p>

---

## 它解决什么问题

每个 AI 辅助开发者都踩过同一个坑：正在做功能 A，脑子里蹦出功能 B 的想法，Agent 为了"给 B 腾地方"重写了半个代码库，结果全崩了。这不是工具问题——是**纪律**问题。Agent 没有纪律。

`三思` 给它加了一套五步决策协议。当你说"我突然想到……"时，Agent 不再推平已有代码，而是暂停、分类、评估影响范围，并在动手前征求你的意见。控制权始终在你手里。

---

## 加载前后对比

<p align="center">
  <img src="assets/demo.png" alt="加载三思前后的 Agent 行为对比" width="100%">
</p>

> 就是一个 Skill 的区别。详见 [`test-prompts.json`](test-prompts.json) 中的 8 个可复现验证场景。

---

## 快速开始

**1. 安装**

```bash
# OpenClaw / WorkBuddy
openclaw skills install think-thrice

# npx（Claude Code / Codex / 任意 runtime）
npx skills add dmlin7777777/think-thrice

# 或直接 clone
git clone https://github.com/dmlin7777777/think-thrice.git ~/.workbuddy/skills/build-forward
```

**2. 正常开发。** 当你说以下关键词时，Skill 自动触发：

| 触发词（中文） | 触发词（English） |
|---------------|-------------------|
| "我突然想到…" | "I just thought of…" |
| "要不要顺便…" | "can we also…" |
| "能不能改成…" | "what if we change…" |
| "加个功能…" | "let's also add…" |

**3. 装完第一句话。** 对 Agent 说：

```text
我突然想到，要不要顺便加个支付系统？
```

Agent 应该暂停、输出 C 类分类 + 24 小时冷却建议——而非直接开始写支付代码。

**4. 更多验证场景** 见 [`test-prompts.json`](test-prompts.json) 和 [`examples/`](examples/)。

---

## 五条铁律

<p align="center">
  <img src="assets/five-laws-card.svg" alt="三思 · 五铁律速查卡" width="100%">
</p>

**铁律一 — 先分类，不动手。** 每个想法先标 A（堵漏，立即修）/ B（优化，问用户）/ C（扩展，进 inbox + 24h 冷却）。C 类冷却是有意设计：大多数功能冲动要么消散，要么沉淀为真实需求。

**铁律二 — 评估破坏性。** 双向门（UI、新字段、新路由）直接继续。单向门（DB schema、公开 API、文件删除、全局状态）→ 🔴 CHECKPOINT，输出影响矩阵，等用户确认。

**铁律三 — 消费者审计。** 动手前先数调用方。0 个不建，1 个内联不抽象，2 个可提取，≥3 个才考虑抽象层。从源头杜绝"为未来而建"。

**铁律四 — 选接入模式。** 按破坏性从低到高：**Wrap** → **Extend** → **Branch** → **Replace**（最后手段）。

**铁律五 — 重复报警。** 同一逻辑 ≥3 次重复 → 🔴 CHECKPOINT。问用户：现在统一，还是进 inbox？

> 完整协议（含边界场景、自纠正规则、降级路径）见 [`SKILL.md`](SKILL.md)。

---

## 安全声明

**本 Skill 是决策协议，不是执行引擎。** 它不会删除文件、修改数据库、执行 shell 命令、自动 commit、发送网络请求、或替你决定单向门操作。最大破坏半径：略微降低开发节奏——这是设计意图，不是 bug。

---

## Skill 生态

`三思` 填补了"要做什么？"和"怎么安全地做？"之间的空白。

| Skill | 角色 | 时机 |
|-------|------|------|
| `brainstorming` / `grill-me` | 需求澄清 | 写代码之前 |
| **`三思`** | **新想法防火墙** | **开发中途新想法到达时** |
| `ideas-inbox` | B/C 类归档 + 冷却跟踪 | 分类完成后 |
| `vibecoding-workflow` | 执行纪律 | 接入路径确定后 |
| `incremental-implementation` | 分步稳妥实现 | 已知需求，如何安全执行 |

完整链路：**brainstorming → 三思 → ideas-inbox → vibecoding-workflow / incremental-implementation**

---

## 优化记录

| 日期 | 版本 | 过尺评分 | 方法 |
|------|------|---------|------|
| 2026-06-15 | v1.2.0 | 81（独立验收） | 鲁班过尺·方案B精雕 |
| 2026-06-15 | v1.1.0 | 82（预估） | 鲁班八步打磨 |
| 2026-05-31 | v1.0.0 | 79.6 → 85.0 | Darwin Skill 九维优化 |

---

## 参与贡献

1. **发现边界场景？** 提 Issue 附上场景描述
2. **有测试用例？** 加到 [`test-prompts.json`](test-prompts.json)
3. **想改进协议？** 先读 [`SKILL.md`](SKILL.md)，再提 PR

---

## 许可证

MIT © [dmlin7777777](https://github.com/dmlin7777777)
