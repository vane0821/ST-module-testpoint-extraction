---
name: module-st-testpoint-extraction
description: 面向 ST 层面从模块 spec、寄存器列表、动态输入描述、接口信号表、debug/performance spec 中提取、整理或审查模块验证 Testpoint。Use when the user asks to generate complete or category-specific module verification TP, manage draft/complete/blocked lifecycle, map coverage strategy, review TP completeness without regenerating TP, or report missing inputs. Covers register access, configuration space, dynamic input parameters, cross relationships, debug, atomic performance, and optional output-result coverage. Default output language is Chinese.
---

# Module ST Testpoint Extraction

默认使用中文输出。目标是从 ST 层面的模块验证视角提取可评审、可落地的 Testpoint（TP），而不是生成完整 block-level DV testplan 或 testcase implementation plan。

## 1. 职责与边界

本 Skill 负责根据输入资料提取模块验证 TP：

- 寄存器访问属性 TP。
- 配置空间 TP。
- 动态输入参数 TP。
- Cross TP。
- Debug 能力 TP。
- 原子工作场景性能 TP。
- 可选输出结果覆盖 TP。
- 覆盖策略映射。
- 输入资料不足报告。

核心 TP 元模型固定为：

- Register Access：一个明确的寄存器访问行为，即 access action + expected behavior。
- Config Space：一个配置对象的完整单对象 value space，即 values / ranges / categories + input-defined semantics。
- Dynamic Input：一个动态输入对象的完整单对象 input space，即 values / ranges / categories + input-defined semantics。
- Cross：两个或多个 Config / Dynamic 对象之间的明确联合关系，即 joint condition -> relation / result。

所有 TP 均以 `TP_ID`、`lifecycle_status`、必要的定位字段、`verification_goal`、`coverage_strategy` 为基础；`verification_scenario`、`expected_result` 和 `coverage_strategy_mapping` 仅在当前 category 或当前 TP 需要时输出。定位字段只定位覆盖对象，不承载验证目标、场景、预期结果或覆盖逻辑。当前固定定位字段为：Dynamic Input 的 `parameter`、`parameter_type`；Config 的 `config_object`、`field`；Debug 的 debug capability。Performance、Register Access、Cross 与 Output Result 默认不增加定位字段。不得为任何 category 新增平行的目标、场景、预期、覆盖或实现映射字段体系。

`verification_goal` 的职责固定为：Register Access 描述当前访问动作和预期结果；Config Space 描述当前 field 的完整 value space 及输入资料明确语义；Dynamic Input 描述当前 parameter 的完整 input space 及输入资料明确语义；Cross 描述完整联合条件及对应关系或结果，并优先使用 `<joint_condition> -> <relation_or_result>`。

TP 不展开 testcase 实现细节：不得写 driver sequence、stimulus 调度或具体构造细节、iteration 内部步骤、handshake 顺序、wait/drain/recovery、寄存器写入时序、scoreboard/checker 实现或 testcase 内部循环。

TP 粒度首先服从当前 category 的固定对象模型；只有该 category 明确允许对象合并时，才依据 verification goal、coverage space、DUT behavior 和验证构造方式判断是否合并。全局合并规则不得覆盖 category 已定义的对象粒度。不得为不存在 `expected_result` 字段的 category 额外生成该字段。

本 Skill 不负责：

- 自动生成真实软件 workload 场景；workload 由独立输入件或独立流程维护。
- 不自动生成任意 Configuration / Dynamic Input 的全量组合；仅对输入资料明确给出的关系生成 Cross TP。
- 默认展开模块压力测试；压力测试作为后续可选增强，不作为当前 ST 层面 module 验证主线。
- 生成通路级非法地址、地址对齐、访问宽度等 TP；这些默认由 Path Verification 覆盖。
- 猜测输入资料没有定义的行为、非法处理、HDL path、采样条件、性能阈值或代表值。

## 2. TP 生成模式与生命周期

各 TP category 独立判断和生成；一个 category 缺资料不得阻断其他 category。输出时按 category 聚合为结构化 TP 数据；默认交付为 Excel workbook 的对应 sheet。category 独立只表示生成逻辑互不阻塞，不表示一个 TP 一个文件或多个同类 TP 数据文件。

### Prompt Target Isolation 与两阶段 gate（强制）

用户 Prompt 中出现的具体指令、功能、场景、opcode 或目标对象，只能解释为 **Scenario Extraction Target**，不得解释为 Base Inventory 的提取过滤条件。Base Inventory 的唯一输入范围是用户提供的全部输入资料：`Base Inventory = F(All Input Documents)`；禁止使用 `Base Inventory = F(All Input Documents, Target Instruction)`。

**Phase 1 — Base Inventory Generation**：暂时忽略目标名称和场景内容，基于全部输入资料独立建立完整 Register Access、Config Space、Dynamic Input、Cross inventory。Config 必须覆盖全部识别出的 `config_object.field`，Dynamic 必须覆盖全部识别出的动态参数空间，Cross 必须覆盖输入资料中全部明确给出的多对象关系。不得使用目标名称搜索或筛选对象；不得判断对象与目标“相关/不相关”；不得因目标未使用某寄存器、字段、参数或关系而跳过、删除或缩小任何基础 TP。出现“该对象与目标无关，因此不生成”的推理，属于 Target Leakage — ERROR。

