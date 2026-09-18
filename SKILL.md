---
name: module-st-testpoint-extraction
description: 面向 ST 层面从模块 spec、寄存器列表、动态输入描述、接口信号表、debug/performance spec 中提取、整理或审查模块验证 Testpoint。Use when the user asks to generate complete or category-specific module verification TP, manage draft/complete/blocked lifecycle, map coverage strategy, review TP completeness without regenerating TP, or report missing inputs. Covers register access, configuration space, dynamic input parameters, cross relationships, debug, atomic performance, and optional output-result coverage. Default output language is Chinese.
---

# Module ST Testpoint Extraction

默认使用中文输出。

## 1. Core Model

### 1.1 Scope and Boundary

从 ST 模块视角提取可评审、可落地的 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance 和可选 Output Result TP，并输出 coverage mapping 与缺失输入报告。

范围限于模块自身语义；通路级地址、对齐和访问宽度属于 path-level 验证，真实软件 workload 与压力测试不自动进入本 Skill。TP 定义验证目标与覆盖要求，不定义 testcase implementation、stimulus scheduling、sequence algorithm 或 checker implementation。

### 1.2 Category Object Model

| Category | 最小对象模型 |
|---|---|
| Register Access | access action + expected behavior |
| Config Space | 单个配置对象的完整 value space + input-defined semantics |
| Dynamic Input | 单个请求输入参数在一个 verification dimension 下的完整 input subspace + input-defined semantics |
| Cross | 两个或多个 Config / Dynamic 对象共同决定的通用约束、映射、合法性或结果 |
| Debug | 输入资料明确支持的异步输入级 debug capability |
| Performance | 原子工作场景的性能目标 |
| Output Result | DUT 输出对象本身的独立结果空间 |

各 category 独立生成，一个 category 缺资料不阻断其他 category。对象粒度和允许的合并方式以对应 Category Rules 为准；通用规则不得覆盖 category-specific object model。

### 1.3 Common Field Responsibilities

所有 TP 以 `TP_ID`、`lifecycle_status`、category-required identifier、`verification_goal` 和 `coverage_strategy` 为基础；`verification_scenario`、`expected_result`、`coverage_strategy_mapping` 是否固定输出由 Output Contract 和 Category Rules 决定。

- identifier 只定位对象；`verification_goal` 定义验证目标；`verification_scenario` 只描述场景、状态或观测范围；`expected_result` 写可观测结果；`coverage_strategy` 写验证/覆盖方式；`coverage_strategy_mapping` 写具体实现对象或其待补实现绑定。
- 不得新增与上述职责平行的目标、场景、结果、覆盖或 mapping 字段体系，也不得让一个字段承担另一个字段的职责。

### 1.4 Lifecycle

- **complete**：对象、`verification_goal`、验证方法及 Category Rules 要求的其他信息均完整；仅缺 HDL signal / path、clock、sample event 或其他非 category-required implementation binding 时仍为 complete，并按 Coverage Responsibility 的 implementation TODO 格式显式记录。
- **draft**：对象和验证方向成立，但完成目标所需的 design semantic，或 Category Rules 明确要求的 implementation input / mapping 缺失。
- **blocked**：对象、目标方向、策略方向或必要行为判定无法成立；仍输出 blocked TP 行并保留已知信息，不猜测未知字段。

`lifecycle_status` 不得删除。draft / blocked 的缺口按 Missing-input Flow 输出；一个 category 的状态不改变其他 category。

### 1.5 Coverage Responsibility and Precedence

`verification_goal` 承载对象、条件与预期语义；`coverage_strategy` 承载 coverage / verification method；`coverage_strategy_mapping` 只承载 testcase、assertion 或 coverage implementation object。method 默认由 verification intent 决定，Category Rules 已固定 method 时以 category-specific rule 为准。

- behavior TP 使用实际需要的 testcase、assertion 或其他 method，不为字段完整性制造 bins。
- assertion 需要明确时序、安全、边界或状态约束；covergroup 的 coverage object 默认由当前 TP 行确定，默认采样和 implementation structure 由生成脚本处理，仅当输入资料定义特殊采样条件时在 `coverage_strategy` 输出实际 `sample event`；testcase 只表示构造方式，不展开实现步骤。
- verification goal 与 method 已完整、仅缺非 category-required implementation binding 时，`coverage_strategy` 只写实际 `<method>`，`coverage_strategy_mapping` 固定写 `TODO（missing <具体 implementation input>）`；不得用笼统的 `missing implementation input`。生成脚本识别该 TODO 后不生成对应 SV，并输出明确的 implementation TODO。此情况不改变 complete、不进入 lifecycle missing-input report，也不属于 Cross Skill Draft。Category Rules 明确要求的 implementation input / mapping 仍按其规则判定 Lifecycle。
- Config / Dynamic TP 声明独立 value/input-space coverage 时读取 [Value/Input-Space Coverage Contract](references/value-input-space-coverage.md)；Cross expression 与 cross coverage 读取 [Cross Expression Contract](references/cross-expression.md)。不适用时不得机械增加 bins 或 covergroup。

### 1.6 TP_ID

TP_ID 使用大写、下划线和三位连续序号；不使用 `ST`、`REF`、`VAL` 或平行旧命名。

- Register Access：`<module>_REG_<register>_<access_type>_<index>`。
- Config Space：`<module>_CFG_<config_object>_<index>`。
- Dynamic Input：`<module>_DYN_<parameter>_<coverage_space>_<index>`。
- Cross：`<module>_CROSS_<object>_<index>`，其中 `object` 表示已明确的跨对象关系焦点。
- 其他：`<module>_<category>_<object>_<index>`，category code 使用 `DBG`、`PERF`、`OUT`。

TP_ID 只表达归属、category、验证焦点和唯一性，不绑定输入资料层次；当前 Prompt 的目标名称不得成为 Base TP 的额外身份。

### 1.7 Global Output Principles

- 不推断输入资料未定义的 design semantic、HDL path、处理行为、阈值、采样或观测方式；描述必须具体，不使用“按输入件定义”等空泛占位。
- 能由 sheet、TP_ID、schema 或固定规则唯一确定，且删除后不影响理解、实现或评审的信息不重复输出。对象 ownership 与 category boundary 以 Category Rules 为准。
- identifier、signal、enum、opcode、encoding、error code 和代码表达式保持输入原文；自然语言 behavior、semantic、explanation 和 missing-input 默认使用中文。
- 所有 TP 表达必须同时满足：简洁、人工可读、脚本可确定性生成 SV，且不在 Excel 中展开大量 implementation syntax；各 category 的具体紧凑表达由对应规则定义。
- Cross 与 Scenario constraint 以 semantic exactness 和人工可读性为先：简单关系直接表达，复杂 count / resource / cardinality 使用结构化中文；每个 independent branch 独立且完整，不使用 `...`、复杂符号压缩或合并独立规则。

## 2. Category Rules

### 2.1 Register Access

**输入要求**

| Access target | 最小必需信息 |
|---|---|
| RESET | register、register width，以及输入资料给出的整体 reset value，或可无歧义拼出该值的完整 field / bit range / reset value |
| R / RO | register、全部同类 field、bit range 和明确 access property |
| RW / RESERVED RAW | register width、全部 RW 与 RESERVED field / bit range，以及足以唯一计算每组写入后整体 rdata 的 access property |
| 纯 RESERVED | register、全部 RESERVED field 及 bit range |
| 其他 access property | 对象、对象粒度、access action、condition（如有）和 expected behavior |

