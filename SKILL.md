---
name: module-st-testpoint-extraction
description: 面向 ST 层面从模块 spec、寄存器列表、动态输入描述、接口信号表、debug/performance spec 中提取、整理或审查模块验证 Testpoint。Use when the user asks to generate complete or category-specific module verification TP, manage draft/complete/blocked lifecycle, map coverage strategy, review TP completeness without regenerating TP, or report missing inputs. Covers register access, configuration space, dynamic input parameters, debug, atomic performance, and optional output-result coverage. Default output language is Chinese.
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

所有 TP category 必须遵循统一验证模型：TP ID、仅用于定位覆盖对象的 category-specific identifier fields、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`。单个 TP 必须包含后四项，使验证人员可据此编写 testcase，并可独立评审覆盖目标。TP 必须明确当前验证对象、覆盖空间、观测对象和 DUT 可验证行为。`verification_goal` 必须点明当前验证对象、参数或功能对象、对应硬件对象和覆盖目标；不得使用“该动态输入参数对应的独立覆盖目标”等无法评审的泛称。输入资料若使用 instruction、task 等具体名称，可自然保留为输入事实，但不是 Skill 固定对象类型。

category-specific 字段只能用于定位覆盖对象，不承载验证语义，也不得替代四个统一字段。当前允许的固定集合为：Dynamic Input 的 `parameter`、`parameter_type`；Config 的 `config_object`、`field`；Debug 的 debug capability。Performance、Register Access 与 Output Result 默认不增加 category-specific 字段。`value_space` 属于配置覆盖语义，只能由 `verification_goal` 和 `coverage_strategy` 表达；性能对象、条件和测量边界也只能由四个统一字段表达。不得为任何 category 新增平行的目标、场景、预期或覆盖字段体系，也不得以新定位字段承载验证语义。

TP 不展开 testcase 实现细节：不得写 driver sequence、stimulus 调度或具体构造细节、iteration 内部步骤、handshake 顺序、wait/drain/recovery、寄存器写入时序、scoreboard/checker 实现或 testcase 内部循环。

TP 数量由独立验证目标决定，不由输入字段数量决定。仅当参数或功能语义、coverage space、`expected_result` 和验证构造方式均一致时可合并；覆盖空间、DUT 行为或预期结果任一不同则必须拆分。

本 Skill 不负责：

- 自动生成真实软件 workload 场景；workload 由独立输入件或独立流程维护。
- 自动生成配置 × 指令 / 配置 × 任务全量组合；组合围绕真实 workload 或项目指定典型任务单独维护。
- 默认展开模块压力测试；压力测试作为后续可选增强，不作为当前 ST 层面 module 验证主线。
- 生成通路级非法地址、地址对齐、访问宽度等 TP；这些默认由 Path Verification 覆盖。
- 猜测输入资料没有定义的行为、非法处理、HDL path、采样条件、性能阈值或代表值。

## 2. TP 生成模式与生命周期

各 TP category 独立判断和生成；一个 category 缺资料不得阻断其他 category。输出时按 category 聚合为结构化 TP 数据；默认交付为 Excel workbook 的对应 sheet。category 独立只表示生成逻辑互不阻塞，不表示一个 TP 一个文件或多个同类 TP 数据文件。根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读完备性审查；未指定时默认完整生成。

提示词示例：

- `根据这些模块资料生成完整的模块 ST TP，并标注生命周期。`
- `仅生成 LOAD 指令的动态输入参数 TP。`
- `将这份 TP 清单按 draft/complete/blocked 整理，并列出缺失输入。`
- `审查 MU 的 TP inventory 是否覆盖全部适用 category；只报告缺口，不要生成新 TP。`

每个候选项只有一种状态：

- **complete**：验证目标、验证场景、预期结果和明确覆盖策略均已具备。动态参数 TP 的覆盖策略至少定义覆盖空间、覆盖对象和 bins；testcase、covergroup、assertion 的实际映射可后续独立维护。
- **draft**：验证目标和验证场景明确，但 HDL path、monitor mapping、sample event 或覆盖策略绑定尚未完整；缺失项和完成条件记录在独立 report，不作为单 TP 字段输出。
- **blocked**：无法形成可验证场景；在对应 Excel sheet 输出 `lifecycle_status=blocked` 的 TP 行，保留已知字段，未知验证字段允许为空；不得为填满字段猜测设计语义。阻塞原因和恢复所需输入仅记录在独立 todo/missing-input report。

`lifecycle_status` 是 TP 的固有字段。每个生成的 TP 必须包含 `tp_id`、`lifecycle_status`、适用的 category-specific identifier fields、`verification_goal`、`verification_scenario`、`expected_result` 和 `coverage_strategy`；不得因最小充分输出而删除 `lifecycle_status`。
## 3. 输入资料与逻辑信息块

输入资料可以是一个或多个文件；同一个文件可以包含多个逻辑信息块，同一个逻辑信息块也可以分散在多个输入资料中。按逻辑信息块检查，不按物理文件数量检查。

常见逻辑信息块：模块 spec、寄存器基础描述和 side effect、字段约束、配置字段到 HDL signal/path 映射、动态输入参数及功能、Debug、Performance、输出覆盖、clock/reset、采样条件和可观测映射。

当生成某类 TP 所需信息缺失时：

1. 已能定义验证目标和验证场景，但缺 HDL path、monitor mapping、sample event 或覆盖策略绑定时，生成受影响的 **draft** TP。
2. 无法定义验证目标或验证场景时，生成对应 category 的 **blocked** TP 行；未知验证字段留空，并在独立 todo/missing-input report 记录阻塞原因和恢复所需输入。
3. 不受影响的 category 继续生成。
4. 每项缺失报告必须列出缺失内容、影响范围、当前状态和恢复所需输入；不得猜测或用空泛描述代替具体内容。

当验证目标可以确定，但某项设计语义无法从输入资料唯一确认时，不得仅根据名称推断。生成受影响的 **draft** TP，并在独立待办/缺失输入报告中说明待确认问题及其影响的 TP 部分。只有该不确定性使验证目标本身无法成立时，才记录为 **blocked**。
## 4. 覆盖策略

TP 的覆盖策略使用结构化描述，只输出实际需要的条目。`coverage_strategy` 描述该 TP 如何被覆盖和判定：可按 category 包含 coverage space、coverage target、bins、illegal_bins、ignore_bins、testcase 映射、covergroup 映射或 assertion 映射。testcase、covergroup、assertion 不是所有 category 的必需项；动态参数可仅以 coverage space、coverage target 和 bins 形成完整覆盖策略。

规则：

- complete TP 必须有明确且可追踪的覆盖策略；按验证目标选择适用的 coverage 内容和实现映射。动态参数 TP 在未建立 testcase 映射时，以覆盖空间、覆盖对象和 bins 作为覆盖策略。
- draft TP 允许覆盖策略未完整映射，也允许缺 HDL path、monitor mapping 或 sample event，但必须已有明确 `verification_goal` 和验证场景；缺失项和完成条件写入独立 report。
- blocked TP 的未知覆盖策略允许为空；不得猜测填充。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。

`coverage_strategy` 中的 coverage space、coverage target、bins、illegal_bins 和 ignore_bins 描述覆盖空间和值分类；其余映射条目描述实现时如何覆盖和判定。`expected_result` 描述 DUT 对输入值的可观测处理行为。合法值由 coverage space、coverage target 和 bins 描述，`expected_result` 不重复枚举全部合法值。非法/reserved 值只在输入资料明确分类时覆盖，且 `expected_result` 必须展开具体 DUT 行为；不得写“按规格处理”“按输入资料定义”或同类泛称。`illegal_bins` 仅表示覆盖模型中的非法采样分类，不代表 DUT 必须报错、拒绝或异常；`ignore_bins` 仅表示不参与覆盖统计，不代表非法或 DUT 行为异常。
## 5. TP_ID 命名规则

统一使用大写、下划线和三位序号。不得保留 `ST`、`REF`、`VAL` 或与本规则并行的旧命名。

- 动态输入参数：`<module>_DYN_<parameter_or_group>_<coverage_space>_<index>`，例如 `VU_DYN_VD_RANGE_003`、`VU_DYN_SRC_REG_RANGE_001`、`VU_DYN_SRC_DATA_002`。
- Cross：`<module>_CROSS_<object>_<index>`，其中 `object` 表示已明确的跨对象关系焦点。
- 模块能力类：`<module>_<category>_<object>_<index>`，例如 `MU_REG_RW_001`、`MU_CFG_CTRL_001`、`MU_DBG_STOP_001`、`MU_PERF_ADD_DUT_LAT_001`、`MU_OUT_STATUS_FLAG_001`。

动态 TP 以 `<module>` 标识归属模块，不使用 `source` 作为 TP_ID 身份或来源追溯。TP_ID 的职责是保证唯一性、表达 category 和快速表达验证焦点；`index` 保证唯一性，ID 不绑定当前输入资料的组织层次。`parameter_or_group` 保留覆盖焦点，但不要求机械地一字段一个 TP。
## 6. 寄存器访问属性 TP

寄存器访问属性 TP 独立于功能场景，用于验证寄存器表本身是否实现正确。不要把寄存器访问属性测试和寄存器配置生效行为混在一起。

### TP_ID 与粒度

Register Access category 的 TP_ID 使用 `<module>_REG_<object>_<index>`。其中 `object` 表示**字段访问类别**，不表示寄存器名或字段名；可使用 `RW`、`RO`、`RESET`、`SIDE_EFFECT`、`FIELD_CONSTRAINT` 等类别。

```text
TS_REG_RW_001
TS_REG_RO_002
TS_REG_RESET_003
```

不得把寄存器名、字段名、访问 Master 放进该 ID，例如不得生成 `TS_REG_DATAIN_TASK_RW_001` 或 `TS_REG_FINISH_RW_001`。寄存器和字段名称必须写在 `verification_goal`、`verification_scenario`、`expected_result` 和覆盖策略中。

同一模块、同一 `REG` category 下，`index` 按现有 TP inventory 连续递增；后续新增 TP 不复用已有编号，也不因寄存器或字段对象变化重新从 `001` 编号。TP 粒度由访问类别和一致的访问语义决定，不由寄存器或字段数量决定。

### 输入检查

寄存器基础描述最少需要：

- 目标模块。
- 寄存器名。
- 地址 / offset。
- 寄存器位宽。
- 字段名。
- bit range。
- 访问 Master。
- 每个访问 Master 对应的 access type。
- reset/default value。

访问 Master 指外部 bus/master 或设计内部模块 master。不同 Master 对同一字段的 access type 可以不同，必须分别处理。

Side effect 描述可以来自寄存器表、模块 spec 或验证人员整理输入。若字段有 side effect，必须描述触发操作、触发条件、触发效果和可观测结果；缺失时报告输入资料不足。

协议级非法地址、越界地址、对齐、访问宽度等默认由通路验证覆盖，本 Skill 不重复生成。

### TP 类型

寄存器访问属性 TP 包括：

- 复位值检查。
- 访问 Master 遍历。
- bit 读写覆盖。
- Side Effect 测试。
- 字段非法 / 非规范写入测试。

### bit 读写覆盖规则

- 核心目标是每个可写 bit 至少写过 0 和 1。
- 不使用整寄存器 `~reset` 作为通用规则。
- 在字段 bit range 内进行 0/1 覆盖。
- Side Effect 字段不纳入普通 bit 读写覆盖。
- 字段约束不明确时，不生成会触发非规范写入的 TP。

### RO 字段规则

RO TP 的 `expected_result` 必须明确可读值及其来源：固定 reset value、固定硬件定义值，或输入资料定义的状态值集合。

- 可读值依赖状态时，必须列出每个状态条件及对应可读值。
- 输入资料未定义可读值或状态条件时，不生成 complete TP；目标和场景仍可确定时生成 draft TP 并列出缺失信息，否则记录为 blocked。

### Register Access 与功能边界

寄存器字段的业务描述不影响 Register Access TP。例如 `STREAM_NUM` 即使描述为“能接受最大的并行用户数量”，Register Access 只验证其字段写入和读回。

以下内容属于 Config Space 或 Side Effect，而非 Register Access：写入后何时生效、合法配置范围、超出配置值的处理，以及配置导致的模块状态变化。不得因为字段名含有 `enable`、`mode`、`number`、`valid`、`finish` 等关键词自动生成功能 TP。

### 字段访问语义组合并

同一目标模块、同一访问 Master、同一寄存器、同一测试类别下，读写规则和预期行为一致的字段可以合并为一个字段访问语义组。

可以不同但仍可合并：字段名、bit range、width、reset value、业务含义。

必须拆分：access type、读行为、写行为、side effect、字段约束处理规则、预期检查方式不同。

### TP 描述

寄存器访问属性 TP 使用简洁格式：

```text
TP ID:
...