在进入 Phase 2 前执行 Base Inventory Completeness Gate：确认四类 inventory 均已按全部输入资料处理，且未使用目标过滤、删除或缩小 inventory item。全部通过时才可设定 `BASE_INVENTORY_COMPLETE = TRUE` 并进入 Scenario Extraction；否则为 `FALSE`，必须先补齐 Base Inventory，不得交付最终 Scenario TP。

**Phase 2 — Scenario Extraction**：仅在 Base Inventory 完整后读取用户指定并命名的目标，即 `Scenario TP = G(Complete Base Inventory, Target Instruction)`。每个 Scenario 使用独立 `Scenario - <scenario_name>` sheet，从既有完整 inventory 提取相关 Register Access、Config Space、Dynamic Input、Cross 及其他适用基础 TP，并以 `related_tp_id` 建立追溯；不得重新生成一套目标专属的 Config、Dynamic 或 Cross TP。Scenario 不是新的 TP category，不得生成、修改、裁剪、补充或重排基础 TP。仅当既有 Output Result 边界适用时，可在此阶段增加场景特有的 Output Result TP；场景特有的缺失信息进入独立 missing-input report。Scenario sheet 不重新定义基础覆盖空间，也不复制完整 TP。

当前功能名或指令名不得作为额外身份信息写入基础 TP_ID。输入资料本身明确包含的功能名、指令名或编码语义可自然保留在 `verification_goal` 等描述字段中，但不得仅因当前 Scenario 向基础 TP 注入额外场景条件。最终输出前必须确认：若删除 Prompt 中的目标名称，四份 Base Inventory 是否完全相同；只有答案为 YES 才可交付。根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读完备性审查；未指定时默认完整生成。

每个候选项只有一种状态：

- **complete**：当前 TP 的必需字段均已具备。Config / Dynamic 的基础空间 TP 在 `verification_goal` 和 `coverage_strategy` 已明确时，不因未输出 `verification_scenario` 或 `expected_result` 降级；多对象关系不写入基础 Dynamic TP，统一归入 Cross。单个 Dynamic 参数自身的 valid / invalid / reserved / unsupported 分类及输入资料明确的对应预期语义可直接写入 `verification_goal`，且不因此输出 `verification_scenario` 或 `expected_result`。动态参数 TP 的覆盖策略至少定义覆盖空间、覆盖对象和 bins；实际实现对象可在 `coverage_strategy_mapping` 中后续维护。
- **draft**：TP 的验证目标可成立，但当前已选择的 coverage strategy 依赖尚未具备的实现映射、HDL path、monitor mapping、sample event 或其他执行/判定信息；缺失项和完成条件记录在独立 report，不作为单 TP 字段输出。仅缺少可后续维护的 `coverage_strategy_mapping` 不导致降级。
- **blocked**：当前 TP 必需的验证目标、覆盖策略方向或行为判定无法成立；在对应 Excel sheet 输出 `lifecycle_status=blocked` 的 TP 行，保留已知字段，未知验证字段允许为空；不得为填满字段猜测设计语义。阻塞原因和恢复所需输入仅记录在独立 todo/missing-input report。

`lifecycle_status` 是 TP 的固有字段。每个生成的 TP 必须包含 `tp_id`、`lifecycle_status`、适用的 category-specific identifier fields、`verification_goal` 和 `coverage_strategy`；仅在适用时输出 `verification_scenario`、`expected_result` 和 `coverage_strategy_mapping`。不得因最小充分输出而删除 `lifecycle_status`。

只要存在任意 `draft` 或 `blocked` TP，必须自动输出独立的 lifecycle missing-input report，不等待用户额外要求。该 report 至少包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`，且每个 draft / blocked TP 均必须存在对应记录；不得出现 TP 已标记 draft / blocked 而 report 缺失的情况。Completeness Review 和 inventory-level missing 仍仅在用户明确要求 review/report 时输出。
## 3. 输入资料与逻辑信息块

输入资料可以是一个或多个文件；同一个文件可以包含多个逻辑信息块，同一个逻辑信息块也可以分散在多个输入资料中。按逻辑信息块检查，不按物理文件数量检查。

常见逻辑信息块：模块 spec、寄存器基础描述和 side effect、字段约束、配置字段到 HDL signal/path 映射、动态输入参数及功能、Debug、Performance、输出覆盖、clock/reset、采样条件和可观测映射。

当生成某类 TP 所需信息缺失时：

1. 先按已选择的 `coverage_strategy` 判断缺失项是否为必需：若 TP 仅定义单对象 coverage space、coverage target 和 bins，且无需具体 HDL、monitor 或 sample event 即可形成完整目标和覆盖策略，相关信息缺失不影响 `complete`；若策略为 assertion、需要明确采样事件的 covergroup，或其他必须依赖具体 HDL、monitor 或 sample mapping 才能执行或判定，则相应缺失时生成 **draft** TP。仅缺少可后续维护的 `coverage_strategy_mapping` 不影响 `complete`。
2. 无法定义当前 TP 必需的验证目标、覆盖策略方向或行为判定时，生成对应 category 的 **blocked** TP 行；未知验证字段留空，并在独立 todo/missing-input report 记录阻塞原因和恢复所需输入。
3. 不受影响的 category 继续生成。
4. lifecycle missing-input report 每项必须填写 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`；不得猜测或用空泛描述代替具体内容。

当验证目标可以确定，但某项设计语义无法从输入资料唯一确认时，不得仅根据名称推断。生成受影响的 **draft** TP，并在独立待办/缺失输入报告中说明待确认问题及其影响的 TP 部分。只有该不确定性使验证目标本身无法成立时，才记录为 **blocked**。
## 4. 覆盖策略

