---
name: module-st-testpoint-extraction
description: 面向 ST 层面从模块 spec、寄存器列表、动态输入描述、接口信号表、debug/performance spec 中提取、整理或审查模块验证 Testpoint。Use when the user asks to generate complete or category-specific module verification TP, manage draft/complete/blocked lifecycle, map coverage strategy, review TP completeness without regenerating TP, or report missing inputs. Covers register access, configuration space, dynamic input parameters, cross relationships, debug, atomic performance, and optional output-result coverage. Default output language is Chinese.
---

# Module ST Testpoint Extraction

默认使用中文输出。

## 1. Core Model

### 1.1 Scope

从 ST 层面的模块验证视角提取可评审、可落地的 Testpoint（TP），包括 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance 和可选 Output Result，并提供覆盖策略映射与输入资料不足报告。

模块 TP 以模块自身的寄存器访问、配置、动态输入、对象关系、Debug、原子性能和输出结果为范围。通路级非法地址、地址对齐和访问宽度属于 path-level 验证，不进入当前模块 TP。真实软件 workload 由独立输入件或流程维护；压力测试不作为当前 ST module 验证主线。用户明确提供压力测试输入件时可作为后续增强处理，但不得自动 cross 配置、输入、Debug、输出或性能空间。

TP 是验证目标，不是完整 block-level DV testplan 或 testcase implementation plan。不得展开 driver sequence、stimulus 调度或具体构造细节、iteration 内部步骤、handshake 顺序、wait/drain/recovery、寄存器写入时序、scoreboard/checker 实现或 testcase 内部循环。

### 1.2 TP Metamodel

TP category 固定为：

- Register Access：一个明确的寄存器访问行为，即 access action + expected behavior。
- Config Space：一个配置对象的完整单对象 value space，即 values / ranges / categories + input-defined semantics。
- Dynamic Input：一个动态输入对象的完整单对象 input space，即 values / ranges / categories + input-defined semantics。
- Cross：两个或多个 Config / Dynamic 对象之间的明确联合关系，即 joint condition -> relation / result。
- Debug：一个输入资料明确支持的异步输入级 debug capability。
- Performance：一个原子工作场景的性能目标。
- Output Result：一个 DUT 输出对象本身的独立结果覆盖空间。

各 category 独立判断和生成；一个 category 缺资料不得阻断其他 category。category 独立只表示生成逻辑互不阻塞，不表示一个 TP 一个文件或多个同类 TP 数据文件。

TP 粒度首先服从当前 category 的对象模型；只有该 category 明确允许对象合并时，才依据 verification goal、coverage space、DUT behavior 和验证构造方式判断是否合并。全局合并规则不得覆盖 category 的对象粒度。

### 1.3 Common Fields

所有 TP 均以 `TP_ID`、`lifecycle_status`、必要的 category-specific identifier fields、`verification_goal` 和 `coverage_strategy` 为基础。`verification_scenario` 和 `expected_result` 仅在当前 category 或当前 TP 需要时输出。对 schema 已定义包含 `coverage_strategy_mapping` 的 category，该字段作为固定列保留，内容未知时允许为空。

- category-specific identifier fields 只定位覆盖对象，不承载验证目标、场景、预期结果或覆盖逻辑。固定定位字段为 Dynamic Input 的 `parameter`、`parameter_type`，Config Space 的 `config_object`、`field`，以及 Debug 的 debug capability。Performance、Register Access、Cross 与 Output Result 默认不增加定位字段。
- `verification_goal` 描述验证什么。Register Access 写当前访问动作和预期结果；Config Space 写当前 field 的完整 value space 及输入资料明确语义；Dynamic Input 写当前 parameter 的完整 input space 及输入资料明确语义；Cross 写完整联合条件及对应关系或结果，并优先使用 `<joint_condition> -> <relation_or_result>`。
- `verification_scenario` 只描述覆盖空间或状态及观测点，不展开 testcase 实现。
- `expected_result` 描述 DUT 可观测行为，不得引用未展开的规格描述。

不得为任何 category 新增平行的目标、场景、预期、覆盖或实现映射字段体系。不得为不存在 `expected_result` 字段的 category 增加该字段。

### 1.4 Lifecycle

每个候选项只有一种状态：

