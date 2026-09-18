# Cross Semantic Regression

- Cross 只承载两个或多个 Config / Dynamic 对象共同决定的 constraint、mapping、legality 或 result；单对象语义不进入 Cross。
- 一条 TP 完整表达同一组参与对象共同决定的一个关系目标；同一目标的多个分支同 TP 分块，独立关系不合并。
- 条件关系使用 `if (<SV-style condition>)` 换行、tab 缩进结果；全域直接关系使用 `<target> = <multi-object expression>`。
- 固定值、range 和集合可进入条件，range 使用 `inside {[lower:upper]}`；不得强制所有关系使用箭头或 `if`。
- Cross 固定使用紧凑 `cross bins` / `illegal cross bins` DSL，对象条件只使用 `==`、`inside` 及其 `&&` / `||` 组合，不在 TP 展开 `binsof(...) intersect`；其他 selector 进入 Cross Skill Draft。
- 需要主动验证报错、忽略、降级或其他已定义行为的 invalid / reserved / unsupported / forbidden 组合使用普通 cross bin；只有输入资料明确规定命中本身即为非法采样状态时使用 illegal cross bin。组合 legality 或行为影响 partition 但无法唯一确定时标记 draft，不得猜测 bin type。
- Cross `coverage_strategy` 不使用 testcase 或 assertion；testcase 只产生组合，REF / scoreboard 依据 `verification_goal` 判定 DUT 结果，Cross 不重复承担结果 checker。
- 输入缺 condition / result / behavior 时按 TP Lifecycle draft；仅缺非 category-required cross coverage implementation binding 时保持 complete，并按通用规则在 `coverage_strategy_mapping` 写具体 TODO；输入关系完整但 cross-bin 表达模型或转换能力不支持时进入 Cross Skill Draft，不生成 TP_ID。
- Cross Skill Draft 不阻断其他可表达 TP，但存在时不得宣称 Cross / Base Inventory complete。
- scenario-independent relation 进入 Base Cross；scenario-specific relation 进入 Scenario；ownership unresolved 进入 relation ownership missing-input。