`coverage_strategy` 是从 `verification_goal` 到实际 coverage 实现的最小映射。空间型 TP（Config Space、Dynamic Input、以及需要 value-space coverage 的 Cross）输出当前需要的 coverage target、bins、illegal_bins、ignore_bins，verification method 仅在需要时输出；行为型 TP（Register Access 及其他以行为判定为主且无独立 value-space bins 的 TP）可仅输出实际 verification method。不得为行为型 TP 制造无意义 bins。`coverage_strategy_mapping` 仅在实际实现对象已知时保存 testcase name、assertion code 或 coverage code；不得将实现映射写入 `coverage_strategy`。

规则：

- complete TP 必须有明确且可追踪的覆盖策略；按验证目标选择适用的验证方式和覆盖内容。
- draft TP 允许当前已选择的 coverage strategy 所需映射、HDL path、monitor mapping、sample event 或行为判定信息未完整，但必须已有明确 `verification_goal`；仅缺少不影响当前策略成立的实现映射不触发 draft。缺失项和完成条件写入独立 report。
- blocked TP 的未知覆盖策略允许为空；不得猜测填充。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
- 每个 bin 必须给出当前 TP 的明确值、范围或集合；不得在 `coverage_strategy` 中重复 `verification_goal` 已表达的设计语义、预期行为或值空间解释。

空间型 TP 的 coverage space、coverage target、bins、illegal_bins 和 ignore_bins 描述覆盖空间和值分类。valid、invalid、reserved、unsupported 是语义分类，不自动对应任何 bin 类型；主动作为验证输入覆盖的 invalid、reserved、unsupported 值使用普通 `bins`。仅当输入资料明确规定某采样值不应出现时使用 `illegal_bins`；`ignore_bins` 仅用于明确不纳入覆盖统计的值。`expected_result` 仅在该 category 输出该字段时描述 DUT 对输入值的可观测处理行为；未定义的 DUT 行为不得据此推断。
## 5. TP_ID 命名规则

统一使用大写、下划线和三位序号。不得保留 `ST`、`REF`、`VAL` 或与本规则并行的旧命名。

- 动态输入参数：`<module>_DYN_<parameter>_<coverage_space>_<index>`，例如 `VU_DYN_VD_RANGE_003`、`VU_DYN_RS1_RANGE_001`、`VU_DYN_RS1_DATA_002`。
- Register Access：`<module>_REG_<register>_<access_type>_<index>`。
- Cross：`<module>_CROSS_<object>_<index>`，其中 `object` 表示已明确的跨对象关系焦点。
- 其他模块能力类：`<module>_<category>_<object>_<index>`，例如 `MU_CFG_CTRL_001`、`MU_DBG_STOP_001`、`MU_PERF_ADD_DUT_LAT_001`、`MU_OUT_STATUS_FLAG_001`。

动态 TP 以 `<module>` 标识归属模块，不使用 `source` 作为 TP_ID 身份或来源追溯。TP_ID 的职责是保证唯一性、表达 category 和快速表达验证焦点；`index` 保证唯一性，ID 不绑定当前输入资料的组织层次。`parameter` 固定表示一个动态输入对象。
## 6. 寄存器访问属性 TP

Register Access TP 按寄存器组织，验证寄存器读写动作及其输入资料明确规定的直接 side effect。command、trigger、start、kick 等由寄存器访问直接触发的明确行为属于 Register Access 语义；与寄存器读写动作无直接关系的独立功能效果不进入 Register Access。不得根据字段名称推断 side effect。

解析寄存器表后，每个适用的 Register Access 对象都必须生成 TP，或按已知信息标记为 draft / blocked；不得因为当前功能场景未引用该寄存器或字段而跳过。若完全没有生成对应 TP，仅在用户要求 Completeness Review 时将该 inventory item 标记为 `missing`。存在寄存器表但未生成 Register Access sheet，视为生成不完整。

每个 Register Access TP 只表达一个独立访问验证目标。`verification_goal` 必须直接写明验证对象、访问条件或操作，以及预期观察结果；不得使用“验证寄存器访问语义”“验证符合访问语义”或其他未展开的泛化描述。RESET、字段读写属性和 side effect 是不同验证目标，不得混合成一条泛化 TP。

### 生成顺序与 TP_ID

按寄存器表顺序逐个处理寄存器：先生成该寄存器的 RESET TP，再生成其字段对应的 R、RW 或其他已定义属性 TP；当前寄存器的 TP 全部完成后才处理下一个寄存器。同一寄存器的 TP 必须在 Register Access sheet 中连续排列。

TP_ID 使用 `<module>_REG_<register>_<access_type>_<index>`。`<register>` 必须为寄存器名称，`<access_type>` 表示该 TP 的访问属性。`index` 按 Register Access sheet 的最终行顺序在整个 sheet 中连续递增；不因 access type 分别编号，也不因进入新寄存器重置。

### TP schema 与实现映射

