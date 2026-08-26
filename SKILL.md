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

所有 TP category 必须遵循统一验证模型：TP ID、仅用于定位覆盖对象的 category-specific identifier fields、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy` 和 `coverage_strategy_mapping`。`verification_goal` 与 `coverage_strategy` 是每个可生成 TP 的基础验证信息；`verification_scenario`、`expected_result` 仅在当前 TP 存在独立场景条件或需要判定 DUT 行为时输出。TP 必须明确当前验证对象、覆盖空间，以及适用时的观测对象和 DUT 可验证行为。`verification_goal` 必须点明当前验证对象、参数或功能对象、对应硬件对象和覆盖目标；不得使用“该动态输入参数对应的独立覆盖目标”等无法评审的泛称。输入资料若使用 instruction、task 等具体名称，可自然保留为输入事实，但不是 Skill 固定对象类型。

category-specific 字段只能用于定位覆盖对象，不承载验证语义，也不得替代基础字段 `verification_goal`、`coverage_strategy`，或适用时的 `verification_scenario`、`expected_result`。当前允许的固定集合为：Dynamic Input 的 `parameter`、`parameter_type`；Config 的 `config_object`、`field`；Debug 的 debug capability。Performance、Register Access 与 Output Result 默认不增加 category-specific 字段。`value_space` 属于配置覆盖语义，必须由 `verification_goal` 和 `coverage_strategy` 表达；性能对象、条件和测量边界由基础字段及适用的按需字段表达。`coverage_strategy_mapping` 仅保存实际实现对象。不得为任何 category 新增平行的目标、场景、预期、覆盖或实现映射字段体系，也不得以新定位字段承载验证语义。

TP 不展开 testcase 实现细节：不得写 driver sequence、stimulus 调度或具体构造细节、iteration 内部步骤、handshake 顺序、wait/drain/recovery、寄存器写入时序、scoreboard/checker 实现或 testcase 内部循环。

TP 数量由独立验证目标决定，不由输入字段数量决定。仅当参数或功能语义、coverage space、`expected_result` 和验证构造方式均一致时可合并；覆盖空间、DUT 行为或预期结果任一不同则必须拆分。

本 Skill 不负责：

- 自动生成真实软件 workload 场景；workload 由独立输入件或独立流程维护。
- 不自动生成任意 Configuration / Dynamic Input 的全量组合；仅对 spec 或具体功能场景明确存在的关系生成 Cross TP。
- 默认展开模块压力测试；压力测试作为后续可选增强，不作为当前 ST 层面 module 验证主线。
- 生成通路级非法地址、地址对齐、访问宽度等 TP；这些默认由 Path Verification 覆盖。
- 猜测输入资料没有定义的行为、非法处理、HDL path、采样条件、性能阈值或代表值。

## 2. TP 生成模式与生命周期

各 TP category 独立判断和生成；一个 category 缺资料不得阻断其他 category。输出时按 category 聚合为结构化 TP 数据；默认交付为 Excel workbook 的对应 sheet。category 独立只表示生成逻辑互不阻塞，不表示一个 TP 一个文件或多个同类 TP 数据文件。用户以功能场景或指令作为生成入口时，默认新增对应 `Scenario` sheet；该 sheet 呈现场景关联的既有 TP 与场景特有 TP。Register Access 与 Config Space 仍按模块基础 inventory 完整生成，不因该功能场景裁剪，也不得将功能名写入基础 TP_ID；Scenario sheet 不得改写、复制或重排基础 sheet 内容。根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读完备性审查；未指定时默认完整生成。

提示词示例：

- `根据这些模块资料生成完整的模块 ST TP，并标注生命周期。`
- `仅生成 LOAD 指令的动态输入参数 TP。`
- `将这份 TP 清单按 draft/complete/blocked 整理，并列出缺失输入。`
- `审查 MU 的 TP inventory 是否覆盖全部适用 category；只报告缺口，不要生成新 TP。`

每个候选项只有一种状态：

- **complete**：当前 TP 的必需字段均已具备。Config / Dynamic 的基础空间 TP 在 `verification_goal` 和 `coverage_strategy` 已明确时，不因未输出 `verification_scenario` 或 `expected_result` 降级；若该 TP 存在独立场景条件或 DUT 行为判定，则相关按需字段也必须已具备。动态参数 TP 的覆盖策略至少定义覆盖空间、覆盖对象和 bins；实际实现对象可在 `coverage_strategy_mapping` 中后续维护。
- **draft**：TP 的验证目标可成立，但当前必需的覆盖策略、行为判定、实现映射、HDL path、monitor mapping 或 sample event 尚未完整；缺失项和完成条件记录在独立 report，不作为单 TP 字段输出。
- **blocked**：当前 TP 必需的验证目标、覆盖策略方向或行为判定无法成立；在对应 Excel sheet 输出 `lifecycle_status=blocked` 的 TP 行，保留已知字段，未知验证字段允许为空；不得为填满字段猜测设计语义。阻塞原因和恢复所需输入仅记录在独立 todo/missing-input report。

`lifecycle_status` 是 TP 的固有字段。每个生成的 TP 必须包含 `tp_id`、`lifecycle_status`、适用的 category-specific identifier fields、`verification_goal` 和 `coverage_strategy`；仅在适用时输出 `verification_scenario`、`expected_result` 和 `coverage_strategy_mapping`。不得因最小充分输出而删除 `lifecycle_status`。
## 3. 输入资料与逻辑信息块

输入资料可以是一个或多个文件；同一个文件可以包含多个逻辑信息块，同一个逻辑信息块也可以分散在多个输入资料中。按逻辑信息块检查，不按物理文件数量检查。

常见逻辑信息块：模块 spec、寄存器基础描述和 side effect、字段约束、配置字段到 HDL signal/path 映射、动态输入参数及功能、Debug、Performance、输出覆盖、clock/reset、采样条件和可观测映射。

当生成某类 TP 所需信息缺失时：

1. 验证目标可定义，但当前必需的覆盖策略、行为判定、HDL path、monitor mapping、sample event 或实现映射尚不完整时，生成受影响的 **draft** TP。
2. 无法定义当前 TP 必需的验证目标、覆盖策略方向或行为判定时，生成对应 category 的 **blocked** TP 行；未知验证字段留空，并在独立 todo/missing-input report 记录阻塞原因和恢复所需输入。
3. 不受影响的 category 继续生成。
4. 每项缺失报告必须列出缺失内容、影响范围、当前状态和恢复所需输入；不得猜测或用空泛描述代替具体内容。

当验证目标可以确定，但某项设计语义无法从输入资料唯一确认时，不得仅根据名称推断。生成受影响的 **draft** TP，并在独立待办/缺失输入报告中说明待确认问题及其影响的 TP 部分。只有该不确定性使验证目标本身无法成立时，才记录为 **blocked**。
## 4. 覆盖策略

TP 的覆盖策略使用结构化描述，只输出实际需要的条目。`coverage_strategy` 描述验证方式或覆盖模型：可按 category 包含 assertion、testcase、covergroup，以及必要的 coverage space、coverage target、bins、illegal_bins、ignore_bins。testcase、covergroup、assertion 不是所有 category 的必需项；动态参数可仅以 coverage space、coverage target 和 bins 形成完整覆盖策略。`coverage_strategy_mapping` 在实际实现对象已知时，用于保存 testcase name、assertion code 或 coverage code；不得将这些实现映射写入 `coverage_strategy`。

规则：

- complete TP 必须有明确且可追踪的覆盖策略；按验证目标选择适用的验证方式和覆盖内容。动态参数 TP 在未定义 testcase name 时，以覆盖空间、覆盖对象和 bins 作为覆盖策略。
- draft TP 允许覆盖策略未完整映射，也允许缺 HDL path、monitor mapping、sample event，或当前 TP 所需的场景条件 / DUT 行为判定，但必须已有明确 `verification_goal`；缺失项和完成条件写入独立 report。
- blocked TP 的未知覆盖策略允许为空；不得猜测填充。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。

`coverage_strategy` 中的 coverage space、coverage target、bins、illegal_bins 和 ignore_bins 描述覆盖空间和值分类。`expected_result` 描述 DUT 对输入值的可观测处理行为。合法值由 coverage space、coverage target 和 bins 描述，`expected_result` 不重复枚举全部合法值。非法/reserved 值只在输入资料明确分类时覆盖，且 `expected_result` 必须展开具体 DUT 行为；不得写“按规格处理”“按输入资料定义”或同类泛称。`illegal_bins` 仅表示覆盖模型中的非法采样分类，不代表 DUT 必须报错、拒绝或异常；`ignore_bins` 仅表示不参与覆盖统计，不代表非法或 DUT 行为异常。
## 5. TP_ID 命名规则

统一使用大写、下划线和三位序号。不得保留 `ST`、`REF`、`VAL` 或与本规则并行的旧命名。

- 动态输入参数：`<module>_DYN_<parameter_or_group>_<coverage_space>_<index>`，例如 `VU_DYN_VD_RANGE_003`、`VU_DYN_SRC_REG_RANGE_001`、`VU_DYN_SRC_DATA_002`。
- Register Access：`<module>_REG_<register>_<access_type>_<index>`。
- Cross：`<module>_CROSS_<object>_<index>`，其中 `object` 表示已明确的跨对象关系焦点。
- 其他模块能力类：`<module>_<category>_<object>_<index>`，例如 `MU_CFG_CTRL_001`、`MU_DBG_STOP_001`、`MU_PERF_ADD_DUT_LAT_001`、`MU_OUT_STATUS_FLAG_001`。

动态 TP 以 `<module>` 标识归属模块，不使用 `source` 作为 TP_ID 身份或来源追溯。TP_ID 的职责是保证唯一性、表达 category 和快速表达验证焦点；`index` 保证唯一性，ID 不绑定当前输入资料的组织层次。`parameter_or_group` 保留覆盖焦点，但不要求机械地一字段一个 TP。
## 6. 寄存器访问属性 TP

Register Access TP 按寄存器组织，用于验证寄存器 reset 与已定义访问属性。不要把寄存器访问验证与配置生效、功能状态变化或未确认的特殊访问语义混在一起。

### 生成顺序与 TP_ID

按寄存器表顺序逐个处理寄存器：先生成该寄存器的 RESET TP，再生成其字段对应的 R、RW 或其他已定义属性 TP；当前寄存器的 TP 全部完成后才处理下一个寄存器。同一寄存器的 TP 必须在 Register Access sheet 中连续排列。

TP_ID 使用 `<module>_REG_<register>_<access_type>_<index>`。`<register>` 必须为寄存器名称，`<access_type>` 表示该 TP 的访问属性。`index` 按 Register Access sheet 的最终行顺序在整个 sheet 中连续递增；不因 access type 分别编号，也不因进入新寄存器重置。

### TP schema 与实现映射

Register Access TP 遵循统一 TP schema，并额外包含 `coverage_strategy_mapping`：`TP_ID`、`lifecycle_status`、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`、`coverage_strategy_mapping`。不得输出独立 `expected_value` 字段；spec 中的 reset/default/value 信息直接写入 `verification_goal` 和 `expected_result`。

