# 覆盖策略规则

每个 **complete TP** 至少包含以下条目之一。draft TP 的例外见“生命周期例外”。

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

## 生命周期例外

complete 至少一项明确覆盖策略；draft 可缺覆盖策略或映射，但必须有目标、场景、缺失项与完成条件；blocked 不生成 TP。