Register Access TP 默认只包含 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`。访问条件、操作和预期结果合并写入一个完整的 `verification_goal`；默认不输出 `verification_scenario` 或 `expected_result`。只有特殊访问语义确实无法自然合并进 `verification_goal` 时，才允许例外保留必要的额外字段；不得将该例外扩展为通用模板。不得输出独立 `expected_value` 字段；reset/default/fixed value 必须实例化为输入资料中的实际值并直接写入 `verification_goal`。

`coverage_strategy` 表示验证方式；`coverage_strategy_mapping` 表示实际验证实现：assertion 对应 assertion code，testcase 对应 testcase name，coverage/covergroup 对应 coverage code。

### 对象归属与 TP 粒度

- RESET 默认是 register-level TP，一个寄存器一条。
- R、RW 等字段访问属性默认是 field-level TP，每条只验证对应字段的访问属性。
- side effect TP 的粒度跟随输入资料中该行为的真实归属对象。整个寄存器的一次访问触发行为时，只生成一条 register-level side effect TP，不得复制到每个 field；只有输入资料明确说明某个 field 的访问独立触发该行为时，才生成 field-level side effect TP。
- 同一对象同时存在普通访问属性与明确 side effect 时，分别生成访问属性 TP 和 side effect TP，不得把 side effect 条件混入普通 R / RW TP。

### 固定访问模型

- RESET：`reset -> default value`。
- R：`write attempt -> no write effect`。
- RW：`write -> readback == write data`。
- SIDE_EFFECT：`access -> defined effect`。

RESET 固定为一个寄存器一条，覆盖该寄存器中具有明确 reset/default value 的字段；R、RW 等字段访问属性默认一字段一条；side effect 按其真实归属对象生成。生成具体 TP 时，`verification_goal` 直接实例化当前对象的实际值、实际行为和实际条件，不保留占位式自然语言。实际读写语义或 side effect 信息不足时，按 lifecycle 处理。不得根据字段名称推导行为。

## 7. 配置空间 TP

### 功能场景输入模型

一个完整功能场景由 Configuration 和 Dynamic Input 组成。Dynamic Input 是当前请求执行所需、并随该请求一起携带的信息；Configuration 不随当前请求携带，在请求开始前预先设置，并在单个请求执行期间保持稳定。分类只依据是否随请求携带，不依据 design spec 中的 static、dynamic 或 dynamic configuration 等命名。

若某项 Configuration 允许在单个请求执行期间变化，不按普通 Configuration 处理，标记为待确认，并要求明确更新时机、生效时机及其对当前请求的影响。

### Config Space 生成

Config Space 固定为单对象 value-space TP。每个 Config TP 只对应一个 `config_object`、一个 `field` 和一个完整单对象 value space；TP_ID 使用 `<module>_CFG_<config_object>_<index>`，`index` 按 Config Space sheet 的最终行顺序连续递增，不按对象或类型分别编号。输入资料识别出的每个 `config_object.field` 都必须生成 Config TP，或按已知信息标记为 draft / blocked；不得因为当前功能场景未引用该 field 而跳过。若完全没有生成对应 TP，仅在用户要求 Completeness Review 时将该 inventory item 标记为 `missing`。基础 Config Space TP 固定只包含 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；不输出 `verification_scenario`、`expected_result`。

`verification_goal` 必须直接展开该 field 的完整定义空间，写出具体 enum、编码、范围或分类，以及输入资料明确定义的 valid / invalid / reserved / unsupported 分类和对应预期语义。不得用“覆盖有效配置状态”“覆盖选择空间”“覆盖所有合法值”或其他无法直接看出待覆盖值的泛化描述。只阅读 `verification_goal` 而不查询原 spec 时，reviewer 必须能知道该 TP 要遍历哪些 field value 与其已定义语义。输入资料明确给出的 valid、invalid、reserved、unsupported 值均属于 Config Space；未定义的 DUT 行为不得自行推断。

单对象自身取值对应的输入资料明确语义保留在当前 Config TP。多对象联合关系进入 Cross；输出对象自身的独立覆盖空间进入 Output Result；寄存器访问动作及直接 side effect 进入 Register Access。缺少完整 value space 或覆盖策略所需信息时，按 lifecycle 生成 draft/blocked。

## 8. 动态输入参数 TP

Dynamic Input 只包含确认随当前请求携带的信息。运行时会变化、名称包含 dynamic 或存放在寄存器中，均不能单独作为 Dynamic Input 的分类依据；随请求携带关系不明确时标记为待确认。接口信号表用于 HDL path、采样条件、输入有效事件和输出观测点映射，不自动扩展为动态参数清单。

Dynamic Input 固定为单对象 input-space TP。基础 Dynamic TP 固定只包含 `TP_ID`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；不输出 `verification_scenario`、`expected_result`、`category`、`source`、`input_basis`、`operation` 或输入资料 traceability 字段。`parameter` 固定表示一个动态输入对象；多个参数即使 parameter_type、coverage_space 或语义相同，也分别生成各自 TP，不得合并。

`parameter_type` 描述参数本身，如 reg、imm、mem、mask、enum、index；`coverage_space` 在 `coverage_strategy` 中描述扫描空间，如 range、data、address、format、mode。不得使用 reference/content、`REF` 或 `VAL`。`coverage_space` 不得替代 `verification_goal` 中的实际值范围。

`verification_goal` 必须直接展开当前 parameter 的完整输入空间，写出具体值、编码、范围、边界或类别，以及输入资料明确的 valid、invalid、reserved、unsupported 分类和对应语义。`coverage_strategy` 只映射该输入空间的 coverage target 与 bins；每个 bin 必须对应明确值、范围或集合。未定义的 DUT 行为不得自行推断。

基础 Dynamic TP 不因当前功能场景中的 Configuration 而裁剪；功能名或指令名不得作为额外身份信息写入基础 TP_ID。输入资料本身明确包含的功能名、指令名或编码语义可自然保留在描述字段中，但不得仅因当前 Scenario 注入额外场景条件。

若字段的定义 bit range 大于实际生效的 bit range，扫描字段定义的完整 bit range；实际有效位、保留位和非法处理方式仅在输入资料明确时写入当前 Dynamic TP 的 `verification_goal`。基础 Dynamic TP 只覆盖单对象输入空间；未扫描参数使用输入资料定义的合法 baseline，且 baseline 不是输出字段。多个 Dynamic 参数之间存在输入资料明确的联合关系时，该关系独立生成 Cross TP。

### 动态输入资料

动态输入资料可能包含参数定义、参数类型、参数范围、参数到 HDL signal/path 的映射、有效接收或采样事件、clock/reset、非法/reserved 行为，以及其他输入资料明确的参数约束。固定 opcode 或其他静态识别字段只作为当前动态输入对象的静态条件，不自动作为动态参数扫描项；跨对象的 opcode decode 覆盖作为独立 decode 目标处理。

## 9. Cross TP

Cross 固定为多对象关系 TP，用于输入资料明确给出的 Configuration / Dynamic Input 联合取值约束、映射或结果。Cross 可覆盖 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input；只有联合取值产生单个 Config 或 Dynamic TP 无法表达的新关系时才生成，不默认展开 Cartesian cross。

Cross TP 默认使用 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，不增加 category-specific identifier fields，不输出 `verification_scenario`。`verification_goal` 必须以输入资料中的对象、字段和值直接表达完整联合条件及对应关系或结果，优先使用 `<joint_condition> -> <relation_or_result>`；能形式化表达时不补写等价长自然语言。输入资料明确定义无效组合时，同样表达无效联合条件及其对应关系或结果；单字段 reserved/illegal encoding 不作为 Cross invalid combination。

输入资料已明确 invalid/unsupported 组合但未定义对应关系或 DUT 行为时，生成 `draft` TP，并在 lifecycle missing-input report 要求补充行为/结果以完成 `verification_goal`。只有组合约束本身无法形成目标时，才标记为 `blocked`。`expected_result` 默认不输出；仅在结果无法自然并入 `verification_goal` 的逻辑关系表达式时才允许额外输出。

Cross 的 `coverage_strategy` 不设固定默认值；按关系选择 testcase、covergroup 或 assertion 等验证方式。实际 testcase name、assertion code 或 coverage code 仅写入 `coverage_strategy_mapping`。Cross 作为基础 inventory 独立生成，不根据用户指定 Scenario 临时生成、裁剪或修改。
## 10. Debug 能力 TP

Debug 属于异步输入级别能力。一个 debug 能力一个 TP。初始模板用于生成候选项，实际项目必须根据 debug spec、状态定义、命令接口和 testbench 信号迭代修正。

Debug 能力包括：

- stop。
- resume。
- single step。
- breakpoint。

### TP 描述

```text
TP ID:
...