- **complete**：当前 TP 的必需验证信息已经完整，`verification_goal` 和 `coverage_strategy` 均可成立。仅尚未生成 testcase、assertion 或 coverage implementation object，或 `coverage_strategy_mapping` 当前为空，不影响 complete。
- **draft**：TP 的验证对象和验证方向已经成立，但完成 `verification_goal`，或使已选择的 `coverage_strategy` 可执行或判定所需的部分设计语义或 coverage implementation inputs 尚未具备。设计语义可包括 relation、result、behavior；implementation inputs 可包括 HDL path、signal、monitor mapping、sample event。不得自行推断缺失设计语义。
- **blocked**：当前 TP 的验证对象、验证目标方向、覆盖策略方向或必要行为判定本身无法成立，无法形成有效 TP。对应 category sheet 仍输出 `lifecycle_status=blocked` 的 TP 行，保留已知字段，未知验证字段允许为空，不得猜测填充。

`lifecycle_status` 是 TP 固有字段，不得因最小充分输出而删除。不受影响的 category 继续生成。

### 1.5 Coverage Model

`coverage_strategy` 描述采用什么验证方式和覆盖内容，是从 `verification_goal` 到实际 coverage 实现的最小映射，不以 testcase、covergroup、assertion 为默认模板。

- 空间型 TP（Config Space、Dynamic Input 和需要 value-space coverage 的 Cross）使用 coverage space、coverage target、bins、illegal_bins、ignore_bins；verification method 仅在需要时输出。
- 行为型 TP（Register Access 及其他以行为判定为主且无独立 value-space bins 的 TP）可只输出实际 verification method，不得制造无意义 bins。
- complete TP 必须有明确、可追踪的覆盖策略。不得只写空标签；每个条目必须有当前 TP 的明确对象、值、范围或集合。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
- testcase 只标识测试构造方式，不展开 testcase 实现步骤。
- 每个 bin 必须给出当前 TP 的明确值、范围或集合，不得重复 `verification_goal` 已表达的设计语义、预期行为或值空间解释。

动态参数的 coverage strategy 至少定义 coverage space（range / data / address / format / mode）、coverage target（参数名或字段定义 bit range）和 bins。`coverage_target` 必须写出具体硬件对象、字段或编码空间，不得使用“对应覆盖空间”等泛称。`illegal_bins` 仅在输入资料明确规定某采样值不应出现时输出；`ignore_bins` 仅在明确不纳入覆盖统计时输出。不得为了字段完整性制造空的或无意义的 `illegal_bins` / `ignore_bins`。

valid、invalid、reserved、unsupported 是语义分类，不自动对应 bin 类型。主动作为验证目标覆盖的 negative / invalid / reserved / unsupported 输入使用普通 `bins`；例如输入非法配置并验证 DUT 返回 CFG_ERROR、RF_IDX_ERROR 或其他输入资料明确的错误行为，属于主动 negative verification target。不得因输入在设计语义上 illegal 就自动使用 `illegal_bins`；只有输入资料明确规定该采样值本身不应在覆盖采样中出现时才使用 `illegal_bins`。error expectation 与 bin type 职责独立。`ignore_bins` 仅用于明确不纳入覆盖统计的值；`illegal_bins` 不表示 DUT 必须报错，`ignore_bins` 不表示 DUT 行为异常。

coverage implementation inputs 包括 HDL path、signal、monitor mapping、sample event 等，用于后续生成或完善具体实现。若已选择的 coverage strategy 必须依赖这些输入才能执行或判定，缺失时按 Lifecycle 标记为 draft。

`coverage_strategy_mapping` 保存已经存在或后续生成的 testcase name、assertion code 或 coverage implementation object/code，是实现结果映射。不得将 coverage implementation inputs 写入该字段，也不得将实现结果写入 `coverage_strategy`。仅 implementation object 尚未生成或 mapping 为空不影响 complete。

### 1.6 TP_ID

统一使用大写、下划线和三位序号。不得保留 `ST`、`REF`、`VAL` 或与本规则并行的旧命名。

