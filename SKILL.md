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

TP 是验证目标，不是完整 block-level DV testplan、testcase implementation plan 或 Case Development。不得定义 randc / random algorithm、iteration、scan order / scheduling、testcase 数量、最小扫描次数、一个 testcase 命中几个 bin、sequence generation，也不得展开 driver sequence、stimulus 调度或具体构造细节、handshake 顺序、wait/drain/recovery、寄存器写入时序、scoreboard/checker 实现或 testcase 内部循环。

### 1.2 TP Metamodel

TP category 固定为：

- Register Access：一个明确的寄存器访问行为，即 access action + expected behavior。
- Config Space：一个配置对象的完整单对象 value space，即 values / ranges / categories + input-defined semantics。
- Dynamic Input：一个动态输入对象的完整单对象 input space，即 values / ranges / categories + input-defined semantics。
- Cross：两个或多个 Config / Dynamic 对象之间，经 Relation Ownership 判断为 scenario-independent 的明确联合关系，即 `[applicability_condition &&] joint_condition -> relation_or_result`；applicability condition 仅在输入资料实际定义时存在，不得人为制造。
- Debug：一个输入资料明确支持的异步输入级 debug capability。
- Performance：一个原子工作场景的性能目标。
- Output Result：一个 DUT 输出对象本身的独立结果覆盖空间。

各 category 独立判断和生成；一个 category 缺资料不得阻断其他 category。category 独立只表示生成逻辑互不阻塞，不表示一个 TP 一个文件或多个同类 TP 数据文件。

TP 粒度首先服从当前 category 的对象模型；只有该 category 明确允许对象合并时，才依据 verification goal、coverage space、DUT behavior 和验证构造方式判断是否合并。全局合并规则不得覆盖 category 的对象粒度。

### 1.3 Common Fields

所有 TP 均以 `TP_ID`、`lifecycle_status`、必要的 category-specific identifier fields、`verification_goal` 和 `coverage_strategy` 为基础。`verification_scenario` 和 `expected_result` 仅在当前 category 或当前 TP 需要时输出。对 schema 已定义包含 `coverage_strategy_mapping` 的 category，该字段作为固定列保留，内容未知时允许为空。

- category-specific identifier fields 只定位覆盖对象，不承载验证目标、场景、预期结果或覆盖逻辑。固定定位字段为 Dynamic Input 的 `parameter`、`parameter_type`，Config Space 的 `config_object`、`field`，以及 Debug 的 debug capability。Performance、Register Access、Cross 与 Output Result 默认不增加定位字段。
- `verification_goal` 描述验证什么。Register Access 写当前访问动作和预期结果；Config Space 写当前 field 的完整 value space 及输入资料明确语义；Dynamic Input 写当前 parameter 的完整 input space 及输入资料明确语义；Cross 写完整 applicability condition（若输入资料实际定义）、joint condition 和 relation / result，多个 independent branch 一条一行，并按 Constraint Expression Rule 选择直接逻辑表达或结构化中文，不得人为制造 applicability condition。
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

- coverage strategy 明确需要参数值扫描的 Config Space / Dynamic Input TP 使用 coverage space、coverage target 和 structured bins；`illegal_bins`、`ignore_bins` 仅按本节既有语义在适用时输出。需要 value-space coverage 的 Cross 继续按实际关系使用相应结构；verification method 仅在需要时输出。
- 行为型 TP（Register Access 及其他以行为判定为主且无独立 value-space bins 的 TP）可只输出实际 verification method，不得制造无意义 bins。
- complete TP 必须有明确、可追踪的覆盖策略。不得只写空标签；每个条目必须有当前 TP 的明确对象、值、范围或集合。
- assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
- covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
- testcase 只标识测试构造方式，不展开 testcase 实现步骤。
- 每个 bin 必须给出当前 TP 的明确值、范围或集合，不得重复 `verification_goal` 已表达的设计语义、预期行为或值空间解释。

**Parameter Scan Output Contract**：仅当 Config Space / Dynamic Input TP 的实际 `coverage_strategy` 明确需要参数值扫描时适用。`verification_goal` 定义完整 value/input space、legal constraint 和设计语义，回答哪些值属于可生成空间；`coverage_strategy.coverage_space` 定义扫描维度，`coverage_target` 定义被覆盖对象，`bins` 定义下游可直接消费的结构化 coverage partitions。`legal space -> structured bins` 是本 Skill 的 consumer-facing output 边界，不定义下游的 randc、random algorithm、iteration、scan order / scheduling、testcase 数量、最小扫描次数或 bin 命中算法。非 parameter-scan TP 继续按实际 verification method 处理，不得为字段完整性制造无意义 bins。

parameter-scan complete TP 的 structured bins 必须完整承载声明覆盖的 coverage space：输入资料明确的特殊值，以及输入资料或明确 coverage intent 指定需独立追踪的典型值，使用明确命名的 explicit bin；明确 numeric range 的端点可直接识别为边界，不属于补充 DUT semantic，但仅在 coverage intent 要求独立覆盖时拆为 explicit bin。合法空间中未被 explicit bins 承接但仍属于 coverage target 的有效值，使用 residual / range bin 准确承接。`ZERO: {0}`、`MIN_NONZERO: {1}`、`TYPICAL: {127}`、`MAX: {255}`、`RESIDUAL: {[2:126], [128:254]}` 仅说明表达格式，不是必须采用的 bin 名称或生成模板。不得自行创造典型值、特殊值、代表值或语义分类，不得仅写 legal range，或用“覆盖边界值、典型值和随机值”等自然语言代替当前 coverage intent 要求的 partition。legal coverage bins 的值必须属于可达到 legal space，不包含明确 unreachable value，不为扫描扩大 legal space；residual/range bin 准确覆盖剩余有效空间，且不与 explicit bins 错误重叠；bins 不得与 `verification_goal` 的合法性语义冲突。下游必须无需重新解释 spec 或从自然语言猜测 coverage partition。

