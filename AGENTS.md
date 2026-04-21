# AGENTS · 项目规则

## 一、字段归属规则

```
[G] Generated   由 EXTRACTION_BLOCK 生成，重编译时可覆盖
[M] Manual      人工维护，编译流水线禁止修改
[C] Computed    从 records 计算，任何人和流水线都不直接写入
[B] Bootstrap   bootstrap 框架的手工填入值，无 compiled artifact 支撑
                （bootstrap 毕业后 [B] → [G]）
```

### [C] 字段的唯一写入路径

1. 扫描该框架的所有 effective records
2. 按判定链计算 internalization_level
3. 全量替换 `frameworks/<id>.md` 的内化状态 section

任何其他路径写入 [C] 字段均为违规。

---

## 二、落盘规则

### training/records/YYYY-MM.md

- **只追加，不修改**
- 每条记录含完整 RECORD_BLOCK
- 发现错误：追加新记录（使用 supersedes_training_id），不修改原记录

### frameworks/<id>.md

- [G] 字段：框架蒸馏流水线写入，重编译时可覆盖
- [M] 字段：人工写入，编译流水线禁止修改
- [C] 字段：仅通过「从 records 全量重算」路径写入

### 派生文件（全量重建，不增量）

- `frameworks/index.md`：从 frameworks/*.md 全量重建
- `library/index.md [D区]`：从 library/compiled/*/meta.md 全量重建（[M区]原样保留）
- `log.md`：从 training/records/ + library/compiled/*/meta.md 全量重建

---

## 三、幂等规则

### 训练落盘幂等

落盘前检查 training_id 是否已存在于 records：
- 已存在 → 跳过追加 records，直接重建派生文件（步骤2-4），跳过 log 重建（步骤5）
- 不存在 → 正常执行全部步骤

### 重建幂等

相同的 canonical state（records + meta.md 集合）→ 重建结果完全确定，无随机性。

---

## 四、Bootstrap 框架规则

**定义**：没有对应 compiled artifact 的框架，[G] 字段由人工填写并标注 [B]。

**bootstrap 不是临时状态**：没有对应原始材料的框架可以永久保持 bootstrap 状态。

**毕业流程**（当 bootstrap 框架对应的材料被正式编译后）：

1. 确认 `library/compiled/<source_id>/meta.md` 已生成且 EXTRACTION_BLOCK 完整
2. 按 `library/DISTILL.md` **模式B** 执行：[G] 覆盖 [B]，`framework_type: bootstrap → compiled`
3. **[M] 字段**（训练补充触发）：原样保留，不得修改
4. **[C] 字段**（内化状态）：原样保留，训练连续性不中断

---

## 五、重建触发规则

重建算法见 `rebuilders/` 目录：

| 派生文件 | 重建算法 |
|---------|---------|
| frameworks/index.md | rebuilders/rebuild-frameworks-index.md |
| library/index.md [D区] | rebuilders/rebuild-library-index.md |
| log.md | rebuilders/rebuild-log.md |

---

## 六、effective records 定义

```python
voided_ids = {
    r.supersedes_training_id
    for r in all_records
    if r.supersedes_training_id is not None
}
effective = [
    r for r in all_records
    if r.record_status == "committed"
    and r.training_id not in voided_ids
]
```

---

## 七、internalization_level 判定链

```
统计窗口：最近5次 effective records
          sessions_total < 3 → L?

变量：
  avg_level        = 最近5次的 training_level 平均值
  complete_ratio   = 最近5次中 (path_correct AND judgment_specific) 的比例
  stuck_streak     = 末尾连续 training_level==3 的次数
  zero_event_types = 历史有效记录中 training_level==0 的不同 event_type 数量

判定链（严格按序）：
  if sessions_total < 3:                                   → L?
  elif stuck_streak >= 3:                                  → L0（强制降级）
  elif avg_level < 0.5
       and complete_ratio >= 0.8
       and zero_event_types >= 3:                          → L3
  elif avg_level < 1.5
       and complete_ratio >= 0.6:                          → L2
  elif avg_level <= 2.5
       and complete_ratio >= 0.3:                          → L1
  else:                                                    → L0
```

### 辅助字段计算规则（落盘时随 [C] 区一并写入）

```
sessions_total      = effective records 的总数

avg_level           = 最近5次 effective records 的 training_level 平均值
                      保留两位小数（Python: round(x, 2)）
                      sessions_total < 5 时取全部有效记录

complete_ratio      = 最近5次中 (path_correct=true AND judgment_specific=true) 的比例
                      保留两位小数
                      sessions_total < 5 时取全部有效记录

stuck_streak        = 末尾连续 training_level==3 的次数

zero_level_event_types = 历史有效记录中 training_level==0 的不同 event_type 列表
                         （落盘格式：["type1", "type2"] 或 []）

最近训练            = 最近一条 effective record 的 training_id 中解出的日期
                      training_id 格式：<framework_id>-<YYYYMMDD>-<HHMMSS>
                      → 提取 <YYYYMMDD> 段，格式化为 YYYY-MM-DD
                      （按 training_id 字典序取最大值，即时间最晚的那条）

变化趋势            = 纯从 records 计算，不读取文件当前状态：
                      level_current  = 用全部 effective records 按判定链算出的等级
                      level_previous = 去掉 training_id 最大的那条 effective record 后
                                       按同一判定链算出的等级（effective records 为空时 → L?）
                      level_current > level_previous → 上升
                      level_current < level_previous → 下降
                      level_current = level_previous → 持平
                      level_current = L?（即 sessions_total < 3）→ 初始
                      等级数值映射：L? → -1，L0 → 0，L1 → 1，L2 → 2，L3 → 3
```

### 空值规则（sessions_total = 0 时的初始状态）

```
当前等级:           L?
sessions_total:     0
avg_level:          -
complete_ratio:     -
stuck_streak:       0
zero_level_event_types: []
最近训练:           -
变化趋势:           初始
```

---

## 八、框架蒸馏流水线

**完整算法**：`library/DISTILL.md`

输入：`library/compiled/<source_id>/meta.md` 全文（YAML header + EXTRACTION_BLOCK）

输出：`frameworks/<framework_id>.md`

三种模式：
- **模式A（新建）**：[G] 字段全量生成，[M] 留空，[C] 初始化为空值规则默认值，标题取 `framework_display_name`
- **模式B（Bootstrap毕业）**：[G] 覆盖 [B]，标题/[M]/[C] 原样保留，`framework_type` 改为 `compiled`
- **模式C（重编译）**：[G] 覆盖原 [G]，标题/[M]/[C] 原样保留，`framework_type` 保持 `compiled`

字段映射（快速参考；详见 DISTILL.md）：

| EXTRACTION_BLOCK 字段 | 框架文件字段 |
|----------------------|------------|
| CORE_MECHANISM | 核心机制（一句话）[G] |
| SHORT_MECHANISM | SHORT_MECHANISM [G] |
| TRIGGER_SENTENCE | 一句话总结（第3层触发句）[G] |
| SHORT_TRIGGER | SHORT_TRIGGER [G] |
| PRIMARY_HOOKS | 主触发 [G] |
| SECONDARY_HOOKS | 次触发 [G] |
| DIAGNOSTIC_ENTRY | 诊断入口 [G] |
| FAILURE_BOUNDARIES | 失效边界 [G] |
| PAIRED_FRAMEWORK | 配对框架 [G] |
