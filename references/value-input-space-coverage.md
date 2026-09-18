# Value/Input-Space Coverage Contract

仅在 Config / Dynamic TP 声明独立 value/input-space coverage 时读取。本文件是单对象 structured bins、coverage tracking 和 negative bin semantics 的唯一详细定义；Cross coverage 使用 [Cross Expression Contract](cross-expression.md)。

## 字段职责与 tracking

- `verification_goal` 定义完整 value/input space、legal constraint 和 design semantic。
- 当前 TP 行的 `config_object.field` 或 `parameter` 定义 coverage target；`coverage_strategy` 只写验证方式及可脚本转换的 coverage partitions，不重复对象信息。
- structured bins 默认由 covergroup tracking；testcase 可承担 stimulus construction，但不能替代 bins tracking。输入资料明确其他 tracking mechanism 时按实际机制处理。
- 仅 testcase 且无其他 tracking mechanism 时，不能宣称 structured bins coverage 完整。
- 本 Contract 不定义 random algorithm、iteration、scan order / scheduling、testcase 数量、最小扫描次数或 bin 命中算法。

## Coverage Strategy 输出格式

structured coverage 固定使用换行 bin block，不使用 YAML、多层 schema 或解释性段落：

```text
cover bins：
<bin_name> = {<concrete values or [lower:upper] ranges>}
...

illegal bins：
<bin_name> = {<concrete values or [lower:upper] ranges>}
```

没有 illegal bin 时固定写 `illegal bins：` 后换行 `无`。输入资料明确 ignore space 时，在其后追加同格式的 `ignore bins：` block；否则不输出该 block。每个 bin 独占一行，多个 value / range 使用逗号分隔；range 固定写成 `[lower:upper]`。不得输出 `<...>` 占位符、`~` 范围、嵌套对象、重复 coverage target，或“完整覆盖”“剩余范围”“逐地址覆盖”等需要二次解释的描述。

生成脚本直接从当前 TP 行取得 coverage target，并将上述 block 转换为 coverage code；bin block 必须包含完成转换所需的实际 bin name、value 和 range。只有输入资料明确存在非默认采样条件时，才在 block 前增加一行 `sample event：<actual condition>`；默认采样由 coverage implementation 决定，不影响 TP complete。具体 coverage implementation object 仍由 `coverage_strategy_mapping` 承载，尚未生成且非 category-required 时允许为空。

## Partition 完整性

- 输入资料明确的 special value 使用 explicit bin。
- 声明独立 numeric value/input-space coverage 时，合法范围的 lower boundary、upper boundary 和一个可表示的 interior typical value 使用 explicit bin。typical value 是 coverage sample，不构成新增 DUT semantic；存在多个内部值时固定选择 `floor((lower + upper) / 2)`，不存在内部值时不生成 typical bin。
- coverage target 中未被 explicit bins 承接的剩余合法空间，必须由 residual / range bin 准确覆盖。
- 所有 bins 必须属于字段可表示且 DUT 可接收的 input domain，不包含 unreachable value；legal explicit / residual bins 不得扩大 legal space，negative bins 不得混入 legal residual，所有 partitions 不错误重叠。
- 除上述确定性 interior typical coverage sample 外，不得自行创造 typical、special、representative value 或 semantic category，也不得用“覆盖边界值、典型值和随机值”等自然语言代替实际 partition。
- coverage partition 必须由当前输出直接确定，无需重新查询 spec 或猜测。

## Numeric Coverage Profile

声明独立 numeric range coverage 时，`verification_goal` 必须直接列出实际 legal range、lower boundary、upper boundary、按上述公式选出的 typical value（若存在），以及字段可表示空间内的 below-range / above-range space（若存在）。不得写“覆盖输入资料定义的范围”“完整 numeric bins”等占位描述。

`coverage_strategy` 按 Coverage Strategy 输出格式逐行给出 `lower_boundary`、`upper_boundary`、可适用时的 `typical`、承接其余合法值的 `legal_remaining`，以及可表示时完整承接非法数值空间的 `below_range`、`above_range` 普通 negative bins。