parameter-scan TP 的验证对象和扫描方向已明确，但缺少形成 structured bins 所需的 value range、classification、boundary 或 design semantic 时，按 Lifecycle 标记为 draft，并进入 lifecycle missing-input report；验证对象、coverage direction 或必要 legal semantic 无法成立时标记 blocked。parameter-scan TP 缺少 structured bins 时不得标记 complete。仅 testcase、assertion、coverage implementation object 尚未生成或 `coverage_strategy_mapping` 为空，仍不影响 complete。

动态参数的 parameter-scan coverage strategy 至少定义 coverage space（range / data / address / format / mode）、coverage target（参数名或字段定义 bit range）和 structured bins。`coverage_target` 必须写出具体硬件对象、字段或编码空间，不得使用“对应覆盖空间”等泛称。`illegal_bins` 仅在输入资料明确规定某采样值不应出现时输出；`ignore_bins` 仅在明确不纳入覆盖统计时输出。不得为了字段完整性制造空的或无意义的 `illegal_bins` / `ignore_bins`。

valid、invalid、reserved、unsupported 是语义分类，不自动对应 bin 类型。主动作为验证目标覆盖的 negative / invalid / reserved / unsupported 输入使用普通 `bins`，并作为明确 negative coverage target 与 legal coverage residual bins 分离；例如输入非法配置并验证 DUT 返回 CFG_ERROR、RF_IDX_ERROR 或其他输入资料明确的错误行为，属于主动 negative verification target。不得因输入在设计语义上 illegal 就自动使用 `illegal_bins`；只有输入资料明确规定该采样值本身不应在覆盖采样中出现时才使用 `illegal_bins`。error expectation 与 bin type 职责独立。`ignore_bins` 仅用于明确不纳入覆盖统计的值；`illegal_bins` 不表示 DUT 必须报错，`ignore_bins` 不表示 DUT 行为异常。

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
- Config Space 和 Dynamic Input 的输入资料明确语义即使包含 DUT 行为描述，也保留在对应单对象空间中；多对象联合关系先进入 Relation Ownership 判断，scenario-independent relation 进入 Base Cross，scenario-specific relation 进入 Scenario；输出对象自身的独立覆盖空间进入 Output Result，寄存器访问动作及其直接 side effect 进入 Register Access。
- **Language Rule**：最终 TP、Scenario 和 Missing-input report 中，register、field、parameter、signal、HDL path、enum、opcode、encoding、instruction、function、error code、macro、constant、逻辑操作符及公式或代码表达式中的 identifier 保持输入资料原文，不翻译。`verification_goal` 的自然语言 behavior / semantic、`scenario_value_or_constraint` 的自然语言结果说明、`why_relevant_to_scenario`、`scenario_application`、`coverage_strategy` 的解释性文字、missing-input 的问题描述与完成条件，以及 comment / meaning / explanation 默认必须使用中文。逻辑表达中的 identifier 可保持原文；relation / result 属于自然语言行为时必须使用中文，除非右侧本身是必须原样保留的正式 identifier、enum 或 error code。不得因采用逻辑表达而将整条 relation 自动改写为英文。
- **Constraint Expression Rule**：Cross `verification_goal` 与 Scenario `scenario_value_or_constraint` 使用同一套 readability-first policy，首要目标是 `semantic exactness + human reviewability`。选择 reviewer 无需二次解码即可理解、且不损失设计语义的最简表达；逻辑表达和结构化中文都是工具，更形式化、更数学化、更短、更少文字、更少行或更少 TP 均不是独立优化目标。简单 equality、inequality、boolean、membership 或 conditional result 在一行即可直接理解时优先逻辑表达；complex count / cardinality、resource limit、port usage、occupancy、multiple-source selection、多对象 mutual exclusion、长集合或多层 boolean 若公式会增加 review 成本，则优先结构化中文。每个 independent constraint / branch 独立表达且一条一行，但不要求每条都是公式；positive 与 negative/error condition 分别表达，完整保留 applicability condition 及其对应 result。最终表达不得使用 `...` 省略对象、条件或 branch，不得使用需 reviewer 手工展开的复杂 `count()`、多层 nested boolean、过长 `&&` / `||` 链或复杂集合语法隐藏具体对象，也不得为单一公式或减少行数合并 independent constraints。Language Rule 同样适用：identifier 保持原文，constraint explanation 与 behavior 默认中文。
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

当前 Config TP 的 coverage strategy 明确需要 parameter scan 时，complete TP 必须按 Parameter Scan Output Contract 提供完整 structured bins；不需要 parameter scan 时不强制 bins，也不得制造无意义 bins。

`index` 按 Config Space sheet 最终行顺序连续递增，不按对象或类型分别编号。

### 2.3 Dynamic Input

