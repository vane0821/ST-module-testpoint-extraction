---
name: module-st-testpoint-extraction
description: 面向 ST 层面从模块 spec、寄存器列表、指令/任务描述、接口信号表、debug/performance spec 中提取、整理或审查模块验证 Testpoint。Use when the user asks to generate complete or category-specific module verification TP, manage draft/complete/blocked lifecycle, map coverage strategy, review TP completeness without regenerating TP, or report missing inputs. Covers register access, configuration space, dynamic input parameters, debug, atomic performance, and optional output-result coverage. Default output language is Chinese.
---

# Module ST Testpoint Extraction

默认使用中文输出。目标是从 ST 层面的模块验证视角提取可评审、可落地的 Testpoint（TP），而不是生成完整 block-level DV testplan。

## 1. 职责与边界

本 Skill 负责根据输入资料提取模块验证 TP：

- 寄存器访问属性 TP。
- 配置空间 TP。
- 动态输入参数 TP。
- Debug 能力 TP。
- 原子工作场景性能 TP。
- 可选输出结果覆盖 TP。
- 覆盖策略映射。
- 输入资料不足报告。

本 Skill 不负责：

- 自动生成真实软件 workload 场景；workload 由独立输入件或独立流程维护。
- 自动生成配置 × 指令 / 配置 × 任务全量组合；组合围绕真实 workload 或项目指定典型任务单独维护。
- 默认展开模块压力测试；压力测试作为后续可选增强，不作为当前 ST 层面 module 验证主线。
- 生成通路级非法地址、地址对齐、访问宽度等 TP；这些默认由 Path Verification 覆盖。
- 猜测输入资料没有定义的行为、非法处理、HDL path、采样条件、性能阈值或代表值。

## 2. TP 生成模式与生命周期

各 TP category 独立判断、独立生成和独立输出；一个 category 缺资料不得阻断其他 category。根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读完备性审查；未指定时默认完整生成。

提示词示例：

- `根据这些模块资料生成完整的模块 ST TP，并标注生命周期。`
- `仅生成 LOAD 指令的动态输入参数 TP。`
- `将这份 TP 清单按 draft/complete/blocked 整理，并列出缺失输入。`
- `审查 MU 的 TP inventory 是否覆盖全部适用 category；只报告缺口，不要生成新 TP。`

每个候选项只有一种状态：

- **complete**：验证目标、验证场景、预期结果和明确覆盖策略均已具备。动态参数 TP 的覆盖策略至少定义覆盖空间、覆盖对象和 bins；testcase、covergroup、assertion 的实际映射可后续独立维护。
- **draft**：验证目标和验证场景明确，但 HDL path、monitor mapping、sample event 或覆盖策略绑定尚未完整；必须列出缺失项和完成条件。
- **blocked**：无法形成可验证场景；不生成验证 TP，仅记录 category/source、阻塞原因和恢复所需输入。
## 3. 输入资料与逻辑信息块

输入资料可以是一个或多个文件；同一个文件可以包含多个逻辑信息块，同一个逻辑信息块也可以分散在多个输入资料中。按逻辑信息块检查，不按物理文件数量检查。

常见逻辑信息块：模块 spec、寄存器基础描述和 side effect、字段约束、配置字段到 HDL signal/path 映射、指令编码及功能、任务/descriptor/command 参数、Debug、Performance、输出覆盖、clock/reset、采样条件和可观测映射。

当生成某类 TP 所需信息缺失时：

1. 已能定义验证目标和验证场景，但缺 HDL path、monitor mapping、sample event 或覆盖策略绑定时，生成受影响的 **draft** TP。
2. 无法定义验证目标或验证场景时，记录 **blocked** 项，不生成验证 TP。
3. 不受影响的 category 继续生成。
4. 每项缺失报告必须列出缺失内容、影响范围、当前状态和恢复所需输入；不得猜测或用空泛描述代替具体内容。
## 4. 覆盖策略

TP 的覆盖策略使用结构化描述。只输出实际需要的条目：