输入质量统一按 Input Processing 判断，缺项统一按 Lifecycle 处理。RESERVED 的固定 access semantic 无需输入资料为每个 register 重复定义。

**生成范围**

- 按寄存器处理 RESET、R / RO、RW、RESERVED 和输入资料明确的其他 access property。
- 验证边界是 access action 与 expected behavior；写 trigger、start、kick 或配置变更后产生的独立模块功能行为不属于 Register Access。
- 只根据输入资料识别访问属性和功能行为，不根据字段名称推断。
- 每个适用对象都进入完整 Base Inventory，并按 Lifecycle 标记 complete、draft 或 blocked；当前功能场景不裁剪该 inventory。
- 存在寄存器表却没有生成对应 inventory 时，生成不完整。只有 Completeness Review 将完全缺失的 inventory item 标记为 `missing`。

**TP 粒度与行为**

| Access type | TP 粒度 | `verification_goal` |
|---|---|---|
| RESET | 每个具有明确整体 reset value 的 register 一条 | `复位值检查：<register> = <actual reset value>`；只写整体寄存器值，不重复展开各 field |
| R / RO | 每个 register + access type 一条 | `<access_type> 属性检查：对 <fields / bit ranges> 尝试写入，确认对应 rdata 保持原值` |
| RW | 每个含 RW field 的 register 一条 | 使用四组固定全宽 wdata 执行 RAW；逐组列出 wdata 和按全部 RW / RESERVED 属性计算的整体 expected rdata |
| RESERVED | 每个含 RESERVED field 的 register 一条 | 与同一 register 的 RW TP 共用 RAW verification goal 和 testcase；无 RW 时独立表达 RESERVED 检查 |
| 其他 access property | 跟随输入资料定义的 register-level 或 field-level 对象 | 写明实际 access action、condition 和 result |

同一 register 的 RW 与 RESERVED 保留独立 TP_ID，但共用一个 RAW verification goal 和 testcase。四组 wdata 固定为寄存器全宽的全 0、全 1、`01` 交织和 `10` 交织；`01` 表示从最高位观察为 `0101...`，`10` 表示 `1010...`。wdata 不按 RW mask 预先裁剪；expected rdata 必须逐 bit 根据 RW 写入生效、RESERVED 读回为 0 及输入资料明确的其他 access property 计算。无法唯一计算任一整体 expected rdata 时按 Lifecycle 处理，不得猜测。R / RO 和其他 access property 仍按各自对象粒度分别生成，不跨 register 合并。

**Coverage 与 Lifecycle**

- RESET TP 固定设置 `coverage_strategy = assertion`，`coverage_strategy_mapping = <module>_reg_reset_assertion`。
- 同一 register 的 RW 与 RESERVED TP 固定设置 `coverage_strategy = testcase`，共同映射 `<module>_reg_rw_test`；R / RO、无 RW 的 RESERVED 和其他 Register Access TP 使用 `<module>_reg_<access_type>_test`。
- mapping 是 Register Access 的 category-required 字段，缺失时 TP 不能标记为 complete。
- mapping 只表示 testcase 名称；信息不足时按 Lifecycle 处理。

**排序与输出**

- 按寄存器表顺序处理；同一 register 的 TP 在 sheet 中连续排列。
- register 内依次生成 RESET、R / RO、RW、RESERVED 和按输入资料粒度生成的其他 access property。
- 同一 register 的 RW 与 RESERVED 行保持相邻；两者内容相同的 `verification_goal`、`coverage_strategy` 和 `coverage_strategy_mapping` 在最终 Excel 中分别纵向合并。`TP_ID` 与 `lifecycle_status` 保持独立，不合并。
- `register` 使用寄存器名称，`access_type` 使用访问属性；RESERVED TP_ID 使用 `<module>_REG_<register>_RESERVED_<index>`。
- `index` 按 Register Access sheet 的最终行顺序连续递增，不按 access type 分组，也不因新 register 重置。
- 实际值、行为和条件写入 `verification_goal`，不单设 `expected_value`；其余字段遵循 TP Sheet Schema。

### 2.2 Config Space

**输入要求**

| 信息 | 最小必需内容 |
|---|---|
| 对象 | `config_object`、field、bit range |
| 值空间 | 完整 enum、encoding、range 或 category |
| 单字段语义 | 输入资料为各 value / range / category 定义的 selection、behavior、legality 或其他 semantic |
| 分类依据 | 该对象在请求前设置并在单个请求执行期间保持稳定 |

字段当前位置未写完整 semantic 时，先从全部输入资料中的定义、表格、功能描述和映射关系归并已有信息。能够无歧义建立完整 value-to-semantic mapping 时使用归并结果；信息冲突、对应关系不唯一或全部输入资料仍缺失时按 Lifecycle 处理。归并不等于推断，不得根据字段名称、经验或常识创造 semantic。

**生成范围**

- Configuration 不随当前请求携带，在请求前设置，并在单个请求执行期间保持稳定；分类不依据 spec 中 static、dynamic 或 dynamic configuration 等名称。更新与生效时机无法确认时按 Lifecycle 处理。
- Config Space 固定为单对象 value-space TP；每个 TP 只对应一个 `config_object`、一个 `field` 和一个完整单对象 value space。每个适用对象都进入 Base Inventory，不受当前功能场景裁剪；完全缺失仅在 Completeness Review 中标记 `missing`。
- 只有 register access semantic 的 RESERVED field 不进入 Config Space，由 Register Access 覆盖；有效配置字段内部的 reserved / unsupported encoding 仍属于该字段 value space。
- 引用其他 Config / Dynamic 对象才能成立的 constraint、mapping、legality 或 result 只进入 Relation Extraction 与 Ownership，不复制进 Config TP。scenario-independent relation 由 Cross 承载，scenario-specific relation 由 Scenario 承载。

**TP 粒度与行为**

| Value-space 类型 | TP 粒度 | `verification_goal` |
|---|---|---|
| Enum / encoding | 每个 field 一条 | 每行 `<value / range> -> <semantic>`，完整列出已定义与 remaining / reserved / unsupported space |
| Address | 每个 field 一条 | 按 Value/Input-Space Coverage Contract 的 Address Coverage Profile 输出实际地址目标；不扩展为 path-level 通路验证 |
| Numeric range | 每个 field 一条 | 使用 `<range> -> <semantic>`；输入资料明确定义特殊值或分类时分别列出 |
| Selector / mapping | 每个 field 一条 | 每行 `<value / range> -> <selected object / behavior>`；只给候选编码但缺实际 selection semantic 时不能标记 complete |
| 单字段条件行为 | 每个 field 一条 | 完整表达 `if (<condition>) -> <result>; else -> <result>` |
| Multi-object relation | 不生成 Config TP | 进入 Relation Extraction 与 Ownership |
| Access-only RESERVED | 不生成 Config TP | 由 Register Access 覆盖 |

`verification_goal` 只保留能改变验证判断的信息，不添加“编码空间为”“输入资料定义的候选值”等前置说明。只阅读当前 TP 而不查询原 spec 时，reviewer 必须能知道当前 field 待遍历的值及其单字段语义。若输入资料明确某 selector 只检查 opaque candidate-ID membership、模块不解释具体 selection，则完整合法集合本身可作为 semantic；否则 selector 必须给出实际 value-to-selection mapping。

