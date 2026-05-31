# build-forward

**An AI agent skill that integrates new ideas without reconstruction.**

> When a new idea appears mid-development, most AI agents default to rewriting. `build-forward` gives agents a protocol to grow the product forward instead.

Compatible with Claude Code, Cursor, Codex CLI, Gemini CLI, OpenClaw, and any agent that supports SKILL.md.

---

## The Problem

In AI-assisted development, ideas arrive faster than architectural planning. The typical agent response to "I want to add X" is to rewrite whatever is in the way.

This creates a painful loop:

```
New idea → Agent rewrites foundation → New idea → Agent rewrites again → ...
```

Existing skills like `incremental-implementation` solve *how to implement a known feature* incrementally. But none of them answer:

> **What do you do the moment a new idea arrives that might conflict with what you're already building?**

`build-forward` fills this gap.

---

## How It Works

Five iron laws, executed in order whenever a new idea appears. Triggered by phrases like "我突然想到" / "要不要顺便" / "能不能改成" / "加个功能."

### Iron Law 1 — Classify first, don't code

Every new idea gets a label before any code is written:

| Type | Criteria | Default action |
|------|----------|----------------|
| **A — Fix** | Breaks the current main path; must fix | Handle now, proceed to Iron Law 2 |
| **B — Polish** | Better UX, but works today | Ask user: now or inbox? |
| **C — Extend** | New feature or new scenario | Default: inbox, 24h cooldown |

The 24-hour cooldown for C-class prevents impulse development. Most feature urges either fade or crystallize into clearer requirements.

**When classification is ambiguous:**

| Trigger | First-line fix | Fallback |
|---------|---------------|----------|
| A vs B unclear | Treat as A (fix-first, err on the strict side) | — |
| B vs C unclear | Ask: "What's the cost of NOT doing this?" | User says "not urgent" → C; "affects UX" → B |
| C-class with security impact | Upgrade to A, handle immediately | — |

### Iron Law 2 — Assess destructiveness (one-way vs two-way doors)

Before touching anything, classify the change:

**Two-way door** (reversible) → proceed to Iron Law 3:
- UI/ style / copy changes
- New fields (don't delete old ones)
- New functions (don't change old signatures)
- New routes (don't change old ones)

**One-way door** (hard to reverse) → 🔴 CHECKPOINT · 🛑 STOP, output impact matrix:

```
单向门影响评估
═══════════════════════════════════════

直接改动
  [文件路径] — [改动内容简述]
  [接口名称] — [签名变更说明]

下游影响
  [消费者1] — [影响方式]
  （grep 确认，不编造）

回滚成本：🔴高 / 🟡中 / 🟢低
  理由：[为什么是这个评级]

建议
  □ 现在做 — [理由]
  □ 开新分支 — [理由]
  □ 记 inbox — [触发条件]
  □ 不接 — [为什么当前方案已足够]
```

See [`references/impact-matrix-template.md`](references/impact-matrix-template.md) for the canonical template with rollback cost rating standards.

### Iron Law 3 — Consumer audit (count callers before building)

Before creating any API, module, component, or utility:

> "Who consumes this? How many actual call sites exist right now?"

| Consumers | Action |
|-----------|--------|
| **0** | Don't build. Log to inbox as C-class. |
| **1** | Simplest inline implementation. No abstraction. |
| **2** | Extract if helpful, but no generalization. |
| **≥ 3** | Now consider an abstraction layer. |

This rule kills "building for the future" before it starts.

**When consumer count is uncertain:**

| Trigger | Action |
|---------|--------|
| External callers (3rd-party API consumers) | Treat as ≥3 conservatively; changes follow one-way door process |
| grep returns nothing but implicit calls suspected | Mark as "consumers unconfirmed", don't change yet, wait for runtime error to locate |
| Cross-repo / cross-project calls | Assume consumers exist. **Never delete or change old signatures**, only append |

### Iron Law 4 — Choose an integration mode

Pick the lowest-destructiveness path that works:

| Mode | When to use | What it means |
|------|-------------|---------------|
| **Wrap** | New feature enhances existing code | Add a layer outside; don't touch the original |
| **Extend** | New scenario shares most existing logic | Add a new branch/param/route; don't change existing paths |
| **Branch** | New and old paths diverge significantly | Implement in parallel, migrate consumers gradually |
| **Replace** | Old implementation is genuine liability | New impl + tests first, then delete old, then migrate. **Forbidden**: mixed old/new beyond one commit |

### Iron Law 5 — Duplication alert

When the same logic appears **≥ 3 times**:

1. 🔴 CHECKPOINT · 🛑 STOP: don't silently copy a third time.
2. Surface it: *"I've found [X logic] duplicated [N] times. Consolidate now or log to inbox?"*
3. User decides. Either way, document it.

1-2 occurrences are normal. 3 is the signal, not 2.

---

## Anti-Patterns This Prevents

- ❌ **默认重写 / Default rewrite** — seeing a new idea and immediately restructuring existing code
- ❌ **为未来消费者设计 / Zero-consumer abstraction** — building APIs/modules that nothing calls yet
- ❌ **C 类即时执行 / Impulse C-class execution** — shipping new feature ideas without 24h cooling
- ❌ **混合模式超期 / Mixed-mode limbo** — running old and new implementations simultaneously across multiple commits
- ❌ **静默蔓延 / Silent scope creep** — "while I'm here, I'll also fix..."

---

## vs. Similar Skills

| Skill | Trigger point | Core question |
|-------|--------------|---------------|
| `incremental-implementation` (Addy Osmani) | Known requirement | How to implement steadily |
| `vibecoding-workflow` | Every dev task | Am I staying in scope? |
| **`build-forward`** | **New idea arrives mid-dev** | **Which path causes least destruction?** |

These skills complement each other. `build-forward` handles the moment of idea arrival; `incremental-implementation` takes over once the integration path is decided.

---

## Darwin Optimization History

| Date | Version | Score | Δ | Changes |
|------|---------|-------|----|---------|
| 2026-05-31 | v1.0.0 | 79.6 → 85.0 | +5.4 | dim3: added classification ambiguity + consumer uncertainty fallback tables; dim4: added 🔴 CHECKPOINT visual markers at 3 checkpoints; dim6: added `references/impact-matrix-template.md` |

Evaluated with the [Darwin Skill](https://github.com/alchaincyf/darwin-skill) 9-dimension rubric (based on Microsoft Research SkillLens + SkillOpt papers).

---

## Contributing

Issues and PRs are welcome. If you've used this skill and found an edge case it doesn't handle, open an issue with the scenario.

## License

MIT
