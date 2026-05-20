<!--
  Step 0 必产出: docs/audits/<subplan-id>/expectations.md
  规范: workflow.md §4.5
  使用方法: 复制本文件到 docs/audits/<subplan-id>/expectations.md, 然后逐节填写。

  填写顺序: §1 → §2 → §3 → ... → §10。
  关键纪律: 必须先读契约文档不读现有代码 (避免 expectations 被代码现状污染)。

  10 节中:
    机械化  (§2/§3/§4/§7/§8/§9 部分)  → scripts/contract_audit/ 自动校验 (项目自备)
    人审    (§5/§6/§10/§9 部分)        → subagent 独立审 + 主 agent 兜底
-->

# Contract Audit Expectations — `<subplan-id>`

**子计划**: `<例: A-X feature-name>`
**契约 review 时间**: `<YYYY-MM-DD>`
**Step 0 完成时间**: `<YYYY-MM-DD>`
**Step 5 验证时间**: `<YYYY-MM-DD, 待填>`

---

## 1. 契约源 (multi-doc)

> 列出所有相关契约文档章节。**这是审核的基准**，下面 9 节都从这里抽取。

| # | 文档 | 章节 | 涵盖契约维度 |
|---|------|------|--------------|
| 1 | `<docs/research/<feature>-design.md>` | `<§X.Y>` | schema / 退出准则 |
| 2 | `<src/<module>/<DATA_FORMAT_SPEC>.md>` | `<§X-§Y>` | 字段 / 枚举 |
| 3 | `<docs/design/<feature>/architecture.md>` | `<§X.Y>` | error 枚举 / 流程 |

---

## 2. Schema 字段（机械化 — JSON Schema 校验）

> 输出文件的字段名 / 类型 / 必选性。每条对应**一个 JSON Schema 字段**。

| 字段名 | 类型 | 必选 | 来源（§:line） | 实现位置（待 Step 5 填） | 一致 |
|--------|------|------|---------------|-------------------------|------|
| `<example_field>` | int | yes | `<§X.Y L42>` | `<file.py:LL>` | `<✅/❌>` |

**JSON Schema 文件**: `<scripts/contract_audit/schemas/<subplan-id>.schema.json>`（机械化校验入口）

---

## 3. 枚举值（机械化 — 共享常量 import）

> 跨实现的枚举集合（kind / layer / level / error 类型 等）。**所有实现必须 import 共享常量**，不可重新定义。

| 枚举集合 | 来源 | 共享常量名 | Import 路径 | 一致性测试 |
|---------|------|-----------|------------|-----------|
| `<example_enum>` | `<DATA_FORMAT §X>` | `<EXAMPLE_ENUM_CONSTANT>` | `<src/<module>/file.py:LL>` | `<test_example_enum_match>` |

---

## 4. 流程步骤（机械化 — `# Step Pn:` 注释 grep）

> 设计文档列出的流程步骤（如 P1-P9），**实现代码必须用 `# Step Pn:` 注释标注每步**。

| 设计步骤 | 来源 | 实现位置（`# Step Pn:` 注释行号）| 验证 |
|---------|------|-----------------------------|------|
| `<P1 输入验证>` | `<§X.Y>` | `<src/<module>/file.py:LL>` | `<run_all 检查 P1-Pn 全在>` |
| `<P2 业务校验>` | `<§X.Y>` | `<src/<module>/file.py:LL>` | |

---

## 5. 行为契约（语义，人审）

> "应该这样行为而不是那样"的语义约束。**机械化抓不到，必须人审。**

| 契约 | 来源 | 验证方式（测试名） | 一致 |
|------|------|------------------|------|
| `<rc != 0 不算 error>` | `<§X.Y>` | `<test_non_zero_exit_code_is_not_error>` | `<✅>` |
| `<新枚举值 X 视作 Y 类，按 Y 行为处理（forward-compat）>` | `<§X forward-compat>` | `<test_forward_compat_new_value>` | |

---

## 6. 时序/状态契约（人审）

> 状态变更顺序、init 顺序、冻结语义等**时间维度**的约束。