- category code 固定为 Register Access=`REG`、Config Space=`CFG`、Dynamic Input=`DYN`、Cross=`CROSS`、Debug=`DBG`、Performance=`PERF`、Output Result=`OUT`。
- Dynamic Input：`<module>_DYN_<parameter>_<coverage_space>_<index>`，例如 `VU_DYN_VD_RANGE_003`、`VU_DYN_RS1_RANGE_001`、`VU_DYN_RS1_DATA_002`。
- Register Access：`<module>_REG_<register>_<access_type>_<index>`。
- Config Space：`<module>_CFG_<config_object>_<index>`。
- Cross：`<module>_CROSS_<object>_<index>`，其中 `object` 表示已明确的跨对象关系焦点。
- 其他模块能力类：`<module>_<category>_<object>_<index>`，例如 `MU_CFG_CTRL_001`、`MU_DBG_STOP_001`、`MU_PERF_ADD_DUT_LAT_001`、`MU_OUT_STATUS_FLAG_001`。

`module` 标识归属模块；TP_ID 保证唯一性、表达 category 并快速表达验证焦点，不绑定输入资料的组织层次。`index` 保证唯一性。Dynamic Input 不使用 `source` 作为身份或来源追溯，`parameter` 固定表示一个动态输入对象。当前功能名或指令名不得作为额外身份信息写入基础 TP_ID；输入资料本身明确包含的功能名、指令名或编码语义可自然保留在描述字段中。

### 1.7 Global Boundaries

- 不推断输入资料未定义的 HDL path、非法处理、输出类别、性能阈值、采样条件、监测方式或其他设计行为。
- TP 描述必须具体，不得使用“输入件定义的代表值”“按输入件定义”等空泛描述。
- 凡是能由上层分组、Excel sheet、TP_ID 或固定规则唯一确定，且删除后不影响 TP 理解、实现或评审的信息，不在更低层重复输出。不得机械输出 scope、输入对象 metadata、assembly、fixed opcode、execution unit list 或规则解释。
- Config Space 和 Dynamic Input 的输入资料明确语义即使包含 DUT 行为描述，也保留在对应单对象空间中；多对象联合关系进入 Cross，输出对象自身的独立覆盖空间进入 Output Result，寄存器访问动作及其直接 side effect 进入 Register Access。
- 一个字段描述可以同时产生 Register Access TP 和 Config Space TP，但必须拆分验证目标；Config Space 验证软件配置状态及配置约束，不验证寄存器存储行为。

## 2. Category Rules

### 2.1 Register Access

Register Access TP 按寄存器组织，验证寄存器读写动作及其输入资料明确规定的直接 side effect。command、trigger、start、kick 等由寄存器访问直接触发的明确行为属于 Register Access；与寄存器读写动作无直接关系的独立功能效果不进入该 category。不得根据字段名称推断 side effect。

每个适用 Register Access 对象都必须生成 TP，或按已知信息标记为 draft / blocked；不得因当前功能场景未引用该寄存器或字段而跳过。存在寄存器表但未生成 Register Access sheet，视为生成不完整。完全没有生成对应 TP 时，仅在用户要求 Completeness Review 时将该 inventory item 标记为 `missing`。

每个 TP 只表达一个独立访问目标。`verification_goal` 直接写明对象、访问条件或操作和预期观察结果，不得使用未展开的泛化描述。RESET、字段读写属性和 side effect 是不同目标，不得混合。

对象粒度固定为：

- RESET 默认是 register-level，一个寄存器一条，覆盖该寄存器中具有明确 reset/default value 的字段。
- R、RW 等字段访问属性默认是 field-level，每条只验证对应字段。
- side effect 跟随输入资料中的真实归属对象。整个寄存器的一次访问触发行为时，只生成一条 register-level TP；仅在输入资料明确某 field 的访问独立触发行为时生成 field-level TP。
- 同一对象同时存在普通访问属性与明确 side effect 时，分别生成 TP，不得把 side effect 条件混入普通 R / RW TP。

固定访问模型为 RESET：`reset -> default value`，`coverage_strategy = assertion`；R：`write attempt -> no write effect`；RW：`write -> readback == write data`；SIDE_EFFECT：`access -> defined effect`。具体 TP 直接实例化实际值、行为和条件；不得输出独立 `expected_value` 字段，reset/default/fixed value 写入 `verification_goal`。信息不足时按 Lifecycle 处理。