**Coverage 与 Lifecycle**

- 需要独立 value-space coverage 时满足 [Value/Input-Space Coverage Contract](references/value-input-space-coverage.md)；否则按实际 intent 选择 method，不制造 structured bins 或 covergroup。
- Config 不固定 testcase 或 `coverage_strategy_mapping` 名称；mapping 不是 category-required 字段，未知时允许为空。
- `complete`：对象、完整值空间及全部输入资料已定义的单字段 semantic 均可直接判定；selection field 已具有完整 mapping，或输入资料明确只验证 opaque candidate-ID membership；address field 存在可表示越界空间时，越界行为也已明确。
- `draft`：对象和值空间成立，但跨文档信息冲突、对应关系不唯一，或仍缺少完成目标所需的 classification、semantic、value-to-selection mapping 或 address 越界行为。
- `blocked`：无法确认对象是否属于 Configuration，或无法建立有效的单字段 value space / verification direction。

**排序与输出**

- 按输入资料中的 `config_object` 和 field 顺序处理；同一 `config_object` 的 TP 在 sheet 中连续排列。
- `index` 按 Config Space sheet 最终行顺序连续递增，不按对象或 value-space 类型分组，也不因新对象重置。
- 最终字段遵循 Config Space TP Sheet Schema，不增加 source、input basis 或解释字段。

### 2.3 Dynamic Input

**输入要求**

| 信息 | 最小必需内容 |
|---|---|
| 对象 | `parameter`、`parameter_type`、bit range 或 encoding width |
| Verification dimension | range、data、address、format、mode 或输入资料定义的其他单一覆盖维度 |
| Input subspace | 当前 dimension 的完整值、范围或类别 |
| 单参数语义 | 输入资料定义的 valid、invalid、reserved、unsupported 或其他行为分类 |
| 分类依据 | 该参数随当前请求携带，并可在不同请求间变化 |

字段当前位置未写完整 semantic 时，先从全部输入资料中的定义、表格、功能描述和映射关系归并已有信息。能够无歧义确定对象、dimension 和 input subspace 时使用归并结果；信息冲突、对应关系不唯一或仍缺失时按 Lifecycle 处理。不得根据参数名称、类型或经验创造 semantic。

**生成范围**

- Dynamic Input 只包含随当前请求携带的 DUT 输入参数。运行时变化、名称包含 dynamic 或存放在寄存器中，均不能单独作为分类依据；请求前设置并在单个请求期间稳定的对象属于 Config Space。
- 接口 signal、HDL path、valid / sample event、clock、reset 和 monitor mapping 是实现信息，不自动形成 Dynamic inventory。内部状态、中间结果和输出不属于 Dynamic Input，按其验证目标进入 Debug、Performance、Output Result 或 Scenario。
- 固定 opcode、mode 或识别字段仅作为 applicability condition 时不自动成为扫描项；输入资料明确要求覆盖其输入空间时，才作为相应 Dynamic parameter 处理。
- 每个适用 `parameter × verification dimension` 都进入 Base Inventory，不受当前 Scenario 或 Config 条件裁剪。
- 多参数联合 constraint、mapping、legality 或 result 进入 Relation Extraction 与 Ownership，不复制进单参数 Dynamic TP。

**TP 粒度与行为**

一个 Dynamic TP 固定对应一个 `parameter × verification dimension`。同一 parameter 的不同 dimension 分别生成；不同 parameter 不得合并。

| Verification dimension | `verification_goal` |
|---|---|
| Range / numeric | 直接列出合法范围、上下边界、确定性典型值、可表示的下越界/上越界空间，以及输入资料定义的越界行为 |
| Data / pattern | 当前 dimension 的完整数据模式空间及输入资料定义的分类语义 |
| Address | 按 Value/Input-Space Coverage Contract 的 Address Coverage Profile 输出实际地址目标；不扩展为 path-level 通路验证 |
| Enum / format / mode | 每个 `<value / range> -> <semantic>` 独占一行，覆盖已定义与 remaining / reserved / unsupported space；不得输出 `<semantic> -> 同名 semantic` 的同义反复 |
| 单参数条件行为 | 完整表达 `if (<condition>) -> <result>; else -> <result>` |

`parameter_type` 只描述参数类型，如 reg、imm、mem、mask、enum、index；verification dimension 由 TP_ID 的 `coverage_space` 与 `coverage_strategy` 一致表达。`verification_goal` 必须写出当前 input subspace 的实际值、范围或类别，不能用 coverage space 名称代替。不得使用 reference/content、`REF` 或 `VAL`。

字段定义 bit range 大于实际生效 bit range 时仍覆盖完整定义范围；实际有效位、保留位和非法处理仅在输入资料明确时表达。

**Coverage 与 Lifecycle**

- 需要独立 input-space coverage 时满足 [Value/Input-Space Coverage Contract](references/value-input-space-coverage.md)；否则按实际 intent 选择 method，不制造 structured bins 或 covergroup。
- Dynamic Input 不固定 testcase 或 `coverage_strategy_mapping` 名称；mapping 不是 category-required 字段，未知时允许为空。
- `complete`：对象、dimension、完整 input subspace 及输入资料定义的单参数 semantic 均可直接判定；numeric range / address 存在可表示越界空间时，越界行为也已明确。
- `draft`：对象与 dimension 成立，但 subspace、classification、semantic、可表示越界空间的行为，或 category-required implementation input 不完整。
- `blocked`：无法确认对象是否随请求携带，或无法建立有效的 dimension / verification direction。

**排序与输出**

- 按输入资料中的 parameter 顺序处理；同一 parameter 的各 dimension 连续排列。
- TP_ID 使用 `<module>_DYN_<parameter>_<coverage_space>_<index>`；`coverage_space` 表示当前 verification dimension，`index` 按 Dynamic Input sheet 最终行顺序连续递增。
- 最终字段遵循 Dynamic Input TP Sheet Schema，不增加 source、input basis、operation 或 traceability 字段。

### 2.4 Cross

**输入与范围**

Cross 只承载两个或多个 Config / Dynamic 对象共同决定的 constraint、mapping、legality 或 result。Config / Dynamic 保留单对象空间；多对象关系不复制进单对象 `verification_goal`。单字段 reserved / illegal encoding 不构成 Cross。

Base Cross 只承载脱离当前 Target Scenario 后仍成立的通用模块关系；仅依赖当前 instruction / function / Scenario 才成立的关系进入 Scenario。关系带 applicability condition 不表示它一定属于 Scenario；ownership 无法唯一确定时进入 relation ownership missing-input。

**TP 粒度与表达**

一条 Cross TP 完整表达同一组参与对象共同决定的一个关系目标。同一关系目标的多个条件分支放在同一 TP 内；参与对象、关系目标不同，或合并后不能逐分支判断覆盖时分别生成。

`verification_goal` 使用 [Cross Expression Contract](references/cross-expression.md) 支持的紧凑形式并保留实际换行。输入关系完整但该 Contract 无法无损表达时进入 Cross Skill Draft；不得改写成近似关系、宽泛概念或伪 Cross TP。

**Coverage 与 Lifecycle**

Cross 固定使用 cross coverage，并按 Cross Expression Contract 输出 `cross bins`；每个 goal branch 均有对应 bin，不写 compatible、mismatch、valid combination 等无法还原实际对象和值的抽象 bin。testcase 只负责产生目标组合，REF / scoreboard 负责依据 `verification_goal` 计算预期并判定 DUT 结果；二者均不作为 Cross TP 的 `coverage_strategy`，Cross 不重复生成 assertion 或结果 checker。

