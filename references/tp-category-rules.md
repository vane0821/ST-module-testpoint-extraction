# TP 分类与覆盖策略规则

各 category 独立生成；Completeness Review 仅审查 inventory。

| Category | ID |
|---|---|
| 寄存器访问属性 | `REG` |
| 配置空间 | `CFG` |
| 动态输入参数 | `DYN` |
| Cross | `CROSS` |
| Debug | `DBG` |
| 性能 | `PERF` |
| 输出结果 | `OUT` |

动态参数使用 `<module>_DYN_<parameter>_<coverage_space>_<index>`；`module` 标识归属模块，`index` 保证唯一性，TP_ID 不使用 `source` 作为身份或来源追溯。`parameter` 固定表示一个动态输入对象；多个参数即使 parameter_type、coverage_space 或语义相同，也分别生成各自 TP，不得合并。Dynamic Input 仅包含确认随当前请求携带的信息；该关系不明确时标记为待确认，不得按名称、运行时变化或寄存器载体分类。基础 Dynamic TP 固定只输出 TP_ID、lifecycle_status、parameter、parameter_type、verification_goal、coverage_strategy、coverage_strategy_mapping；不输出 verification_scenario、expected_result。verification_goal 必须展开参数实际遍历的 enum、编码、范围、类别或边界，以及输入资料明确定义的 valid、invalid、reserved、unsupported 分类和对应语义。coverage_strategy 映射 coverage target 与 bins。功能名或指令名不得作为额外身份信息写入基础 Dynamic TP_ID；输入资料本身明确包含的功能名、指令名或编码语义可自然保留在描述字段中。固定 opcode 只是动态输入对象的静态识别条件，不属于动态参数。禁止 REF/VAL/reference/content。

寄存器访问属性使用 `<module>_REG_<register>_<access_type>_<index>`；`register` 必须为寄存器名称，`access_type` 表示访问属性。按寄存器表顺序生成：每个寄存器先 RESET，再处理其字段的 R、RW 或其他已定义属性；同一寄存器的 TP 在 Register Access sheet 中连续排列，`index` 按整个 sheet 的最终行顺序连续递增且不复用。Register Access 默认只输出 TP_ID、lifecycle_status、verification_goal、coverage_strategy、coverage_strategy_mapping；访问条件、操作和预期结果直接合并进 verification_goal，不输出 verification_scenario 或 expected_result。固定模型为 RESET：`reset -> default value`，R：`write attempt -> no write effect`，RW：`write -> readback == write data`，SIDE_EFFECT：`access -> defined effect`。RESET 默认按 register-level，字段访问属性默认按 field-level；side effect 按输入资料中的真实归属对象独立生成，register-level 行为不得复制到每个 field，也不得混入普通 R / RW TP。具体 TP 直接实例化实际值、行为和条件；信息不足时按 lifecycle 处理。

配置空间使用 `<module>_CFG_<config_object>_<index>`；每个 Config TP 只对应一个 `config_object`、一个 `field` 和一个完整单对象 value space，`index` 按 Config Space sheet 最终行顺序连续递增。Config Space 覆盖 field 的完整定义空间；输入资料明确的 valid、invalid、reserved、unsupported 均属于该空间，verification_goal 必须展开其具体值空间和对应语义，未定义的 DUT 行为不得推断。基础 Config TP 固定只输出 TP_ID、lifecycle_status、config_object、field、verification_goal、coverage_strategy、coverage_strategy_mapping；不输出 verification_scenario、expected_result。Config 由“不随当前请求携带、在请求开始前设置且单个请求期间保持稳定”定义，不由 static/dynamic 命名定义；完整 Config Space 不受功能场景裁剪。

complete 至少有一项明确覆盖策略。draft 仅在当前已选择的 coverage strategy 依赖缺失的实现映射、HDL path、monitor mapping、sample event 或其他执行/判定信息时生成；仅缺少不影响策略成立的 `coverage_strategy_mapping` 不降级。必须具备当前 TP 所需的明确目标；blocked 以 `lifecycle_status=blocked` 进入对应 Excel sheet，已知字段保留、未知验证字段允许为空。只要存在 draft / blocked TP，就必须自动输出独立 lifecycle missing-input report，并为每个受影响 TP 记录 affected_tp_id、lifecycle_status、missing_content、completion_or_unblock_condition；不得出现 draft / blocked TP 没有对应 report 的情况。Completeness Review 和 inventory-level missing 仍仅按用户明确要求输出。

Excel lifecycle 状态使用统一高亮：complete 不特殊高亮，draft 使用统一提醒型高亮，blocked 使用统一强警示高亮；至少覆盖 lifecycle_status 单元格，且同一 workbook 的范围和样式一致。Scenario 引用 draft / blocked TP 时，related_tp_id 保留对应状态高亮；纵向 merge 不得使状态不可见。

Cross 与 Register Access、Config Space、Dynamic Input 一样属于基础 TP inventory，仅用于输入资料明确给出、且产生单个 Config/Dynamic TP 无法表达的新联合约束或新行为的组合关系；可发生在 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input。Cross 独立于 Scenario 生成，其完整性不依赖当前用户场景。Cross 默认只输出 TP_ID、lifecycle_status、verification_goal、coverage_strategy、coverage_strategy_mapping；verification_goal 使用输入资料中的对象、字段和值优先按 `<joint_condition> -> <relation_or_result>` 表达完整联合条件及对应关系/结果，不重复等价长自然语言。仅在输入资料明确定义无效组合时表达无效联合条件及其对应关系/结果。单字段 reserved/illegal encoding 不作为 Cross invalid combination。输入资料已明确 invalid/unsupported 组合但未定义 DUT 行为时生成 draft；只有组合约束本身无法形成目标时才 blocked。不得默认展开全量 Cartesian cross。