按寄存器表顺序处理：每个寄存器先 RESET，再处理字段 R、RW 或其他已定义属性；当前寄存器全部完成后处理下一个寄存器。同一寄存器 TP 在 sheet 中连续排列。`register` 必须是寄存器名称，`access_type` 表示访问属性；`index` 按最终行顺序在整个 sheet 中连续递增，不按 access type 分别编号，也不因新寄存器重置。

### 2.2 Config Space

Configuration 不随当前请求携带，在请求前设置，并在单个请求执行期间保持稳定；分类不依据 spec 中 static、dynamic 或 dynamic configuration 等命名。某 Configuration 若允许在单个请求期间变化，标记为待确认，并要求明确更新时机、生效时机及其对当前请求的影响。配置空间不引入请求或激励。

Config Space 固定为单对象 value-space TP。每个 TP 只对应一个 `config_object`、一个 `field` 和一个完整单对象 value space。每个识别出的 `config_object.field` 都必须生成 TP，或标记为 draft / blocked；不得因当前功能场景未引用而跳过。完全没有生成对应 TP 时，仅在用户要求 Completeness Review 时标记为 `missing`。

`verification_goal` 直接展开完整定义空间，写出具体 enum、编码、范围或分类，以及输入资料明确定义的 valid / invalid / reserved / unsupported 分类和对应预期语义。只阅读该字段而不查询原 spec 时，reviewer 必须能知道待遍历值及其已定义语义。不得使用无法看出待覆盖值的泛化描述，不得推断未定义 DUT 行为。

`index` 按 Config Space sheet 最终行顺序连续递增，不按对象或类型分别编号。

### 2.3 Dynamic Input

Dynamic Input 是当前请求执行所需并随请求携带的信息。运行时变化、名称包含 dynamic 或存放在寄存器中，均不能单独作为分类依据；随请求携带关系不明确时标记为待确认。接口信号表用于 HDL path、采样条件、输入有效事件和输出观测点映射，不自动扩展为动态参数清单。

Dynamic Input 固定为单对象 input-space TP。`parameter` 固定表示一个动态输入对象；多个参数即使 `parameter_type`、coverage space 或语义相同，也分别生成，不得合并。`parameter_type` 描述参数本身，如 reg、imm、mem、mask、enum、index；coverage space 在 `coverage_strategy` 中描述扫描空间，如 range、data、address、format、mode。不得使用 reference/content、`REF` 或 `VAL`，coverage space 不得替代 `verification_goal` 中的实际值范围。

`verification_goal` 直接展开当前 parameter 的完整输入空间，写出具体值、编码、范围、边界或类别，以及输入资料明确的 valid、invalid、reserved、unsupported 分类和对应语义。`coverage_strategy` 只映射该输入空间的 coverage target 与 bins。不得推断未定义 DUT 行为。

若字段定义 bit range 大于实际生效 bit range，扫描字段定义的完整 bit range；实际有效位、保留位和非法处理方式仅在输入资料明确时写入 `verification_goal`。未扫描参数使用输入资料定义的合法 baseline，baseline 不是输出字段。

动态输入资料可包含参数定义、参数类型、参数范围、参数到 HDL signal/path 的映射、有效接收或采样事件、clock/reset、非法/reserved 行为及其他明确约束。固定 opcode 或其他静态识别字段只作为当前动态对象的静态条件，不自动作为扫描项；opcode decode 本质上属于输入资料明确的多对象联合关系时，按 Cross 处理。

基础 Dynamic TP 不因当前功能场景中的 Configuration 而裁剪，也不得仅因 Scenario 注入额外条件。多个 Dynamic 参数间的明确联合关系生成 Cross TP。

### 2.4 Cross

Cross 固定为多对象关系 TP，只用于输入资料明确存在、且脱离当前 instruction / function / Scenario 后仍成立的 Configuration / Dynamic Input 联合取值约束、映射或结果，可覆盖 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input。只有 scenario-independent 的联合取值产生单个 Config 或 Dynamic TP 无法表达的新关系时才生成，不默认展开 Cartesian cross。仅在特定 instruction / function / Scenario 上下文成立的关系进入 Scenario legality / Scenario expression，不生成 Base Cross。单字段 reserved/illegal encoding 不作为 Cross invalid combination。