`coverage_strategy` 表示验证方式；`coverage_strategy_mapping` 表示实际验证实现：assertion 对应 assertion code，testcase 对应 testcase name，coverage/covergroup 对应 coverage code。

### RESET 标准属性模板

RESET TP 固定为一个寄存器一条，不按 field 拆分；覆盖该寄存器中所有具有明确 reset/default value 的字段。

- `verification_goal`：验证系统解复位后，该寄存器的值符合 spec 定义的 reset/default value。
- `verification_scenario`：系统解复位，并在 spec 定义的有效检查时机观察该寄存器。
- `expected_result`：寄存器各字段值等于 spec 定义的 reset/default value。
- `coverage_strategy`：`assertion`。
- `coverage_strategy_mapping`：生成该寄存器对应的 assertion code。

reset/default value 必须从 spec 解析提取；assertion 使用实际寄存器/字段映射；检查时机必须来自 reset、clock 或 spec 定义，不得自行假设固定检查周期。

生成具体 RESET TP 时，必须将模板中的“spec 定义的 reset/default value”替换为从 spec 提取的实际值，并直接写入 `verification_goal` 和 `expected_result`；不得保留“spec 定义值”“默认值见 spec”“按 spec”等未展开表述。

### R 标准属性模板

R 属性的验证意图是写操作不得改变字段自身的值或状态来源。

