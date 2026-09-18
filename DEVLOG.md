# Development Log

本文件记录 skill 开发历史，不参与 runtime 规则解释。

## 2026-09-17 | Cross Negative and Illegal Bin Boundary

### Problem

Cross Contract 提供 `illegal cross bins` 格式，但未定义 negative combination 与 illegal sampling state 的边界，导致需要主动验证错误行为的组合可能被写成命中即报错的 illegal bin。

### Root Cause

Cross 的组合 DSL 缺少与上位 Value/Input-Space negative bin semantics 一致的 category-specific 落地规则。

### Change

主动验证报错、忽略、降级或其他已定义行为的 invalid / reserved / unsupported / forbidden 组合使用普通 cross bin；仅当输入资料明确规定组合在当前 sampling domain 中不应出现时使用 illegal cross bin。影响 partition 的 legality 或行为无法唯一确定时标记 draft。

### Preserved Behavior

Cross 固定使用 cross coverage；REF / scoreboard 结果判定、Cross 对象模型、关系表达、bin DSL、selector 限制、Lifecycle、Cross Skill Draft、schema 及其他 category 的 negative bin 规则保持不变。

### Validation

Observed Behavior Change 仅为 Cross negative combination 的 bin-type 判定与缺失输入处理。Cross Contract 与 regression 使用同一规则，并与上位 negative bin semantics 一致；Final Coverage Gate 继续通过 Contract 验收，Preserved Behavior 未回退。

### Pending

无。

## 2026-09-17 | Cross Coverage and REF Responsibility Boundary

### Problem

Cross 可在 testcase、assertion 和 cross coverage 之间选择，导致同一关系既由 Cross assertion 判定、又由 REF / scoreboard 判定，且 testcase stimulus 被误当成 Cross verification method。

### Root Cause

Cross 的组合覆盖职责与下游 stimulus、结果预测和 checker 职责没有形成排他边界。

### Change

Cross 固定使用 cross coverage；`verification_goal` 保留完整关系语义，`cross bins` 确认多对象组合发生。testcase 只负责产生组合，REF / scoreboard 负责计算预期并判定 DUT 结果，二者不进入 Cross `coverage_strategy`。

### Preserved Behavior

Cross 对象模型、Relation Extraction、Base / Scenario ownership、关系分支、紧凑 cross-bin DSL、Lifecycle、Cross Skill Draft、schema 和 implementation TODO 规则保持不变。其他 category 的 testcase / assertion 选择不变。

### Validation

Observed Behavior Change 仅为 Cross method 从可选 testcase / assertion / cross coverage 收敛为固定 cross coverage。Cross Category Rules、Cross Expression Contract、Final Coverage Gate 与 regression 使用同一职责边界；关系语义仍完整保留，Preserved Behavior 未回退。

### Pending

无。

## 2026-09-17 | Implementation TODO Lifecycle Boundary

### Problem

Cross 关系、验证目标和 assertion 方法已经完整时，仅因 HDL signal / path 等实现绑定尚未提供，TP 仍会被标记 draft，混淆了设计语义完整性与 SV 实现就绪度。

### Root Cause

通用 Lifecycle 将所选策略依赖的所有 implementation input 都作为 complete 的前置条件，未区分 category-required implementation input 与非必需 implementation binding。

### Change

设计语义、验证目标和 method 完整时，仅缺非 category-required implementation binding 的 TP 保持 complete；`coverage_strategy` 只写 method，`coverage_strategy_mapping` 使用 `TODO（missing <具体 implementation input>）`，脚本不生成对应 SV 并输出实现待办。Category-required implementation input / mapping 缺失仍按 Lifecycle 判定。

### Preserved Behavior

缺 condition、result、behavior 或其他 design semantic 的 TP 仍为 draft；Register Access 的 category-required mapping 不放宽；Cross Skill Draft 仍只处理表达模型或脚本转换能力缺口。TP 粒度、Cross 表达、schema、missing-input report 类型和其他 category 行为不变。

### Validation

Observed Behavior Change 仅为非 category-required implementation binding 缺失时由 draft 改为 complete，并由 `coverage_strategy_mapping` 承载 implementation TODO。Lifecycle、字段职责、Coverage Responsibility、Dynamic / Cross 局部规则、Input Processing、Output Contract、Final Gate 与 regression 使用同一边界；Preserved Behavior 未回退。

### Pending

无。

## 2026-09-17 | Cross Expression Governance

### Problem

Cross 使用抽象 Relation Atom 和宽泛 lossless merge 描述，缺少直观、紧凑且可由脚本确定性展开的表达；直接写 SV `binsof/intersect` 又会使 Excel 冗长。输入关系完整但现有模型无法表达时，也容易被误标为 TP draft 或被生硬套用。

### Root Cause

Cross semantic expression 与 implementation syntax 未分层，也没有定义稳定的目标 DSL、cross-bin DSL 和 Skill 自身模型缺口处理。

### Change

在全局输出原则中明确 TP 必须简洁、人工可读、可由脚本确定性生成 SV，且不在 Excel 展开大量 implementation syntax。本轮仅治理 Cross：支持换行缩进的 `if` 条件关系和直接多对象等式；cross bins 使用简洁条件 DSL，由脚本展开为 `binsof/intersect`；新增 Cross Skill Draft report。

### Preserved Behavior

其他 category 的业务规则不变。Cross 的多对象边界、Base / Scenario ownership、Relation Extraction 完整性、No Inference、TP Lifecycle、coverage branch 完整性和 schema 保持不变。Cross Skill Draft 不替代 design missing-input，也不生成伪 TP_ID。

### Validation

Observed Behavior Change 仅为全局表达质量要求，以及 Cross 粒度、表达格式、cross coverage 序列化和 unsupported-model 处理。Cross Category Rules、Input Processing、Missing-input Flow、Output Contract、Final Gate 与 regression 使用同一模型。

### Pending

无。

## 2026-09-16 | Compact Scriptable Coverage Strategy

### Problem