验证目标:
...

验证场景:
...

预期结果:
...

覆盖策略:
- <按验证目标选择 testcase / covergroup / assertion 映射>
```

不要在 Skill 中保留完整 TP demo，避免复制固定字段或 testcase 名称；仅保留命名规则、category 边界和容易误判的反例。

## 7. 配置空间 TP

配置空间 TP 描述软件预先设定的配置字段取值及组合约束。不引入请求、指令、任务或激励。

配置空间验证软件配置状态及配置约束，不验证寄存器存储行为。每个 Config Space TP 必须明确三层定位：**配置对象**（register/config block）、**field** 和 **value space**。field 是覆盖焦点，不能作为配置对象的唯一定位。这里的 **value space 是 valid configuration state space**：即软件可选择且由规格定义为有效的配置状态集合，不是字段 bit 的全部 encoding space。

### TP_ID 与对象定位

Config Space TP 使用 `<module>_CFG_<config_object>_<index>`；其中 `config_object` 是 register 或 config block，不是 field 名称。例如 `MU_CFG_CTRL_001` 表示配置对象 `CTRL`，具体 field 和 value space 必须在 TP 描述中写明。

```text
TP ID: MU_CFG_CTRL_001
配置对象: CTRL
字段: mode
有效配置状态: normal、debug、perf
```

不得使用仅以 field 定位 object 的 `MU_CFG_MODE_001` 或 `MU_CFG_CTRL_MODE_001`。生成结果必须能够确定被配置的对象、覆盖的字段及字段的 value space。

配置空间包括：

- 合法配置。
- 非法配置。
- 保留配置。
- 不支持配置。
- 约束冲突配置。

### 分类

配置空间 TP 分为：

- 单字段配置覆盖。
- 多字段组合配置覆盖。

单字段配置覆盖扫描某个配置字段自身的有效配置状态空间。多字段组合配置覆盖字段之间的依赖、互斥、范围、模式相关约束。

字段描述必须先拆解验证语义，再决定 TP category：

- 访问属性、写入限制、非法写入行为、保持值、side effect 属于 Register Access TP。
- 软件可配置状态、合法配置值、模式选择、配置组合关系属于 Config Space TP。
- 字段依赖、互斥关系、合法组合空间属于 Config Space combination TP。

不得把字段中的所有 enum、invalid 或 reserved 描述直接归类为 Config Space；一个字段描述可以拆分为 Register Access TP 与 Config Space TP 的不同验证目标。

Config Space 只覆盖静态配置状态，不得把动态输入参数合并进 Config TP。输入资料若明确动态输入对静态配置存在 override、precedence 或 selection 关系，应将该关系提取为独立验证目标；它不是默认的 Dynamic × Config cross，也不得展开全量笛卡尔积。

### 多实例配置

同类型多实例配置对象不要求每个 instance 单独生成重复 TP，但 TP 必须显式描述 instance 覆盖范围。覆盖策略必须包含 `instance` 维度，并表达 `instance × field × value` 覆盖；不得只覆盖 `field × value`。

```text
配置对象: QUEUE_CFG
实例范围: QUEUE_CFG[0..3]
字段: mode
覆盖空间: normal、debug、perf
覆盖交叉: instance × mode × value
```

无法确认实例范围时，输出缺失输入报告；不得默认选择单个 instance。

### covergroup 要求

配置空间 TP 默认覆盖策略为 `covergroup`。complete TP 必须提供配置对象及 field 到 HDL signal/path 的映射和 sample event；对象、field、value space、实例范围或组合约束不明确时按 draft/blocked 生命周期处理，不得推测。仅缺 HDL 映射或 sample event 时，可生成 draft TP 并列出完成条件。

同一寄存器的配置字段尽量合并到同一个 covergroup。TP 中 `覆盖策略` 填 covergroup 名称，covergroup 代码可在后续统一生成。

配置 TP 的覆盖策略至少写明配置对象、field、value space；多实例时还必须写明 instance 范围和 `instance × field × value` 覆盖关系。

默认采样语义：

- reset 默认值计入配置覆盖。
- 后续在寄存器字段有效值变化时采样。
- 若字段有特殊生效事件，以输入资料定义为准。

### bins 规则

- 小规模离散空间可全覆盖。
- 大范围字段按输入资料指定边界 / 类别 / 代表集合覆盖。
- 不盲目展开超过 SV 自动建仓限制的大空间。
- `value space` 的 bins 只覆盖有效配置状态，不以字段 bit 的全部编码自动建 bins。非法、reserved、不支持编码是需要单独定义 DUT 行为的异常输入，不自动并入 valid configuration state space。
- 非法配置如需主动测试，不默认写成 `illegal_bins`；是否使用普通 bins、illegal_bins 或 ignore_bins 由输入资料定义。
- 非法、reserved 或不支持编码的 `expected_result` 必须写明具体 DUT 可观测行为，例如保持原值、写入忽略、映射默认值、产生错误状态或其他明确硬件行为；不得写“按规格定义”“按输入资料处理”或“按要求处理”。
- 若输入资料只标记非法/reserved 而未定义 DUT 行为，不生成该行为对应的 complete TP；输出缺失输入并要求补充非法配置处理规则。

### 生成前检查

生成 Config Space TP 前依次检查：

1. 配置对象是否明确。
2. field 是否属于该配置对象。
3. 是否存在多实例覆盖需求，以及实例范围是否明确。
4. value space 是否明确。
5. 非法/reserved value 是否定义具体 DUT 行为。
6. 是否存在字段组合约束。
7. HDL path、sample event 等 complete 条件是否满足。

检查失败时按 complete、draft、blocked 生命周期处理，不允许推测缺失行为。

### TP 描述

```text
TP ID:
...

