# Development Log

本文件记录 skill 开发历史，不参与 runtime 规则解释。

## 2026-09-09 | Scenario/Base Boundary and Coverage Semantics Cleanup

### Problem

Base Cross 可能被仅在特定 Scenario 成立的 relation 污染；Scenario dependency completeness 与 legality completeness 的边界可能混淆；`related_tp_id` 可能按对象参与关系而非 semantic source 建立追溯；主动 negative verification target 可能与 `illegal_bins` 混用；Performance allowed range 可能被误写为 coverage target。

### Root Cause

Base / Scenario source-of-truth 边界不够明确；traceability 字段职责不够明确；coverage intent 与 bin semantics 未被严格区分；Performance measurement target 与 acceptance criteria 职责混淆。

### Change

收敛 Base Cross 为脱离 instruction / function / Scenario 后仍成立的多对象关系；保持 Scenario legality completeness 与 dependency completeness 分离；将 `related_tp_id` 固定为 Scenario expression 的 Base semantic provenance；区分主动 negative verification target 与“不应出现”采样值的 bin 语义；区分 Performance measurement target 与 expected acceptance criteria。

### Preserved Behavior

TP category、TP granularity、lifecycle、TP/Scenario schema、`coverage_strategy_mapping`、Excel display rules、output order 和 Completeness Review 行为保持不变。

### Validation

使用以下典型场景进行规则回归：同一多对象关系分别在全局与特定 Scenario 下成立；单对象 semantic、Base Cross relation 和多源 semantic basis 的 Scenario traceability；非法输入主动触发明确错误行为与覆盖采样值不应出现；Performance metric 观测与 allowed range/threshold 判定。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。
## 2026-09-09 | Scenario Source Model and Traceability Fix

### Problem

scenario-specific relation 已正确从 Base Cross 排除后，Scenario 缺少合法 semantic source；`related_tp_id` 被要求承担全部 semantic provenance，导致 scenario-specific relation 没有合法 Base TP 可追溯。

### Root Cause

Scenario Extraction 输入模型把 Base Inventory 错当成 Scenario semantic source 全集；Base TP traceability 与 design semantic provenance 职责混淆。

### Change

将 Scenario Extraction 收敛为 `G(Complete Base Inventory, All Input Documents, Target Scenario)`；将 Scenario semantic source 区分为 Base-derived 与 Scenario-specific；将 `related_tp_id` 收敛为 Base TP traceability，不再承担完整 semantic provenance；同步调整 Scenario Applicable Legality Set 的输入来源与 Final Gate。

### Preserved Behavior

Base Cross 仍只允许 scenario-independent relations；TP category、granularity、lifecycle、coverage model、negative target / illegal_bins、Performance target / allowed range、schema、Excel display、output order 和 Completeness Review 保持不变。

### Validation

验证 scenario-independent relation 正常进入 Base Cross 并被 Scenario 复用；instruction-specific relation 不进入 Base Cross，但可从输入资料进入 Scenario；scenario-specific relation 不需要伪造 Base semantic-source TP；`related_tp_id` 不再把参与对象错误宣称为 relation semantic source。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。