structured coverage 虽要求可直接实现，但输出模板包含 method、coverage space、target、sample event 和嵌套 bins 等多层信息，重复当前 TP 行已有对象信息，Excel 中冗长且不便评审。

### Root Cause

将 coverage implementation schema 误放入 TP 的 `coverage_strategy`，没有区分“脚本所需的最小 bin 输入”和“脚本生成的 covergroup 结构”。

### Change

structured coverage 收敛为保留实际换行的 `cover bins` / `illegal bins` block；每个 bin 一行，直接给出 name、具体 value 或 `[lower:upper]` range。target 从当前 TP 行取得，默认采样和 implementation structure 由脚本处理；只有明确的特殊 sample event 与 ignore space 才额外输出。

### Preserved Behavior

bin partition、boundary、typical、legal residual、negative coverage、illegal/ignore 语义、地址 64-value threshold、越界行为、Lifecycle 和 implementation mapping 职责保持不变。简化格式不减少 coverage 信息，也不把预期报错的 negative input 改为 illegal bin。

### Validation

Observed Behavior Change 仅为 `coverage_strategy` 的序列化和显示格式；Config、Dynamic、共享 Contract、Excel display、Final Coverage Gate 与 regression 均接受同一 bin block，字段职责无冲突。

### Pending

无。

## 2026-09-16 | Shared Address Coverage Profile

### Problem

Config Space 与 Dynamic Input 的地址对象缺少共同、确定的输出规则；大地址空间容易产出描述性解释，小地址空间也没有明确何时逐地址展开。

### Root Cause

Address 仅作为通用 numeric dimension 处理，未建立跨 category 的识别规则、展开阈值和固定输出模板。

### Change

在 Value/Input-Space Coverage Contract 中建立唯一 Address Coverage Profile：识别明确 address 语义或独立 `地址` / `addr` / `address` token；合法地址值不超过 64 时逐地址建 bin，超过 64 时固定输出边界、典型地址、legal residual 和完整可表示越界 partitions。Config 与 Dynamic 仅路由到该共享 Profile。

### Preserved Behavior

Config 与 Dynamic 的对象模型、TP 粒度、schema、category ownership 和 Scenario 行为不变；越界行为、sample event 和设计 semantic 仍不得猜测；无可表示越界空间时不制造越界值；地址 coverage 不扩展为 path-level 通路验证。

### Validation

Observed Behavior Change 仅为两类 address TP 共用 64-value threshold 与固定输出模板。共享 Contract、两类 Category Rules、Lifecycle、Final Coverage Gate 与两份 regression 一致，未发现额外行为变化。

### Pending

无。

## 2026-09-16 | Numeric Dynamic Coverage Closure

### Problem

Dynamic numeric/address TP 仍可能用“覆盖输入资料定义的范围”“完整 address bins”等占位表达，未稳定输出上下边界、典型值、越界空间和越界行为；coverage strategy 不足以直接实现，enum mapping 的显示也可能挤在同一行。

### Root Cause

共享 Value/Input-Space Contract 禁止自动选择 typical value，且未定义 numeric coverage profile；Dynamic Category Rules 只要求完整范围，没有把数值目标、partition、negative behavior、Lifecycle 和 Excel display 闭合为同一模型。

### Change

为独立 numeric range/address coverage 建立确定性 profile：显式 lower/upper boundary、按中点公式选择 interior typical coverage sample、legal residual、完整可表示的 below/above-range negative partitions，并要求输入资料定义越界行为。缺少越界行为时标记 draft。Enum/format/mode mapping 改为 value-to-semantic 且逐行显示。

### Preserved Behavior

typical sample 不构成 DUT semantic；special semantic、越界行为、采样方式和 mapping 仍不得猜测；无可表示越界空间时不制造越界值；negative target 仍使用普通 bins，只有输入资料规定采样值本身不应出现时才使用 illegal_bins。Dynamic TP 粒度、category 名称、schema、relation ownership 和 Scenario 行为不变。

### Validation

Observed Behavior Change 仅包含 numeric/address 目标与 coverage partition 的确定化、越界行为缺失时的 draft 判定，以及 enum mapping 的逐行显示。共享 Contract、Dynamic Category Rules、Lifecycle、Output Display、Final Coverage Gate 和 regression 使用同一模型；未发现额外行为变化。

### Pending

无。

## 2026-09-16 | Dynamic Input Dimension Model

### Problem

Dynamic Input 以“单参数完整 input space”为粒度，range、data、address、format、mode 等不同验证维度容易被压进同一 TP；分类边界、输入要求、Lifecycle 和输出排序也缺少统一结构。

### Root Cause

旧模型区分了 parameter 与 coverage space，但没有把 verification dimension 提升为 TP 粒度的一部分，导致对象定义、Base Inventory 和 structured coverage contract 使用不同粒度。

### Change

Dynamic Input 改为一个 `parameter × verification dimension` 一个 TP；明确随请求携带的分类依据、输入要求、生成边界、各 dimension 的目标表达、Lifecycle、排序和 TP_ID 中 `coverage_space` 的职责，并同步 Base Inventory、Relation Extraction、Value/Input-Space Coverage Contract 与 regression。

### Preserved Behavior

Dynamic Input 仍只覆盖单参数 input semantic；多个 parameter 不合并，多参数关系仍进入 Relation Extraction；Base Inventory 不受 Scenario 裁剪；完整 bit range、No Inference、structured coverage contract、现有 schema、mapping 可选性和 Scenario parameter disposition 行为保持不变。

### Validation

Observed Behavior Change 仅为 TP 粒度从单 parameter 调整为 `parameter × verification dimension`，以及由该粒度直接决定的 inventory、目标和排序行为。Affected Rule Set 内的 Core Model、Category Rules、Lifecycle、coverage reference、Output Contract 与 regression 使用同一模型；Preserved Behavior 未回退。

### Pending

无。

## 2026-09-16 | Core Model Impact Closure

### Problem