legal range 覆盖字段全部可表示空间时不生成越界 partition。below-range / above-range 的实际 DUT behavior 必须来自输入资料，并写入 `verification_goal`；若 negative space 存在但 behavior 未定义，对象和 coverage direction 已成立，TP 标记 draft，并在 lifecycle missing-input report 中记录缺失的越界行为。不得把 error expectation 自动转换为 `illegal_bins`。

## Address Coverage Profile

本 Profile 同时适用于 Config Space 和 Dynamic Input，是 address value/input-space coverage 的唯一模板，并优先于 Numeric Coverage Profile。满足任一条件即识别为 address coverage：输入资料明确对象类型或 verification dimension 为 address；或者 field / parameter identifier 含独立的 `地址`、`addr`、`address` token。英文 token 不区分大小写，按下划线、非字母数字分隔符或 camel-case 边界识别；输入资料明确该 token 表示其他非地址语义时，以明确语义为准。名称命中只选择 coverage profile，不得据此创造 legal range、地址分类或越界行为。

以 address coverage target 中的合法地址值总数为阈值：

- `≤ 64`：每个合法地址分别生成 single-value explicit bin；可表示的 below-range / above-range 仍各自使用完整 range negative bin。
- `> 64`：不得逐地址展开，也不得输出解释性概述；固定使用下述目标与 partition 模板，并以实际值替换全部占位符。

大地址空间的 `verification_goal` 固定为：

```text
合法地址范围：<legal_range>；
下边界：<lower>；
上边界：<upper>；
典型地址：<floor((lower + upper) / 2)>；
下越界范围：<below_range> -> <input-defined behavior>；
上越界范围：<above_range> -> <input-defined behavior>。
```

不存在下越界或上越界空间时，删除对应行；合法范围无内部值时删除典型地址行。存在多个不连续合法区间时，每个区间分别保留上下边界；`typical` 对整个合法集合选择一个实际属于 legal space 的中部地址，普通中点落在 hole 时选择距离中点最近的合法地址，距离相同时选择较小者。

大地址空间的 `coverage_strategy` 固定为：

```text
cover bins：
boundary = {<lower>, <upper>}
typical = {<typical>}
legal = {<all remaining legal ranges>}
out_of_range = {<complete representable below and above ranges>}

illegal bins：
<input-defined illegal values/ranges or 无>
```

生成结果必须以实际值替换全部占位符；不存在 typical 或 out-of-range 时删除对应 bin 行。小地址空间按同一 block 每个合法地址输出一个 `addr_<value> = {<value>}`，并输出适用的完整 `out_of_range`；不得写“逐地址覆盖”“完整 address bins”等占位描述。越界行为、缺失处理、negative bin type、tracking 和 implementation mapping 继续遵守 Numeric Coverage Profile、Negative 与 bin type、Lifecycle Gate；地址对象仍不得扩展为 path-level 通路验证。

## Negative 与 bin type

- valid、invalid、reserved、unsupported 是 design semantic，不自动决定 bin type。
- 主动验证 negative / invalid / reserved / unsupported 输入及其 DUT 行为时使用普通 bins，并与 legal residual bins 分离。
- 只有输入资料明确规定某采样值本身不应出现时使用 `illegal_bins`；DUT 应返回 error 不等于该值应进入 `illegal_bins`。
- `ignore_bins` 只用于输入资料明确不纳入覆盖统计的值。不得为字段完整性制造空的 `illegal_bins` 或 `ignore_bins`。

## Lifecycle Gate

- complete TP 的 structured bins 必须完整承载声明的 coverage space，且 tracking method 满足本 Contract。
- 对象和 coverage direction 已成立，但缺少 partition 所需的 range、classification、boundary 或 design semantic 时标记 draft，并进入 lifecycle missing-input report。
- 对象、coverage direction 或必要 legal semantic 无法成立时标记 blocked。
- implementation object 尚未生成或非 category-required mapping 为空，不单独影响 complete。

## Dynamic Input 最小输出

每个 `parameter × verification dimension` 的独立 input-space coverage 必须按 Coverage Strategy 输出格式给出当前 dimension 的完整 bins。bins 只承载当前 dimension 的 input subspace；numeric range dimension 满足 Numeric Coverage Profile，address dimension 满足 Address Coverage Profile。