`verification_goal` 直接使用输入资料中的对象、字段和值表达完整联合条件及对应关系或结果，优先采用 `<joint_condition> -> <relation_or_result>`；能形式化时不补写等价长自然语言。输入资料明确定义无效组合时，同样表达无效联合条件及对应关系或结果。

输入资料已明确 invalid/unsupported 组合，且验证对象和方向成立，但 relation、result 或 behavior 缺失导致目标不完整时生成 draft，并在 lifecycle missing-input report 记录待补内容。验证对象、目标方向、策略方向或必要行为判定本身无法成立时标记 blocked。

Cross 的 coverage strategy 不设固定默认值，按关系选择 testcase、covergroup 或 assertion。Cross 属于 Base Inventory，独立于 Scenario 生成；不得因为 Prompt 指定 instruction / function / Scenario 而将 target-specific relation 写入 Base Cross，也不根据 Scenario 临时生成、裁剪或修改。`expected_result` 默认不输出，仅在结果无法自然并入 `verification_goal` 的逻辑关系表达式时允许输出。

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

输入资料按实际内容读取 module spec、寄存器基础描述和 side effect、字段约束、配置到 HDL 的映射、动态输入描述、Debug、Performance、可选输出覆盖、clock/reset、采样条件、可观测映射等逻辑信息块；这些是可能出现的信息类型，不构成固定文件要求或统一 mandatory input checklist。

某项信息是否为当前 TP 必需，由 Core Model、Lifecycle、Coverage Model 和对应 Category Rules 判断。coverage implementation inputs 仅在已选择的 coverage strategy 必须依赖它们执行或判定时，其缺失才导致 draft；不得因本节列举的逻辑信息块改变 complete / draft / blocked 判定。

根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读 Completeness Review；未指定时默认完整生成。

### 3.2 Base Inventory

`Base Inventory = F(All Input Documents)`；禁止使用 `Base Inventory = F(All Input Documents, Target Instruction)`。Base Inventory 包含完整 Register Access、Config Space、Dynamic Input、Cross inventory。Base 不受 Target Scenario 过滤，但 Base Cross 的 relation 仍必须脱离当前 instruction / function / Scenario 后成立；不受 Target 过滤不表示允许 target-specific relation 进入 Base。

用户 Prompt 中的具体指令、功能、场景、opcode 或目标对象只作为 Scenario Extraction Target，不得作为 Base Inventory 过滤条件。Phase 1 暂时忽略目标名称和场景内容，基于全部输入资料建立 inventory：覆盖所有适用 Register Access 对象、全部 `config_object.field`、全部 Dynamic 参数空间和全部明确且 scenario-independent 的多对象关系。不得搜索或筛选与目标“相关”的对象，也不得因当前目标未使用而跳过、删除或缩小基础 TP；否则属于 Target Leakage。

### 3.3 Base Inventory Completeness Gate

进入 Scenario Extraction 前，确认四份 Base Inventory 均已按全部输入资料处理，且未使用目标过滤、删除或缩小 inventory item。全部通过时设置 `BASE_INVENTORY_COMPLETE = TRUE`；否则设置 `BASE_INVENTORY_COMPLETE = FALSE`，必须先补齐 Base Inventory，不得交付最终 Scenario Extraction 结果。

删除 Prompt 中目标名称后，四份 Base Inventory 必须完全相同。

### 3.4 Scenario Extraction

仅在 Base Inventory 完整后执行：`Scenario Extraction = G(Complete Base Inventory, All Input Documents, Target Scenario)`。

三类输入职责固定为：`Complete Base Inventory` 提供 scenario-independent Base TP 与 Base semantics，用于 Scenario 关联、复用和 legality closure，但不包含仅在当前 instruction / function / Scenario 下成立的 relation；`All Input Documents` 提供完整设计语义，包括 scenario-independent 和 instruction / function / Scenario-specific semantics，scenario-specific relation 可直接进入 Scenario expression，但不得因此生成 Base Cross；`Target Scenario` 提供当前场景上下文，用于判断输入资料中的 semantics / relations 是否 applicable。Base Inventory 不是 Scenario semantic source 的全集，不得为给 Scenario 制造 semantic source 而把 scenario-specific relation 放回 Base Cross。