Core Model 迁移 structured coverage 详细规则后，Config、Dynamic 和 Output Contract Gate 仍残留旧职责展开，导致新 Contract 之外继续存在部分重复定义。

### Root Cause

上一轮优先完成第一章压缩和 Final Coverage Gate 路由，但没有彻底清理所有 downstream caller 的重复说明。

### Change

Config 与 Dynamic 只保留“何时读取 Contract”的 category routing 和各自对象规则；structured bins、tracking 与非适用行为统一由 Contract 和上位 Coverage Responsibility 定义；Output Contract Gate 删除与 Coverage Integrity 重复的 Contract 检查。

### Preserved Behavior

Config / Dynamic 对象模型、是否需要独立 value/input-space coverage 的判断、structured bins、tracking、Lifecycle、schema、Final Coverage Gate 和其他 category 行为保持不变。

### Validation

检查 SKILL 全部 structured coverage 调用点后，详细语义仅存在于 Contract；Core Model、Config、Dynamic 和 Final Gate 只承担各自 routing / acceptance 职责。Observed Behavior Change 为空，Affected Rule Set 已闭合。

### Pending

Dynamic Input 章节自身的业务结构治理仍作为独立 change。

## 2026-09-16 | Core Model Responsibility Compression

### Problem

第一章同时承载通用模型、category-specific 行为、详细 structured bins contract 和部分 Output Contract，形成第二套 Category Rules，与第 2、4 章重复且加载成本过高。

### Root Cause

持续演进时将 coverage regression 直接追加到 Core Model，未维持“通用职责、category 行为、输出 schema、条件性详细 contract”的分层。

### Change

将第一章收敛为 Scope、Category Object Model、Common Field Responsibilities、Lifecycle、Coverage Responsibility and Precedence、TP_ID、Global Output Principles；删除 category-specific 重复；将 structured bins、tracking、negative bin semantics 和对应 Lifecycle Gate 迁移为按需读取的 `references/value-input-space-coverage.md` 唯一详细定义；Final Gate 改为引用该 Contract。

### Preserved Behavior

七类 TP 对象模型、Lifecycle、coverage method precedence、stimulus / tracking 分工、structured bins completeness、explicit / residual partition、negative target / illegal_bins / ignore_bins、Dynamic coverage target、mapping responsibility、TP_ID、语言、constraint readability、Category Rules、schema 和 final gate 行为保持不变。

### Validation

Observed Behavior Change 仅为规则职责与加载位置调整；详细 coverage semantics 已完整迁移并由 SKILL 和 Final Gate 路由。Core Model 不再重复第 2、4 章职责，Category Rules 与 Output Contract 未被改写，Preserved Behavior 未回退。

### Pending

Dynamic Input 及后续 category 章节的结构治理作为独立 change。

## 2026-09-16 | Config Space Execution Closure and Cross-Document Semantics

### Problem

Config Space 虽已建立单字段 ownership 与目标格式，但章节仍缺少与 Register Access 同级的输入、粒度、Lifecycle、排序和输出闭环；字段局部缺少 semantic 时可能被过早判为 draft，忽略其他输入资料中的分散定义。

### Root Cause

Config 仍以原则描述为主，没有形成可逐项执行的 category model；输入判断停留在字段当前位置，没有明确全部输入资料范围内的 semantic closure。

### Change

将 Config Space 重构为“输入要求、生成范围、TP 粒度与行为、Coverage 与 Lifecycle、排序与输出”；增加跨文档 semantic 归并，要求在全部输入资料中建立 value-to-semantic mapping；明确 selector、opaque candidate-ID、complete / draft / blocked 和 mapping optional 的判定。

### Preserved Behavior

Config 单字段粒度、完整 value space、access-only RESERVED 边界、reserved / unsupported encoding、multi-object Relation Ownership、structured bins、covergroup tracking、schema、Base Inventory target isolation及其他 category 行为保持不变。

### Validation

Config 与 Register Access 达到相同章节成熟度但未强制相同 verification method；跨文档归并只使用已有事实；selector semantic 缺失不会伪装 complete；mapping optional 与通用 Lifecycle 一致。Affected Rule Set 中 Category Rules、Input Processing、Relation Extraction、Lifecycle、Output Contract 与 regression 无冲突。

### Pending

Dynamic Input 的章节结构治理作为独立后续 change。

## 2026-09-16 | Config Space Single-Field Ownership and Goal Format

### Problem

Config `verification_goal` 混合单字段值空间、条件行为和跨字段关系，同一 relation 在多个 Config TP 中重复；输出包含“编码空间为”“输入资料定义的候选来源”等无判定价值文字；只有访问语义的 RESERVED field 被错误保留为长期 draft Config TP。

### Root Cause

Config 单对象职责与 Relation Ownership 没有在 category source of truth 中形成排他边界，value-to-semantic mapping 也缺少稳定的 reviewer-facing 表达。

### Change

Config 固定使用一行一个 `<value / range> -> <semantic>`；单字段条件使用完整 `if / else`；multi-object semantic 只进入 Relation Extraction 与 Ownership，不复制进 Config TP；只有 register access semantic 的 RESERVED field 不进入 Config，而配置字段内部的 reserved / unsupported encoding 继续保留。

### Preserved Behavior

Config 单字段粒度、完整 value space、Lifecycle、value-space structured bins、covergroup tracking、schema、连续 index、Base Inventory target isolation、Cross / Scenario ownership 和其他 category 行为保持不变。

### Validation

Config TP 可独立读出当前字段全部值及单字段语义；multi-object relation 由 Cross / Scenario 唯一承载；access-only RESERVED 不再制造 Config draft；reserved encoding 未被误删。Affected Rule Set 中 Category Rules、Relation Extraction、Base Inventory、Lifecycle、Output Contract 与 regression 使用同一模型。

### Pending

Dynamic Input 的目标表达与冗余治理作为独立后续 change。

## 2026-09-15 | Separate RW and RESERVED Traceability with Shared RAW Evidence

### Problem