Debug capability:
...

验证目标:
...

验证场景:
覆盖 debug capability 的触发条件；在 <debug state / 硬件对象> 观察。

预期结果:
描述 capability 对应的 DUT 状态更新或输入资料定义的错误状态。

覆盖策略:
- <按验证目标选择 testcase / covergroup / assertion>
```

规则：

- 触发时机使用逻辑表达式描述。
- 默认预期行为只要求 debug 状态更新。
- drain、上下文保持、禁止新交易、resume 后结果一致等是可选增强；只有输入资料提供明确规则时生成。
- assertion 只在输入资料提供状态更新时序或安全约束时生成。
- 触发时机模板是候选，不支持的能力或时机不生成 TP。

### 触发时机建议模板

Stop：

```text
- idle_stop: dbg_stop_req && dut_idle
- before_execute_stop: dbg_stop_req && instr_or_task_accept
- during_execute_stop: dbg_stop_req && instr_or_task_executing
- wait_or_stall_stop: dbg_stop_req && wait_or_stall
- txn_pending_stop: dbg_stop_req && (outstanding_txn_cnt != 0)
- completion_edge_stop: dbg_stop_req && instr_or_task_done
- error_pending_stop: dbg_stop_req && error_pending
- already_stopped_stop: dbg_stop_req && (debug_status == STOPPED)
- debug_disabled_stop: dbg_stop_req && !debug_enable
```

Resume：

```text
- stopped_resume: dbg_resume_req && (debug_status == STOPPED)
- stopped_by_breakpoint_resume: dbg_resume_req && (debug_status == STOPPED_BY_BREAKPOINT)
- stopped_by_step_resume: dbg_resume_req && (debug_status == STOPPED_BY_STEP)
- stopped_with_error_resume: dbg_resume_req && (debug_status == STOPPED_WITH_ERROR)
- running_resume: dbg_resume_req && (debug_status == RUNNING)
- idle_not_stopped_resume: dbg_resume_req && dut_idle && (debug_status != STOPPED)
- executing_not_stopped_resume: dbg_resume_req && instr_or_task_executing && (debug_status != STOPPED)
- debug_disabled_resume: dbg_resume_req && !debug_enable
```

Single step：

```text
- stopped_step: dbg_step_req && (debug_status == STOPPED)
- stopped_by_breakpoint_step: dbg_step_req && (debug_status == STOPPED_BY_BREAKPOINT)
- stopped_after_step_step: dbg_step_req && (debug_status == STOPPED_BY_STEP)
- running_step: dbg_step_req && (debug_status == RUNNING)
- executing_step: dbg_step_req && instr_or_task_executing
- idle_not_stopped_step: dbg_step_req && dut_idle && (debug_status != STOPPED)
- debug_disabled_step: dbg_step_req && !debug_enable
```

Breakpoint：

```text
- bp_enabled_hit: breakpoint_en && breakpoint_match
- bp_disabled_hit_condition: !breakpoint_en && breakpoint_match
- bp_enabled_not_hit: breakpoint_en && !breakpoint_match
- multi_bp_hit: breakpoint_en && (breakpoint_hit_count > 1)
- already_stopped_bp_hit: (debug_status == STOPPED) && breakpoint_en && breakpoint_match
- debug_disabled_bp_hit: !debug_enable && breakpoint_en && breakpoint_match
```

## 11. 性能验证 TP

性能验证描述原子工作场景性能。`Level0` 是外层规划概念，TP 内不反复写 Level0。

原子工作场景包括：

- 单个可执行对象。
- 同类原子单元连续流，用于 throughput / bandwidth 测量。

### TP 描述

```text
TP ID:
...