```text
覆盖策略:
- testcase: <测试构造方式或 testcase 名称>
- covergroup: <covergroup / coverpoint 名称>
- assertion: <assertion 名称或生成规则>
```

规则：

- complete TP 必须有明确且可追踪的覆盖策略；寄存器、配置、Debug、性能和输出 TP 适用 `testcase`、`covergroup`、`assertion` 映射。动态参数 TP 在未建立 testcase 映射时，以覆盖空间、覆盖对象和 bins 作为覆盖策略。
- draft TP 允许覆盖策略未完整映射，也允许缺 HDL path、monitor mapping 或 sample event，但必须已有明确 `verification_goal` 和验证场景，并列出缺失项和完成条件。
- blocked 项不生成 TP，也不填写覆盖策略。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
## 5. TP_ID 命名规则

统一使用大写、下划线和三位序号。不得保留 `ST`、`REF`、`VAL` 或与本规则并行的旧命名。

- 动态输入参数：`<source>_DYN_<parameter_or_group>_<coverage_space>_<index>`，例如 `VFMV_S_F_DYN_FS1_RANGE_001`、`ADD_DYN_SRC_REG_RANGE_001`、`ADD_DYN_SRC_DATA_002`。
- 模块能力类：`<module>_<category>_<object>_<index>`，例如 `MU_REG_CTRL_ENABLE_RW_001`、`MU_CFG_CTRL_MODE_001`、`MU_DBG_STOP_001`、`MU_PERF_ADD_DUT_LAT_001`、`MU_OUT_STATUS_FLAG_001`。

`source` 只作为 TP_ID 中 instruction/task/descriptor/command 的来源对象；模块能力类以模块名为来源。动态 TP 不输出独立 `source` 字段。`parameter_or_group` 保留覆盖焦点，但不要求机械地一字段一个 TP。
## 6. 寄存器访问属性 TP

寄存器访问属性 TP 独立于功能场景，用于验证寄存器表本身是否实现正确。不要把寄存器访问属性测试和寄存器配置生效行为混在一起。

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

### 字段访问语义组合并

同一目标模块、同一访问 Master、同一寄存器、同一测试类别下，读写规则和预期行为一致的字段可以合并为一个字段访问语义组。

可以不同但仍可合并：字段名、bit range、width、reset value、业务含义。

必须拆分：access type、读行为、写行为、side effect、字段约束处理规则、预期检查方式不同。

### TP 描述

寄存器访问属性 TP 使用简洁格式：

```text
TP ID:
...

验证场景:
...

预期结果:
...

覆盖策略:
- testcase: ...
```

示例：

```text
TP ID:
MU_APB_CTRL_RW_001

验证场景:
APB 对 CTRL 中 RW 字段组执行 bit 0/1 写入覆盖。

预期结果:
每个可写 bit 均至少成功写入 0 和 1，读回值与对应字段写入值一致。

覆盖策略:
- testcase: reg_rw_bit_toggle_test
```

## 7. 配置空间 TP

配置空间 TP 描述软件预先设定的配置字段取值及组合约束。不引入请求、指令、任务或激励。

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

单字段配置覆盖扫描某个配置字段自身取值空间。多字段组合配置覆盖字段之间的依赖、互斥、范围、模式相关约束。

### covergroup 要求

配置空间 TP 默认覆盖策略为 `covergroup`。必须提供配置字段到 HDL signal/path 的映射；缺失时报告输入资料不足，不生成该 TP。

同一寄存器的配置字段尽量合并到同一个 covergroup。TP 中 `覆盖策略` 填 covergroup 名称，covergroup 代码可在后续统一生成。

默认采样语义：

- reset 默认值计入配置覆盖。
- 后续在寄存器字段有效值变化时采样。
- 若字段有特殊生效事件，以输入资料定义为准。

### bins 规则

- 小规模离散空间可全覆盖。
- 大范围字段按输入资料指定边界 / 类别 / 代表集合覆盖。
- 不盲目展开超过 SV 自动建仓限制的大空间。
- 非法配置如需主动测试，不默认写成 `illegal_bins`；是否使用普通 bins、illegal_bins 或 ignore_bins 由输入资料定义。