Scenario expression 的 semantic source 分为两类：Base-derived semantic 已存在于 Base TP，Scenario 直接复用；Scenario-specific semantic 只在当前 instruction / function / Scenario 下成立，直接来自 All Input Documents 中明确的 scenario-specific definition，不要求对应 Base Cross 或伪造 Base TP。Scenario 结合 `All Input Documents + Target Scenario` 判定 scenario-specific semantics，并与 Complete Base Inventory 中 applicable 的 Base semantics 一起形成 Scenario expression。

每个 Scenario 使用独立 `Scenario - <scenario_name>` sheet，从完整 inventory 提取相关 Register Access、Config Space、Dynamic Input、Cross 及其他适用基础 TP，并通过 `related_tp_id` 追溯。不得重新生成目标专属 Config、Dynamic 或 Cross TP。Scenario 不是 TP category，不得生成、修改、裁剪、补充或重排基础 TP，也不得复制完整 TP。仅当 Output Result 的既有边界适用时，可增加场景特有 Output Result TP。

基础 sheet 保持完整内容和原 TP_ID。Scenario sheet 只做场景化组织和解释，不重新定义基础覆盖空间。多个命名 Scenario 不得混合，也不新增 `scenario_name` 行字段。

**Scenario Applicable Legality Closure**：Scenario legality completeness 的输入集合固定为 `Scenario Applicable Legality Set = H(Complete Base Inventory, All Input Documents, Target Scenario)`。该集合由 Complete Base Inventory 中 applicable 的 Base legality semantics / Base Cross relations、All Input Documents 中明确的 scenario-specific legality semantics / relations，以及 Target Scenario 上下文共同确定；不得限定为只能来自 Base TP。Scenario 生成必须先确定该集合，再生成对应 Scenario 表达和 Base TP `related_tp_id` 关联；不得以已经生成的 `related_tp_id` 反向决定该集合。该集合是生成过程中的判定结果，不新增持久化模型或输出对象。

`Scenario Applicable Legality Set` 仅覆盖当前 Scenario 涉及的 Config Space / Dynamic Input applicable legality semantics / constraints，以及 applicable Base Cross relations 和输入资料明确的 scenario-specific legality relations。集合中的每个 legality semantic / relation 都必须在 Scenario 中表达；实际值、范围、联合条件及对应行为继续写入现有 `scenario_value_or_constraint`。Base-derived semantic 按其 Base TP 建立 `related_tp_id`；scenario-specific legality relation 不生成 Base Cross，也不要求 semantic-source TP_ID，`related_tp_id` 仅关联该 expression 实际依赖或约束的 Base verification object（若存在）。不得只摘录合法值而遗漏同一 Scenario 下成立的 negative / reserved / unsupported / conditional-invalid 等已定义语义。`Scenario legality completeness != Scenario dependency completeness`；当前不建立通用 Scenario dependency completeness。Register Access、Debug、Performance、Output Result 及其他 category 的场景关联继续按现有 Scenario Extraction 和对应 Category Rules 处理，不自动并入该集合，也不新增 dependency set、graph、mapping、字段或 sheet。

当值的 legality 或 behavior 依赖其他 Config / Dynamic 条件时，必须表达输入资料明确给出的完整条件与结果，例如 `SRC1_SEL=0x06 && DATA_TYPE=FP32 -> legal`、`SRC1_SEL=0x06 && DATA_TYPE=BF16 -> CFG_ERROR`；不得只写该值及简短注释，也不得自行推断关系。

该 closure 不重新展开 Base coverage space：不复制完整 Base TP，不要求 Scenario 重新列出全部 Base values 或 bins，不要求 legal / reserved / unsupported / illegal 等类别逐项输出 `N/A`，不修改或收缩 Base Config / Dynamic / Cross TP，也不新增 Base-bin 到 Scenario-bin 的 mapping 字段或中间持久化模型。Base semantic / relation 在当前 Scenario 下明确不适用时无需输出；applicability 无法由输入资料唯一确定时，按 Scenario missing-input report 规则处理，不得猜测。

### 3.5 Base Legality / Scenario Legality

- **Base legality**：Config Space / Dynamic Input 中单对象自身的完整 value/input space 及通用 valid / invalid / reserved / unsupported 语义，以及 Cross 中输入资料明确存在、脱离当前 Scenario 后仍成立的多对象通用关系。
- **Scenario legality**：Base Config / Dynamic 参数本身合法，但当前 Scenario 的操作结构、对象组合、选择关系或其他上下文只允许其中部分取值或组合；该限制仅在当前目标 Scenario 成立。