config_object:
...

field:
...

验证目标:
...

验证场景:
...

预期结果:
...

覆盖策略:
- <按验证目标选择 testcase / covergroup / assertion 映射>
```

不要在 Skill 中保留完整 Config TP demo；仅保留对象定位、状态空间和非法行为的规则。

## 8. 动态输入参数 TP

动态输入参数 TP 描述软件或上游在一次操作中可动态提交到模块的输入参数。不要默认展开底层接口 transaction 信号；接口信号表主要用于 HDL path、采样条件、输入有效事件和输出观测点映射。

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

`range` 覆盖字段定义的编码或数值范围；`data` 覆盖参数承载的数据；`address` 覆盖内存地址空间；`format` 和 `mode` 仅在输入明确要求时使用。TP 必须显式列举编码、范围、边界或类别；小规模离散空间全覆盖，大空间仅使用输入明确的集合。当前不默认生成 opcode × parameter、parameter × parameter 或动态参数 × 配置空间组合。

若字段的定义 bit range 大于实际生效的 bit range，扫描字段**定义**的完整 bit range；实际有效位、保留位和非法处理方式只能按输入规格写在 `expected_result`，不得默认推导截断、alias 或高位忽略。

除非 TP 明确覆盖多个参数的组合关系，未扫描参数均取输入资料定义的合法 baseline。这是 TP 生成约束，不是 `verification_scenario` 的固定输出模板。只有 baseline 影响 `expected_result`、改变覆盖语义或输入资料明确要求固定配置时，才在场景中写出具体 baseline；否则不得自动输出“未扫描字段取合法基线值”。

### TP 描述

动态输入参数 TP 遵循统一 TP schema：`tp_id`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`。其中 category-specific identifier fields 仅为 `parameter`、`parameter_type`；不输出 `category`、`source`、`input_basis`、`operation`、testcase implementation 字段或输入资料 traceability 字段。`parameter` 可以是单个参数，也可以是同语义参数组。`verification_goal`、`verification_scenario` 与 `coverage_target` 必须保留当前动态输入对象、参数名称及被覆盖的硬件对象或编码空间，不得使用无法定位对象的泛化措辞。complete TP 不以 testcase 映射为必要条件。

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
覆盖 <当前验证对象> 的 <parameter> 在 <硬件对象、字段或编码空间> 的 <具体覆盖目标>。