验证目标:
...

验证场景:
覆盖输入资料定义的性能条件；在 <DUT monitor / 软件可观察对象> 观察。

预期结果:
描述输入资料定义的可观测性能指标、测量边界和阈值。

覆盖策略:
- <按验证目标选择 testcase / covergroup / assertion>
```

规则：

- 验证场景写明性能条件和 DUT monitor 或软件可观察对象；不展开监测实现步骤。
- expected_result 写明指标公式 / 统计方式、测量边界和阈值，且必须与观测对象匹配。
- 外部干扰条件只有影响覆盖空间、预期行为或输入资料明确要求时，才在验证场景中写出。
- 性能 TP 可与功能 TP 共用 testcase，但 TP 不合并。
- covergroup 用于性能分布/趋势分析时暂不默认考虑；只有输入资料明确要求时输出。

不要在 Skill 中保留完整 Performance TP demo；性能场景、监测方式和测量边界按输入资料生成。

## 12. 可选输出结果覆盖 TP

Output Result TP 只用于 DUT 输出对象本身存在的独立结果覆盖空间，例如 destination data、calculation result、output data/packet、status/flag、error code 或 result register state。若输入资料未提供该输出对象的独立结果覆盖要求，不生成 Output Result TP，也不报错。

其他 category TP 中用于判定当前行为是否正确的输出，按该 category 的既有字段规则表达，不因此自动生成 Output Result TP；Register Access 默认将访问条件、操作和预期结果合并写入 `verification_goal`。只有输出对象本身需要独立遍历或覆盖其结果空间时，才生成 Output Result TP。

输入与输出对象可以拆分：`vd index` 属于 Dynamic Input，`vd data` 属于 DUT Output；仅当 `vd data` 存在独立结果覆盖目标时生成 Output Result TP。同样，error/status 若只用于当前 category TP 的判定，按该 category 的既有字段规则表达；只有 error/status 自身存在需要独立覆盖的类别或状态空间时，才生成 Output Result TP。

不得从“支持浮点”自动推出 NaN / Inf / subnormal；不得从“有 status/error register”自动推出状态覆盖。

格式：

```text
TP ID:
...

验证目标:
计算结果 / 寄存器状态 / error code / flag：覆盖 ...

验证场景:
覆盖输入资料定义的结果类别；在 <结果寄存器 / status / error code / flag> 观察。

预期结果:
...