- `verification_goal`：验证对 R 字段执行写操作不会改变该字段自身的值或状态来源。
- `verification_scenario`：对该字段分别尝试写入全 0、全 1、01 交织、10 交织，随后观察读值。
- `expected_result`：写操作不得改变 R 字段；字段为固定值时，读值仍为 spec 定义值；字段由硬件状态驱动时，读值仍由其正常硬件状态来源决定，不要求保持为固定常数。
- `coverage_strategy`：`testcase`。
- `coverage_strategy_mapping`：对应 testcase name。

生成具体 R TP 时，字段为固定值必须从 spec 提取并直接写入 `expected_result`；字段由硬件状态驱动时，继续描述其状态来源，不强制写固定常数。

### RW 标准属性模板

RW 属性的验证意图是字段能够正确写入并读回。

- `verification_goal`：验证 RW 字段能够正确写入并读回。
- `verification_scenario`：对该字段分别写入全 0、全 1、01 交织、10 交织，随后读回。
- `expected_result`：读回值等于写入值。
- `coverage_strategy`：`testcase`。
- `coverage_strategy_mapping`：对应 testcase name。

### 已定义属性与待确认机制

当前已确认的标准属性模板仅为 RESET、R、RW。只有字段语义与对应模板完全一致时才可使用；不得因字段名称或 access type 相近而自动套用。

