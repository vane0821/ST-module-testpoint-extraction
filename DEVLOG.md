# Development Log

本文件记录 skill 开发历史，不参与 runtime 规则解释。

## 2026-09-11 | Dynamic Coverage and Scenario Missing-Input Consistency

### Problem

Dynamic Input 中残留 coverage strategy 默认包含 bins 的旧口径；Scenario missing-input 已支持 parameter disposition，但 Output Order 和 report summary 仍只描述 legality；semantic / relation 不适用规则可能被用于跳过 parameter disposition closure。

### Root Cause

Parameter Scan Output Contract 落地后，Dynamic Input 旧描述未完全清理；Missing-input Flow 扩展职责后 Output Contract 未同步；Scenario semantic applicability 与 parameter disposition applicability 的边界未显式区分。

### Change

Dynamic coverage strategy 改为映射实际 verification method，仅 parameter-scan 时要求 structured bins；Scenario missing-input 统一覆盖 legality 与 parameter disposition；明确 semantic / relation omission 不得绕过 parameter disposition closure。

### Preserved Behavior

Constraint Readability、Parameter Scan Output Contract、Scenario Parameter Disposition、Relation Extraction / Ownership、Lifecycle、Coverage Model、Excel merge 和 Skill / downstream boundary 保持不变。

## 2026-09-11 | Parameter Disposition and Bin Semantics Cleanup

### Problem

free disposition 被错误绑定到 parameter-scan structured bins；explicit bin 规则可能诱导模型制造输入资料或 coverage intent 未定义的值；inactive/default 的抽象继承表述无法保证下游唯一解析最终约束。

### Root Cause

Scenario disposition 与 Base coverage strategy 的职责边界不够明确；bin 示例与生成要求未充分区分；inactive/default 共享规则缺少可直接判定的表达要求。

### Change

明确 free 只表示使用 Base legal space，并仅在对应 Base TP 已采用 parameter-scan strategy 时使用其 structured bins；将 explicit bins 收敛为输入资料或 coverage intent 明确要求的目标，格式示例不再构成模板；要求 inactive/default rule expression 自身唯一确定适用 parameter 集合、applicability condition 和每个 parameter 的最终约束。

### Preserved Behavior

其余 TP、Relation、Scenario、Coverage、Lifecycle、schema、Excel display 和 Output Order 行为保持不变。

### Validation

检查 free 不会反向创建 parameter-scan；非 parameter-scan Base TP 不强制 bins；未定义的 typical、special、representative value 或 semantic category 不会被生成；共享 inactive/default rule 可由现有 Scenario 表达直接且唯一解析，否则进入 Scenario missing-input report。

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

## 2026-09-09 | Cross Relation Completeness and Review Readability

### Problem

明确的 multi-object relation 可能被遗漏，或只存在于 Config / Dynamic 描述而未形成 Cross；Base / Scenario relation ownership 判断不一致；多个具体 relation 可能被过度概括为宽泛 Cross；Cross `verification_goal` 退化成长段自然语言；Scenario 连续相同 `related_tp_id` 不再 merge，人工 review 可读性回退。

### Root Cause

Cross 缺少统一的 `Relation Extraction -> Ownership -> Lossless Merge -> Coverage Closure` 生成闭环；target-independent 被错误理解为 condition-free；结构治理过程中已有 display / review capability 未被完整保留。

### Change

建立生成时 Relation Atom 识别过程，明确 Base / Scenario ownership，建立 Cross completeness closure，限制 Cross lossless merge，恢复逻辑表达优先和具体 branch coverage，恢复 Scenario `related_tp_id` / object role Excel merge，并补充人工评审可读性约束。

### Preserved Behavior

TP category、其他 TP metamodel、Scenario source model `G(Complete Base Inventory, All Input Documents, Target Scenario)`、Scenario Applicable Legality Set、Scenario dependency completeness 当前边界、lifecycle、negative target / illegal_bins、coverage model、`coverage_strategy_mapping`、TP/Scenario schema、output order 和 Completeness Review 保持不变。

### Validation

重新检查同一输入集应满足：Config / Dynamic 中明确双对象 relation 可被识别；所有 scenario-independent relation 均有 Base Cross 承载；带 opcode / mode 条件的通用 relation 不因有条件而移入 Scenario；真正 scenario-specific relation 不污染 Base Cross；合并不丢 relation branch；Cross `verification_goal` 优先逻辑表达；coverage strategy 对应具体 branch；Scenario 连续相同 `related_tp_id` 恢复纵向 merge；merge 不改变底层 Scenario / lifecycle 数据。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。