### TP 描述

```text
TP ID:
...

配置覆盖:
...

预期结果:
...

覆盖策略:
- covergroup: ...
```

示例：

```text
TP ID:
MU_CFG_CTRL_MODE_001

配置覆盖:
CTRL.mode 覆盖 0:normal、1:debug、2:perf、3:reserved。

预期结果:
0/1/2 作为合法配置覆盖；3 作为 reserved 配置覆盖，并按输入资料定义的 reserved 处理规则验证。

覆盖策略:
- covergroup: cg_mu_ctrl_cfg
```

## 8. 动态输入参数 TP

动态输入参数 TP 描述软件可提交到模块的动态输入空间，包括 instruction、task、descriptor、command/request 参数。不要默认展开底层接口 transaction 信号；接口信号表主要用于 HDL path、采样条件、输入有效事件和输出观测点映射。

动态输入 TP 的粒度是一个**可独立构造、独立观测、独立判定的覆盖目标**。TP 数量由覆盖目标决定，不由输入字段数量决定。同一 instruction/task 可以有多个 DYN TP，但不机械地“一字段一个 TP”：仅当参数语义、覆盖空间、预期结果和测试构造方式均一致时允许合并；任一项不同则拆分。

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

TP_ID 保留覆盖焦点；同一来源对象下按覆盖目标直接递增：

```text
VFMV_S_F_DYN_FS1_RANGE_001  # fs1 定义编码范围
ADD_DYN_SRC_REG_RANGE_001   # rs1、rs2 的寄存器编号范围
ADD_DYN_SRC_DATA_002        # rs1、rs2 的寄存器数据集合
```

`range` 覆盖字段定义的编码或数值范围；`data` 覆盖参数承载的数据；`address` 覆盖内存地址空间；`format` 和 `mode` 仅在输入明确要求时使用。TP 必须显式列举编码、范围、边界或类别；小规模离散空间全覆盖，大空间仅使用输入明确的集合。当前不默认生成 opcode × parameter、parameter × parameter 或动态参数 × 配置空间组合。

若字段的定义 bit range 大于实际生效的 bit range，扫描字段**定义**的完整 bit range；实际有效位、保留位和非法处理方式只能按输入规格写在 `expected_result`，不得默认推导截断、alias 或高位忽略。

除非 TP 明确覆盖多个参数的组合关系，未扫描参数均取输入资料定义的合法基线值。不要在每个场景重复“其余字段合法”；只有该基线值影响预期结果时，才显式列出具体值。

### TP 描述

动态输入参数 TP 不输出 `category`、`source`、`input_basis` 或 `operation`；它们分别由 TP 所在章节/TP_ID、TP_ID、输入资料清单和参数语义规则承担。`parameter` 可以是单个参数，也可以是同语义参数组。虽然不输出 `source` 字段，`verification_goal`、`verification_scenario` 与 `coverage_target` 必须保留 instruction/task/descriptor 名称、参数名称及被覆盖的硬件对象或编码空间，不得使用无法定位对象的泛化措辞。complete TP 使用：

```text
TP ID:
...

生命周期:
complete

参数:
- <parameter 或同语义参数组成员>

参数类型:
<reg / imm / mem / ...>

验证目标:
覆盖 <instruction/task/descriptor 名称> 的 <parameter> 在 <硬件对象、字段或编码空间> 的 <具体覆盖空间>。

验证场景:
向 <instruction/task/descriptor 名称> 构造 <parameter> 的 <具体值、范围或类别>；在 <硬件对象或观测点> 采样。未扫描参数取输入资料定义的合法基线值。

预期结果:
只描述被扫描参数到 DUT 输入语义的映射、合法值处理、输入资料明确的非法/保留处理，或定义 bit range 与实际有效位的关系；不得描述 instruction 完整执行结果、算法计算、destination 数据结果或数据通路功能正确性。

覆盖策略:
- coverage_space: <range / data / address / format / mode>
- coverage_target: <参数名或字段 bit range>
- bins:
  - <需统计的值/范围/类别>
- illegal_bins: <仅在规格定义该采样值不应出现时输出>
- ignore_bins: <仅在值不纳入覆盖统计时输出>
```