- `complete`：参与对象、完整关系、结果或合法性均明确，且现有 Contract 可将各关系分支无损转换为 cross bins；仅缺非 category-required implementation binding 时按通用 implementation TODO 规则记录。
- `draft`：关系方向成立，但输入资料缺少完成 cross bins 所需的条件、结果、行为或 category-required implementation input。
- `blocked`：参与对象、关系方向、策略方向或必要行为判定无法成立。
- **Cross Skill Draft**：输入关系已经明确，但现有 Cross 表达模型或脚本转换能力无法无损承载；它不是 TP lifecycle，不生成 TP_ID。

Cross 属于 Base Inventory，不根据 Scenario 临时生成、裁剪或修改。`expected_result` 默认不输出，仅在结果无法自然并入受支持的 `verification_goal` 表达时允许输出。

### 2.5 Debug

Debug 属于异步输入级别能力，一个 debug capability 一个 TP。capability 只包括输入资料实际支持的 stop、resume、single step、breakpoint；不支持或未定义的 capability 与触发条件不生成。

`verification_goal` 根据输入资料明确的触发条件和状态行为生成，触发条件使用逻辑表达式。明确的 drain、上下文保持、禁止新交易、resume 后结果一致或错误状态等行为纳入对应目标，不得补充未定义行为。

按验证目标选择 testcase、covergroup 或 assertion。仅在输入资料明确状态更新时序、安全约束或状态约束时生成 assertion。

### 2.6 Performance

Performance 描述原子工作场景性能；`Level0` 是外层规划概念，不在 TP 内反复书写。原子工作场景包括单个可执行对象，以及用于 throughput / bandwidth 测量的同类原子单元连续流。

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

- coverage / observation target 描述测量什么，即 measured latency、throughput、bandwidth 等 performance metric；不得承载 allowed range、threshold、acceptance interval 或其他 pass/fail criteria。
- `verification_scenario` 写明输入资料定义的性能条件和 DUT monitor 或软件可观察对象，不展开监测实现步骤。
- `expected_result` 描述允许什么结果，必须承载输入资料定义的指标公式或统计方式、测量边界、allowed range、threshold 或 acceptance rule，并与观测对象匹配。
- 外部干扰条件仅在影响覆盖空间、预期行为或输入资料明确要求时写入。
- 性能 TP 可与功能 TP 共用 testcase，但 TP 不合并。
- coverage strategy 仅描述如何覆盖或观测 performance metric，并与监测方式、测量边界一致。性能分布或趋势分析的 covergroup 不默认生成，仅在输入资料明确要求时输出。

性能场景、监测方式和测量边界按输入资料生成，不使用固定完整 demo。

### 2.7 Output Result

Output Result 只用于 DUT 输出对象本身存在的独立结果覆盖空间，例如 destination data、calculation result、output data/packet、status/flag、error code 或 result register state。输入资料未提供独立结果覆盖要求时不生成，也不报错。

其他 category 中用于判定当前行为是否正确的输出按该 category 规则表达，不因此生成 Output Result；只有输出对象本身需要独立遍历或覆盖其结果空间时才生成。`vd index` 可属于 Dynamic Input，`vd data` 属于 DUT Output；仅当 `vd data` 有独立结果覆盖目标时生成 Output Result。error/status 仅用于当前 TP 判定时也不生成；只有其自身存在待覆盖类别或状态空间时才生成。

不得从“支持浮点”自动推出 NaN / Inf / subnormal，不得从“有 status/error register”自动推出状态覆盖。

`verification_goal` 写计算结果、寄存器状态、error code 或 flag 的明确覆盖目标；`verification_scenario` 写输入资料定义的结果类别及结果寄存器、status、error code 或 flag 等观测点；`expected_result` 写明确的 DUT 可观测结果；coverage strategy 按目标选择 testcase、covergroup 或 assertion。

## 3. Execution Flow

### 3.1 Input Processing

输入资料可以是一个或多个文件；同一个文件可以包含多个逻辑信息块，同一逻辑信息块也可分散在多个文件中。按逻辑信息块检查，不按物理文件数量检查。

输入资料按实际内容读取 module spec、寄存器基础描述和 register access property、字段约束、配置到 HDL 的映射、动态输入描述、Debug、Performance、可选输出覆盖、clock/reset、采样条件、可观测映射等逻辑信息块；这些是可能出现的信息类型，不构成固定文件要求或统一 mandatory input checklist。

某项信息是否为当前 TP 必需，由 Core Model、Lifecycle、Coverage Responsibility and Precedence 和对应 Category Rules 判断。仅缺非 category-required coverage implementation binding 时保持 complete 并记录 implementation TODO；只有 Category Rules 明确要求的 implementation input / mapping 缺失才因此进入 draft。不得因本节列举的逻辑信息块改变 complete / draft / blocked 判定。

所有 Config / Dynamic multi-object relation 使用唯一处理流：`All Input Documents -> Relation Extraction -> Relation Atom -> Ownership(Base Cross / Scenario-specific / Pending) -> Cross Expression Fit -> Cross TP / Scenario Expression / Cross Skill Draft / relation ownership missing-input`。Relation Extraction 不使用 Target Scenario 过滤，从 module spec、Config、Dynamic Input、功能描述、instruction/function 描述等实际内容中识别两个或多个 Config / Dynamic 对象共同决定的 constraint、mapping、legality、DUT result 或其他明确关系；multi-object semantic 不得由任何 category 旁路决定归属或复制进单对象 TP。

内部最小判断单位为 `Relation Atom = participants + relation`。relation 可以是带条件的结果，也可以是多个对象直接决定目标对象的等式或映射；必须保留输入资料给出的完整 applicability、条件、关系和结果，不强制改写为箭头形式。Relation Atom 只用于生成中的识别、归属和完整性判断，不新增 TP 字段或持久化中间文件。

在生成 Base Cross 前必须完成一次 transient exhaustive Relation Extraction Pass：

- **Config Space pass**：逐个扫描每个已识别 `config_object.field` 的输入资料 semantic，检查是否显式引用其他 Config / Dynamic 对象；形成 multi-object condition 到 relation / result 时产生 Relation Atom，并从 Config TP 的单字段 `verification_goal` 中分离。
- **Dynamic Input pass**：逐个扫描每个 Dynamic parameter 及其 verification dimensions 的定义、有效条件、encoding semantic、selection rule 和 legality，检查是否显式引用其他 Config / Dynamic 对象；形成 multi-object relation 时产生 Relation Atom。
- **Explicit Relation pass**：扫描 All Input Documents 中独立描述的 mapping、selection、mutual exclusion、source/destination compatibility、resource limitation、producer/consumer relation、count/occupancy constraint、format/datatype compatibility、conditional legality、conditional result 和 mode-dependent behavior；实际涉及两个或多个 Config / Dynamic 对象时产生 Relation Atom。
- **Scenario semantic pass**：instruction / function / Scenario-specific 描述中的 multi-object relation 同样先提取 Relation Atom，再进入 Ownership；提取阶段不得按 Target Scenario 过滤。