遇到 side effect、W1C、W1S、RC、特殊写行为、状态依赖、合法值限制或其他 RESET/R/RW 无法覆盖的语义时，不得强行套用、根据属性名称推导行为，或为当前寄存器临时创建一次性规则。标记该属性或语义为待确认，并要求验证人员补充可泛化的属性模板；该模板至少定义 `verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`、`coverage_strategy_mapping`。模板确认后，才作为该类属性的统一生成规则，且不得绑定某个寄存器实例。

## 7. 配置空间 TP

### 功能场景输入模型

一个完整功能场景由 Configuration 和 Dynamic Input 组成。Dynamic Input 是当前请求执行所需、并随该请求一起携带的信息；Configuration 不随当前请求携带，在请求开始前预先设置，并在单个请求执行期间保持稳定。分类只依据是否随请求携带，不依据 design spec 中的 static、dynamic 或 dynamic configuration 等命名。

若某项 Configuration 允许在单个请求执行期间变化，不按普通 Configuration 处理，标记为待确认，并要求明确更新时机、生效时机及其对当前请求的影响。

### Config Space 生成

Config Space 必须先独立生成完整配置空间，不受当前功能场景裁剪。每个 Config TP 只对应一个 `config_object`、一个 `field` 和一个独立 value-space verification goal；TP_ID 使用 `<module>_CFG_<config_object>_<index>`，`index` 按 Config Space sheet 的最终行顺序连续递增，不按对象或类型分别编号。基础 Config Space TP 默认不强制输出 `verification_scenario` 或 `expected_result`；只有存在独立场景条件或需要判定独立 DUT 行为时才输出。

`verification_goal` 必须直接展开该 field 要遍历的完整**有效配置状态**空间，写出具体 enum、编码、范围或分类。不得用“覆盖有效配置状态”“覆盖选择空间”“覆盖所有合法值”或其他无法直接看出待覆盖值的泛化描述。只阅读 `verification_goal` 而不查询原 spec 时，reviewer 必须能知道该 TP 要遍历哪些 field value。reserved、illegal、unsupported 编码不属于正常 Config Space 遍历，由 Register Access 或明确的异常行为验证覆盖。

