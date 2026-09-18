# Register Access Semantic Regression

只检查语义，不比较措辞或全文快照。

## 对象与粒度

- 每个具有明确整体 reset value 的 register 生成一条 register-level RESET TP；整体值可来自输入资料直接定义，或由覆盖完整 register width 的 field / bit range / reset value 无歧义拼出。
- R / RO 按每个 register + access type 各生成一条；RW 与 RESERVED 各自保留 register-level TP_ID。
- 同一 register 的 RW 与 RESERVED TP 共用完整 RAW verification goal 和 testcase；无 RW 的 RESERVED TP 独立表达 RESERVED 检查。
- 其他 access property 按输入资料确定的 register-level 或 field-level 生成。
- 普通访问属性、其他 access property 和独立模块功能行为不合并为同一个 access target。

## 行为与覆盖

- RESET 的最终 `verification_goal` 只写整体寄存器 reset value，不重复展开 field；R / RO 与纯 RESERVED 使用简洁中文表达属性检查；RW / RESERVED RAW 逐组列出固定全宽 wdata 和按全部 access property 计算的整体 expected rdata。
- RESET 使用 `coverage_strategy = assertion` 和 `<module>_reg_reset_assertion` mapping。
- 同一 register 的 RW 与 RESERVED TP 使用 `coverage_strategy = testcase` 并共同映射 `<module>_reg_rw_test`；其他非 RESET access type 使用对应 `<module>_reg_<access_type>_test` mapping。
- RAW 的四组 wdata 固定为寄存器全宽的全 0、全 1、`01` 交织、`10` 交织，不按 RW mask 裁剪；expected rdata 逐 bit 应用 RW、RESERVED 及其他明确 access property，无法唯一计算时不得标记 complete。
- Register Access mapping 缺失时不能标记 complete；其他 category 是否要求 mapping 仍由各自 Category Rules 决定。
- mapping 只表示 testcase 名称，不展开 testcase 实现。

## 输出与顺序

- Register Access schema 保持 `TP_ID`、`lifecycle_status`、`verification_goal`、`coverage_strategy`、`coverage_strategy_mapping`。
- 不单设 `expected_value`；实际 reset/default/fixed value 位于 `verification_goal`。
- 每个 register 内按 RESET、R / RO、RW、RESERVED、其他 access property 排序；同一 register 的 TP 连续，全 sheet index 连续递增。
- 同一 register 相邻 RW 与 RESERVED 行的相同 `verification_goal`、`coverage_strategy`、`coverage_strategy_mapping` 在最终 Excel 中实际纵向 merge；`TP_ID` 和 `lifecycle_status` 不 merge，两个 TP 的 traceability 保持独立。
