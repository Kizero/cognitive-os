# Rebuilder：library/index.md

## 触发条件

以下任一情况发生时执行本重建（仅 [D] 区）：

- 新的 `library/compiled/<source_id>/` 目录被创建
- 任何 `library/compiled/*/meta.md` 的内容发生变化

[M] 区（待编译队列）永远不被重建覆盖，只由人工维护。

## 输入

- [D] 区来源：所有 `library/compiled/*/meta.md`
- [M] 区来源：当前 `library/index.md` 的 [M] 区内容（原样保留）

## 算法

```
1. 读取当前 library/index.md，提取 [M] 区域内容
   识别方式：<!-- [M] 以下为人工维护区，重建程序不修改 --> 标记行
   到文件末尾（含 <!-- [M] 结束 --> 行）

2. 扫描所有 library/compiled/*/meta.md，提取：
   - source_id        → YAML 头 source_id 字段
   - 书名/文章        → YAML 头 source_title 字段
   - 类型             → YAML 头 source_type 字段
   - 编译日期         → YAML 头 compiled_date 字段
   - 框架ID           → YAML 头 framework_id 字段
   - 核心机制         → EXTRACTION_BLOCK.SHORT_MECHANISM
   - 路径             → 拼接：library/compiled/<source_id>/

3. 生成新的 [D] 区内容（按 source_id 字母序排列）

4. 输出 = [D] 区（新生成）+ [M] 区（原样保留）
   全量写入 library/index.md
```

## 输出格式

```markdown
# 编译产物索引

更新时间：[YYYY-MM-DD]

<!-- [D] 以下由重建程序全量替换，请勿手动编辑 -->
## 已编译

| source_id | 书名/文章 | 类型 | 编译日期 | 框架ID | 核心机制 | 路径 |
|-----------|---------|------|---------|--------|---------|------|
| [source_id] | [title] | [type] | [date] | [framework_id] | [SHORT_MECHANISM] | library/compiled/[source_id]/ |
<!-- [D] 结束 -->

<!-- [M] 以下为人工维护区，重建程序不修改 -->
## 待编译队列

[原样保留 [M] 区内容]
<!-- [M] 结束 -->
```

## 幂等性

相同的 `compiled/*/meta.md` 内容集合 + 相同的 [M] 区内容 → 相同的 `library/index.md` 输出。