Scenario Extraction 必须检查目标 instruction / function / scenario 是否引入 Scenario legality。资料明确时，将具体允许值、禁止值、范围或联合条件写入 Scenario sheet 的 `scenario_value_or_constraint`。Base-derived semantic 关联提供该 semantic 的 Base TP；scenario-specific semantic 直接来自 All Input Documents，不要求 Base semantic source，其 `related_tp_id` 仅关联实际依赖或约束的 Base verification object（若存在），不得将该对象宣称为 relation semantic source。不得为满足 traceability 生成 Base Cross、伪造 Base TP，或因对象仅出现在联合条件中就机械加入其单对象 Base TP。不得回写或收缩 Base Config / Dynamic value space，不得修改、补充或重排 Base TP，也不得仅因 Scenario 生成新的基础 TP、通用字段或 category。

### 3.6 Missing-input Flow

只要存在任意 draft 或 blocked TP，就自动输出独立 lifecycle missing-input report，不等待用户额外要求。每个受影响 TP 必须有对应记录，包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。draft 记录缺失内容和完成条件；blocked 记录阻塞原因和恢复所需输入。不得使用空泛描述。

Scenario legality 无法由输入资料唯一确定时，不得推断，自动输出对应 Scenario missing-input report，记录具体待确认问题、受约束对象和场景上下文。

两类 report 相互独立：任意 TP 的 draft / blocked 缺失进入 lifecycle missing-input report；Scenario-specific legality 缺失进入对应 Scenario missing-input report。来源依据、traceability、Completeness Review、inventory-level missing、generation summary 和 completion summary 仅在用户明确要求时输出。

### 3.7 Completeness Review

仅在用户明确要求 review/report 时，基于既有 TP inventory 审查 Register Access、Config Space、Dynamic Input、Cross、Debug、Performance、Output Result 七个适用 category。每项标记 `covered`、`draft`、`blocked`、`not_applicable` 或 `missing`，并列出现有 TP ID、缺口或阻塞原因；没有明确多对象关系时 Cross 为 `not_applicable`。

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

结构化输出中的 `coverage_strategy` 可包含 testcase、covergroup、assertion、coverage_space、coverage_target、bins、illegal_bins、ignore_bins；`coverage_strategy_mapping` 可包含 testcase、assertion、coverage。Excel 单元格可使用简洁、可读的结构化表达，不得新增大量扁平字段。

### 4.2 Scenario Sheet Schema

每条 Scenario 关联至少包含 `related_tp_id`、对象角色、`scenario_value_or_constraint`、`why_relevant_to_scenario`、`scenario_application`。

- `related_tp_id` 用于关联当前 Scenario expression 所依赖或约束的 Base TP，承担 Base TP traceability，不承担全部设计语义来源追溯。Base-derived semantic 指向提供该 semantic 的 Base Config / Dynamic TP；复用 scenario-independent Base Cross relation 时指向该 Base Cross TP；scenario-specific semantic 不要求 Base semantic source，仅在存在相关 Base verification object 时关联该对象。它不是设计资料 source reference、完整 semantic provenance、dependency list 或对象参与关系的机械枚举；不得为满足该字段生成 Base Cross、伪造 Base TP，或将参与对象宣称为 scenario-specific relation 的 semantic source。确需关联多个 Base TP 时，多个 TP_ID 一条一行。
- `scenario_value_or_constraint` 表达当前 Scenario 的实际取值、范围或 constraint。输入资料已明确值语义时，必须按 `<value> /* <meaning> */` 附最小注释；多个值或 constraint 一条一行。Scenario legality 的允许值、禁止值、范围或联合条件统一写入该字段。
- `why_relevant_to_scenario` 只说明该基础 TP 为什么与当前 Scenario 相关。
- `scenario_application` 只说明该对象或约束在当前 Scenario 中起什么作用，不承载具体值、范围或联合条件，不重复 value semantics，不复制基础 TP 的 `verification_goal` 或 `coverage_strategy`，也不得只写“沿用基础 TP 定义的覆盖空间”“不重新定义 bins”等无场景语义信息。

### 4.3 Missing-input Report Schema

