# Dynamic Input Regression

- Dynamic Input 的分类依据是参数随当前请求携带并可在请求间变化，而不是名称、运行时变化或寄存器存放位置。
- 内部状态、中间结果和输出不得进入 Dynamic Input；接口 signal、HDL path、采样事件和 monitor mapping 不自动形成 Dynamic inventory。
- 一个 Dynamic TP 恰好对应一个 `parameter × verification dimension`；同一 parameter 的 range、data、address、format、mode 等独立 dimension 分别生成，不得合并。
- TP_ID 中的 `coverage_space` 与 `coverage_strategy` 表达同一 verification dimension；`verification_goal` 必须给出该 dimension 的实际完整 input subspace。
- numeric range / address TP 明确列出合法范围、上下边界、确定性 interior typical value、可表示的越界空间及输入资料定义的越界行为；coverage strategy 给出对应 explicit、residual 和 negative bins，不得使用“完整 bins”等占位描述。
- identifier 含独立 `地址` / `addr` / `address` token，或输入资料明确为 address 的 Dynamic parameter，使用共享 Address Coverage Profile；合法地址值 `≤64` 时逐地址建 bin，`>64` 时使用固定大地址模板。
- 名称命中只选择 Address Coverage Profile，不得由名称推断 legal range、地址分类或越界行为。
- typical value 只按 `floor((lower + upper) / 2)` 选择 coverage sample，不增加 DUT semantic；无内部值时不生成 typical bin。
- 存在可表示越界空间但越界行为未定义时标记 draft；legal range 已覆盖全部可表示空间时不制造越界值。
- Enum / format / mode 使用原始 value / encoding 到 semantic 的映射，每个 mapping 独占一行，不输出 semantic 到同名 semantic 的同义反复。
- 多参数关系必须进入 Relation Extraction 与 Ownership，不复制进单参数 TP。
- 跨文档信息只能归并明确 semantic；冲突、不唯一或缺失按 Lifecycle 处理，不得从名称或经验推断。
- complete / draft / blocked 判定必须检查对象、dimension、input subspace、单参数 semantic 及 category-required implementation input；仅缺非 category-required implementation binding 时保持 complete，并在 `coverage_strategy_mapping` 显式记录 implementation TODO。
- Dynamic mapping 非必需且可为空；需要独立 input-space coverage 时满足 Value/Input-Space Coverage Contract。
- legal bins 与 negative bins 均位于可表示、可接收的 input domain，彼此不重叠；negative bins 不混入 legal residual，非 category-required implementation mapping 为空不影响 complete。
- structured coverage 使用实际换行的 `cover bins` / `illegal bins` block，每个 bin 独占一行并具有具体 name、value 或 `[lower:upper]` range；不重复 target，不输出 YAML 或解释性段落。
- 同一 parameter 的各 dimension 连续排列，schema 不增加 source、input basis、operation 或 traceability 字段。
