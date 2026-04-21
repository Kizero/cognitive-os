# training/records · 训练记录

## 规则

- 每个月一个文件：`YYYY-MM.md`
- **只追加，不修改**
- 发现错误：追加新记录（使用 supersedes_training_id），不修改原记录
- 任何不一致时，records 赢

## 这个目录是什么

训练记录是系统的核心权威来源（Source of Truth）：

- `frameworks/*.md` 的 [C] 字段（内化状态）从这里全量重算
- `log.md` 从这里重建
- effective records 的计算定义见 `AGENTS.md` 第六节

## RECORD_BLOCK 字段说明

```
training_id              <framework_id>-<YYYYMMDD>-<HHMMSS>，秒级，零碰撞
framework_id             框架ID
event_type               policy|geopolitical|social|market|tech|historical
source_ref               news|book|article|history|other
training_level           0-3（0=裸触发走通，3=看了触发句才走通）
hint_level1_used         true|false
hint_level2_used         true|false
hint_level3_used         true|false
path_correct             true|false（推导路径是否正确，见 PROTOCOL.md）
judgment_specific        true|false（判断是否具体，见 PROTOCOL.md）
record_status            committed|voided
supersedes_training_id   null 或被替换的 training_id
```

## quality（展示层计算，不落盘）

```
path_correct=true  AND judgment_specific=true  → 完整
path_correct=true  XOR judgment_specific=true  → 部分
path_correct=false AND judgment_specific=false → 形式
```