覆盖策略:
- <按验证目标选择 testcase / covergroup / assertion>
```

## 13. 压力测试与 Workload 边界

压力测试暂不作为当前 ST 层面 module 验证主线。

真实软件 workload 场景由独立输入件或独立流程维护，描述单个任务或多个任务组合。ST 压力测试关注多模块联动场景，不在本 Skill 的模块基础 TP 中默认展开。

如果用户明确提供压力测试输入件，可作为后续增强处理；不得自动 cross 配置、输入、debug、输出或性能空间。

## 14. 结构化输出与顺序

### 输出最小充分原则

Skill 的核心输出是按统一 TP schema 生成、按 category 分组的结构化 TP 数据；序列化格式不得反向决定 TP 模型。JSON/YAML 仅用于中间结构或用户明确要求的格式，默认最终交付为 Excel workbook。

默认 workbook 使用 `Register Access`、`Config Space`、`Dynamic Input`、`Cross`、`Debug`、`Performance`、`Output Result` sheet；仅生成实际适用且有 TP 的 sheet。category 由 sheet 表达，不在每行重复输出。每个 TP（包括 blocked TP）在对应 sheet 占一行：`Register Access` 默认包含 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，不输出 `verification_scenario`、`expected_result`，且同一寄存器连续排列、RESET 优先、TP_ID index 与行顺序一致；基础 `Dynamic Input` 固定包含 `TP_ID`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，不输出 `verification_scenario`、`expected_result`；基础 `Config Space` 固定包含 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，不输出 `verification_scenario`、`expected_result`；`Cross` 固定包含 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，不输出 `verification_scenario`，仅在必要时输出 `expected_result`；其他 sheet 按统一 TP 模型和必要 identifier fields 输出。Register Access 仅在特殊访问语义确实无法合并进 `verification_goal` 时允许例外输出必要的额外字段。`coverage_strategy` 可在单元格中使用简洁、可读的结构化表达，不得为 Excel 新增大量扁平字段。

Excel workbook 必须按 `lifecycle_status` 使用统一状态高亮：`complete` 正常显示且不做特殊高亮；`draft` 使用统一的黄色或琥珀色提醒型高亮；`blocked` 使用统一的红色强警示高亮。高亮至少覆盖 `lifecycle_status` 单元格，也可覆盖整行，但同一 workbook 必须固定采用同一种范围和样式。若存在 draft / blocked TP，lifecycle missing-input report 在 Excel 中使用独立 sheet 输出。

Excel 单元格中的文本若包含中文分号 `；` 或英文分号 `;`，输出时在分号处分行显示，使每个分句单独一行；仅改变 Excel 展示格式，不改变字段内容和语义。

当用户以功能场景或指令作为生成入口时，每个命名 Scenario 使用独立 `Scenario - <scenario_name>` sheet，从完整 Base Inventory 中提取相关 Register Access、Config Space、Dynamic Input、Cross 及其他适用基础 TP，做场景化组织和解释；不新增 `scenario_name` 行字段，多个 Scenario 不得混合。每条场景关联至少输出 `related_tp_id`、对象角色、场景取值含义或约束关系、`why_relevant_to_scenario`、`scenario_application`；`scenario_application` 必须说明该基础 TP 对象在当前 Scenario 中实际决定、影响或约束什么，不得只写“沿用基础 TP 定义的覆盖空间”“不重新定义 bins”或同类无场景语义信息，也不得复制基础 TP 的 `verification_goal` 或 `coverage_strategy`。连续多行中 `why_relevant_to_scenario` 或 `scenario_application` 内容完全一致时，允许纵向合并对应 Excel 单元格；该合并仅用于展示，不改变每行与 `related_tp_id` 的逻辑对应关系。内容仅语义相近时不得合并。基础 sheet 保持模块 inventory 的完整内容和原有 TP_ID。Scenario sheet 不重新定义基础覆盖空间，不得复制、改写、裁剪或重排基础 sheet 内容；功能名称不得作为额外身份信息写入基础 TP_ID。输入资料本身明确包含的功能名、指令名或编码语义可自然保留在基础 TP 描述字段中，但不得仅因当前 Scenario 注入额外场景条件。

Scenario sheet 引用 draft / blocked 基础 TP 时，必须在对应 `related_tp_id` 单元格保留与基础 sheet 相同的提醒型或强警示型高亮，使状态可直接识别；不得因 `why_relevant_to_scenario` 或 `scenario_application` 的纵向 merge 而遮蔽、合并或丢失该状态提示。Scenario 展示层高亮不改变基础 TP 的 lifecycle 状态，也不新增 Scenario 行字段。

凡是能由上层分组、Excel sheet、TP_ID 或 Skill 固定规则唯一确定，且删除后不影响 TP 的理解、实现或评审的信息，不在更低层重复输出。不得机械输出 scope、输入对象 metadata、assembly、fixed opcode、execution unit list 或规则解释；只有信息本身确实对 TP 评审或实现必要时才保留。来源依据、Completeness Review、inventory-level missing、generation summary 和 completion summary 属于 inventory/report 层，仅在用户明确要求 review/report 时独立输出，且 report 不得复制完整 TP inventory。lifecycle missing-input report 不受该按需限制：存在 draft / blocked TP 时必须自动独立输出。

### 生成/输出前自检

生成或输出前逐项检查：

1. 每个 TP 是否符合统一 TP schema。
2. `lifecycle_status` 是否存在。
3. category-specific identifier fields 是否只用于定位。
4. `verification_goal` 是否明确验证对象和覆盖目标。
5. 输出 `verification_scenario` 时，是否只描述覆盖空间/状态和观测点。
6. 输出 `verification_scenario` 时，是否混入 testcase implementation。
7. 输出 `expected_result` 时，是否描述 DUT 可观测行为。
8. `coverage_strategy` 是否只是 `verification_goal` 的具体 coverage 映射，且实际实现对象仅写入 `coverage_strategy_mapping`。
9. 是否存在未确认设计语义却被模型自行推断。
10. 是否存在可以由 sheet、TP_ID 或上层结构确定的重复字段。
11. 是否新增未经定义的 TP 字段体系。
12. 每个 TP 是否被放入正确 category sheet。
13. category 是否被不必要地重复成每行字段。
14. 用户指定 Scenario 时，基础 inventory 是否保持完整，且 Scenario 名称未作为额外身份信息写入基础 TP_ID；输入资料本身明确存在的功能名、指令名或编码语义可保留在基础 TP 描述字段中，且未仅因当前 Scenario 注入额外场景条件。
15. 存在寄存器表时是否已生成 Register Access sheet，且每个识别出的适用 Register Access 对象均已有 TP（complete、draft 或 blocked）；完全没有对应 TP 时，仅在用户要求 Completeness Review 时标记为 `missing`。
16. 每个识别出的 `config_object.field` 是否均已有 Config TP（complete、draft 或 blocked）；完全没有对应 TP 时，仅在用户要求 Completeness Review 时标记为 `missing`。
17. 每个命名 Scenario 是否使用独立 `Scenario - <scenario_name>` sheet，包含 `related_tp_id`、对象角色、场景取值含义或约束关系、`why_relevant_to_scenario`、`scenario_application`，且未复制完整基础 TP。
18. Cross 是否仅按输入资料中明确存在的多对象关系生成，并作为独立基础 inventory 完整保留，不受当前 Scenario 影响。
19. 每个识别出的 Dynamic Input 参数空间是否均已有基础 TP（complete、draft 或 blocked）；完全没有对应 TP 时，仅在用户要求 Completeness Review 时标记为 `missing`。
20. 输入资料中每个明确的多对象关系是否均已有 Cross TP（complete、draft 或 blocked）；完全没有对应 TP 时，仅在用户要求 Completeness Review 时标记为 `missing`。
21. Register Access、Config Space、Dynamic Input、Cross 是否均已按输入资料完整处理；否则 Base Inventory 不得标记为完整。
22. 是否已在进入每个 `Scenario - <scenario_name>` sheet 前通过 Base Inventory Completeness Gate，即 `BASE_INVENTORY_COMPLETE = TRUE`。
23. 删除用户 Prompt 中的目标名称后，Register Access、Config Space、Dynamic Input、Cross 四份 Base Inventory 是否仍完全相同；若否，存在 Target Leakage，必须重新生成。
24. Register Access 的 `verification_goal` 是否已直接实例化访问对象、实际条件和实际行为，且符合固定访问模型并无占位式描述。
25. RESET 是否保持 register-level，R / RW 是否默认保持 field-level，side effect 是否按输入资料中的真实归属对象独立生成且未复制或污染普通 R / RW TP。
26. Register Access 是否默认只输出 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`，且未无理由恢复 `verification_scenario` 或 `expected_result`。
27. 每个 draft / blocked TP 是否均在 lifecycle missing-input report 中有对应记录，且四个必需字段完整；若有任一遗漏，不得交付。
28. lifecycle missing-input report 是否在存在 draft / blocked TP 时自动输出，同时未把 Completeness Review 或 inventory-level missing 错误改为自动输出。
29. Excel 中 complete、draft、blocked 是否按统一样式分别正常显示、提醒型高亮和强警示高亮，且高亮至少覆盖 `lifecycle_status` 单元格。
30. Scenario sheet 引用 draft / blocked TP 时，`related_tp_id` 是否保留对应状态高亮，且纵向 merge 未使状态不可见。