Dynamic Input 是当前请求执行所需并随请求携带的信息。运行时变化、名称包含 dynamic 或存放在寄存器中，均不能单独作为分类依据；随请求携带关系不明确时标记为待确认。接口信号表用于 HDL path、采样条件、输入有效事件和输出观测点映射，不自动扩展为动态参数清单。

Dynamic Input 固定为单对象 input-space TP。`parameter` 固定表示一个动态输入对象；多个参数即使 `parameter_type`、coverage space 或语义相同，也分别生成，不得合并。`parameter_type` 描述参数本身，如 reg、imm、mem、mask、enum、index；coverage space 在 `coverage_strategy` 中描述扫描空间，如 range、data、address、format、mode。不得使用 reference/content、`REF` 或 `VAL`，coverage space 不得替代 `verification_goal` 中的实际值范围。

`verification_goal` 直接展开当前 parameter 的完整输入空间，写出具体值、编码、范围、边界或类别，以及输入资料明确的 valid、invalid、reserved、unsupported 分类和对应语义。`coverage_strategy` 映射当前 Dynamic Input 的实际覆盖方式；仅在 coverage strategy 明确需要 parameter scan 时，按 Parameter Scan Output Contract 输出 `coverage_space`、`coverage_target` 和 structured bins；其他 verification method 按 Coverage Model 处理。不得推断未定义 DUT 行为。

当前 Dynamic TP 的 coverage strategy 明确需要 parameter scan 时，complete TP 必须按 Parameter Scan Output Contract 提供完整 structured bins；不需要 parameter scan 时不强制 bins，也不得制造无意义 bins。

若字段定义 bit range 大于实际生效 bit range，扫描字段定义的完整 bit range；实际有效位、保留位和非法处理方式仅在输入资料明确时写入 `verification_goal`。未扫描参数使用输入资料定义的合法 baseline，baseline 不是输出字段。

动态输入资料可包含参数定义、参数类型、参数范围、参数到 HDL signal/path 的映射、有效接收或采样事件、clock/reset、非法/reserved 行为及其他明确约束。固定 opcode 或其他静态识别字段只作为当前动态对象的静态条件，不自动作为扫描项；opcode decode 本质上属于输入资料明确的多对象联合关系时，进入 Relation Extraction 和 Ownership 判断。

基础 Dynamic TP 不因当前功能场景中的 Configuration 而裁剪，也不得仅因 Scenario 注入额外条件。多个 Dynamic 参数间的明确联合关系进入 Relation Extraction 和 Ownership 判断，不得在 Dynamic Input 阶段直接决定生成 Base Cross。

### 2.4 Cross

Cross 固定为多对象关系 TP，只承载经 Input Processing 的 Relation Ownership 判断为 scenario-independent 的 Configuration / Dynamic Input 联合取值 constraint、mapping、legality 或 result，可覆盖 Configuration × Configuration、Dynamic Input × Dynamic Input、Configuration × Dynamic Input。Config / Dynamic 仍只负责单对象完整 value/input space；relation 即使已出现在单对象 `verification_goal` 中，也不表示已完成关系覆盖。

Base Cross 只承载 scenario-independent Relation Atom：relation 脱离当前用户指定的 Target Scenario 后仍是输入资料定义的通用模块行为。`target-independent != condition-free`；relation 包含 opcode、mode、instruction class、producer type 或其他 applicability condition，并不自动成为 Scenario-specific relation。只有 relation 本身仅依赖当前 instruction / function / Scenario 定义才成立时，才进入 Scenario legality / Scenario expression。不得一看到条件就移入 Scenario，不得因 Prompt 指定目标而把 target-specific relation 放入 Base Cross，也不默认展开 Cartesian cross；ownership 无法唯一确定时不得猜测，按 Missing-input Flow 处理。单字段 reserved/illegal encoding 不作为 Cross invalid combination。

`verification_goal` 直接表达每个 Relation Atom 在存在时的 applicability condition，以及完整 joint condition 与 relation / result，并遵守 Global Boundaries 的 Constraint Expression Rule。多个 independent branch 一条一行；输入资料明确定义无效组合时，同样分别表达完整条件及对应关系或结果。

允许多个 Relation Atom 合并为一个 Cross TP，但必须 lossless：relation 结构一致，且 applicability condition、joint condition、relation / result 均未丢失，coverage strategy 能证明各 relation branch 已覆盖。不得使用 compatible/incompatible producer、valid/invalid combination、legal/illegal source 等宽泛概括隐藏输入资料明确的 consumer、source field、producer encoding、opcode、datatype、mode、type matching 或其他联合条件；若 reviewer 无法仅根据 Cross TP 判断原始各 branch 是否覆盖，则不得合并。

Cross 的 coverage strategy 不设固定默认值，按关系选择 testcase、covergroup 或 assertion，并必须对应 `verification_goal` 中的具体 relation branch。包含多个 branch 时，策略必须能证明每个 branch 均被覆盖，不得只写 compatible、mismatch 等抽象类别。输入资料已明确 invalid/unsupported 组合，且验证对象和方向成立，但 relation、result 或 behavior 缺失导致目标不完整时生成 draft，并在 lifecycle missing-input report 记录待补内容；验证对象、目标方向、策略方向或必要行为判定本身无法成立时标记 blocked。Cross 属于 Base Inventory，独立于 Scenario 生成，不根据 Scenario 临时生成、裁剪或修改。`expected_result` 默认不输出，仅在结果无法自然并入 `verification_goal` 的逻辑关系表达式时允许输出。

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