同一 field 的不同 value 类别若对应不同 DUT 行为，拆分为不同 TP；DUT 行为一致的值空间可以合并。缺少完整 value space、对应行为或实现所需信息时，按 lifecycle 生成 draft/blocked，且不得推测。

## 8. 动态输入参数 TP

Dynamic Input 只包含确认随当前请求携带的信息。运行时会变化、名称包含 dynamic 或存放在寄存器中，均不能单独作为 Dynamic Input 的分类依据；随请求携带关系不明确时标记为待确认，不得自行分类。不要默认展开底层接口 transaction 信号；接口信号表主要用于 HDL path、采样条件、输入有效事件和输出观测点映射。

动态输入 TP 的粒度是一个**可独立构造、独立观测、独立判定的覆盖目标**。TP 数量由覆盖目标决定，不由输入字段数量决定。同一动态输入对象可以有多个 DYN TP，但不机械地“一字段一个 TP”：仅当参数语义、覆盖空间、预期结果和测试构造方式均一致时允许合并；任一项不同则拆分。

每个动态参数覆盖项使用两个正交属性：

```text
parameter_type: reg | imm | mem | mask | enum | index | ...
coverage_space: range | data | address | format | mode | ...
```

`parameter_type` 描述参数本身；`coverage_space` 描述扫描空间。不得使用 reference/content、`REF` 或 `VAL`。

同语义参数组可以共同表达一个覆盖焦点，例如：

```text
参数:
- rs1
- rs2

参数类型:
reg
```

TP_ID 保留覆盖焦点；同一模块下按覆盖目标直接递增：

```text
VU_DYN_FS1_RANGE_001      # fs1 定义编码范围
VU_DYN_SRC_REG_RANGE_001  # rs1、rs2 的寄存器编号范围
VU_DYN_SRC_DATA_002       # rs1、rs2 的寄存器数据集合
```

`range` 覆盖字段定义的编码或数值范围；`data` 覆盖参数承载的数据；`address` 覆盖内存地址空间；`format` 和 `mode` 仅在输入明确要求时使用。TP 必须显式列举编码、范围、边界或类别；小规模离散空间全覆盖，大空间仅使用输入明确的集合。Dynamic Input TP 按参数自身的完整 coverage space 生成，不因当前功能场景中的 Configuration 而裁剪。

若字段的定义 bit range 大于实际生效的 bit range，扫描字段**定义**的完整 bit range；实际有效位、保留位和非法处理方式只能按输入规格写在 `expected_result`，不得默认推导截断、alias 或高位忽略。

除非 TP 明确覆盖多个参数的组合关系，未扫描参数均取输入资料定义的合法 baseline。这是 TP 生成约束，不是 `verification_scenario` 的固定输出模板。只有 baseline 影响 `expected_result`、改变覆盖语义或输入资料明确要求固定配置时，才在场景中写出具体 baseline；否则不得自动输出“未扫描字段取合法基线值”。

### TP 描述

动态输入参数 TP 遵循统一 TP schema：`tp_id`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`coverage_strategy`，并仅在适用时输出 `verification_scenario`、`expected_result`、`coverage_strategy_mapping`。其中 category-specific identifier fields 仅为 `parameter`、`parameter_type`；不输出 `category`、`source`、`input_basis`、`operation`、testcase implementation 字段或输入资料 traceability 字段。`parameter` 可以是单个参数，也可以是同语义参数组。`verification_goal`、适用的 `verification_scenario` 与 `coverage_target` 必须保留当前动态输入对象、参数名称及被覆盖的硬件对象或编码空间，不得使用无法定位对象的泛化措辞。基础动态参数空间 TP 默认不强制输出 `verification_scenario` 或 `expected_result`；只有存在独立场景条件或需要判定独立 DUT 行为时才输出。complete TP 不以 testcase name 为必要条件。

