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

动态参数使用 `<module>_DYN_<parameter_or_group>_<coverage_space>_<index>`；`module` 标识归属模块，`index` 保证唯一性，TP_ID 不使用 `source` 作为身份或来源追溯。`parameter_or_group` 表示覆盖焦点，可表达单个参数或参数组；仅当参数语义、覆盖空间、预期结果和测试构造方式均一致时允许合并。Dynamic Input 仅包含确认随当前请求携带的信息；该关系不明确时标记为待确认，不得按名称、运行时变化或寄存器载体分类。verification_goal 必须展开参数实际遍历的 enum、编码、范围、类别或边界，parameter type（reg/imm/mem 等）和 coverage space 不能替代该展开。基础动态参数空间 TP 仅在存在独立场景条件或独立 DUT 行为判定时输出 verification_scenario、expected_result。固定 opcode 只是动态输入对象的静态识别条件，不属于动态参数。禁止 REF/VAL/reference/content。

寄存器访问属性使用 `<module>_REG_<register>_<access_type>_<index>`；`register` 必须为寄存器名称，`access_type` 表示访问属性。按寄存器表顺序生成：每个寄存器先 RESET，再处理其字段的 R、RW 或其他已定义属性；同一寄存器的 TP 在 Register Access sheet 中连续排列，`index` 按整个 sheet 的最终行顺序连续递增且不复用。

配置空间使用 `<module>_CFG_<config_object>_<index>`；每个 Config TP 只对应一个 `config_object`、一个 `field` 和一个独立 value-space verification goal，`index` 按 Config Space sheet 最终行顺序连续递增。verification_goal 必须展开该 field 的完整有效配置状态空间；reserved、illegal、unsupported 编码不属于正常 Config Space 遍历，由 Register Access 或明确的异常行为验证覆盖。基础 Config Space TP 仅在存在独立场景条件或独立 DUT 行为判定时输出 verification_scenario、expected_result。Config 由“不随当前请求携带、在请求开始前设置且单个请求期间保持稳定”定义，不由 static/dynamic 命名定义；完整 Config Space 不受功能场景裁剪。

complete 至少有一项明确覆盖策略。draft 可以缺策略、HDL path、monitor mapping 或 sample event，但必须具备明确目标、场景、缺失项和完成条件。blocked 以 `lifecycle_status=blocked` 进入对应 Excel sheet，已知字段保留、未知验证字段允许为空；阻塞原因和恢复所需输入写入独立 todo/missing-input report。

Cross 仅用于 spec 明确定义、且产生单个 Config/Dynamic TP 无法表达的新联合约束或新行为的组合关系；可发生在 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input。verification_goal 必须展开参与对象、有效联合取值空间及其对应 DUT 行为/结果；仅在 spec 明确定义无效组合时展开无效联合取值空间及其对应 DUT 行为/结果。单字段 reserved/illegal encoding 不作为 Cross invalid combination。spec 已明确 invalid/unsupported 组合但未定义 DUT 行为时生成 draft；只有组合约束本身无法形成目标或场景时才 blocked。不得默认展开全量 Cartesian cross。