所有 Config / Dynamic multi-object relation 使用唯一处理流：`All Input Documents -> Relation Extraction -> Relation Atom -> Ownership(Base Cross / Scenario-specific / Pending) -> Cross TP / Scenario Expression / relation ownership missing-input -> Lossless Coverage Closure`。Relation Extraction 不使用 Target Scenario 过滤，从 module spec、Config、Dynamic Input、功能描述、instruction/function 描述等实际内容中识别两个或多个 Config / Dynamic 对象共同决定的 constraint、mapping、legality、DUT result 或其他明确关系；Config / Dynamic `verification_goal` 中出现的 multi-object semantic 也必须进入该流程，不得由任何 category 旁路决定归属。

内部最小判断单位为 `Relation Atom = [applicability_condition &&] joint_condition -> relation_or_result`：`applicability_condition` 是可选的设计规则适用条件；relation 不需要额外适用条件时可以为空，此时 Relation Atom 仅由 `joint_condition` 决定，例如 `A=0 || B=0 -> RESULT`，不得人为制造 applicability condition。`joint_condition` 表示两个或多个 Config / Dynamic 对象的联合条件，`relation_or_result` 表示输入资料明确的 constraint、mapping、legality、DUT result 或其他关系。Relation Atom 只用于生成中的识别、归属和完整性判断，不新增 category、sheet、输出字段或持久化中间文件。

在生成 Base Cross 前必须完成一次 transient exhaustive Relation Extraction Pass：

- **Config Space pass**：逐个扫描每个已识别 `config_object.field` 的输入资料 semantic，检查是否显式引用其他 Config / Dynamic 对象；形成 multi-object condition 到 relation / result 时产生 Relation Atom，即使该 semantic 已写入 Config TP 也不得跳过。
- **Dynamic Input pass**：逐个扫描每个 Dynamic parameter 的定义、有效条件、encoding semantic、selection rule 和 legality，检查是否显式引用其他 Config / Dynamic 对象；形成 multi-object relation 时产生 Relation Atom。
- **Explicit Relation pass**：扫描 All Input Documents 中独立描述的 mapping、selection、mutual exclusion、source/destination compatibility、resource limitation、producer/consumer relation、count/occupancy constraint、format/datatype compatibility、conditional legality、conditional result 和 mode-dependent behavior；实际涉及两个或多个 Config / Dynamic 对象时产生 Relation Atom。
- **Scenario semantic pass**：instruction / function / Scenario-specific 描述中的 multi-object relation 同样先提取 Relation Atom，再进入 Ownership；提取阶段不得按 Target Scenario 过滤。

Relation Extraction 必须逐对象、逐参数、逐明确 semantic 完成，不得用“根据文档整体理解总结几个主要 Cross”代替 exhaustive scan，也不得因 relation 相似、TP 已很多、希望减少输出、Target 仅使用部分内容或已有抽象 Cross 而提前停止。Relation Extraction 的单位是输入资料中的明确 Relation Atom；merge 仅可在 extraction 完成后执行，并继续遵守 Cross 的 lossless merge。

Ownership 只判断 relation 是否必须依赖当前 Target Scenario 才成立：脱离当前 Target Scenario 仍成立的通用模块关系进入 Base Cross；仅在当前 instruction / function / Scenario 定义下存在的关系进入 Scenario；无法唯一判断时进入 relation ownership Pending。Pending 不生成猜测性 Base Cross、Scenario expression、虚假 TP_ID，也不借用既有 TP 的 lifecycle。
根据请求选择完整生成、指定 category 生成、生命周期整理、覆盖策略映射、缺失输入报告或只读 Completeness Review；未指定时默认完整生成。

### 3.2 Base Inventory

`Base Inventory = F(All Input Documents)`；禁止使用 `Base Inventory = F(All Input Documents, Target Instruction)`。Base Inventory 包含完整 Register Access、Config Space、Dynamic Input、Cross inventory。Base 不受 Target Scenario 过滤，但 Base Cross 的 relation 仍必须脱离当前 instruction / function / Scenario 后成立；不受 Target 过滤不表示允许 target-specific relation 进入 Base。

用户 Prompt 中的具体指令、功能、场景、opcode 或目标对象只作为 Scenario Extraction Target，不得作为 Base Inventory 过滤条件。Phase 1 暂时忽略目标名称和场景内容，基于全部输入资料建立 inventory：覆盖所有适用 Register Access 对象、全部 `config_object.field`、全部 Dynamic 参数空间，并按 Cross Category Rules 对全部已识别 Relation Atom 完成 ownership 判断，承载所有明确且 scenario-independent 的多对象关系。不得搜索或筛选与目标“相关”的对象，也不得因当前目标未使用而跳过、删除或缩小基础 TP；否则属于 Target Leakage。

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

当值的 legality 或 behavior 依赖其他 Config / Dynamic 条件时，必须表达输入资料明确给出的完整条件与结果，不得只写该值及简短注释，也不得自行推断关系。`scenario_value_or_constraint` 中的 legality、joint condition、resource limit、producer/consumer condition、count constraint、mutual exclusion 或 conditional result 均遵守 Global Boundaries 的 Constraint Expression Rule；一个可独立理解或判断的 scenario expression 单独成行，不得为减少 Scenario 行数合并 independent constraints。

