# TP 分类与覆盖策略规则

各 category 独立生成；Completeness Review 仅审查 inventory。

| Category | ID |
|---|---|
| 寄存器访问属性 | `REG` |
| 配置空间 | `CFG` |
| 动态输入参数 | `DYN` |
| Debug | `DBG` |
| 性能 | `PERF` |
| 输出结果 | `OUT` |

动态参数使用 `<source>_DYN_<parameter>_<coverage_space>_<index>`；parameter type（reg/imm/mem 等）不进入 ID，禁止 REF/VAL/reference/content。

complete 至少有一项明确覆盖策略。draft 可以缺策略、HDL path、monitor mapping 或 sample event，但必须具备明确目标、场景、缺失项和完成条件。blocked 不生成 TP，仅记录阻塞原因和所需输入。