验证场景:
覆盖 <parameter> 的 <值/范围/类别>，并在 <硬件对象或观测点> 观察对应映射或状态。

预期结果:
只描述被扫描参数到 DUT 输入语义的映射、合法值处理、输入资料明确的非法/保留处理、定义 bit range 与实际有效位的关系，及对应可观测状态；不得描述当前验证对象的完整执行结果、算法计算、destination 数据结果或数据通路功能正确性。

覆盖策略:
- coverage_space: <range / data / address / format / mode>
- coverage_target: <参数名或字段 bit range>
- bins:
  - <需统计的值/范围/类别>
- illegal_bins: <仅在规格定义该采样值不应出现时输出>
- ignore_bins: <仅在值不纳入覆盖统计时输出>
```

`illegal_bins` 仅表示覆盖模型中的非法采样值，不代表 DUT 必须报错；`ignore_bins` 仅表示不参与覆盖统计的采样值，不代表 DUT 行为异常。DUT 对这些值的处理方式只能通过 `expected_result` 描述。testcase、covergroup、assertion 可以后续维护为实现映射，但不是动态参数 TP 为 complete 的必要字段。

### 反例检查

- `VU_DYN_OPCODE_RANGE_001` 不得因某个固定 opcode 而生成：固定 opcode 只是当前动态输入对象的静态识别条件，不是该对象的动态参数。跨对象的 opcode decode 扫描属于独立 decode 覆盖目标。
- `VU_DYN_FS1_RANGE_001` 可生成：`fs1` 是动态参数；定义 bit range 内的所有编码参与扫描，实际有效位、保留位和非法处理方式只按输入规格写在 `expected_result`。

draft TP 使用相同的 TP 字段，`coverage_strategy` 可为 `null`；缺失信息和完成条件仅写入独立 report。

## 9. Cross TP

Cross TP 只用于输入资料明确规定的 override、precedence、selection、互斥或依赖关系。例如，`TYPE_VL override static_TYPE_VL` 的选择关系属于 Cross TP。不得默认生成 Dynamic × Config、parameter × parameter 或 category × category 的全量 cross。

Cross TP 遵循统一 TP schema，不增加 category-specific identifier fields；关系对象、关系条件、观测对象和判定方式分别写入四个统一验证字段。

### 动态输入资料

动态输入资料可能包含参数定义、参数类型、参数范围、参数到 HDL signal/path 的映射、有效接收或采样事件、clock/reset、非法/reserved 行为，以及其他输入资料明确的参数约束。固定 opcode 或其他静态识别字段只作为当前动态输入对象的静态条件，不自动作为动态参数扫描项；跨对象的 opcode decode 覆盖作为独立 decode 目标处理。
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
- <按验证目标选择 testcase / covergroup / assertion 映射>
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
- <按验证目标选择 testcase / covergroup / assertion 映射>
```