Relation Extraction 必须逐对象、逐参数、逐明确 semantic 完成，不得用“根据文档整体理解总结几个主要 Cross”代替 exhaustive scan，也不得因 relation 相似、TP 已很多、希望减少输出、Target 仅使用部分内容或已有抽象 Cross 而提前停止。Relation Extraction 的单位是输入资料中的明确 Relation Atom；merge 仅可在 extraction 完成后执行，并继续遵守 Cross 的 lossless merge。

Ownership 只判断 relation 是否必须依赖当前 Target Scenario 才成立：脱离当前 Target Scenario 仍成立的通用模块关系进入 Base Cross；仅在当前 instruction / function / Scenario 定义下存在的关系进入 Scenario；无法唯一判断时进入 relation ownership Pending。Pending 不生成猜测性 Base Cross、Scenario expression、虚假 TP_ID，也不借用既有 TP 的 lifecycle。
根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读 Completeness Review；未指定时默认完整生成。

### 3.2 Base Inventory

`Base Inventory = F(All Input Documents)`；禁止使用 `Base Inventory = F(All Input Documents, Target Instruction)`。Base Inventory 包含完整 Register Access、Config Space、Dynamic Input、Cross inventory。Base 不受 Target Scenario 过滤，但 Base Cross 的 relation 仍必须脱离当前 instruction / function / Scenario 后成立；不受 Target 过滤不表示允许 target-specific relation 进入 Base。

用户 Prompt 中的具体指令、功能、场景、opcode 或目标对象只作为 Scenario Extraction Target，不得作为 Base Inventory 过滤条件。Phase 1 暂时忽略目标名称和场景内容，基于全部输入资料建立 inventory：覆盖所有适用 Register Access 对象、全部适用 Config `config_object.field`、全部适用 Dynamic `parameter × verification dimension`，并按 Cross Category Rules 对全部已识别 Relation Atom 完成 ownership 判断，承载所有明确且 scenario-independent 的多对象关系。不得搜索或筛选与目标“相关”的对象，也不得因当前目标未使用而跳过、删除或缩小基础 TP；否则属于 Target Leakage。

### 3.3 Base Inventory Completeness Gate

进入 Scenario Extraction 前，确认四份 Base Inventory 均已按全部输入资料处理，且未使用目标过滤、删除或缩小 inventory item。先完成 transient exhaustive Relation Extraction Pass：所有 Config object、Dynamic parameter 及 All Input Documents 中的明确 multi-object semantic 均已扫描，已识别 semantic 未仅停留在 Config / Dynamic `verification_goal`，且每个 Relation Atom 均进入 Ownership；全部满足时设置生成过程状态 `RELATION_EXTRACTION_COMPLETE = TRUE`，否则为 `FALSE`。该状态不是数量目标或 Cartesian expansion，不新增输出字段或 sheet。只有 `RELATION_EXTRACTION_COMPLETE = TRUE` 后，才可按 `Extracted Base Relation Atoms == Covered Base Relation Atoms` 判断 Base Cross completeness，确认所有明确的 scenario-independent Relation Atom 均由 Base Cross lossless 承载。每个 Relation Atom 必须归属于 Base Cross、Scenario 或 Pending / missing-input，不得已识别却没有承载。全部通过时设置 `BASE_INVENTORY_COMPLETE = TRUE`；否则设置 `BASE_INVENTORY_COMPLETE = FALSE`，必须先补齐 Base Inventory，不得交付最终 Scenario Extraction 结果。

删除 Prompt 中目标名称后，四份 Base Inventory 必须完全相同。

### 3.4 Scenario Extraction

仅在 Base Inventory 完整后执行：`Scenario Extraction = G(Complete Base Inventory, All Input Documents, Target Scenario)`。

三类输入职责固定为：`Complete Base Inventory` 提供 scenario-independent Base TP 与 Base semantics，用于 Scenario 关联、复用和 legality closure，但不包含仅在当前 instruction / function / Scenario 下成立的 relation；`All Input Documents` 提供完整设计语义，包括 scenario-independent 和 instruction / function / Scenario-specific semantics，scenario-specific relation 可直接进入 Scenario expression，但不得因此生成 Base Cross；`Target Scenario` 提供当前场景上下文，用于判断输入资料中的 semantics / relations 是否 applicable。Base Inventory 不是 Scenario semantic source 的全集，不得为给 Scenario 制造 semantic source 而把 scenario-specific relation 放回 Base Cross。

Scenario expression 的 semantic source 分为两类：Base-derived semantic 已存在于 Base TP，Scenario 直接复用；Scenario-specific semantic 只在当前 instruction / function / Scenario 下成立，直接来自 All Input Documents 中明确的 scenario-specific definition，不要求对应 Base Cross 或伪造 Base TP。Scenario 结合 `All Input Documents + Target Scenario` 判定 scenario-specific semantics，并与 Complete Base Inventory 中 applicable 的 Base semantics 一起形成 Scenario expression。

每个 Scenario 使用独立 `Scenario - <scenario_name>` sheet，从完整 inventory 提取相关 Register Access、Config Space、Dynamic Input、Cross 及其他适用基础 TP，并通过 `related_tp_id` 追溯。不得重新生成目标专属 Config、Dynamic 或 Cross TP。Scenario 不是 TP category，不得生成、修改、裁剪、补充或重排基础 TP，也不得复制完整 TP。仅当 Output Result 的既有边界适用时，可增加场景特有 Output Result TP。

基础 sheet 保持完整内容和原 TP_ID。Scenario sheet 只做场景化组织和解释，不重新定义基础覆盖空间。多个命名 Scenario 不得混合，也不新增 `scenario_name` 行字段。

**Scenario Applicable Legality Closure**：Scenario legality completeness 的输入集合固定为 `Scenario Applicable Legality Set = H(Complete Base Inventory, All Input Documents, Target Scenario)`。该集合由 Complete Base Inventory 中 applicable 的 Base legality semantics / Base Cross relations、All Input Documents 中明确的 scenario-specific legality semantics / relations，以及 Target Scenario 上下文共同确定；不得限定为只能来自 Base TP。Scenario 生成必须先确定该集合，再生成对应 Scenario 表达和 Base TP `related_tp_id` 关联；不得以已经生成的 `related_tp_id` 反向决定该集合。该集合是生成过程中的判定结果，不新增持久化模型或输出对象。

`Scenario Applicable Legality Set` 仅覆盖当前 Scenario 涉及的 Config Space / Dynamic Input applicable legality semantics / constraints，以及 applicable Base Cross relations 和输入资料明确的 scenario-specific legality relations。集合中的每个 legality semantic / relation 都必须在 Scenario 中表达；实际值、范围、联合条件及对应行为继续写入现有 `scenario_value_or_constraint`。Base-derived semantic 按其 Base TP 建立 `related_tp_id`；scenario-specific legality relation 不生成 Base Cross，也不要求 semantic-source TP_ID，`related_tp_id` 仅关联该 expression 实际依赖或约束的 Base verification object（若存在）。不得只摘录合法值而遗漏同一 Scenario 下成立的 negative / reserved / unsupported / conditional-invalid 等已定义语义。`Scenario legality completeness != Scenario dependency completeness`；当前不建立通用 Scenario dependency completeness。Register Access、Debug、Performance、Output Result 及其他 category 的场景关联继续按现有 Scenario Extraction 和对应 Category Rules 处理，不自动并入该集合，也不新增 dependency set、graph、mapping、字段或 sheet。

