# 覆盖策略规则

每个非动态的 **complete TP** 至少包含以下条目之一。动态参数 TP 的例外见“动态参数 TP”。draft TP 的例外见“生命周期例外”。

```text
覆盖策略:
- testcase: <测试构造方式或 testcase 名称>
- covergroup: <covergroup / coverpoint 名称>
- assertion: <assertion 名称或生成规则>
```

规则：

1. 不得只写空标签。
2. 每个条目必须有可追踪的名称、构造方式或规则。
3. assertion 只在输入资料给出明确时序、安全、边界或状态约束时生成。
4. covergroup 需要明确覆盖对象、采样事件和相关 HDL / monitor 映射。
5. 性能 TP 的覆盖策略必须和监测方式、测量边界一致。 

## 动态参数 TP

动态参数 TP 未建立 testcase/covergroup/assertion 映射时，仍可为 complete；其覆盖策略必须至少包含：

```text
- coverage_space: range / data / address / format / mode
- coverage_target: 参数名或字段定义 bit range
- bins: 需统计的值、范围或类别
- illegal_bins: 覆盖模型中的非法采样值
- ignore_bins: 不参与覆盖统计的采样值
```

`coverage_target` 必须写出具体硬件对象、字段或编码空间，不得只写“对应覆盖空间”等泛称。`illegal_bins` 仅表示覆盖模型中的非法采样值，不代表 DUT 必须报错。`ignore_bins` 仅表示不参与覆盖统计的采样值，不代表 DUT 行为异常。二者不用于描述 DUT 对参数值的功能处理；DUT 的合法映射、非法/保留处理和有效 bit 关系只能写在 `expected_result`。testcase、covergroup、assertion 可以作为后续实现映射维护，但不是动态参数 TP 为 complete 的必要字段。

## 生命周期例外

complete 必须有明确覆盖策略；draft 可缺覆盖策略或映射，但必须有目标、场景、缺失项与完成条件；blocked 不生成 TP。
