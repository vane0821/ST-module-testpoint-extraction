# Cross Expression Contract

Cross TP 保留关系语义，但不在 Excel 中展开冗长 SV implementation syntax。

## Verification Goal

条件关系固定为：

```text
if (<SV-style boolean expression>)
	<result expression>
```

条件支持 `==`、`!=`、`<`、`<=`、`>`、`>=`、`&&`、`||`、`!` 和 `inside {value, [lower:upper]}`。`if` 与结果换行，结果使用一个 tab 缩进；固定值、范围和集合均写实际值。

两个或多个对象在全部适用输入下直接决定目标对象时，固定为：

```text
<target object> = <multi-object expression>
```

同一关系目标的多个分支在同一 TP 中依次成块，并以空行分隔。无法由上述两种形式无损表达的明确关系进入 Cross Skill Draft。

## Cross Coverage

Cross TP 固定使用紧凑条件 DSL，不在 TP 中展开 `binsof(...) intersect`：

```text
cross bins：
<bin_name> =
	<object condition>
	&& <object condition>

illegal cross bins：
<bin_name> =
	<object condition>
```

没有 illegal cross bin 时固定写 `illegal cross bins：` 后换行 `无`。每个 bin 独占一个表达块。Cross coverage 的对象条件只使用 `A == value`、`A inside {value, [lower:upper]}` 及其 `&&` / `||` 组合；Verification Goal 中其他可读比较运算不得直接复制为 cross bin selector。

### Negative 与 illegal cross bins

- valid、invalid、reserved、unsupported 或 forbidden 是 relation semantic，不自动决定 bin type。
- 需要主动构造并由 REF / scoreboard 检查报错、忽略、降级或其他已定义行为的组合，使用普通 `cross bins`。
- 只有输入资料明确规定某组合在当前 coverage sampling domain 中不应出现、命中本身即表示非法采样状态时，才使用 `illegal cross bins`。
- DUT 对某输入组合返回 error，不等于该组合应进入 `illegal cross bins`。
- 组合 legality 或对应行为无法由输入资料唯一确定，且该信息影响 cross partition 时，TP 标记 draft；不得自行归入普通 bin、illegal cross bin 或 `无`。

生成脚本固定展开：

- `A == value` → `binsof(cp_A) intersect {value}`；
- `A inside {value, [lower:upper]}` → `binsof(cp_A) intersect {value, [lower:upper]}`；
- `&&` / `||` 保持对应的 cross bin select 逻辑。

`verification_goal` 保留关系语义，供 REF / scoreboard 计算预期结果；Cross coverage 只确认多对象组合发生，不重复生成 assertion、结果 checker 或 testcase。testcase 仅作为下游 stimulus implementation，不写入 Cross TP 的 `coverage_strategy`。

需要 `!=`、大小比较、`with`、动态集合或其他当前映射不支持的 cross bin selector 时进入 Cross Skill Draft，不输出近似 bin 或冗长手写 SV。

## Cross Skill Draft

Cross Skill Draft 只处理“输入关系已明确，但现有 Cross 模型无法无损表达或转换”的缺口，不替代输入缺失导致的 TP draft，也不生成 TP_ID。每条记录包含：

- `involved_objects`；
- `observed_relation`；
- `model_gap`；
- `required_skill_decision`。

受影响关系停止生成；其他可表达 TP 继续处理。确认新模型后作为独立 Skill evolution 治理。