## 2026-09-09 | Relation Extraction, Constraint Expression and Review Readability

### Problem

Base Cross relation 提取不完整；Config / Dynamic 中已识别的 multi-object semantic 未必进入关系覆盖；Base / Scenario ownership 判断不一致；Cross lossless merge 不稳定；AI 可能为压缩文字把多个 independent constraints 合并成难读自然语言；可逻辑表达的 relation 可能退化成长段散文；relation ownership Pending 缺少正式 missing-input 落点；Scenario 相同 `related_tp_id` merge 能力回退。

### Root Cause

1. multi-object relation 缺少统一 Relation Extraction / Ownership source of truth。
2. Cross completeness 过去以最终 TP 为中心，而不是以原始 Relation Atom 为中心。
3. Skill 未明确规定简洁不能破坏 constraint structure。
4. Missing-input model 未覆盖尚未形成 TP 的 ownership uncertainty。
5. 结构治理过程中 display / review capability 未被完整保持。

### Change

统一 Relation Extraction，清理旧 multi-object relation 直接进入 Cross 的旁路规则，明确 Base Cross / Scenario-specific / Pending ownership，增加 relation ownership missing-input report，建立 lossless merge 和逐 constraint 逻辑表达规则，使 Cross 与 Scenario 共用可读 constraint expression 原则，建立 relation completeness closure，并恢复按列判断的 Scenario `related_tp_id` / object role merge。

### Preserved Behavior

TP category 与其他 TP model、Lifecycle、Coverage Model 的 implementation input / mapping 边界、negative target / illegal_bins、`coverage_strategy_mapping`、Scenario source model `G(Complete Base Inventory, All Input Documents, Target Scenario)`、Scenario Applicable Legality Set、Scenario dependency completeness 当前 pending、`related_tp_id` Base TP traceability、Register Access、Config、Dynamic、Debug、Performance、Output Result、TP/Scenario schema 和 Completeness Review 保持不变。原 output order 仅插入按需生成的 relation ownership missing-input report。

### Validation

一致性检查覆盖：Config 中双对象 relation 进入 Relation Extraction；Dynamic multi-object relation 不再绕过 Ownership；通用 opcode/mode 条件 relation 可进入 Base Cross；target-specific relation 不污染 Base Cross；ownership unresolved 有独立 missing-input；Relation Atom 合并保持 branch；independent constraints 一条一行；可形式化约束不退化成长段自然语言；count、mutual exclusion、membership 等表达可直接评审；coverage strategy 对应具体 branch；Scenario 相同 `related_tp_id` 恢复 merge；merge 不影响 `scenario_value_or_constraint` 的独立表达。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。

## 2026-09-09 | Relation Atom Optional Applicability and Review Pending Handling

### Problem

Relation Atom 定义可能误导模型认为 `applicability_condition` 必填；存在 relation ownership Pending 时，Cross 可能被错误判为 `not_applicable`。

### Root Cause

Relation Atom 形式定义不够精确；Completeness Review 的 `not_applicable` 判据未考虑 unresolved ownership。

### Change

将 Relation Atom 定义收敛为 `[applicability_condition &&] joint_condition -> relation_or_result`，明确 applicability condition 可为空；Cross `not_applicable` 增加不存在 ownership Pending 的必要条件，并在 Pending 存在时引用对应 relation ownership missing-input。

### Preserved Behavior

其余模型、字段、流程、schema、coverage、Scenario、Excel display 和 output order 保持不变。

## 2026-09-09 | Final Gate Decoupling and Cross Expression Alignment

### Problem

Final Gate 错误依赖可选 Completeness Review；Core Model 的 Cross 公式未同步可选 applicability condition。

### Root Cause

generation gate 与 optional review flow 职责混淆；上层 Core Model 与下层 Relation Atom 表达漂移。

### Change

Final Gate 改为直接检查所有 ownership unresolved Relation Atom 是否进入 relation ownership missing-input；Core Model 的 Cross 表达统一为 `[applicability_condition &&] joint_condition -> relation_or_result`。

### Preserved Behavior

其余 Relation、Scenario、Coverage、Lifecycle、Output 行为保持不变。