该 closure 不重新展开 Base coverage space：不复制完整 Base TP，不要求 Scenario 重新列出全部 Base values 或 bins，不要求 legal / reserved / unsupported / illegal 等类别逐项输出 `N/A`，不修改或收缩 Base Config / Dynamic / Cross TP，也不新增 Base-bin 到 Scenario-bin 的 mapping 字段或中间持久化模型。Base semantic / relation 在当前 Scenario 下明确不适用时无需输出；该规则只决定 semantic、relation、legality expression 是否需要出现在 Scenario 中，不得用于绕过 Scenario Parameter Disposition Closure。属于 Parameter Disposition Closure 处理范围但当前 Scenario 不使用的 Config / Dynamic parameter，仍必须判断为 inactive，并明确最终 inactive/default constraint、使用完整 shared rule expression，或在 unresolved 时进入 Scenario missing-input；不得把 semantic / relation 不适用解释为 parameter omission 或直接推导 inactive。parameter disposition 仍必须通过 `Scenario Parameter Disposition Set = D(Complete Base Inventory, All Input Documents, Target Scenario)` 判断。applicability 无法由输入资料唯一确定时，按 Scenario missing-input report 规则处理，不得猜测。

**Scenario Parameter Disposition Output Contract**：对可进入 testcase generation 的 Scenario，必须先求得 transient `Scenario Parameter Disposition Set = D(Complete Base Inventory, All Input Documents, Target Scenario)`，再决定 Scenario 输出；该集合不新增字段、sheet 或持久化中间模型。Complete Base Inventory 提供 parameter Base legal space、Base semantics，以及对应 Base TP 采用 parameter-scan strategy 时已有的 structured bins；All Input Documents 提供 applicability、inactive/default rule 和 Scenario-specific semantics；Target Scenario 提供当前上下文。不得通过 Scenario sheet 当前是否已有该 parameter 的行反向推导 disposition，`Scenario 未提及 parameter` 本身不表示 free、inactive、fixed 或 default。

处理范围为所有可能影响当前 Scenario behavior、legal space、path activation、producer/consumer activation、resource usage、testcase executability 或 parameter generation constraint 的相关 Config / Dynamic parameter。每个相关 parameter 必须唯一归入：1) **constrained/fixed**：存在 fixed value、legal sub-range、allowed set、Scenario-specific baseline 或 joint constraint，实际 constraint 写入 `scenario_value_or_constraint`；2) **free**：Scenario 没有进一步限制，显式写 `FREE /* 使用 Base legal space */`，通过 `related_tp_id` 关联 Base TP，不复制 Base bins；free 不等于 parameter-scan，仅当对应 Base TP 的 coverage strategy 已明确为 parameter-scan 时，才同时使用该 Base TP 的 structured bins，且不得仅因 disposition 为 free 将 Base TP 改为 parameter-scan；3) **inactive**：Scenario 不使用该 parameter，存在明确值时写 `INACTIVE /* 固定为 <default_value> */`。不得仅写无法确定最终约束的 `INACTIVE`，不得默认 reset value。

多个 inactive parameter 可以共享输入资料明确的同一条 rule expression，但该 expression 本身必须明确且唯一确定：适用的 parameter 集合、applicability condition，以及每个适用 parameter 的最终 inactive/default constraint。若共享 expression 不能同时确定这三项，必须使用现有字段逐 parameter 明确表达，或进入 Scenario missing-input report；不得使用“继承 spec 默认值”“按设计 inactive rule”“使用默认 inactive 配置”“沿用 base definition”等无法确定最终约束的表述。Scenario omission 不能充当 inactive/default rule。本 Skill 不新增 provenance、source、dependency、inheritance ID 或其他追溯模型。constrained/fixed、free、inactive 均优先复用 `related_tp_id`、对象角色、`scenario_value_or_constraint`、`why_relevant_to_scenario` 和 `scenario_application`，不新增 `disposition` 字段。复杂 constraint 遵守同一 readability-first Constraint Expression Rule。

**Scenario Parameter Disposition Closure**：相关 parameter 全部具有唯一 disposition，constrained/fixed 有明确 constraint，free 可追溯 Base legal space；仅当对应 Base TP 已明确采用 parameter-scan strategy 时，free 才还必须可使用该 Base TP 的 structured bins；inactive 必须由逐 parameter expression 或共享 rule expression 明确且唯一确定适用 parameter 集合、applicability condition 和每个 parameter 的最终 inactive/default constraint，才能设置生成过程状态 `testcase-generation-ready = TRUE`。任一 disposition、inactive/default semantic 或适用的 parameter-scan structured bins unresolved 时设为 `FALSE`，并按 Missing-input Flow 输出；该状态不新增 workbook 字段，只表示当前 Skill 输出足以被下游无歧义消费，不表示 testcase、generation algorithm、iteration、scan scheduling 或 testcase 数量已经确定。该 closure 只解决 Config / Dynamic parameter disposition，不等于也不扩展为通用 Scenario dependency completeness；不建立 generic dependency set、graph 或非参数 dependency mapping。

### 3.5 Base Legality / Scenario Legality

- **Base legality**：Config Space / Dynamic Input 中单对象自身的完整 value/input space 及通用 valid / invalid / reserved / unsupported 语义，以及 Cross 中输入资料明确存在、脱离当前 Scenario 后仍成立的多对象通用关系。
- **Scenario legality**：Base Config / Dynamic 参数本身合法，但当前 Scenario 的操作结构、对象组合、选择关系或其他上下文只允许其中部分取值或组合；该限制仅在当前目标 Scenario 成立。