上一版为避免重复执行而删除 RESERVED TP，导致 RESERVED 覆盖目标失去独立 TP_ID 和 completeness traceability。

### Root Cause

“共享一次 testcase execution”被错误等同为“共享一个 TP identity”，混淆了验证目标追踪与验证实现复用。

### Change

同一 register 的 RW 与 RESERVED 各自保留 TP_ID，但共用完整 RAW verification goal、testcase method 和 `<module>_reg_rw_test` mapping；最终 Excel 纵向合并三个共享列，TP_ID 与 lifecycle_status 保持独立。

### Preserved Behavior

四组固定全宽 wdata、整体 expected rdata 计算、RESET assertion、R / RO 与其他 access property 边界、Lifecycle、schema、顺序、连续 index 及其他 category 行为保持不变。

### Validation

RW 与 RESERVED 均具有独立 TP traceability，且没有重复 testcase 或重复展示相同 RAW goal；Excel merge 仅改变展示，不删除 TP_ID 或改变 lifecycle。

### Pending

R / RO 与其他特殊 access property 是否未来并入同一全寄存器 RAW 模型，作为独立后续 change。

## 2026-09-15 | Fixed-Pattern RW and RESERVED RAW Closure

### Problem

RW pattern 被预先按 RW mask 裁剪，导致 RESERVED 位没有真正接收写入；RW 与 RESERVED 被拆为两个 TP，但实际由同一个 register RAW testcase 完成。

### Root Cause

TP 模型按字段属性拆分，而 testcase 的实际判定对象是固定全宽 wdata 作用于整个寄存器后的整体 rdata。

### Change

含 RW field 的 register 将 RW 与 RESERVED 合为一个 RAW TP；固定使用全宽全 0、全 1、`01` 交织和 `10` 交织四组 wdata，不按 RW mask 裁剪，并逐组按全部 access property 计算整体 expected rdata。此类 TP 统一映射 `<module>_reg_rw_test`；仅无 RW 时保留纯 RESERVED TP。

### Preserved Behavior

RESET 的整体值检查与 assertion method、R / RO 的独立属性检查、纯 RESERVED 覆盖、其他 access property 粒度、Lifecycle、mapping 必需性、TP schema、寄存器顺序、连续 index、功能边界及其他 category 行为保持不变。

### Validation

四组 wdata 均为固定全宽 pattern，RESERVED 位实际被写入；expected rdata 同时体现 RW 写入生效与 RESERVED 读回为 0。无法唯一计算整体 rdata 时进入 Lifecycle，未引入猜测；Category Rules、Lifecycle、Output Contract 与 regression 一致。

### Pending

R / RO 与其他特殊 access property 是否未来并入同一全寄存器 RAW 模型，作为独立后续 change。

## 2026-09-15 | Register Reset Assertion and Concise Access Goals

### Problem

RESET TP 重复抄写 field 明细，且使用 testcase 与适合持续检查复位状态的 assertion 不一致；普通访问目标的展示仍偏规则说明，不够像可执行检查项。

### Root Cause

RESET 的证据来源与最终输出形式没有分离；Register Access coverage method 被错误统一为 testcase；RAW 的比较范围未显式限定到对应 access 位域。

### Change

RESET 最终只输出整体寄存器 reset value，并固定使用 assertion；整体值必须由输入资料直接给出或由覆盖完整 register width 的字段值无歧义拼出。R / RO、RW、RESERVED 改为简洁的属性检查表达；RW 使用 RAW 测试并只比较对应 RW 位域。RESET mapping 使用 `<module>_reg_reset_assertion`，其他 access type 保持 testcase mapping。

### Preserved Behavior

每个 register + access type 的聚合粒度、字段与 bit range 完整性、RESET 和 RESERVED 的 register-level 边界、其他 access property 粒度、Lifecycle、mapping 必需性、TP schema、寄存器顺序、连续 index、功能边界及其他 category 行为保持不变。

### Validation

Observed Behavior Change 仅包含 RESET 展示形式与 coverage method，以及各 access goal 的简洁表达。RESET 整体值仍可追溯到完整输入；RW 未错误比较其他 access type 位域；Category Rules、Lifecycle、Output Contract 与 regression 一致，Preserved Behavior 未回退。

### Pending

Coverage Model、Scenario、Cross、其余 demo、模板与 Skill 目录结构的治理仍作为独立后续 change。

## 2026-09-15 | Register Access Aggregation and Reviewer-Facing Output

### Problem

普通 R / RO、RW 按 field 拆成多条 TP，与同一 access type 共用 testcase 的执行模型不一致，并产生大量重复行；最终 `verification_goal` 使用内部公式式英文，人工评审不直观；Register Access 的最小输入未在 category source of truth 中直接闭合。

### Root Cause

TP 粒度被绑定到 field，而不是实际独立 access target；内部行为模型直接泄漏为最终展示文本；category-specific 生成规则缺少与粒度对应的输入清单。

### Change

将普通 R / RO、RW 收敛为每个 register + access type 一条 TP，并完整列出对应 field / bit range；最终 `verification_goal` 改为可直接评审的中文；Register Access mapping 统一为 `<module>_reg_<access_type>_test`；在 Register Access 唯一权威章节增加与各 access target 对应的最小输入要求。

### Preserved Behavior

RESET 和 RESERVED 保持 register-level；其他 access property 仍按输入资料定义的真实对象粒度独立生成；Base Inventory、Lifecycle、Register Access 固定 testcase method、mapping 必需性、TP schema、寄存器顺序、全 sheet 连续 index、功能边界及其他 category 行为保持不变。

### Validation

Observed Behavior Change 仅包含 R / RO、RW 聚合粒度、最终展示语言、mapping 命名和显式输入闭合；同一 access type 的字段没有丢失或跨 register 合并。Category Rules、Lifecycle、Output Contract 与 regression 使用同一模型，Preserved Behavior 未回退，Affected Rule Set 内未发现语义冲突。

### Pending

Coverage Model、Scenario、Cross、其余 demo、模板与 Skill 目录结构的治理仍作为独立后续 change。

