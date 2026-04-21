# Rebuilder：log.md

## 触发条件

以下任一情况发生时执行本重建：

- 新的训练记录被追加到 `training/records/YYYY-MM.md`
- 新的 `library/compiled/*/meta.md` 被创建
- 任何 `training/records/` 内容发生变化（如纠错追加）

## 输入

- 所有 `training/records/YYYY-MM.md` 文件中的 RECORD_BLOCK
- 所有 `library/compiled/*/meta.md` 文件的 YAML 头

## 算法

```
1. 收集训练/作废事件：
   扫描所有 training/records/YYYY-MM.md，解析每个 RECORD_BLOCK：

   record_status=committed，supersedes_training_id=null：
     → type=train
     → 时间戳从 training_id 解出（格式：<id>-<YYYYMMDD>-<HHMMSS> → YYYY-MM-DD HH:MM:SS）
     → 行格式：## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality>

   record_status=committed，supersedes_training_id≠null：
     → type=train（带 supersedes 后缀）
     → 时间戳从新记录的 training_id 解出
     → 行格式：## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality> | supersedes:<old_id>

   record_status=voided：
     → type=void
     → 时间戳从该记录 training_id 解出
     → 行格式：## [YYYY-MM-DD HH:MM:SS] void | <training_id>

2. 收集编译事件：
   扫描所有 library/compiled/*/meta.md，从 YAML 头读取：
   compiled_date / source_id / source_title / framework_id
   → type=compile，时间戳=compiled_date（日级，只到日）
   → 行格式：## [YYYY-MM-DD] compile | <source_id>（<source_title>）→ <framework_id>

3. quality 展示层计算（仅用于 log 行，不落盘）：
   path_correct=true  AND judgment_specific=true  → 完整
   path_correct=true  XOR judgment_specific=true  → 部分
   path_correct=false AND judgment_specific=false → 形式

4. 合并排序：
   主排序：时间戳升序
   同时间戳优先级：compile > train > void
   同类型相同时间戳：按 framework_id 字母序（void 事件的 framework_id 从 training_id 前缀解出）

5. 全量写入 log.md
```

## 输出格式

```markdown
# 操作审计日志

来源：`training/records/` + `library/compiled/*/meta.md` 全量重建

> 派生文件，不参与任何计算，仅用于人类可读审计。
> 重建算法见 `rebuilders/rebuild-log.md`。

---

## [YYYY-MM-DD] compile | <source_id>（<source_title>）→ <framework_id>
## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality>
## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality> | supersedes:<old_id>
## [YYYY-MM-DD HH:MM:SS] void | <training_id>
```

## 幂等性

相同的 records 数据集 + 相同的 meta.md 数据集 → 相同的 `log.md` 输出，完全确定，可随时验证。