规则：

- 验证场景写明性能条件和 DUT monitor 或软件可观察对象；不展开监测实现步骤。
- expected_result 写明指标公式 / 统计方式、测量边界和阈值，且必须与观测对象匹配。
- 外部干扰条件只有影响覆盖空间、预期行为或输入资料明确要求时，才在验证场景中写出。
- 性能 TP 可与功能 TP 共用 testcase，但 TP 不合并。
- covergroup 用于性能分布/趋势分析时暂不默认考虑；只有输入资料明确要求时输出。

不要在 Skill 中保留完整 Performance TP demo；性能场景、监测方式和测量边界按输入资料生成。

## 12. 可选输出结果覆盖 TP

输出结果覆盖 TP 完全由输入资料驱动。若输入资料没有提供输出结果覆盖要求，不生成输出结果覆盖 TP，也不报错。

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
- <按验证目标选择 testcase / covergroup / assertion 映射>
```

## 13. 压力测试与 Workload 边界

压力测试暂不作为当前 ST 层面 module 验证主线。

真实软件 workload 场景由独立输入件或独立流程维护，描述单个任务或多个任务组合。ST 压力测试关注多模块联动场景，不在本 Skill 的模块基础 TP 中默认展开。

如果用户明确提供压力测试输入件，可作为后续增强处理；不得自动 cross 配置、输入、debug、输出或性能空间。

## 14. 结构化输出与顺序

### 输出最小充分原则

Skill 的核心输出是按统一 TP schema 生成、按 category 分组的结构化 TP 数据；序列化格式不得反向决定 TP 模型。JSON/YAML 仅用于中间结构或用户明确要求的格式，默认最终交付为 Excel workbook。

默认 workbook 使用 `Register Access`、`Config Space`、`Dynamic Input`、`Cross`、`Debug`、`Performance`、`Output Result` sheet；仅生成实际适用且有 TP 的 sheet。category 由 sheet 表达，不在每行重复输出。每个 TP（包括 blocked TP）在对应 sheet 占一行：`Dynamic Input` 至少包含 `TP_ID`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`；`Config Space` 至少包含 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`verification_scenario`、`expected_result`、`coverage_strategy`；其他 sheet 按统一 TP 模型和必要 identifier fields 输出。`coverage_strategy` 可在单元格中使用简洁、可读的结构化表达，不得为 Excel 新增大量扁平字段。