## 2026-09-09 | Relation Extraction Execution and Output Regression Fix

### Problem

最终输出中的 explanation / behavior 大量使用英文；Scenario merge 规则存在但最终 workbook 未实际 merge；Relation Atom 模型存在，但 Base Cross extraction 仍明显不足。

### Root Cause

1. 中文规则只定义全局默认语言，没有定义 identifier 与 prose 的语言边界。
2. merge 只定义 display rule，没有 post-generation workbook validation。
3. Relation Extraction 只有抽象模型，没有 exhaustive execution pass，模型可能少提取后自行通过 completeness。

### Change

增加字段级 Language Rule；增加 transient exhaustive Relation Extraction Pass 和 `RELATION_EXTRACTION_COMPLETE` generation gate；限定 Cross completeness 只能在 extraction complete 后判断；为 Scenario merge 增加最终 workbook merged-range validation。

### Preserved Behavior

Relation Atom、Ownership、Lifecycle、Scenario source model、Scenario Applicable Legality Set、Coverage Model、negative bins、`related_tp_id`、TP/Scenario schema、Performance、Completeness Review 状态体系及 Cross 业务定义保持不变。

### Validation

回归检查覆盖：自然语言 explanation 默认中文；identifier、opcode、signal、error code 保持原文；Config 与 Dynamic 中所有 multi-object semantic 完成 relation scan；Base Cross 来自完整 Relation Extraction 而非主要关系总结；scenario-specific relation 正确 ownership；Cross 不做 Cartesian expansion；Scenario `related_tp_id` 和符合条件的 object role 实际出现在 merged-cell ranges；`scenario_value_or_constraint` 保持独立。

## 2026-09-11 | Constraint Readability and Generation Input Contract

### Problem

count / set / cardinality relation 被过度公式化，人工 review 困难；最终表达可能出现 `...` 等不完整压缩；object role merge 存在 optional / mandatory 漂移；parameter-scan Config / Dynamic TP 缺少稳定 structured bins；Scenario 未完整说明 parameter disposition，omission 可能被误解释为 free / inactive；描述下游消费时存在侵入 Case implementation algorithm 的风险。

### Root Cause

1. “逻辑表达优先”被错误提升为“形式化程度优先”。
2. Excel display rule 与 post-generation validation 存在口径漂移。
3. legal space 与 coverage partition 职责没有完全分开。
4. parameter-scan TP 尚未明确 structured bins 是 downstream-consumable output。
5. Scenario legality closure 没有完整回答每个相关 parameter 在当前 Scenario 中如何被约束。
6. “输出必须可消费”被错误扩展为“Skill 应定义 consumer implementation”。

### Change

将 Constraint Expression 改为 readability-first：simple relation 使用直接逻辑表达，complex count / resource / cardinality 优先结构化中文，禁止省略和复杂符号压缩；将 object role merge 统一为 mandatory；建立 Parameter Scan Output Contract，要求 parameter-scan complete TP 使用 explicit 与 residual/range structured bins；建立 Scenario Parameter Disposition Output Contract 和 Closure，定义 constrained/fixed、free、inactive，并将 unresolved disposition 接入现有 Scenario missing-input；`testcase-generation-ready` 仅表示输出充分且无歧义；明确排除 randc、iteration、testcase count、scan scheduling 等 Case implementation 规则。

### Preserved Behavior

Language Rule、Relation Extraction、Relation Ownership、Cross completeness、Lifecycle、negative target / illegal_bins、Scenario source model、Scenario Applicable Legality Set、`related_tp_id`、Performance、TP / Scenario schema、Output Order 和 Completeness Review 状态体系保持不变。

### Validation

一致性检查覆盖：simple relation 保持清晰逻辑表达；complex count/resource relation 使用可评审的结构化中文；最终 constraint 不使用省略；explanation / behavior 默认中文；`related_tp_id` / object role 在最终 workbook 中实际 merge；parameter-scan Config / Dynamic complete TP 提供 structured bins，special / typical / boundary explicit bins 与 residual legal space 清晰且一致；negative target 与 legal bins 分离；非 parameter-scan TP 不强制 bins；constrained/fixed、free、inactive 均可唯一识别且可追溯；Scenario omission 不推导 disposition；unresolved disposition 进入 Scenario missing-input；`testcase-generation-ready` 不包含下游算法含义；Relation Extraction exhaustive pass 未回退。
