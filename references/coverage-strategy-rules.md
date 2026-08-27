# 覆盖策略规则

complete TP 必须有明确覆盖策略。`coverage_strategy` 是从 `verification_goal` 到实际 coverage 实现的最小映射，不以 testcase、covergroup、assertion 为默认模板。空间型 TP（Config Space、Dynamic Input、以及需要 value-space coverage 的 Cross）使用 coverage target、bins、illegal_bins、ignore_bins，verification method 仅在需要时输出；行为型 TP（Register Access 及其他以行为判定为主且无独立 value-space bins 的 TP）可仅使用 testcase、covergroup 或 assertion 等 verification method，不得制造无意义 bins。coverage_strategy_mapping 在实际实现对象已知时保存 testcase name、assertion code 或 coverage code；不得将实现映射写入 coverage_strategy。

规则：

1. 不得只写空标签。
2. 每个条目必须有当前 TP 的明确对象、值、范围或集合。
3. assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
4. covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
5. 性能 TP 的覆盖策略必须和监测方式、测量边界一致。 
6. `testcase` 只标识测试构造方式，不展开 driver、handshake、等待、恢复、checker 或循环等 testcase 实现步骤；具体 testcase name 仅在适用的 `coverage_strategy_mapping` 中保存。
7. coverage_strategy 定义值分类与统计空间。valid、invalid、reserved、unsupported 是语义分类，不自动对应 bin 类型；主动作为验证输入覆盖的 invalid、reserved、unsupported 值使用普通 bins；仅在输入资料明确规定采样值不应出现时使用 illegal_bins；ignore_bins 仅用于明确不纳入覆盖统计的值。

## 动态参数 TP

动态参数 TP 未建立 testcase/covergroup/assertion 的实际实现映射时，仍可为 complete；其覆盖策略必须至少包含：

```text
- coverage_space: range / data / address / format / mode
- coverage_target: 参数名或字段定义 bit range
- bins: 需统计的值、范围或类别
- illegal_bins: 覆盖模型中的非法采样值
- ignore_bins: 不参与覆盖统计的采样值
```

`coverage_target` 必须写出具体硬件对象、字段或编码空间，不得只写“对应覆盖空间”等泛称。`illegal_bins` 仅表示输入资料明确不应出现的采样值，不代表 DUT 必须报错；invalid、reserved、unsupported 不得仅因语义分类进入 `illegal_bins`。`ignore_bins` 仅表示明确不参与覆盖统计的值，不代表 DUT 行为异常。testcase、covergroup、assertion 可以作为后续实现方式维护；具体实现对象仅在适用的 `coverage_strategy_mapping` 中保存，且不是动态参数 TP 为 complete 的必要字段。

## 生命周期例外

complete 必须有明确覆盖策略；draft 仅在当前已选择的 coverage strategy 所需映射、HDL path、monitor mapping、sample event 或其他执行/判定信息缺失时生成，且必须有当前 TP 所需的明确目标；仅缺少不影响策略成立的 coverage_strategy_mapping 不降级。blocked 进入对应 Excel sheet，未知覆盖策略允许为空，不得猜测填充。每个 draft / blocked TP 必须在自动输出的 lifecycle missing-input report 中记录 affected_tp_id、lifecycle_status、missing_content、completion_or_unblock_condition；Completeness Review 和 inventory-level missing 仍仅按需输出。