| 契约 | 来源 | 实现位置 | 验证 |
|------|------|---------|------|
| `<冻结后再次 init silent no-op>` | `<README §X.Y>` | `<src/<module>/file.py:LL>` | `<test_freeze_semantics>` |
| `<mkdir 必须在 spawn 之前>` | `<§X.Y 流程>` | `<src/<module>/file.py:LL>` | `<test_mkdir_before_spawn>` |

---

## 7. 不变量契约（property-based — 不变量测试覆盖）

> 对任意输入恒成立的约束，用 property-based testing 框架（如 Python `hypothesis`、Haskell `QuickCheck`、Rust `proptest`、Java `jqwik`、JavaScript `fast-check`）自动 fuzz 验证。

| 不变量 | 来源 | 测试 |
|--------|------|------|
| `<sum(parts) == total>` | `<§X.Y 隐含>` | `<test_invariant_sum_equals_total>` |
| `<rate ∈ [0.0, 1.0]>` | `<§X.Y>` | `<test_invariant_rate_bounds>` |
| `<counter ≤ cap + N_concurrent>` | `<§X.Y>` | `<test_invariant_counter_bounded>` |

---

## 8. 性能契约（机械化 — 单元测试时长断言）

> 时间 / 空间复杂度的硬约束，写为单元测试中的 `assert elapsed < T`。

| 契约 | 来源 | 测试 | 实测 |
|------|------|------|------|
| `<100K 行输入 ≤ 10s>` | `<§X.Y 退出准则>` | `<test_100k_lines_under_10s>` | `<待 Step 5 填: 1.2s ✅>` |

---

## 9. 安全/副作用契约（部分机械化）

> stdout 隔离、零修改外部模块、failure-tolerant 等。

| 契约 | 来源 | 验证（机械化/人审）|
|------|------|------------------|
| `<src/<protected-module>.py 零修改>` | `<§X.Y 退出准则>` | `<git diff --quiet src/<protected-module>.py>` 机械化 |
| `<stdout 仅协议数据，不泄漏日志>` | `<server 头部约定>` | `<grep 检查无 print() 到 stdout>` 机械化 |
| `<mkdir / open / write 失败不抛异常>` | `<§X.Y failure-tolerant>` | `<test_open_failure_does_not_propagate>` 人审 |

---

## 10. 跨实现一致性（机械化 — 参考向量 / 跨语言对照）

> 多语言 / 多实现的同一契约必须 bit-exact 一致。

| 项 | 来源 | 验证 |
|----|------|------|
| `<<算法名> N 个参考向量跨实现 bit-exact>` | `<README §X.Y>` | `<test_reference_vectors 跨 Python / C / Java>` |
| `<<rounding 模式 / 数值约定>>` | `<README §X 实现注解>` | `<test_rounding_consistency>` |

---

## 反向扩展声明

> 实现**有但契约没有**的字段 / 行为 / 接口。说明动机和向后兼容策略。

| 扩展项 | 类型 | 动机 | 向后兼容性 |
|-------|------|------|-----------|
| `<<output>.json 加 <new_field> 字段>` | schema 扩展 | `<下游消费者需要标识 X>` | `<<DATA_FORMAT_SPEC> §X 允许扩展，旧消费者忽略未知字段>` |
| `<<output>.json 加 <diagnostic_field> 字段>` | schema 扩展 | `<诊断 X 损坏 / 异常>` | 同上 |

---

## Step 5 验证记录（待填）

### 路径 A 自动检查结果
- [ ] `scripts/contract_audit/run_all.py <subplan-id>` 全过（项目自备脚本）

### 路径 B Subagent 独立审结果
- [ ] `docs/audits/<subplan-id>/audit-report.md` 已产出
- [ ] 0 个 critical / unresolved 项
- [ ] ⚠️ 项处置已记录到 `docs/audits/<subplan-id>/decisions.md`

### §4.2 五维度兜底
- [ ] 一致性
- [ ] 风格
- [ ] 正确性
- [ ] 性能
- [ ] 可维护性

---

*本 expectations 由 Step 0 **从契约文档独立抽取**（禁止从代码回填）。任何后续修改必须先改契约文档，再同步本表。*