凡是能由上层分组、Excel sheet、TP_ID 或 Skill 固定规则唯一确定，且删除后不影响 TP 的理解、实现或评审的信息，不在更低层重复输出。不得机械输出 scope、输入对象 metadata、assembly、fixed opcode、execution unit list 或规则解释；只有信息本身确实对 TP 评审或实现必要时才保留。来源依据、Completeness Review、missing input report、generation summary 和 completion summary 属于 inventory/report 层，仅在用户明确要求 review/report 时独立输出，且 report 不得复制完整 TP inventory。

### 生成/输出前自检

生成或输出前逐项检查：

1. 每个 TP 是否符合统一 TP schema。
2. `lifecycle_status` 是否存在。
3. category-specific identifier fields 是否只用于定位。
4. `verification_goal` 是否明确验证对象和覆盖目标。
5. `verification_scenario` 是否只描述覆盖空间/状态和观测点。
6. `verification_scenario` 是否混入 testcase implementation。
7. `expected_result` 是否描述 DUT 可观测行为。
8. `coverage_strategy` 是否描述该 TP 如何被覆盖和判定。
9. 是否存在未确认设计语义却被模型自行推断。
10. 是否存在可以由 sheet、TP_ID 或上层结构确定的重复字段。
11. 是否新增未经定义的 TP 字段体系。
12. 每个 TP 是否被放入正确 category sheet。
13. category 是否被不必要地重复成每行字段。