`illegal_bins` 仅表示覆盖模型中的非法采样值，不代表 DUT 必须报错；`ignore_bins` 仅表示不参与覆盖统计的采样值，不代表 DUT 行为异常。DUT 对这些值的处理方式只能通过 `expected_result` 描述。testcase、covergroup、assertion 可以后续维护为实现映射，但不是动态参数 TP 为 complete 的必要字段。

示例：

```text
TP ID:
VFMV_S_F_DYN_FS1_RANGE_001

生命周期:
complete

参数:
- fs1

参数类型:
reg

验证目标:
覆盖 VFMV_S_F 的 fs1 在 SRF_RD_P1_IDX 定义编码空间中的寄存器索引范围。

验证场景:
向 VFMV_S_F 提交 fs1 覆盖 SRF_RD_P1_IDX 定义 bit range 的编码；在 SRF_RD_P1_IDX 输入处采样。未扫描字段取输入资料定义的合法基线值。

预期结果:
定义 bit range 内的所有编码均参与扫描；实际有效位、保留位和非法处理方式按照输入规格定义。

覆盖策略:
- coverage_space: range
- coverage_target: SRF_RD_P1_IDX 的定义 bit range
- bins:
 - 定义 bit range 的覆盖集合
```

### 反例检查

- `VFMV_S_F_DYN_OPCODE_RANGE_001` 不得生成：`OPCODE=0x21` 是识别 `VFMV_S_F` 的固定编码，不是该 instruction 的动态参数。跨 instruction set 的 opcode decode 扫描属于独立 decode 覆盖目标。
- `VFMV_S_F_DYN_FS1_RANGE_001` 应生成：`fs1` 是动态参数；定义 bit range 内的所有编码参与扫描，实际有效位、保留位和非法处理方式只按输入规格写在 `expected_result`。

draft TP 使用相同格式，但 `覆盖策略` 可为 `null`；必须追加：

```text
缺失信息:
- <HDL path / monitor mapping / sample event / 覆盖策略绑定>

完成条件:
- <补齐缺失信息后转为 complete 的条件>
```

### 指令输入资料

需要指令编码、功能描述、字段到 HDL signal/path 映射、有效接收/decode/issue 采样事件、clock/reset，以及非法/reserved 编码处理规则。固定 opcode 用于识别 instruction，不视为该 instruction 的动态参数；跨 instruction set 的 opcode decode 覆盖应作为独立 decode 目标处理。任务/descriptor/command 按相同模型处理：先定义参数类型和覆盖空间，再显式列举值、合法/非法分类和处理规则。
## 9. Debug 能力 TP

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

debug能力:
...

触发时机:
- <name>: <logic expression>

预期行为:
状态更新：...

覆盖策略:
- testcase: ...
- covergroup: ...
- assertion: ...
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

## 10. 性能验证 TP

性能验证描述原子工作场景性能。`Level0` 是外层规划概念，TP 内不反复写 Level0。

原子工作场景包括：

- 单条指令。
- 单个任务。
- 单个 command / descriptor。
- 同类原子单元连续流，用于 throughput / bandwidth 测量。

### TP 描述

```text
TP ID:
...

性能场景:
...

性能监测方式:
DUT monitor / 软件行为监测 / DUT monitor + 软件行为监测。

测量边界:
...

外部干扰条件:
...

性能目标:
...

覆盖策略:
- testcase: ...
- assertion: ...
```

规则：

- `性能监测方式` 只选择 DUT monitor、软件行为监测或二者同时使用。
- 软件行为监测必须由输入资料说明支持方式，例如 rdcycle、driver timestamp、polling、software trace、perf API，并映射到 TP。
- `测量边界` 必须与监测方式匹配，不得把 DUT event 的阈值套到软件行为监测上。
- `外部干扰条件` 可选；默认无外部干扰。若关心 ready pattern、wait state、带宽限制、外部模块干扰，必须在这里明确。
- `性能目标` 必须同时包含指标公式 / 统计方式和阈值。
- 性能 TP 可与功能 TP 共用 testcase，但 TP 不合并。
- covergroup 用于性能分布/趋势分析时暂不默认考虑；只有输入资料明确要求时输出。