```text
tp_id:
...

lifecycle_status:
complete

参数:
- <parameter 或同语义参数组成员>

参数类型:
<reg / imm / mem / ...>

验证目标:
覆盖 <当前验证对象> 的 <parameter> 在 <硬件对象、字段或编码空间> 的实际输入空间：<具体 enum、编码、范围、类别或边界>。

验证场景（仅在适用时）:
覆盖 <parameter> 的 <值/范围/类别>，并在 <硬件对象或观测点> 观察对应映射或状态。

预期结果（仅在适用时）:
只描述被扫描参数到 DUT 输入语义的映射、合法值处理、输入资料明确的非法/保留处理、定义 bit range 与实际有效位的关系，及对应可观测状态；不得描述当前验证对象的完整执行结果、算法计算、destination 数据结果或数据通路功能正确性。

覆盖策略:
- coverage_space: <range / data / address / format / mode>
- coverage_target: <参数名或字段 bit range>
- bins:
  - <需统计的值/范围/类别>
- illegal_bins: <仅在规格定义该采样值不应出现时输出>
- ignore_bins: <仅在值不纳入覆盖统计时输出>
```

Dynamic TP 的 `verification_goal` 必须直接展开当前参数需要遍历的实际输入空间，写出具体 enum、编码、范围、类别或边界。不得只写“覆盖某参数的 range space”“覆盖某参数的有效输入”“覆盖某参数对应的编码空间”或其他无法直接看出实际待覆盖值的泛化描述；只阅读 `verification_goal` 时，reviewer 必须能够知道该 TP 实际遍历哪些输入值。`parameter_type` 与 `coverage_space` 只描述参数类型和覆盖空间类别，不能替代该展开。

`illegal_bins` 仅表示覆盖模型中的非法采样值，不代表 DUT 必须报错；`ignore_bins` 仅表示不参与覆盖统计的采样值，不代表 DUT 行为异常。DUT 对这些值的处理方式只能通过 `expected_result` 描述。testcase、covergroup、assertion 可以后续维护为实现方式，但不是动态参数 TP 为 complete 的必要字段。

### 反例检查

- `VU_DYN_OPCODE_RANGE_001` 不得因某个固定 opcode 而生成：固定 opcode 只是当前动态输入对象的静态识别条件，不是该对象的动态参数。跨对象的 opcode decode 扫描属于独立 decode 覆盖目标。
- `VU_DYN_FS1_RANGE_001` 可生成：`fs1` 是动态参数；定义 bit range 内的所有编码参与扫描，实际有效位、保留位和非法处理方式只按输入规格写在 `expected_result`。

draft TP 使用相同的 TP 字段，`coverage_strategy` 可为 `null`；缺失信息和完成条件仅写入独立 report。

### 动态输入资料

动态输入资料可能包含参数定义、参数类型、参数范围、参数到 HDL signal/path 的映射、有效接收或采样事件、clock/reset、非法/reserved 行为，以及其他输入资料明确的参数约束。固定 opcode 或其他静态识别字段只作为当前动态输入对象的静态条件，不自动作为动态参数扫描项；跨对象的 opcode decode 覆盖作为独立 decode 目标处理。

## 9. Cross TP

Cross 用于验证两个或多个 Configuration / Dynamic Input 对象的联合取值约束，以及这些组合对应的 DUT 行为。生成 Cross 时固定检查 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input 三类联合空间；每类必须确定有效组合及其 DUT 行为/结果。只有 spec 明确定义存在无效组合时，才同时覆盖无效组合及其 DUT 行为/结果；不得为了补齐 Cross TP 主动推导或制造无效组合。只有联合取值产生单个 Config TP 或 Dynamic TP 无法表达的新约束或新行为时才生成 Cross；不得仅因多个对象同时存在而生成，也不得默认展开全量 Cartesian cross。

Cross 中的无效组合是指各对象单独取值可能合法，但组合后违反 spec 定义的联合约束；不得将单个 Config field 自身的 reserved/illegal encoding 重新作为 Cross invalid combination。spec 已明确某组合 invalid/unsupported 而未定义 DUT 可观测行为时，不得猜测 `expected_result`；生成 `draft` TP，并在 missing-input/todo report 要求补充该组合的异常处理行为。只有组合约束本身无法确定，导致 `verification_goal` 或 `verification_scenario` 无法形成时，才标记为 `blocked`。

