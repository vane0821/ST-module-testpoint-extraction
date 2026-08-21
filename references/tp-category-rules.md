# TP 分类规则

Module ST Testpoint Extraction 输出以下 TP 类别：

## 寄存器访问属性 TP

用于验证寄存器表本身实现正确，包括复位值、访问 Master 遍历、bit 读写覆盖、Side Effect、字段非法 / 非规范写入。

## 配置空间 TP

用于验证配置字段写入后是否在 DUT 内部产生定义的配置效果。配置空间不引入请求/激励。

## 动态输入参数 TP

用于验证指令、任务、descriptor、command 等动态输入参数对模块行为的影响。不默认和配置空间组合。

## Debug 能力 TP

用于验证 debug 命令、状态、触发时机、breakpoint/step 等能力。候选可以来自模板，但必须根据项目输入件确认。

## 性能验证 TP

用于验证原子工作场景性能。监测方式、测量边界和性能目标必须一致。

## 可选输出结果覆盖 TP

完全由输入资料驱动。输入资料未提供时不生成，也不报错。

## 压力测试与 Workload

压力测试和真实软件 workload 不作为当前 ST 层面 module 基础 TP 默认展开。