Scenario Extraction 必须检查目标 instruction / function / scenario 是否引入 Scenario legality。资料明确时，将具体允许值、禁止值、范围或联合条件写入 Scenario sheet 的 `scenario_value_or_constraint`。Base-derived semantic 关联提供该 semantic 的 Base TP；scenario-specific semantic 直接来自 All Input Documents，不要求 Base semantic source，其 `related_tp_id` 仅关联实际依赖或约束的 Base verification object（若存在），不得将该对象宣称为 relation semantic source。不得为满足 traceability 生成 Base Cross、伪造 Base TP，或因对象仅出现在联合条件中就机械加入其单对象 Base TP。不得回写或收缩 Base Config / Dynamic value space，不得修改、补充或重排 Base TP，也不得仅因 Scenario 生成新的基础 TP、通用字段或 category。

### 3.6 Missing-input Flow

只要存在任意 draft 或 blocked TP，就自动输出独立 lifecycle missing-input report，不等待用户额外要求。每个受影响 TP 必须有对应记录，包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。draft 记录缺失内容和完成条件；blocked 记录阻塞原因和恢复所需输入。不得使用空泛描述。

Scenario legality 无法由输入资料唯一确定时，不得推断，自动输出对应 Scenario missing-input report，记录具体待确认问题、受约束对象和场景上下文。对可进入 testcase generation 的 Scenario，若相关 Config / Dynamic parameter 无法唯一判断 free / inactive / constrained/fixed，或 inactive rule expression 无法明确且唯一确定适用 parameter 集合、applicability condition、每个适用 parameter 的最终 inactive/default constraint，同样进入现有 Scenario missing-input report，记录 affected parameter / parameter set、当前 Scenario、unresolved disposition、缺失的 applicability / inactive / default / constraint semantic 和 completion condition；不得默认 free、inactive 或 reset value，也不得交由下游自行判断。

Relation Atom 已明确识别但根据当前资料无法唯一判断 Base Cross / Scenario-specific ownership 时，自动输出独立 relation ownership missing-input report。记录当前已知的最具体逻辑 `relation`、缺少的 scope / applicability 定义 `missing_content`，以及可使 ownership 唯一确定的设计信息 `completion_condition`。此时尚无 TP，不输出 `lifecycle_status`，不生成 TP_ID，也不错误关联既有 draft / blocked TP；仅在存在 ownership Pending 时生成该 report。

三类 report 相互独立：任意 TP 的 draft / blocked 缺失进入 lifecycle missing-input report；Scenario legality 或 parameter disposition 缺失进入对应 Scenario missing-input report，其中 parameter disposition missing 包括 free / inactive / constrained/fixed 无法唯一判断、inactive/default applicability 不明确或最终 inactive/default constraint 不明确；Relation Atom ownership 缺失进入 relation ownership missing-input report。来源依据、traceability、Completeness Review、inventory-level missing、generation summary 和 completion summary 仅在用户明确要求时输出。

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

结构化输出中的 `coverage_strategy` 可包含 testcase、covergroup、assertion、coverage_space、coverage_target、bins、illegal_bins、ignore_bins；`coverage_strategy_mapping` 可包含 testcase、assertion、coverage。Excel 单元格可使用简洁、可读的结构化表达，不得新增大量扁平字段。

### 4.2 Scenario Sheet Schema

每条 Scenario 关联至少包含 `related_tp_id`、对象角色、`scenario_value_or_constraint`、`why_relevant_to_scenario`、`scenario_application`。

- `related_tp_id` 用于关联当前 Scenario expression 所依赖或约束的 Base TP，承担 Base TP traceability，不承担全部设计语义来源追溯。Base-derived semantic 指向提供该 semantic 的 Base Config / Dynamic TP；复用 scenario-independent Base Cross relation 时指向该 Base Cross TP；scenario-specific semantic 不要求 Base semantic source，仅在存在相关 Base verification object 时关联该对象。它不是设计资料 source reference、完整 semantic provenance、dependency list 或对象参与关系的机械枚举；不得为满足该字段生成 Base Cross、伪造 Base TP，或将参与对象宣称为 scenario-specific relation 的 semantic source。确需关联多个 Base TP 时，多个 TP_ID 一条一行。
- `scenario_value_or_constraint` 表达当前 Scenario 的实际取值、范围或 constraint。输入资料已明确值语义时，必须按 `<value> /* <meaning> */` 附最小注释；多个值或 independent constraint 一条一行，并遵守 Global Boundaries 的 Constraint Expression Rule。Scenario legality 的允许值、禁止值、范围、联合条件或对应行为统一写入该字段。
- 对 testcase-generation Scenario，parameter disposition 复用上述字段表达：constrained/fixed 写实际 constraint；free 写 `FREE /* 使用 Base legal space */` 并追溯 Base TP；仅当对应 Base TP 已采用 parameter-scan strategy 时使用其 structured bins，Scenario 不复制 Base bins，也不得因 free 将 Base TP 改为 parameter-scan；inactive 使用逐 parameter expression，或使用能明确且唯一确定适用 parameter 集合、applicability condition 和每个适用 parameter 最终 inactive/default constraint 的共享 rule expression。omission 不具有 disposition 语义。
- `why_relevant_to_scenario` 只说明该基础 TP 为什么与当前 Scenario 相关。
- `scenario_application` 只说明该对象或约束在当前 Scenario 中起什么作用，不承载具体值、范围或联合条件，不重复 value semantics，不复制基础 TP 的 `verification_goal` 或 `coverage_strategy`，也不得只写“沿用基础 TP 定义的覆盖空间”“不重新定义 bins”等无场景语义信息。