示例：

```text
TP ID:
MU_PERF_ADD_DUT_LAT_001

性能场景:
MU 单条 ADD 指令执行。

性能监测方式:
DUT monitor。

测量边界:
start = DUT 接收 ADD 指令；end = DUT 产生 ADD done。

外部干扰条件:
无外部干扰。

性能目标:
DUT latency = end_cycle - start_cycle，要求 <= 输入件定义的 MU ADD DUT latency 阈值。

覆盖策略:
- testcase: mu_add_dut_latency_test
```

```text
TP ID:
MU_PERF_ADD_SW_LAT_001

性能场景:
软件提交 MU 单条 ADD 指令并等待结果可见。

性能监测方式:
软件行为监测：rdcycle。

测量边界:
start = 软件提交 ADD 前读取 rdcycle；end = 软件观察到 ADD 完成后读取 rdcycle。

外部干扰条件:
无外部软件 workload 干扰。

性能目标:
软件可见 latency = end_cycle - start_cycle，要求 <= 输入件定义的 MU ADD 软件可见 latency 阈值。

覆盖策略:
- testcase: mu_add_sw_latency_test
```

## 11. 可选输出结果覆盖 TP

输出结果覆盖 TP 完全由输入资料驱动。若输入资料没有提供输出结果覆盖要求，不生成输出结果覆盖 TP，也不报错。

不得从“支持浮点”自动推出 NaN / Inf / subnormal；不得从“有 status/error register”自动推出状态覆盖。

格式：

```text
TP ID:
...

验证目标:
计算结果 / 寄存器状态 / error code / flag：覆盖 ...

预期结果:
...

覆盖策略:
- testcase: ...
- covergroup: ...
```

## 12. 压力测试与 Workload 边界

压力测试暂不作为当前 ST 层面 module 验证主线。

真实软件 workload 场景由独立输入件或独立流程维护，描述单个任务或多个任务组合。ST 压力测试关注多模块联动场景，不在本 Skill 的模块基础 TP 中默认展开。

如果用户明确提供压力测试输入件，可作为后续增强处理；不得自动 cross 配置、输入、debug、输出或性能空间。

## 13. 输出顺序

建议按以下顺序输出：

1. 输入资料完整性检查。
2. 寄存器访问属性 TP。
3. 配置空间 TP。
4. 动态输入参数 TP。
5. Debug 能力 TP。
6. 性能验证 TP。
7. 可选输出结果覆盖 TP。
8. draft TP 清单。
9. blocked 项与输入资料不足报告。
10. Module TP Completeness Review。

## 14. Module TP Completeness Review

最后基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Debug、Performance、Output Result 六个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因。

Completeness Review **只检查和报告**，不得修改、补充或重新生成已有 TP；发现 `missing` 仅输出待办或缺失资料。
## 15. 核心原则

1. 中文为主，避免不必要英文术语。
2. TP 描述必须具体，不得使用“输入件定义的代表值”这类空泛描述。
3. 不猜测 HDL path、非法处理、输出类别、性能阈值、采样条件或监测方式。
4. 配置空间不引入请求/激励。
5. 动态输入参数不默认和配置空间组合。
6. Debug 使用模板生成候选，并依赖项目输入件迭代。
7. 性能 TP 的测量边界、监测方式和性能目标必须一致。
8. 输出结果覆盖和压力测试均为可选输入件驱动项。
9. category 独立生成；complete、draft、blocked 的覆盖策略要求不得混用。
10. 动态参数使用 parameter type + coverage space，不得回退到 reference/content 模型。
11. verification_goal 描述覆盖目标。动态参数 TP 的 expected_result 只描述参数到 DUT 输入语义的映射、合法/非法处理以及有效位关系，不描述 instruction 算法结果。
12. Completeness Review 只报告既有 inventory 的缺口，不生成新 TP。