## 2026-09-15 | Register Access Rule Governance

### Problem

Register Access 的当前规则被项目实例、历史解释和 Final Gate 重复定义包围，正向生成路径不清晰，同一行为需要在 Category Rules、Lifecycle 与 Final Gates 多处维护。

### Root Cause

Register Access category-specific completeness 条件没有被通用 Lifecycle 直接容纳；Final Gates 重新展开 Category Rules；项目实例和 testcase implementation 边界重复留在局部规则中。

### Change

将 Register Access 的对象、粒度、固定访问模型、coverage method、mapping 和顺序收敛到 Category Rules，并重排为“生成范围、TP 粒度与行为、Coverage 与 Lifecycle、排序与输出”四个可扫描区块；Lifecycle 统一容纳 category-required 字段；Final Gates 改为引用 Category Rules；删除 Register Access mapping 的项目实例和已由 Scope 定义的 testcase implementation 重复说明。

### Preserved Behavior

RESET 保持 register-level 并直接展开实际 reset/default value；R / RW 保持 field-level；每个含 RESERVED field 的 register 恰好生成一个 register-level RESERVED TP，完整列出 field / bit range 并验证 `write no effect / read as 0`；其他 access property 按真实对象粒度生成；不同 access target 分别生成；Register Access 只承载访问语义；全部 Register Access TP 使用 testcase，同一 access type 共用 `<module>_reg_access_<access_type>_test`；mapping 仍是 complete 的必需项；寄存器顺序和全 sheet 连续 index 不变。其他 category、Scenario、Cross、Coverage Model、Output Contract 和 Excel display 行为不变。

### Validation

Intended Delta 仅改变规则组织、表达与可扫描性；Register Access 的对象识别、粒度、verification goal、coverage method、mapping、lifecycle、schema、顺序与 gate 语义均保持。Affected Rule Set 内由 Category Rules 提供唯一业务定义，Lifecycle 接受 category-required 字段，Final Gates 只引用相应规则；章节已形成清晰执行顺序，未发现非预期行为变化。

### Pending

Coverage Model、Scenario、Cross、其余 demo、模板与 Skill 目录结构的治理作为独立后续 change。

## 2026-09-14 | Coverage Method Precedence and Field Responsibility

### Problem

Coverage Model 的通用 method 选择和内容完整性规则与 Register Access 的 category-specific fixed testcase method 冲突。

### Root Cause

通用规则未明确 category-specific method 的优先级，并要求 `coverage_strategy` 重复承载验证对象或取值信息。

### Change

明确 coverage method 默认由 verification intent 决定，Category Rules 已固定 method 时以 category-specific rule 为准；固定 method 本身可构成完整 `coverage_strategy`，对象、操作与预期行为留在 `verification_goal`，具体实现名称留在 `coverage_strategy_mapping`。

### Preserved Behavior

Register Access 的 testcase method、mapping、RESET / R / RW / RESERVED 粒度与功能边界，以及 Config / Dynamic structured bins、Cross、Scenario、Lifecycle 和 Output Contract 保持不变。

### Validation

Register Access 的 `coverage_strategy = testcase` 无需重复对象或访问动作，mapping 仍为 `<module>_reg_access_<access_type>_test`；无 category-specific rule 时仍由 verification intent 选择 method；其他 category 无语义变化。

### Pending

无。

## 2026-09-14 | Register Access Reset, Reserved, and Testcase Mapping

### Problem

RESET TP 可能未直接实例化实际 reset/default value；RESERVED access 缺少独立 register-level target；Register Access 混入模块功能行为，且 coverage method 与 testcase mapping 不统一。

### Root Cause

Register Access 的 access property 与 module functional behavior 边界不够明确；RESET、RESERVED 和 coverage mapping 的生成约束未形成闭环。

### Change

要求 RESET 直接展开实际 reset/default value；新增每 register 一个 RESERVED TP，并将 `write no effect / read as 0` 定义为无需输入资料逐 register 重复声明的固定 access semantic；Register Access 只承载访问语义；全部 Register Access TP 固定使用 testcase，并按 access type 共用 `<module>_reg_access_<access_type>_test`。该 mapping 是 Register Access complete TP 的 category-specific required output，不改变其他 category 的通用 Lifecycle 规则。

### Preserved Behavior

寄存器表顺序、RESET register-level、R / RW field-level、单 TP 单 access target、Lifecycle、Missing-input、TP_ID 连续 index、其他 category、Config/Dynamic coverage、Language、Constraint Readability 和 Excel display 保持不变。

### Validation

检查 RESET 实际值及目标隔离；识别 RESERVED field 后无需额外 semantic 声明即可生成每 register 恰好一条 RESERVED TP，并验证 write no effect / read as 0；R / RW 不含模块功能行为；全部 Register Access TP 使用 testcase，同 access type mapping 相同，complete TP 不缺失 mapping，且不展开 testcase implementation；其他 category 的 mapping 与 Lifecycle 语义无变化。

### Pending

无。

## 2026-09-11 | Remove Downstream Consumer Semantics

### Problem

TP Skill 混入 downstream testcase consumption 语义。

### Root Cause

structured bins 的用途被错误扩展为 Case consumer 规则。

### Change

TP Skill 只定义 TP / Scenario 输出要求；移除 parameter scan 与 downstream consumer 语义，并将 Scenario free 和 Parameter Disposition Closure 收敛为 Base TP、Base legal space 与三分类完整性检查。

### Preserved Behavior

structured bins、covergroup tracking、testcase 不替代 coverage tracking、free / inactive / constrained/fixed、Lifecycle、Relation Extraction / Ownership、Constraint Readability、Excel merge、Missing-input 和 Skill / Case Development 边界保持不变。

### Validation

Value/Input-Space Coverage Output Contract 与 Final Gate 只检查 TP 自身 coverage 定义和 tracking；Scenario free 只追溯 Base TP 与 Base legal space；未保留 parameter-scan consumer 或 downstream testcase generation 判断；Preserved Behavior 未回退。