当值的 legality 或 behavior 依赖其他 Config / Dynamic 条件时，必须表达输入资料明确给出的完整条件与结果，不得只写该值及简短注释，也不得自行推断关系。`scenario_value_or_constraint` 中的 legality、joint condition、resource limit、producer/consumer condition、count constraint、mutual exclusion 或 conditional result 均遵守 Global Output Principles；一个可独立理解或判断的 scenario expression 单独成行，不得为减少 Scenario 行数合并 independent constraints。

该 closure 不重新展开 Base coverage space：不复制完整 Base TP，不要求 Scenario 重新列出全部 Base values 或 bins，不要求 legal / reserved / unsupported / illegal 等类别逐项输出 `N/A`，不修改或收缩 Base Config / Dynamic / Cross TP，也不新增 Base-bin 到 Scenario-bin 的 mapping 字段或中间持久化模型。Base semantic / relation 在当前 Scenario 下明确不适用时无需输出；该规则只决定 semantic、relation、legality expression 是否需要出现在 Scenario 中，不得用于绕过 Scenario Parameter Disposition Closure。属于 Parameter Disposition Closure 处理范围但当前 Scenario 不使用的 Config / Dynamic parameter，仍必须判断为 inactive，并明确最终 inactive/default constraint、使用完整 shared rule expression，或在 unresolved 时进入 Scenario missing-input；不得把 semantic / relation 不适用解释为 parameter omission 或直接推导 inactive。parameter disposition 仍必须通过 `Scenario Parameter Disposition Set = D(Complete Base Inventory, All Input Documents, Target Scenario)` 判断。applicability 无法由输入资料唯一确定时，按 Scenario missing-input report 规则处理，不得猜测。

**Scenario Parameter Disposition Output Contract**：Scenario 必须先求得 transient `Scenario Parameter Disposition Set = D(Complete Base Inventory, All Input Documents, Target Scenario)`，再决定 Scenario 输出；该集合不新增字段、sheet 或持久化中间模型。Complete Base Inventory 提供 parameter Base legal space 和 Base semantics；All Input Documents 提供 applicability、inactive/default rule 和 Scenario-specific semantics；Target Scenario 提供当前上下文。不得通过 Scenario sheet 当前是否已有该 parameter 的行反向推导 disposition，`Scenario 未提及 parameter` 本身不表示 free、inactive、fixed 或 default。

处理范围为所有可能影响当前 Scenario behavior、legal space、path activation、producer/consumer activation、resource usage、Scenario executability 或 parameter constraint 的相关 Config / Dynamic parameter。每个相关 parameter 必须唯一归入：1) **constrained/fixed**：存在 fixed value、legal sub-range、allowed set、Scenario-specific baseline 或 joint constraint，实际 constraint 写入 `scenario_value_or_constraint`；2) **free**：Scenario 没有进一步限制，显式写 `FREE /* 使用 Base legal space */`，通过 `related_tp_id` 关联 Base TP，不复制 Base bins；3) **inactive**：Scenario 不使用该 parameter，存在明确值时写 `INACTIVE /* 固定为 <default_value> */`。不得仅写无法确定最终约束的 `INACTIVE`，不得默认 reset value。

多个 inactive parameter 可以共享输入资料明确的同一条 rule expression，但该 expression 本身必须明确且唯一确定：适用的 parameter 集合、applicability condition，以及每个适用 parameter 的最终 inactive/default constraint。若共享 expression 不能同时确定这三项，必须使用现有字段逐 parameter 明确表达，或进入 Scenario missing-input report；不得使用“继承 spec 默认值”“按设计 inactive rule”“使用默认 inactive 配置”“沿用 base definition”等无法确定最终约束的表述。Scenario omission 不能充当 inactive/default rule。本 Skill 不新增 provenance、source、dependency、inheritance ID 或其他追溯模型。constrained/fixed、free、inactive 均优先复用 `related_tp_id`、对象角色、`scenario_value_or_constraint`、`why_relevant_to_scenario` 和 `scenario_application`，不新增 `disposition` 字段。复杂 constraint 遵守同一 readability-first Constraint Expression Rule。

**Scenario Parameter Disposition Closure**：相关 parameter 全部具有唯一 disposition；constrained/fixed 有明确 constraint；free 通过 `related_tp_id` 追溯对应 Base TP，且 Base legal space 明确；inactive 由逐 parameter expression 或共享 rule expression 明确且唯一确定适用 parameter 集合、applicability condition 和每个 parameter 的最终 inactive/default constraint。任一 disposition 或 inactive/default semantic unresolved 时，按 Missing-input Flow 输出。该 closure 只解决 Config / Dynamic parameter disposition，不等于也不扩展为通用 Scenario dependency completeness；不建立 generic dependency set、graph 或非参数 dependency mapping。

### 3.5 Base Legality / Scenario Legality

- **Base legality**：Config Space / Dynamic Input 中单对象自身的完整 value/input space 及通用 valid / invalid / reserved / unsupported 语义，以及 Cross 中输入资料明确存在、脱离当前 Scenario 后仍成立的多对象通用关系。
- **Scenario legality**：Base Config / Dynamic 参数本身合法，但当前 Scenario 的操作结构、对象组合、选择关系或其他上下文只允许其中部分取值或组合；该限制仅在当前目标 Scenario 成立。

Scenario Extraction 必须检查目标 instruction / function / scenario 是否引入 Scenario legality。资料明确时，将具体允许值、禁止值、范围或联合条件写入 Scenario sheet 的 `scenario_value_or_constraint`。Base-derived semantic 关联提供该 semantic 的 Base TP；scenario-specific semantic 直接来自 All Input Documents，不要求 Base semantic source，其 `related_tp_id` 仅关联实际依赖或约束的 Base verification object（若存在），不得将该对象宣称为 relation semantic source。不得为满足 traceability 生成 Base Cross、伪造 Base TP，或因对象仅出现在联合条件中就机械加入其单对象 Base TP。不得回写或收缩 Base Config / Dynamic value space，不得修改、补充或重排 Base TP，也不得仅因 Scenario 生成新的基础 TP、通用字段或 category。

### 3.6 Missing-input Flow

只要存在任意 draft 或 blocked TP，就自动输出独立 lifecycle missing-input report，不等待用户额外要求。每个受影响 TP 必须有对应记录，包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。draft 记录缺失内容和完成条件；blocked 记录阻塞原因和恢复所需输入。不得使用空泛描述。

Scenario legality 无法由输入资料唯一确定时，不得推断，自动输出对应 Scenario missing-input report，记录具体待确认问题、受约束对象和场景上下文。若相关 Config / Dynamic parameter 无法唯一判断 free / inactive / constrained/fixed，或 inactive rule expression 无法明确且唯一确定适用 parameter 集合、applicability condition、每个适用 parameter 的最终 inactive/default constraint，同样进入现有 Scenario missing-input report，记录 affected parameter / parameter set、当前 Scenario、unresolved disposition、缺失的 applicability / inactive / default / constraint semantic 和 completion condition；不得默认 free、inactive 或 reset value。

Relation Atom 已明确识别但根据当前资料无法唯一判断 Base Cross / Scenario-specific ownership 时，自动输出独立 relation ownership missing-input report。记录当前已知的最具体逻辑 `relation`、缺少的 scope / applicability 定义 `missing_content`，以及可使 ownership 唯一确定的设计信息 `completion_condition`。此时尚无 TP，不输出 `lifecycle_status`，不生成 TP_ID，也不错误关联既有 draft / blocked TP；仅在存在 ownership Pending 时生成该 report。

