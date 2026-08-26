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

动态参数使用 `<module>_DYN_<parameter_or_group>_<coverage_space>_<index>`；`module` 标识归属模块，`index` 保证唯一性，TP_ID 不使用 `source` 作为身份或来源追溯。`parameter_or_group` 表示覆盖焦点，可表达单个参数或参数组；仅当参数语义、覆盖空间、预期结果和测试构造方式均一致时允许合并。固定 opcode 只是动态输入对象的静态识别条件，不属于动态参数。parameter type（reg/imm/mem 等）不进入 ID，禁止 REF/VAL/reference/content。

寄存器访问属性使用 `<module>_REG_<object>_<index>`；`object` 仅表示字段访问类别（如 `RW`、`RO`、`RESET`、`SIDE_EFFECT`、`FIELD_CONSTRAINT`），不使用寄存器名、字段名或访问 Master。相同模块的 `REG` index 按既有 inventory 连续递增且不复用。

配置空间使用 `<module>_CFG_<config_object>_<index>`；`config_object` 是 register 或 config block，field 只在 TP 描述中标识覆盖焦点。多实例对象必须描述实例范围，并覆盖 `instance × field × value`；实例范围不明时输出缺失输入，不默认取单实例。

complete 至少有一项明确覆盖策略。draft 可以缺策略、HDL path、monitor mapping 或 sample event，但必须具备明确目标、场景、缺失项和完成条件。blocked 以 `lifecycle_status=blocked` 进入对应 Excel sheet，已知字段保留、未知验证字段允许为空；阻塞原因和恢复所需输入写入独立 todo/missing-input report。

Cross 仅用于输入资料明确的 override、precedence、selection、互斥或依赖关系；不得默认展开 Dynamic × Config、parameter × parameter 或 category × category 全量 cross。