### Pending

无。

## 2026-09-11 | Generalize Value/Input-Space Coverage Contract

### Problem

当前 coverage model 已支持独立 value-space / input-space coverage，但权威 Contract 仍命名并限定为 Parameter Scan，导致 abstraction scope 与实际调用范围不一致。

### Root Cause

structured bins contract 最初从 parameter-scan 场景引入；恢复 value-space covergroup tracking 后，上位抽象未同步提升。

### Change

将 `Parameter Scan Output Contract` 统一提升为 `Value/Input-Space Coverage Output Contract`；独立 value/input-space coverage 成为上位 source of truth；parameter scan 保留为具体 downstream generation intent；structured bins 与 covergroup tracking 行为保持不变，并修正 Final Gate 中的 `coggverage` typo。

### Preserved Behavior

Config / Dynamic value-space coverage、structured bins、covergroup tracking、testcase stimulus construction、free 与 parameter-scan 的边界、Scenario disposition、Lifecycle、Relation Extraction、Constraint Readability、Excel display 和 Skill / downstream boundary 保持不变。

## 2026-09-11 | Restore Value-Space Coverage Tracking Semantics

### Problem

结构治理后 Config / Dynamic value-space TP 的 coverage method 可从 covergroup 漂移为 testcase，导致 stimulus construction 与 coverage tracking 职责混淆。

### Root Cause

“verification method 不机械套模板”被过度放宽，误删了 value-space coverage 原有的 covergroup tracking 能力。

### Change

明确 stimulus construction 与 coverage tracking 分工；value-space / input-space 加 structured bins 默认由 covergroup tracking；testcase 可作为 stimulus construction，但不能替代 coverage tracking；behavior-only TP 继续按实际 intent 选择 method。

### Preserved Behavior

Parameter Scan Output Contract、structured bins、Lifecycle、Scenario disposition、Relation Extraction、Constraint Readability 和 Skill / downstream boundary 保持不变。

## 2026-09-11 | Dynamic Coverage and Scenario Missing-Input Consistency

### Problem

Dynamic Input 中残留 coverage strategy 默认包含 bins 的旧口径；Scenario missing-input 已支持 parameter disposition，但 Output Order 和 report summary 仍只描述 legality；semantic / relation 不适用规则可能被用于跳过 parameter disposition closure。

### Root Cause

Parameter Scan Output Contract 落地后，Dynamic Input 旧描述未完全清理；Missing-input Flow 扩展职责后 Output Contract 未同步；Scenario semantic applicability 与 parameter disposition applicability 的边界未显式区分。

### Change

Dynamic coverage strategy 改为映射实际 verification method，仅 parameter-scan 时要求 structured bins；Scenario missing-input 统一覆盖 legality 与 parameter disposition；明确 semantic / relation omission 不得绕过 parameter disposition closure。

### Preserved Behavior

Constraint Readability、Parameter Scan Output Contract、Scenario Parameter Disposition、Relation Extraction / Ownership、Lifecycle、Coverage Model、Excel merge 和 Skill / downstream boundary 保持不变。

## 2026-09-11 | Parameter Disposition and Bin Semantics Cleanup

### Problem

free disposition 被错误绑定到 parameter-scan structured bins；explicit bin 规则可能诱导模型制造输入资料或 coverage intent 未定义的值；inactive/default 的抽象继承表述无法保证下游唯一解析最终约束。

### Root Cause

Scenario disposition 与 Base coverage strategy 的职责边界不够明确；bin 示例与生成要求未充分区分；inactive/default 共享规则缺少可直接判定的表达要求。

### Change

明确 free 只表示使用 Base legal space，并仅在对应 Base TP 已采用 parameter-scan strategy 时使用其 structured bins；将 explicit bins 收敛为输入资料或 coverage intent 明确要求的目标，格式示例不再构成模板；要求 inactive/default rule expression 自身唯一确定适用 parameter 集合、applicability condition 和每个 parameter 的最终约束。

### Preserved Behavior

其余 TP、Relation、Scenario、Coverage、Lifecycle、schema、Excel display 和 Output Order 行为保持不变。

### Validation

检查 free 不会反向创建 parameter-scan；非 parameter-scan Base TP 不强制 bins；未定义的 typical、special、representative value 或 semantic category 不会被生成；共享 inactive/default rule 可由现有 Scenario 表达直接且唯一解析，否则进入 Scenario missing-input report。

## 2026-09-09 | Scenario/Base Boundary and Coverage Semantics Cleanup

### Problem

Base Cross 可能被仅在特定 Scenario 成立的 relation 污染；Scenario dependency completeness 与 legality completeness 的边界可能混淆；`related_tp_id` 可能按对象参与关系而非 semantic source 建立追溯；主动 negative verification target 可能与 `illegal_bins` 混用；Performance allowed range 可能被误写为 coverage target。

### Root Cause

Base / Scenario source-of-truth 边界不够明确；traceability 字段职责不够明确；coverage intent 与 bin semantics 未被严格区分；Performance measurement target 与 acceptance criteria 职责混淆。

### Change

收敛 Base Cross 为脱离 instruction / function / Scenario 后仍成立的多对象关系；保持 Scenario legality completeness 与 dependency completeness 分离；将 `related_tp_id` 固定为 Scenario expression 的 Base semantic provenance；区分主动 negative verification target 与“不应出现”采样值的 bin 语义；区分 Performance measurement target 与 expected acceptance criteria。

### Preserved Behavior

TP category、TP granularity、lifecycle、TP/Scenario schema、`coverage_strategy_mapping`、Excel display rules、output order 和 Completeness Review 行为保持不变。

### Validation

使用以下典型场景进行规则回归：同一多对象关系分别在全局与特定 Scenario 下成立；单对象 semantic、Base Cross relation 和多源 semantic basis 的 Scenario traceability；非法输入主动触发明确错误行为与覆盖采样值不应出现；Performance metric 观测与 allowed range/threshold 判定。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。
## 2026-09-09 | Scenario Source Model and Traceability Fix