关系输入完整、ownership 已确定，但现有 Cross Expression Contract 无法无损表达或脚本无法确定性转换时，输出独立 Cross Skill Draft report。它记录 `involved_objects`、`observed_relation`、`model_gap`、`required_skill_decision`；不生成 TP_ID，不使用 `lifecycle_status`，也不伪装为 design missing-input。

各 report 相互独立：TP 的 draft / blocked 缺失进入 lifecycle missing-input report；Scenario legality 或 parameter disposition 缺失进入 Scenario missing-input report；Relation Atom ownership 缺失进入 relation ownership missing-input report；Cross 表达模型或脚本转换能力缺口进入 Cross Skill Draft report。来源依据、traceability、Completeness Review、inventory-level missing、generation summary 和 completion summary 仅在用户明确要求时输出。

### 3.7 Completeness Review

仅在用户明确要求 review/report 时，基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance、Output Result 七个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因。只有既不存在已判定为 scenario-independent 的 Relation Atom，也不存在 ownership unresolved 的 Relation Atom 时，Cross 才可标记为 `not_applicable`；存在 relation ownership Pending 时不得判定为 `not_applicable`，review 结果须引用或列出对应 relation ownership missing-input 作为未决项，不新增 review status，也不猜测最终 ownership。

Completeness Review 只检查和报告，不修改、补充或重新生成已有 TP；发现 `missing` 只输出待办或缺失资料。

## 4. Output Contract

### 4.1 TP Sheet Schema

核心输出是按 category 分组的结构化 TP 数据；序列化格式不得反向决定 TP 模型。JSON/YAML 仅用于中间结构或用户明确要求的格式，默认最终交付为 Excel workbook。

默认 workbook 使用 `Register Access`、`Config Space`、`Dynamic Input`、`Cross`、`Debug`、`Performance`、`Output Result` sheet，只生成实际适用且有 TP 的 sheet。category 由 sheet 表达，不在每行重复；每个 TP（包括 blocked）占一行。

最终列 schema：