### 输出顺序

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. 以功能场景或指令作为生成入口时，按每个命名 Scenario 输出独立 `Scenario - <scenario_name>` sheet。
6. Debug sheet。
7. Performance sheet。
8. Output Result sheet。
9. 存在 draft / blocked TP 时，自动输出 lifecycle missing-input report；不存在时不生成。
10. 用户明确要求时，独立输出 inventory-level missing 或 Completeness Review report。

## 15. Module TP Completeness Review

仅在用户明确要求 review/report 时，基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance、Output Result 七个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因；没有明确跨对象关系时，Cross 标记为 `not_applicable`。

Completeness Review **只检查和报告**，不得修改、补充或重新生成已有 TP；发现 `missing` 仅输出待办或缺失资料。
## 16. 核心原则

1. 中文为主，避免不必要英文术语。
2. TP 描述必须具体，不得使用“输入件定义的代表值”这类空泛描述。
3. 不猜测 HDL path、非法处理、输出类别、性能阈值、采样条件或监测方式。
4. 配置空间不引入请求/激励。
5. Configuration 与 Dynamic Input 只按是否随请求携带分类；二者均按单对象完整定义空间建模，不受功能场景裁剪；输入资料明确给出的多对象关系归入基础 Cross TP。
6. Debug 使用模板生成候选，并依赖项目输入件迭代。
7. 性能 TP 的测量边界、监测方式和性能目标必须一致。
8. 输出结果覆盖和压力测试均为可选输入件驱动项。
9. category 独立生成；complete、draft、blocked 的覆盖策略要求不得混用。
10. 动态参数使用 parameter type + coverage space，不得回退到 reference/content 模型。
11. verification_goal 描述覆盖目标。Config Space 和 Dynamic Input 保留单对象完整 value space，以及输入资料明确的 valid / invalid / reserved / unsupported 及其对应预期语义；不得因这些语义包含 DUT 行为描述而自动迁移到其他 category。只有多对象联合关系进入 Cross，输出对象自身的独立覆盖空间进入 Output Result，寄存器访问动作及其直接 side effect 进入 Register Access。
12. Completeness Review 只报告既有 inventory 的缺口，不生成新 TP。
13. 配置空间验证软件配置状态及配置约束，不验证寄存器存储行为。
14. 一个字段描述可能同时产生 Register Access TP 和 Config Space TP，必须拆分验证目标。
15. expected_result 必须描述 DUT 可观测行为，不允许引用未展开的规格描述。
16. Config Space 与 Dynamic Input 的 value space 必须直接展开单对象完整定义空间，以及输入资料明确的 valid、invalid、reserved、unsupported 分类与对应预期语义；不得自行推断未定义 DUT 行为。
17. TP 是可评审验证目标，不是 testcase implementation plan；验证场景不得展开 testcase 内部步骤。
18. 默认按 category sheet 输出 TP；存在 draft / blocked TP 时自动输出 lifecycle missing-input report；inventory-level missing、traceability 和 review 仍仅作为按需独立 report 输出。
19. TP 粒度首先服从当前 category 的固定对象模型；只有该 category 明确允许对象合并时，才按 verification goal、coverage space、DUT behavior 和验证构造方式判断是否合并；全局合并规则不得覆盖 category 已定义的对象粒度。



