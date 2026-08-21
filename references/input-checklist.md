# Module ST 输入检查清单

按逻辑信息块检查，不按物理文件数量检查。

## 通用检查

1. 是否明确目标模块和 ST/module 验证范围。
2. 是否存在生成目标 TP 类别所需的逻辑信息块。
3. 是否存在可观测结果、采样条件或状态映射。
4. 覆盖策略是否至少包含 testcase、covergroup、assertion 中一项。
5. 不得用“按输入件定义”“代表值”等空泛描述代替具体内容。

## 分类别检查

- 寄存器访问属性 TP：检查寄存器名、offset、位宽、字段、bit range、访问 Master、access type、reset/default value。
- Side Effect：检查触发操作、触发条件、触发效果、可观测结果。
- 配置空间 TP：检查配置字段、合法取值或边界、HDL signal/path 映射、采样事件。
- 动态输入参数 TP：检查指令/任务/descriptor/command 的参数、编码、合法取值、预期行为。
- Debug 能力 TP：检查 debug 命令、状态、触发时机表达式、enable/disable 条件。
- 性能验证 TP：检查原子工作场景、监测方式、测量边界、外部干扰条件、指标公式和阈值。
- 输出结果覆盖 TP：仅在输入资料明确提供覆盖要求时生成。

缺失必要信息时，不生成受影响 TP，只输出输入资料不足报告；不得猜测缺失信息。