Cross TP 遵循统一 TP schema，不增加 category-specific identifier fields。`verification_goal` 必须展开或清晰描述参与对象、有效联合取值空间及其对应的结果或关系；spec 明确定义无效组合时，还必须展开无效联合取值空间及其对应的结果或关系。不得只写“覆盖相关配置组合”“覆盖 selection/override 关系”“覆盖有效 cross space”或其他无法看出实际组合约束的泛化描述。

Cross 的 `coverage_strategy` 不设固定默认值；按已明确的关系选择 testcase、covergroup 或 assertion 等验证方式。实际 testcase name、assertion code 或 coverage code 仅写入 `coverage_strategy_mapping`。

生成顺序固定为：

1. 基于 spec 独立建立完整 Config Space，展开每个 config field 的完整有效配置状态空间。
2. 根据用户要求生成的具体功能场景，识别该场景实际涉及的 Configuration 和 Dynamic Input。
3. 仅当 spec 明确定义这些对象之间的关系时，生成对应 Cross TP。

功能场景只用于选择参与 Cross 的对象和关系，不得反向裁剪 Config Space。功能场景通过 opcode 选择执行单元并关联 operand/config field 时，各 config field 仍先独立生成完整 Config TP；再生成这些相关 field 的场景级 Cross，且 Cross 只覆盖 spec 定义的有效组合关系。用户以功能场景或指令作为生成入口时，默认在独立 `Scenario` sheet 呈现场景关联 TP 与场景特有 TP；功能场景名称不写入基础 Register Access / Config Space TP_ID，也不改写、复制或重排这些基础 sheet。
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

其他 category TP 中用于判定当前输入、配置或 Cross 行为是否正确的输出，只写入 `expected_result`，不因此自动生成 Output Result TP。只有输出对象本身需要独立遍历或覆盖其结果空间时，才生成 Output Result TP。

输入与输出对象可以拆分：`vd index` 属于 Dynamic Input，`vd data` 属于 DUT Output；仅当 `vd data` 存在独立结果覆盖目标时生成 Output Result TP。同样，非法配置导致的 error/status 若只用于当前 Config/Register TP 的判定，写入 `expected_result`；只有 error/status 自身存在需要独立覆盖的类别或状态空间时，才生成 Output Result TP。

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

