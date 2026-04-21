# Rebuilder：frameworks/index.md

## 触发条件

以下任一情况发生时执行本重建：

- 任何 `frameworks/*.md` 的内化状态 [C] 发生变化（训练落盘后）
- 任何 `frameworks/*.md` 的 SHORT_TRIGGER 或 SHORT_MECHANISM 发生变化（包括：bootstrap初次填写、重编译覆盖[G]字段、bootstrap毕业）
- 新框架文件被创建或删除

## 输入

所有 `frameworks/*.md`（排除 `index.md` 自身和 `README.md`）

## 算法

```
1. 扫描所有 frameworks/*.md（排除 index.md、README.md）

2. 对每个框架文件，提取：
   - framework_id    → 从 **框架ID** 字段读取
   - 触发域          → SHORT_TRIGGER 字段值
   - 一句话          → SHORT_MECHANISM 字段值
   - 等级            → 内化状态 section 的 **当前等级** 字段
   - 最近训练        → 内化状态 section 的 **最近训练** 字段（无记录时为 -）
   - 优先级          → 按以下规则计算：
       L? 或 L0  → ★★★
       L1        → ★★
       L2 或 L3  → ★

3. 按 framework_id 字母序排列

4. 全量替换 frameworks/index.md
```

## 输出格式

```markdown
# 框架快速检索表

更新时间：[YYYY-MM-DD HH:MM]
来源：frameworks/*.md 全量重建

| 框架ID | 触发域 | 一句话 | 等级 | 最近训练 | 优先级 |
|--------|--------|--------|------|---------|--------|
| [id] | [SHORT_TRIGGER] | [SHORT_MECHANISM] | [等级] | [日期|-] | [★] |

优先级含义：
  ★★★ L? 或 L0（优先安排训练）
  ★★  L1（维持训练频率）
  ★   L2-L3（偶尔激活即可）
```

## 幂等性

相同的 `frameworks/*.md` 内容集合 → 相同的 `index.md` 输出，无随机性。