### 4.3 Missing-input Report Schema

- lifecycle missing-input report：每条记录固定包含 `affected_tp_id`、`lifecycle_status`、`missing_content`、`completion_or_unblock_condition`。
- Scenario missing-input report：每条记录包含具体待确认问题、受约束对象和场景上下文。parameter disposition 未决时，同一现有 report 还需明确 affected parameter、unresolved disposition、缺失的 applicability / inactive / default / constraint semantic 和 completion condition，不新增 report 类型。
- relation ownership missing-input report：每条记录固定包含 `relation`、`missing_content`、`completion_condition`，不包含 `lifecycle_status` 或 TP_ID。
- inventory-level missing 或 Completeness Review report：按需独立输出，不混入 TP sheet，也不复制完整 TP inventory。

### 4.4 Excel Display Rules

最终输出同时保证 semantic completeness、generation correctness 和人工 review readability。结构治理、去重、字段调整、格式简化或文字压缩不得造成信息能力、生成行为、Excel display 或人工 review 能力回退；减少文字、行数或 TP 数量不是独立优化目标，任何压缩都必须满足 lossless semantics 和 clear reviewability。展示规则只负责 presentation，不得反向改变 TP / Scenario 数据模型。

- lifecycle 高亮：complete 正常显示且不特殊高亮；draft 使用统一黄色或琥珀色提醒型高亮；blocked 使用统一红色强警示高亮。至少覆盖 `lifecycle_status` 单元格；同一 workbook 的范围和样式保持一致。
- Scenario `related_tp_id` 只引用一个 Base TP 时，沿用该 Base TP 的 lifecycle 高亮；引用多个 Base TP 时保留全部 TP_ID 并一条一行，单元格按 `blocked > draft > complete` 使用最高严重级别高亮。该展示不改变 Base TP lifecycle。
- Scenario sheet 的 merge 按列独立判断：连续多行的 `related_tp_id` 内容完全相同时必须纵向 merge 该列；包含多个 TP_ID 时，仅 TP_ID 集合和顺序均完全相同才视为内容相同。连续多行的 object role 内容完全相同时必须独立纵向 merge。即使各行的 `scenario_value_or_constraint`、relation branch 或 scenario semantic 不同，也不阻止相同 `related_tp_id` 或 object role 列的 merge；不同的 `scenario_value_or_constraint` 本身不得 merge，必须保持每个 independent expression 独立。`why_relevant_to_scenario` 或 `scenario_application` 内容完全相同且连续时继续允许 merge。merge 只影响展示，不删除 TP_ID，不改变每行 Scenario expression 的逻辑关联、lifecycle 或 `related_tp_id` 的 Base TP traceability 语义。
- 文本包含中文分号 `；` 或英文分号 `;` 时，在分号处分行显示；只改变展示，不改变字段内容和语义。

- lifecycle missing-input report 在 Excel 中使用独立 sheet。
- workbook 写完后必须验证最终 Scenario sheet 的实际 merged-cell ranges。每个长度大于 1 的连续相同 `related_tp_id` group 必须存在覆盖该 group 的 merged range；连续完全相同的 object role group 同样必须有覆盖该 group 的实际 merged range。不得只检查源数据内容相同。`scenario_value_or_constraint` 仍保持独立且不 merge；任一必需 merged range 缺失时 Output Contract Gate 失败。

### 4.5 Output Order

建议按以下顺序输出：

1. Register Access sheet。
2. Config Space sheet。
3. Dynamic Input sheet。
4. Cross sheet。
5. 以功能场景或指令作为入口时，每个命名 Scenario 的独立 `Scenario - <scenario_name>` sheet。
6. 任一 Scenario legality 或 parameter disposition 无法唯一确定时，输出对应 Scenario missing-input report；parameter disposition unresolved 包括 disposition 无法唯一判断、inactive/default rule 不完整，或 `testcase-generation-ready` 所需 disposition 信息不充分；不存在时不生成。
7. 存在 Relation Atom ownership Pending 时，relation ownership missing-input report；不存在时不生成。
8. Debug sheet。
9. Performance sheet。
10. Output Result sheet。
11. 存在任意 draft / blocked TP 时，lifecycle missing-input report；不存在时不生成。
12. 用户明确要求时，inventory-level missing 或 Completeness Review report。

### 4.6 Final Gates

交付前检查：