### Problem

scenario-specific relation 已正确从 Base Cross 排除后，Scenario 缺少合法 semantic source；`related_tp_id` 被要求承担全部 semantic provenance，导致 scenario-specific relation 没有合法 Base TP 可追溯。

### Root Cause

Scenario Extraction 输入模型把 Base Inventory 错当成 Scenario semantic source 全集；Base TP traceability 与 design semantic provenance 职责混淆。

### Change

将 Scenario Extraction 收敛为 `G(Complete Base Inventory, All Input Documents, Target Scenario)`；将 Scenario semantic source 区分为 Base-derived 与 Scenario-specific；将 `related_tp_id` 收敛为 Base TP traceability，不再承担完整 semantic provenance；同步调整 Scenario Applicable Legality Set 的输入来源与 Final Gate。

### Preserved Behavior

Base Cross 仍只允许 scenario-independent relations；TP category、granularity、lifecycle、coverage model、negative target / illegal_bins、Performance target / allowed range、schema、Excel display、output order 和 Completeness Review 保持不变。

### Validation

验证 scenario-independent relation 正常进入 Base Cross 并被 Scenario 复用；instruction-specific relation 不进入 Base Cross，但可从输入资料进入 Scenario；scenario-specific relation 不需要伪造 Base semantic-source TP；`related_tp_id` 不再把参与对象错误宣称为 relation semantic source。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。

## 2026-09-09 | Cross Relation Completeness and Review Readability

### Problem

明确的 multi-object relation 可能被遗漏，或只存在于 Config / Dynamic 描述而未形成 Cross；Base / Scenario relation ownership 判断不一致；多个具体 relation 可能被过度概括为宽泛 Cross；Cross `verification_goal` 退化成长段自然语言；Scenario 连续相同 `related_tp_id` 不再 merge，人工 review 可读性回退。

### Root Cause

Cross 缺少统一的 `Relation Extraction -> Ownership -> Lossless Merge -> Coverage Closure` 生成闭环；target-independent 被错误理解为 condition-free；结构治理过程中已有 display / review capability 未被完整保留。

### Change

建立生成时 Relation Atom 识别过程，明确 Base / Scenario ownership，建立 Cross completeness closure，限制 Cross lossless merge，恢复逻辑表达优先和具体 branch coverage，恢复 Scenario `related_tp_id` / object role Excel merge，并补充人工评审可读性约束。

### Preserved Behavior

TP category、其他 TP metamodel、Scenario source model `G(Complete Base Inventory, All Input Documents, Target Scenario)`、Scenario Applicable Legality Set、Scenario dependency completeness 当前边界、lifecycle、negative target / illegal_bins、coverage model、`coverage_strategy_mapping`、TP/Scenario schema、output order 和 Completeness Review 保持不变。

### Validation

重新检查同一输入集应满足：Config / Dynamic 中明确双对象 relation 可被识别；所有 scenario-independent relation 均有 Base Cross 承载；带 opcode / mode 条件的通用 relation 不因有条件而移入 Scenario；真正 scenario-specific relation 不污染 Base Cross；合并不丢 relation branch；Cross `verification_goal` 优先逻辑表达；coverage strategy 对应具体 branch；Scenario 连续相同 `related_tp_id` 恢复纵向 merge；merge 不改变底层 Scenario / lifecycle 数据。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。

## 2026-09-09 | Relation Extraction, Constraint Expression and Review Readability

### Problem

Base Cross relation 提取不完整；Config / Dynamic 中已识别的 multi-object semantic 未必进入关系覆盖；Base / Scenario ownership 判断不一致；Cross lossless merge 不稳定；AI 可能为压缩文字把多个 independent constraints 合并成难读自然语言；可逻辑表达的 relation 可能退化成长段散文；relation ownership Pending 缺少正式 missing-input 落点；Scenario 相同 `related_tp_id` merge 能力回退。

### Root Cause

1. multi-object relation 缺少统一 Relation Extraction / Ownership source of truth。
2. Cross completeness 过去以最终 TP 为中心，而不是以原始 Relation Atom 为中心。
3. Skill 未明确规定简洁不能破坏 constraint structure。
4. Missing-input model 未覆盖尚未形成 TP 的 ownership uncertainty。
5. 结构治理过程中 display / review capability 未被完整保持。

### Change

统一 Relation Extraction，清理旧 multi-object relation 直接进入 Cross 的旁路规则，明确 Base Cross / Scenario-specific / Pending ownership，增加 relation ownership missing-input report，建立 lossless merge 和逐 constraint 逻辑表达规则，使 Cross 与 Scenario 共用可读 constraint expression 原则，建立 relation completeness closure，并恢复按列判断的 Scenario `related_tp_id` / object role merge。

### Preserved Behavior

TP category 与其他 TP model、Lifecycle、Coverage Model 的 implementation input / mapping 边界、negative target / illegal_bins、`coverage_strategy_mapping`、Scenario source model `G(Complete Base Inventory, All Input Documents, Target Scenario)`、Scenario Applicable Legality Set、Scenario dependency completeness 当前 pending、`related_tp_id` Base TP traceability、Register Access、Config、Dynamic、Debug、Performance、Output Result、TP/Scenario schema 和 Completeness Review 保持不变。原 output order 仅插入按需生成的 relation ownership missing-input report。

### Validation

一致性检查覆盖：Config 中双对象 relation 进入 Relation Extraction；Dynamic multi-object relation 不再绕过 Ownership；通用 opcode/mode 条件 relation 可进入 Base Cross；target-specific relation 不污染 Base Cross；ownership unresolved 有独立 missing-input；Relation Atom 合并保持 branch；independent constraints 一条一行；可形式化约束不退化成长段自然语言；count、mutual exclusion、membership 等表达可直接评审；coverage strategy 对应具体 branch；Scenario 相同 `related_tp_id` 恢复 merge；merge 不影响 `scenario_value_or_constraint` 的独立表达。

