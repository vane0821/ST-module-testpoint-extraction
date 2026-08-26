# 覆盖策略规则

每个非动态的 **complete TP** 至少包含以下条目之一。动态参数 TP 的例外见“动态参数 TP”。draft TP 的例外见“生命周期例外”。

coverage_strategy 描述验证方式或覆盖模型，不以 testcase、covergroup、assertion 为默认模板。动态参数 TP 使用 coverage space、coverage target、bins、illegal_bins、ignore_bins；其他 category 根据验证目标选择 testcase、covergroup 或 assertion。coverage_strategy_mapping 仅在 category 已定义需要实际实现映射时保存 testcase name、assertion code 或 coverage code。

规则：

1. 不得只写空标签。
2. 每个条目必须有可追踪的名称、构造方式或规则。
3. assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
4. covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
5. 性能 TP 的覆盖策略必须和监测方式、测量边界一致。 
6. `testcase` 只标识测试构造方式，不展开 driver、handshake、等待、恢复、checker 或循环等 testcase 实现步骤；具体 testcase name 仅在适用的 `coverage_strategy_mapping` 中保存。
7. coverage_strategy 定义值分类与统计空间；expected_result 定义 DUT 对合法、非法或 reserved 值的可观测行为。不得用 expected_result 重复列举全部合法 bins，也不得以“按规格处理”等泛称代替非法行为。

## 动态参数 TP

动态参数 TP 未建立 testcase/covergroup/assertion 的实际实现映射时，仍可为 complete；其覆盖策略必须至少包含：

```text
- coverage_space: range / data / address / format / mode
- coverage_target: 参数名或字段定义 bit range
- bins: 需统计的值、范围或类别
- illegal_bins: 覆盖模型中的非法采样值
- ignore_bins: 不参与覆盖统计的采样值
```

`coverage_target` 必须写出具体硬件对象、字段或编码空间，不得只写“对应覆盖空间”等泛称。`illegal_bins` 仅表示覆盖模型中的非法采样值，不代表 DUT 必须报错。`ignore_bins` 仅表示不参与覆盖统计的采样值，不代表 DUT 行为异常。二者不用于描述 DUT 对参数值的功能处理；DUT 的合法映射、非法/保留处理和有效 bit 关系只能写在 `expected_result`。testcase、covergroup、assertion 可以作为后续实现方式维护；具体实现对象仅在适用的 `coverage_strategy_mapping` 中保存，且不是动态参数 TP 为 complete 的必要字段。

## 生命周期例外

complete 必须有明确覆盖策略；draft 可缺覆盖策略或映射，但必须有目标、场景、缺失项与完成条件；blocked 进入对应 Excel sheet，未知覆盖策略允许为空，不得猜测填充。
