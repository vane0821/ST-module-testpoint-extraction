# Config Space Semantic Regression

只检查语义，不比较措辞或全文快照。

## 对象与准入

- 每个 Config TP 只对应一个 `config_object.field` 和一个完整单对象 value space。
- 只有 register access semantic 的 RESERVED field 不生成 Config TP；有效配置字段内部的 reserved / unsupported encoding 仍保留在该字段 value space。
- 适用 Config field 不因当前 Scenario 未引用而遗漏。
- 按输入资料中的 `config_object` 和 field 顺序生成，同一 `config_object` 连续，全 sheet index 连续。

## 跨文档语义归并

- 字段当前位置缺少 semantic 时，先检查全部输入资料；其他位置存在可唯一对应的定义时归并为最终 value-to-semantic mapping。
- 归并只能使用输入资料已有事实，不能根据字段名、经验或常识创造 semantic。
- 信息冲突、对应关系不唯一或全部输入仍缺必要 semantic 时进入 Lifecycle，不用局部缺失直接宣称完整，也不用局部缺失跳过跨文档检查。

## 目标表达

- enum、encoding、range 和 classification 使用一行一个 `<value / range> -> <semantic>`，不使用“候选值”“按输入资料定义”等内容替代实际 mapping。
- 单字段条件只引用当前 field 或非其他 Config / Dynamic 对象的 applicability condition，并完整表达 `if / else` branch。
- 引用其他 Config / Dynamic 对象的 constraint、mapping、legality 或 result 不复制进 Config TP，而是进入 Relation Extraction 和 Ownership。
- 缺少 value 对应的实际 semantic 时不能用编码清单伪装 complete，按 Lifecycle 处理。
- selector 必须提供 value-to-selection mapping；只有输入资料明确该字段仅验证 opaque candidate-ID membership 时，完整合法集合才足以 complete。

## Lifecycle

- 对象、完整值空间和全部已定义单字段 semantic 均明确时为 complete。
- 对象和值空间成立，但 semantic 冲突、不唯一或缺失时为 draft。
- 无法确认 Configuration 身份，或无法建立有效单字段 value space / verification direction 时为 blocked。
- Config mapping 不是 category-required，implementation mapping 未生成不单独导致 draft。

## Coverage 与输出

- 需要独立 value-space coverage 的 complete Config TP 满足 Value/Input-Space Coverage Contract，structured bins 与 legal space 及 `verification_goal` 一致。
- identifier 含独立 `地址` / `addr` / `address` token，或输入资料明确为 address 的 Config field，使用共享 Address Coverage Profile；合法地址值 `≤64` 时逐地址建 bin，`>64` 时使用固定大地址模板。
- 名称命中只选择 Address Coverage Profile，不得由名称推断 legal range、地址分类或越界行为。
- address field 存在可表示越界空间但越界行为缺失时标记 draft；不得输出“逐地址覆盖”“完整 address bins”等占位描述。
- structured coverage 使用实际换行的 `cover bins` / `illegal bins` block，每个 bin 独占一行并具有具体 name、value 或 `[lower:upper]` range；不重复 target，不输出 YAML 或解释性段落。
- Config schema 保持 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`。
- Config sheet index 按最终行顺序连续递增。