- **Schema**：每个 TP、Scenario 和 report 均符合本 Output Contract，字段职责唯一，未新增字段体系或重复 category 字段。
- **Base Inventory Completeness**：Register Access、Config Space、Dynamic Input、Cross 已按全部输入资料处理；适用对象均有 complete、draft 或 blocked TP。Cross generation 前必须达到 `RELATION_EXTRACTION_COMPLETE = TRUE`：所有 Config / Dynamic 显式 multi-object semantic、独立 constraint / mapping / legality 描述和 scenario-specific semantic 均已完成 extraction 并进入 Ownership，且未以已有 Cross 数量作为停止条件。只有该状态成立后才可用 `Extracted Base Relation Atoms == Covered Base Relation Atoms` 宣称 Cross complete。所有明确 Relation Atom 均完成唯一处理：scenario-independent Atom 由 Base Cross lossless 承载，scenario-specific Atom 进入 Scenario，ownership unresolved Atom 必须全部进入 relation ownership missing-input，不得遗漏、猜测 Base Cross / Scenario ownership，或生成虚假 TP_ID；不得只停留在 Config / Dynamic 描述。任一条件不满足时不得通过 Gate。
- **Target Isolation**：删除 Prompt 目标名称后四份 Base Inventory 保持相同；Scenario 未改写、裁剪、补充或重排基础 TP。
- **Lifecycle Closure**：状态符合 Lifecycle；每个 draft / blocked TP 均有完整 lifecycle missing-input 记录，仅 mapping 为空未触发 draft。
- **Scenario Isolation / Legality**：每个命名 Scenario 独立，Scenario 字段各守职责，Scenario legality 未进入或收缩 Base Config、Dynamic、Cross。分别检查：1) **Scenario semantic completeness**：由 Complete Base Inventory、All Input Documents 和 Target Scenario 确定的 applicable Base-derived 与 scenario-specific legality semantics / relations 是否全部表达，只有 `Scenario Covered Legality Set == Scenario Applicable Legality Set` 时通过；2) **Base TP traceability**：与 Base verification object 相关的 Scenario expression 是否通过 `related_tp_id` 正确关联对应 Base TP。不得把参与对象宣称为 semantic source，也不得要求 scenario-specific semantic 必须有 Base semantic-source TP。任何 applicable semantic / relation 未表达或 Base TP 关联错误时，`Scenario legality completeness = FAIL`，不得交付最终 workbook。该 closure 不代表 Scenario dependency completeness；Scenario legality unresolved 时不得猜测，并按现有规则生成独立 Scenario missing-input report；3) **Scenario Parameter Disposition Integrity**：仅对可进入 testcase generation 的 Scenario，所有影响 behavior、legality、executability 或 parameter generation 的相关 Config / Dynamic parameter 均已唯一归入 constrained/fixed、free 或 inactive；semantic / relation 当前不适用不得让该范围内的 parameter 跳过三分类判断，omission 未被当作 disposition，也不得由 semantic / relation 不适用直接推导 inactive；constrained/fixed 有明确 constraint，free 通过 `related_tp_id` 可追溯 Base legal space，且仅在对应 Base TP 已采用 parameter-scan strategy 时检查其 structured bins；inactive 由逐 parameter expression 或共享 rule expression 明确且唯一确定适用 parameter 集合、applicability condition 和每个适用 parameter 的最终 inactive/default constraint。parameter disposition 或 inactive/default rule unresolved 时必须进入 Scenario missing-input，下游无需重新读取 spec 或猜测。任一条件不满足时 `testcase-generation-ready = FALSE`；不检查下游 implementation algorithm。
- **Coverage Integrity**：coverage strategy 与验证 intent、category、监测和测量边界一致；Cross merge 必须 lossless，每个 independent branch 仍明确可见并映射到实际 coverage，不得以宽泛抽象概念或 bins 替代输入资料明确 relation。对 parameter-scan Config / Dynamic complete TP，必须存在完整 structured bins：输入资料或明确 coverage intent 要求独立追踪的 special / typical target，以及 coverage intent 要求独立覆盖的 numeric boundary，已使用 explicit bins；未被 explicit bins 承接但仍属于 coverage target 的有效空间由 residual/range bin 承接，bins 与 legal space 一致且不含 unreachable value，explicit 与 residual 不错误重叠，且未自行创造 typical、special、representative value 或 semantic category，下游无需从自然语言重新推导 partition；否则 complete 状态下 Gate 失败。主动 negative verification target 使用普通 bins，并与 legal coverage bins 分离；仅在采样值本身被明确规定不应出现时使用 illegal_bins，error expectation 不决定 bin type。非 parameter-scan TP 不强制 bins。implementation inputs 与 implementation object mapping 未混用；不检查 randc、iteration、testcase count、minimum scan count 或 scheduling。
- **No Inference**：未补充输入资料未定义的设计语义、行为、路径、采样、阈值、输出类别或 debug capability。
- **Output Contract**：sheet 组成、schema、输出顺序、高亮、换行和 merge 均符合本章。最终 explanation、semantic 和 behavior 默认中文，identifier、signal、enum、opcode、encoding、error code 和代码表达式保持原文；不得因逻辑表达自动生成整句英文。Constraint expression 语义完整，applicability / condition / result 对应清楚，每个 independent constraint 独立可读；简单 relation 使用直接逻辑表达，complex count / resource / cardinality 选择 reviewer 无需拆解公式即可理解的结构化中文，不存在省略对象或 branch 的 `...`、复杂符号压缩或为减少文字/行数合并独立规则。若逻辑公式与结构化中文语义等价，选择更易 review 的形式。parameter-scan complete TP 提供 downstream-consumable structured bins，testcase-generation Scenario 提供无歧义 parameter disposition。workbook 写完后已检查实际 merged-cell ranges：Scenario 连续相同 `related_tp_id` 和连续相同 object role 均已实际 merge，不同 `scenario_value_or_constraint` 保持独立；display optimization 未改变底层数据关系。若任一 Gate 不通过，不得交付。
