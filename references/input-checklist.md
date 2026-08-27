# Module ST 输入检查清单

按逻辑信息块检查，不按物理文件数量检查。

## 通用检查

1. 是否明确目标模块和 ST/module 验证范围。
2. 是否存在生成目标 TP 类别所需的逻辑信息块。
3. 是否存在可观测结果、采样条件或状态映射。
4. 覆盖策略是否已按 category 明确：动态参数使用 coverage space/target/bins，其他适用 category 使用 testcase、covergroup 或 assertion；具体实现对象仅在适用的 `coverage_strategy_mapping` 中保存。
5. 不得用“按输入件定义”“代表值”等空泛描述代替具体内容。

## 分类别检查

- 寄存器访问属性 TP：检查寄存器名、字段、bit range、access type、reset/default value、实际寄存器/字段映射，以及 reset、clock 或 spec 定义的有效检查时机。
- Side Effect：检查触发操作、触发条件、触发效果、可观测结果。
- 配置空间 TP：检查不随请求携带、请求前设置且单个请求期间保持稳定的配置对象；检查 config_object、field、完整定义 value space，以及输入资料明确的 valid/invalid/reserved/unsupported 分类与对应预期语义。不得因当前功能场景只使用部分值而裁剪完整 Config Space，也不得推断未定义 DUT 行为。
- 动态输入参数 TP：检查随当前请求携带的参数、可合并的同语义参数组、参数类型、字段定义 bit range、实际遍历的 enum/编码/范围/类别/边界，以及输入资料明确定义的 valid/invalid/reserved/unsupported 分类与 bins。运行时变化、名称包含 dynamic 或存放在寄存器中不构成分类依据；固定 opcode 或其他静态识别字段只用于识别当前动态输入对象，不纳入动态参数扫描。不得因这些分类推断 DUT 行为或自动生成行为型 TP。
- Cross TP：检查 spec 是否明确 Configuration × Configuration、Dynamic Input × Dynamic Input 或 Configuration × Dynamic Input 的联合约束；仅在产生单个 Config/Dynamic TP 无法表达的新约束或新行为时生成。verification_goal 优先以 `<joint_condition> -> <relation_or_result>` 直接表达输入资料中的对象、字段和值。单字段 reserved/illegal encoding 不作为 Cross invalid combination；已知 invalid/unsupported 组合未定义 DUT 行为时生成 draft，不得猜测。
- Debug 能力 TP：检查 debug 命令、状态、触发时机表达式、enable/disable 条件。
- 性能验证 TP：检查原子工作场景、监测方式、测量边界、外部干扰条件、指标公式和阈值。
- 输出结果覆盖 TP：仅在 DUT 输出对象本身存在独立结果覆盖空间、且输入资料明确提供覆盖要求时生成；当前 Config/Register/Dynamic/Cross TP 的判定输出只写入 `expected_result`。

缺失必要信息时按生命周期生成 draft 或记录 blocked；缺失输入、completion summary 和 Completeness Review 仅作为独立 report 输出，不混入单 TP 或 category TP 文件；不得猜测缺失信息。

## 生命周期判定

能定义目标和场景但缺映射/采样/策略时为 draft；无法定义目标或场景时为 blocked；不受影响 category 继续生成。