- lifecycle missing-input report：每条记录固定包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。
- Scenario missing-input report：每条记录包含具体待确认问题、受约束对象和场景上下文。
- inventory-level missing 或 Completeness Review report：按需独立输出，不混入 TP sheet，也不复制完整 TP inventory。

### 4.4 Excel Display Rules

- lifecycle 高亮：complete 正常显示且不特殊高亮；draft 使用统一黄色或琥珀色提醒型高亮；blocked 使用统一红色强警示高亮。至少覆盖 `lifecycle_status` 单元格；同一 workbook 的范围和样式保持一致。
- Scenario `related_tp_id` 只引用一个 Base TP 时，沿用该 Base TP 的 lifecycle 高亮；引用多个 Base TP 时保留全部 TP_ID 并一条一行，单元格按 `blocked > draft > complete` 使用最高严重级别高亮。该展示不改变 Base TP lifecycle。
- 文本包含中文分号 `；` 或英文分号 `;` 时，在分号处分行显示；只改变展示，不改变字段内容和语义。
- 连续多行中 `why_relevant_to_scenario` 或 `scenario_application` 内容完全一致时允许纵向 merge；语义仅相近时不得合并。merge 只影响展示，不改变每行与 `related_tp_id` 的逻辑对应，也不得遮蔽或丢失 lifecycle 状态。
- lifecycle missing-input report 在 Excel 中使用独立 sheet。

### 4.5 Output Order

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. 以功能场景或指令作为入口时，每个命名 Scenario 的独立 `Scenario - <scenario_name>` sheet。
6. 任一 Scenario-specific legality 无法唯一确定时，对应 Scenario missing-input report；不存在时不生成。
7. Debug sheet。
8. Performance sheet。
9. Output Result sheet。
10. 存在任意 draft / blocked TP 时，lifecycle missing-input report；不存在时不生成。
11. 用户明确要求时，inventory-level missing 或 Completeness Review report。

### 4.6 Final Gates

交付前检查：

- **Schema**：每个 TP、Scenario 和 report 均符合本 Output Contract，字段职责唯一，未新增字段体系或重复 category 字段。
- **Base Inventory Completeness**：Register Access、Config Space、Dynamic Input、Cross 已按全部输入资料处理；适用对象均有 complete、draft 或 blocked TP，否则不得通过 Gate。
- **Target Isolation**：删除 Prompt 目标名称后四份 Base Inventory 保持相同；Scenario 未改写、裁剪、补充或重排基础 TP。
- **Lifecycle Closure**：状态符合 Lifecycle；每个 draft / blocked TP 均有完整 lifecycle missing-input 记录，仅 mapping 为空未触发 draft。
- **Scenario Isolation / Legality**：每个命名 Scenario 独立，Scenario 字段各守职责，Scenario legality 未进入或收缩 Base Config、Dynamic、Cross。分别检查：1) **Scenario semantic completeness**：由 Complete Base Inventory、All Input Documents 和 Target Scenario 确定的 applicable Base-derived 与 scenario-specific legality semantics / relations 是否全部表达，只有 `Scenario Covered Legality Set == Scenario Applicable Legality Set` 时通过；2) **Base TP traceability**：与 Base verification object 相关的 Scenario expression 是否通过 `related_tp_id` 正确关联对应 Base TP。不得把参与对象宣称为 semantic source，也不得要求 scenario-specific semantic 必须有 Base semantic-source TP。任何 applicable semantic / relation 未表达或 Base TP 关联错误时，`Scenario legality completeness = FAIL`，不得交付最终 workbook。该 closure 不代表 Scenario dependency completeness；applicability 无法唯一确定时不得猜测，并按现有规则生成独立 Scenario missing-input report。
- **Coverage Integrity**：coverage strategy 与验证 intent、category、监测和测量边界一致；主动 negative verification target 使用普通 bins，仅在采样值本身被明确规定不应出现时使用 illegal_bins；error expectation 不决定 bin type。implementation inputs 与 implementation object mapping 未混用。
- **No Inference**：未补充输入资料未定义的设计语义、行为、路径、采样、阈值、输出类别或 debug capability。
- **Output Contract**：sheet 组成、schema、输出顺序、高亮、换行和 merge 均符合本章；若任一 Gate 不通过，不得交付。