默认 workbook 使用 `Register Access`、`Config Space`、`Dynamic Input`、`Cross`、`Debug`、`Performance`、`Output Result` sheet；仅生成实际适用且有 TP 的 sheet。category 由 sheet 表达，不在每行重复输出。每个 TP（包括 blocked TP）在对应 sheet 占一行：`Register Access` 至少包含 `TP_ID`、`lifecycle_status`、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`、`coverage_strategy_mapping`，且同一寄存器连续排列、RESET 优先、TP_ID index 与行顺序一致；`Dynamic Input` 至少包含 `TP_ID`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`coverage_strategy`，并仅在适用时输出 `verification_scenario`、`expected_result`、`coverage_strategy_mapping`；`Config Space` 至少包含 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`coverage_strategy`，并仅在适用时输出 `verification_scenario`、`expected_result`、`coverage_strategy_mapping`；其他 sheet 按统一 TP 模型和必要 identifier fields 输出。`coverage_strategy` 可在单元格中使用简洁、可读的结构化表达，不得为 Excel 新增大量扁平字段。

当用户以功能场景或指令作为生成入口时，默认新增独立 `Scenario` sheet，呈现场景关联的 TP 和场景特有 TP；基础 `Register Access` 与 `Config Space` sheet 保持模块 inventory 的完整内容和原有 TP_ID。Scenario sheet 不得复制、改写或重排基础 sheet 内容；功能名称不得写入基础 TP_ID。

凡是能由上层分组、Excel sheet、TP_ID 或 Skill 固定规则唯一确定，且删除后不影响 TP 的理解、实现或评审的信息，不在更低层重复输出。不得机械输出 scope、输入对象 metadata、assembly、fixed opcode、execution unit list 或规则解释；只有信息本身确实对 TP 评审或实现必要时才保留。来源依据、Completeness Review、missing input report、generation summary 和 completion summary 属于 inventory/report 层，仅在用户明确要求 review/report 时独立输出，且 report 不得复制完整 TP inventory。

### 生成/输出前自检

生成或输出前逐项检查：

1. 每个 TP 是否符合统一 TP schema。
2. `lifecycle_status` 是否存在。
3. category-specific identifier fields 是否只用于定位。
4. `verification_goal` 是否明确验证对象和覆盖目标。
5. 输出 `verification_scenario` 时，是否只描述覆盖空间/状态和观测点。
6. 输出 `verification_scenario` 时，是否混入 testcase implementation。
7. 输出 `expected_result` 时，是否描述 DUT 可观测行为。
8. `coverage_strategy` 是否描述该 TP 如何被覆盖和判定，且实际实现对象是否仅写入 `coverage_strategy_mapping`。
9. 是否存在未确认设计语义却被模型自行推断。
10. 是否存在可以由 sheet、TP_ID 或上层结构确定的重复字段。
11. 是否新增未经定义的 TP 字段体系。
12. 每个 TP 是否被放入正确 category sheet。
13. category 是否被不必要地重复成每行字段。
14. 用户指定功能或指令时，基础 Register Access / Config Space sheet 是否仍保持完整 inventory，且功能名称未写入基础 TP_ID。

### 输出顺序

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. Debug sheet。
6. Performance sheet。
7. Output Result sheet。
8. 以功能场景或指令作为生成入口时，输出 Scenario sheet。
9. 用户明确要求时，独立输出 todo/missing-input 或 Completeness Review report。

## 15. Module TP Completeness Review

仅在用户明确要求 review/report 时，基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance、Output Result 七个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因；没有明确跨对象关系时，Cross 标记为 `not_applicable`。

Completeness Review **只检查和报告**，不得修改、补充或重新生成已有 TP；发现 `missing` 仅输出待办或缺失资料。
## 16. 核心原则

1. 中文为主，避免不必要英文术语。
2. TP 描述必须具体，不得使用“输入件定义的代表值”这类空泛描述。
3. 不猜测 HDL path、非法处理、输出类别、性能阈值、采样条件或监测方式。
4. 配置空间不引入请求/激励。
5. Configuration 与 Dynamic Input 只按是否随请求携带分类；完整 Config Space 不受功能场景裁剪，spec 明确定义的对象关系归入 Cross TP。
6. Debug 使用模板生成候选，并依赖项目输入件迭代。
7. 性能 TP 的测量边界、监测方式和性能目标必须一致。
8. 输出结果覆盖和压力测试均为可选输入件驱动项。
9. category 独立生成；complete、draft、blocked 的覆盖策略要求不得混用。
10. 动态参数使用 parameter type + coverage space，不得回退到 reference/content 模型。
11. verification_goal 描述覆盖目标。动态参数 TP 的 expected_result 只描述参数到 DUT 输入语义的映射、合法/非法处理以及有效位关系，不描述完整算法结果。
12. Completeness Review 只报告既有 inventory 的缺口，不生成新 TP。
13. 配置空间验证软件配置状态及配置约束，不验证寄存器存储行为。
14. 寄存器访问约束验证字段读写语义，不验证配置状态影响。
15. 一个字段描述可能同时产生 Register Access TP 和 Config Space TP，必须拆分验证目标。
16. expected_result 必须描述 DUT 可观测行为，不允许引用未展开的规格描述。
17. Config Space 的 value space 是有效配置状态集合，不是字段 bit 的全部 encoding space；reserved、非法和不支持编码不进入正常 Config Space 遍历，由 Register Access 或明确的异常行为验证覆盖。
18. TP 是可评审验证目标，不是 testcase implementation plan；验证场景不得展开 testcase 内部步骤。
19. 默认按 category sheet 输出 TP；inventory、traceability、缺失输入和 review 仅作为按需独立 report 输出。
20. TP 粒度由独立验证目标决定；不能因输入字段数量机械拆分，也不能合并覆盖空间、DUT 行为或预期结果不同的目标。