### 输出顺序

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. Debug sheet。
6. Performance sheet。
7. Output Result sheet。
8. 用户明确要求时，独立输出 todo/missing-input 或 Completeness Review report。

## 15. Module TP Completeness Review

仅在用户明确要求 review/report 时，基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance、Output Result 七个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因；没有明确跨对象关系时，Cross 标记为 `not_applicable`。

Completeness Review **只检查和报告**，不得修改、补充或重新生成已有 TP；发现 `missing` 仅输出待办或缺失资料。
## 16. 核心原则

1. 中文为主，避免不必要英文术语。
2. TP 描述必须具体，不得使用“输入件定义的代表值”这类空泛描述。
3. 不猜测 HDL path、非法处理、输出类别、性能阈值、采样条件或监测方式。
4. 配置空间不引入请求/激励。
5. 动态输入参数不默认和配置空间组合；输入资料明确的 override、precedence、selection、互斥或依赖关系归入 Cross TP。
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
17. Config Space 的 value space 是有效配置状态集合，不是字段 bit 的全部 encoding space；reserved、非法和不支持编码单独定义处理行为。
18. TP 是可评审验证目标，不是 testcase implementation plan；验证场景不得展开 testcase 内部步骤。
19. 默认按 category sheet 输出 TP；inventory、traceability、缺失输入和 review 仅作为按需独立 report 输出。
20. TP 粒度由独立验证目标决定；不能因输入字段数量机械拆分，也不能合并覆盖空间、DUT 行为或预期结果不同的目标。