- Register Access：固定为 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；默认不输出 `verification_scenario`、`expected_result`。特殊访问语义确实无法自然合并进 `verification_goal` 时才允许保留必要额外字段，不扩展为通用模板。
- Config Space：固定为 `TP_ID`、`lifecycle_status`、`config_object`、`field`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；不输出 `verification_scenario`、`expected_result`。
- Dynamic Input：固定为 `TP_ID`、`lifecycle_status`、`parameter`、`parameter_type`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；不输出 `verification_scenario`、`expected_result`、`category`、`source`、`input_basis`、`operation` 或输入资料 traceability 字段。
- Cross：固定为 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`；不输出 `verification_scenario`，仅在 Category Rules 规定的必要情形输出 `expected_result`。
- Debug、Performance、Output Result：按 Common Fields 和对应 Category Rules 输出必要字段及固定定位字段。

行为型 `coverage_strategy` 使用 testcase、assertion 或实际 method。Config / Dynamic structured coverage 使用 Value/Input-Space Coverage Contract 的换行 `cover bins` / `illegal bins` block；Cross coverage 使用 Cross Expression Contract 的 `cross bins` / `illegal cross bins` block。仅在输入资料明确时增加适用的 ignore bins 或特殊 sample event。`coverage_strategy_mapping` 可包含 testcase、assertion、coverage implementation object，或按通用规则记录具体 implementation TODO。不得新增大量扁平字段。

### 4.2 Scenario Sheet Schema

每条 Scenario 关联至少包含 `related_tp_id`、对象角色、`scenario_value_or_constraint`、`why_relevant_to_scenario`、`scenario_application`。

- `related_tp_id` 用于关联当前 Scenario expression 所依赖或约束的 Base TP，承担 Base TP traceability，不承担全部设计语义来源追溯。Base-derived semantic 指向提供该 semantic 的 Base Config / Dynamic TP；复用 scenario-independent Base Cross relation 时指向该 Base Cross TP；scenario-specific semantic 不要求 Base semantic source，仅在存在相关 Base verification object 时关联该对象。它不是设计资料 source reference、完整 semantic provenance、dependency list 或对象参与关系的机械枚举；不得为满足该字段生成 Base Cross、伪造 Base TP，或将参与对象宣称为 scenario-specific relation 的 semantic source。确需关联多个 Base TP 时，多个 TP_ID 一条一行。
- `scenario_value_or_constraint` 表达当前 Scenario 的实际取值、范围或 constraint。输入资料已明确值语义时，必须按 `<value> /* <meaning> */` 附最小注释；多个值或 independent constraint 一条一行，并遵守 Global Output Principles。Scenario legality 的允许值、禁止值、范围、联合条件或对应行为统一写入该字段。
- parameter disposition 复用上述字段表达：constrained/fixed 写实际 constraint；free 写 `FREE /* 使用 Base legal space */` 并通过 `related_tp_id` 追溯 Base TP；Scenario 不复制 Base bins；inactive 使用逐 parameter expression，或使用能明确且唯一确定适用 parameter 集合、applicability condition 和每个适用 parameter 最终 inactive/default constraint 的共享 rule expression。omission 不具有 disposition 语义。
- `why_relevant_to_scenario` 只说明该基础 TP 为什么与当前 Scenario 相关。
- `scenario_application` 只说明该对象或约束在当前 Scenario 中起什么作用，不承载具体值、范围或联合条件，不重复 value semantics，不复制基础 TP 的 `verification_goal` 或 `coverage_strategy`，也不得只写“沿用基础 TP 定义的覆盖空间”“不重新定义 bins”等无场景语义信息。

### 4.3 Missing-input Report Schema

- lifecycle missing-input report：每条记录固定包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。
- Scenario missing-input report：每条记录包含具体待确认问题、受约束对象和场景上下文。parameter disposition 未决时，同一现有 report 还需明确 affected parameter、unresolved disposition、缺失的 applicability / inactive / default / constraint semantic 和 completion condition，不新增 report 类型。
- relation ownership missing-input report：每条记录固定包含 `relation`、`missing_content`、`completion_condition`，不包含 `lifecycle_status` 或 TP_ID。
- Cross Skill Draft report：每条记录固定包含 `involved_objects`、`observed_relation`、`model_gap`、`required_skill_decision`，不包含 `lifecycle_status` 或 TP_ID。
- inventory-level missing 或 Completeness Review report：按需独立输出，不混入 TP sheet，也不复制完整 TP inventory。

### 4.4 Excel Display Rules

最终输出同时保证 semantic completeness、generation correctness 和人工 review readability。结构治理、去重、字段调整、格式简化或文字压缩不得造成信息能力、生成行为、Excel display 或人工 review 能力回退；减少文字、行数或 TP 数量不是独立优化目标，任何压缩都必须满足 lossless semantics 和 clear reviewability。展示规则只负责 presentation，不得反向改变 TP / Scenario 数据模型。

- lifecycle 高亮：complete 正常显示且不特殊高亮；draft 使用统一黄色或琥珀色提醒型高亮；blocked 使用统一红色强警示高亮。至少覆盖 `lifecycle_status` 单元格；同一 workbook 的范围和样式保持一致。

- Scenario `related_tp_id` 只引用一个 Base TP 时，沿用该 Base TP 的 lifecycle 高亮；引用多个 Base TP 时保留全部 TP_ID 并一条一行，单元格按 `blocked > draft > complete` 使用最高严重级别高亮。该展示不改变 Base TP lifecycle。

- Scenario sheet 的 merge 按列独立判断：连续多行的 `related_tp_id` 内容完全相同时必须纵向 merge 该列；包含多个 TP_ID 时，仅 TP_ID 集合和顺序均完全相同才视为内容相同。连续多行的 object role 内容完全相同时必须独立纵向 merge。即使各行的 `scenario_value_or_constraint`、relation branch 或 scenario semantic 不同，也不阻止相同 `related_tp_id` 或 object role 列的 merge；不同的 `scenario_value_or_constraint` 本身不得 merge，必须保持每个 independent expression 独立。`why_relevant_to_scenario` 或 `scenario_application` 内容完全相同且连续时继续允许 merge。merge 只影响展示，不删除 TP_ID，不改变每行 Scenario expression 的逻辑关联、lifecycle 或 `related_tp_id` 的 Base TP traceability 语义。

- 文本包含中文分号 `；` 或英文分号 `;` 时，在分号处分行显示；只改变展示，不改变字段内容和语义。
- Enum / format / mode 的每个独立 `<value / range> -> <semantic>` mapping 必须在 Excel 单元格中独占一行；不得因原始文本未使用分号而挤在同一显示行。
- structured coverage 的 `cover bins：`、每个 bin、`illegal bins：`、`ignore bins：` 和 `无` 必须按 Coverage Strategy 输出格式保留实际换行，不得折叠为同一行。
- Cross 的 `if` / result、direct relation、`cross bins` 和 `illegal cross bins` 必须按 Cross Expression Contract 保留换行与 tab 缩进；不得在 Excel 中展开 `binsof(...) intersect` implementation code。

- lifecycle missing-input report 在 Excel 中使用独立 sheet。

- workbook 写完后必须验证最终 Scenario sheet 的实际 merged-cell ranges。每个长度大于 1 的连续相同 `related_tp_id` group 必须存在覆盖该 group 的 merged range；连续完全相同的 object role group 同样必须有覆盖该 group 的实际 merged range。不得只检查源数据内容相同。`scenario_value_or_constraint` 仍保持独立且不 merge；任一必需 merged range 缺失时 Output Contract Gate 失败。

### 4.5 Output Order

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. 以功能场景或指令作为入口时，每个命名 Scenario 的独立 `Scenario - <scenario_name>` sheet。
6. 任一 Scenario legality 或 parameter disposition 无法唯一确定时，输出对应 Scenario missing-input report；parameter disposition unresolved 包括 disposition 无法唯一判断或 inactive/default rule 不完整；不存在时不生成。
7. 存在 Relation Atom ownership Pending 时，relation ownership missing-input report；不存在时不生成。
8. 存在 Cross expression / script conversion model gap 时，Cross Skill Draft report；不存在时不生成。
9. Debug sheet。
10. Performance sheet。
11. Output Result sheet。
12. 存在任意 draft / blocked TP 时，lifecycle missing-input report；不存在时不生成。
13. 用户明确要求时，inventory-level missing 或 Completeness Review report。

### 4.6 Final Gates

交付前检查：

- **Schema**：每个 TP、Scenario 和 report 均符合本 Output Contract，字段职责唯一，未新增字段体系或重复 category 字段。
- **Base Inventory Completeness**：Register Access、Config Space、Dynamic Input、Cross 已按全部输入资料处理；适用对象均有 complete、draft 或 blocked TP。Cross generation 前必须达到 `RELATION_EXTRACTION_COMPLETE = TRUE`。所有明确 Relation Atom 均完成唯一处理：scenario-independent Atom 由 Base Cross 承载，scenario-specific Atom 进入 Scenario，ownership unresolved Atom 进入 relation ownership missing-input，表达或脚本转换不支持的 Atom 进入 Cross Skill Draft。存在 Cross Skill Draft 时可交付其他 TP 和该报告，但不得宣称 Cross / Base Inventory complete，也不得基于受影响关系执行最终 Scenario completeness closure。
- **Target Isolation**：删除 Prompt 目标名称后四份 Base Inventory 保持相同；Scenario 未改写、裁剪、补充或重排基础 TP。
- **Lifecycle Closure**：状态符合 Lifecycle；每个 complete TP 满足对应 Category Rules 的必需信息；仅缺非 category-required implementation binding 的 complete TP 在 `coverage_strategy_mapping` 使用规定的具体 implementation TODO，且 `coverage_strategy` 仍只承载 method，脚本不为该项生成 SV；每个 draft / blocked TP 均有完整 lifecycle missing-input 记录。
- **Scenario Isolation / Legality**：每个命名 Scenario 独立，Scenario 字段各守职责，Scenario legality 未进入或收缩 Base Config、Dynamic、Cross。分别检查：1) **Scenario semantic completeness**：由 Complete Base Inventory、All Input Documents 和 Target Scenario 确定的 applicable Base-derived 与 scenario-specific legality semantics / relations 是否全部表达，只有 `Scenario Covered Legality Set == Scenario Applicable Legality Set` 时通过；2) **Base TP traceability**：与 Base verification object 相关的 Scenario expression 是否通过 `related_tp_id` 正确关联对应 Base TP。不得把参与对象宣称为 semantic source，也不得要求 scenario-specific semantic 必须有 Base semantic-source TP。任何 applicable semantic / relation 未表达或 Base TP 关联错误时，`Scenario legality completeness = FAIL`，不得交付最终 workbook。该 closure 不代表 Scenario dependency completeness；Scenario legality unresolved 时不得猜测，并按现有规则生成独立 Scenario missing-input report；3) **Scenario Parameter Disposition Integrity**：所有影响 behavior、legality、executability 或 parameter constraint 的相关 Config / Dynamic parameter 均已唯一归入 constrained/fixed、free 或 inactive；semantic / relation 当前不适用不得让该范围内的 parameter 跳过三分类判断，omission 未被当作 disposition，也不得由 semantic / relation 不适用直接推导 inactive；constrained/fixed 有明确 constraint；free 通过 `related_tp_id` 追溯对应 Base TP，且 Base legal space 明确；inactive 由逐 parameter expression 或共享 rule expression 明确且唯一确定适用 parameter 集合、applicability condition 和每个适用 parameter 的最终 inactive/default constraint。parameter disposition 或 inactive/default rule unresolved 时必须进入 Scenario missing-input。
- **Coverage Integrity**：coverage strategy 与 intent、category、监测和测量边界一致；Register Access 满足其 Category Rules；单对象 structured coverage 满足 [Value/Input-Space Coverage Contract](references/value-input-space-coverage.md)；Cross 固定为 cross coverage，其 expression 与 bins 满足 [Cross Expression Contract](references/cross-expression.md)，未混入 testcase、assertion 或 REF / scoreboard 结果判定。Cross merge lossless，每个 branch 明确可见并映射到实际 coverage；implementation inputs 与 implementation object mapping 未混用。
- **No Inference**：未补充输入资料未定义的设计语义、行为、路径、采样、阈值、输出类别或 debug capability。
- **Output Contract**：sheet、schema、输出顺序、高亮、换行和 merge 符合本章；语言与 constraint expression 符合 Global Output Principles；Scenario parameter disposition 无歧义；workbook 写完后实际 merged-cell ranges 已通过验证。任一条件失败均不得交付。