### Pending

`Scenario dependency completeness` 是否需要独立模型，当前不纳入 legality closure。

## 2026-09-09 | Relation Atom Optional Applicability and Review Pending Handling

### Problem

Relation Atom 定义可能误导模型认为 `applicability_condition` 必填；存在 relation ownership Pending 时，Cross 可能被错误判为 `not_applicable`。

### Root Cause

Relation Atom 形式定义不够精确；Completeness Review 的 `not_applicable` 判据未考虑 unresolved ownership。

### Change

将 Relation Atom 定义收敛为 `[applicability_condition &&] joint_condition -> relation_or_result`，明确 applicability condition 可为空；Cross `not_applicable` 增加不存在 ownership Pending 的必要条件，并在 Pending 存在时引用对应 relation ownership missing-input。

### Preserved Behavior

其余模型、字段、流程、schema、coverage、Scenario、Excel display 和 output order 保持不变。

## 2026-09-09 | Final Gate Decoupling and Cross Expression Alignment

### Problem

Final Gate 错误依赖可选 Completeness Review；Core Model 的 Cross 公式未同步可选 applicability condition。

### Root Cause

generation gate 与 optional review flow 职责混淆；上层 Core Model 与下层 Relation Atom 表达漂移。

### Change

Final Gate 改为直接检查所有 ownership unresolved Relation Atom 是否进入 relation ownership missing-input；Core Model 的 Cross 表达统一为 `[applicability_condition &&] joint_condition -> relation_or_result`。

### Preserved Behavior

其余 Relation、Scenario、Coverage、Lifecycle、Output 行为保持不变。

## 2026-09-09 | Relation Extraction Execution and Output Regression Fix

### Problem

最终输出中的 explanation / behavior 大量使用英文；Scenario merge 规则存在但最终 workbook 未实际 merge；Relation Atom 模型存在，但 Base Cross extraction 仍明显不足。

### Root Cause

1. 中文规则只定义全局默认语言，没有定义 identifier 与 prose 的语言边界。
2. merge 只定义 display rule，没有 post-generation workbook validation。
3. Relation Extraction 只有抽象模型，没有 exhaustive execution pass，模型可能少提取后自行通过 completeness。

### Change

增加字段级 Language Rule；增加 transient exhaustive Relation Extraction Pass 和 `RELATION_EXTRACTION_COMPLETE` generation gate；限定 Cross completeness 只能在 extraction complete 后判断；为 Scenario merge 增加最终 workbook merged-range validation。

### Preserved Behavior

Relation Atom、Ownership、Lifecycle、Scenario source model、Scenario Applicable Legality Set、Coverage Model、negative bins、`related_tp_id`、TP/Scenario schema、Performance、Completeness Review 状态体系及 Cross 业务定义保持不变。

### Validation

回归检查覆盖：自然语言 explanation 默认中文；identifier、opcode、signal、error code 保持原文；Config 与 Dynamic 中所有 multi-object semantic 完成 relation scan；Base Cross 来自完整 Relation Extraction 而非主要关系总结；scenario-specific relation 正确 ownership；Cross 不做 Cartesian expansion；Scenario `related_tp_id` 和符合条件的 object role 实际出现在 merged-cell ranges；`scenario_value_or_constraint` 保持独立。

## 2026-09-11 | Constraint Readability and Generation Input Contract

### Problem

count / set / cardinality relation 被过度公式化，人工 review 困难；最终表达可能出现 `...` 等不完整压缩；object role merge 存在 optional / mandatory 漂移；parameter-scan Config / Dynamic TP 缺少稳定 structured bins；Scenario 未完整说明 parameter disposition，omission 可能被误解释为 free / inactive；描述下游消费时存在侵入 Case implementation algorithm 的风险。

### Root Cause

1. “逻辑表达优先”被错误提升为“形式化程度优先”。
2. Excel display rule 与 post-generation validation 存在口径漂移。
3. legal space 与 coverage partition 职责没有完全分开。
4. parameter-scan TP 尚未明确 structured bins 是 downstream-consumable output。
5. Scenario legality closure 没有完整回答每个相关 parameter 在当前 Scenario 中如何被约束。
6. “输出必须可消费”被错误扩展为“Skill 应定义 consumer implementation”。

### Change

将 Constraint Expression 改为 readability-first：simple relation 使用直接逻辑表达，complex count / resource / cardinality 优先结构化中文，禁止省略和复杂符号压缩；将 object role merge 统一为 mandatory；建立 Parameter Scan Output Contract，要求 parameter-scan complete TP 使用 explicit 与 residual/range structured bins；建立 Scenario Parameter Disposition Output Contract 和 Closure，定义 constrained/fixed、free、inactive，并将 unresolved disposition 接入现有 Scenario missing-input；`testcase-generation-ready` 仅表示输出充分且无歧义；明确排除 randc、iteration、testcase count、scan scheduling 等 Case implementation 规则。

### Preserved Behavior

Language Rule、Relation Extraction、Relation Ownership、Cross completeness、Lifecycle、negative target / illegal_bins、Scenario source model、Scenario Applicable Legality Set、`related_tp_id`、Performance、TP / Scenario schema、Output Order 和 Completeness Review 状态体系保持不变。

### Validation

一致性检查覆盖：simple relation 保持清晰逻辑表达；complex count/resource relation 使用可评审的结构化中文；最终 constraint 不使用省略；explanation / behavior 默认中文；`related_tp_id` / object role 在最终 workbook 中实际 merge；parameter-scan Config / Dynamic complete TP 提供 structured bins，special / typical / boundary explicit bins 与 residual legal space 清晰且一致；negative target 与 legal bins 分离；非 parameter-scan TP 不强制 bins；constrained/fixed、free、inactive 均可唯一识别且可追溯；Scenario omission 不推导 disposition；unresolved disposition 进入 Scenario missing-input；`testcase-generation-ready` 不包含下游算法含义；Relation Extraction exhaustive pass 未回退。
