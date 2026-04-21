# 框架蒸馏 Prompt

## 用途

将 `library/compiled/<source_id>/meta.md` 中的 EXTRACTION_BLOCK 蒸馏为 `frameworks/<framework_id>.md`。

**输入**：meta.md 全文（YAML header + EXTRACTION_BLOCK）
**输出**：完整的 `frameworks/<framework_id>.md`

---

## 编译引擎选择

**首次编译默认使用 `shen-reading` 引擎**。详见 `library/README.md`。

meta.md 的 YAML header 中 `engine_version` 字段记录实际使用的引擎及版本（如 `shen-v3.4`）。

---

## 运行模式

### 模式 A：新建框架文件

目标路径 `frameworks/<framework_id>.md` 不存在。

- 全量生成所有 [G] 字段
- 文件标题（`# …`）使用 `framework_display_name`（YAML header 必填字段，与 `source_title` 可以不同）
- [M] 字段（训练补充触发）留空白行
- [C] 区按空值规则初始化

### 模式 B：Bootstrap 框架毕业

目标路径已存在，且 `framework_type: bootstrap`。

- 用 [G] 字段**逐段覆盖**原 [B] 字段
- `framework_type` 从 `bootstrap` 改为 `compiled`
- **文件标题（`# …`）**：原样保留，不修改（框架名 ≠ source_title 时尤其重要）
- **[M] 字段**（训练补充触发）：原样保留，一字不改
- **[C] 字段**（内化状态 section 全部内容）：原样保留，一字不改

### 模式 C：重编译已存在的 compiled 框架

目标路径已存在，且 `framework_type: compiled`。

触发场景：EXTRACTION_BLOCK 内容有修正、重新编译同一材料。

- 用新 [G] 字段**逐段覆盖**原 [G] 字段
- `framework_type` 保持 `compiled`
- **文件标题（`# …`）**：原样保留，不修改
- **[M] 字段**（训练补充触发）：原样保留，一字不改
- **[C] 字段**（内化状态 section 全部内容）：原样保留，一字不改

---

## 字段来源映射

### 文件头部（标题 + 元数据行）

| 目标字段 | 来源 | 推导规则 |
|---------|------|---------|
| 文件标题（`# …`） | 模式A：YAML → `framework_display_name`；模式B/C：原文件标题 | 见运行模式 |
| `框架ID` | YAML → `framework_id` | 直接复制 |
| `来源` | YAML → `source_title` | 直接复制 |
| `framework_type` | 模式A/C：`compiled`；模式B：`bootstrap → compiled` | — |
| `compiled_source_id` | YAML → `source_id` | 直接复制 |
| `compiled路径` | YAML → `source_id` | `library/compiled/<source_id>/` |
| `建立时间` | YAML → `compiled_date` | 取前7字符（`YYYY-MM`格式） |

所有元数据行右侧标注 `[G]`，对齐到第70列（用空格补齐）。

### 短标签

| 目标字段 | 来源 |
|---------|------|
| `SHORT_TRIGGER` | EXTRACTION → `SHORT_TRIGGER` |
| `SHORT_MECHANISM` | EXTRACTION → `SHORT_MECHANISM` |

两行均标注 `[G]`，对齐到第70列。

### Hook 注册

| 目标字段 | 来源 | 格式规则 |
|---------|------|---------|
| `主触发 [G]` | EXTRACTION → `PRIMARY_HOOKS` | 列表逐行展开为 `- 内容` |
| `次触发 [G]` | EXTRACTION → `SECONDARY_HOOKS` | 列表逐行展开为 `- 内容` |
| `训练补充触发 [M]` | — | 模式A：空白行；模式B/C：原样保留 |

### 内容区

| 目标字段 | 来源 | 格式规则 |
|---------|------|---------|
| `核心机制（一句话）[G]` | EXTRACTION → `CORE_MECHANISM` | 单段，去引号 |
| `诊断入口 [G]` | EXTRACTION → `DIAGNOSTIC_ENTRY` | 多行原样，行首 `→` 换为 `- ` |
| `一句话总结（第3层触发句）[G]` | EXTRACTION → `TRIGGER_SENTENCE` | 单段，去引号 |
| `失效边界 [G]` | EXTRACTION → `FAILURE_BOUNDARIES` | 列表逐行展开为 `- 内容`，去引号 |
| `配对框架 [G]` | EXTRACTION → `PAIRED_FRAMEWORK` | 单段，去引号 |

`DIAGNOSTIC_ENTRY` 格式规范：
- EXTRACTION_BLOCK 中 YAML `|` 块写法保留原有缩进和换行
- 行首的 `→ ` 在输出时统一替换为 `- `
- 行内的 `→` 保留不变

### 内化状态 [C]

**模式A（新建）**：按空值规则初始化：

```
**当前等级**: L?
**sessions_total**: 0
**avg_level**: -
**complete_ratio**: -
**stuck_streak**: 0
**zero_level_event_types**: []
**最近训练**: -
**变化趋势**: 初始
```

**模式B/C**：原样保留现有 [C] section，不修改任何字段。

---

## 输出模板

以下为模式A的完整文件结构（`<PLACEHOLDER>` 为待填入值）：

```markdown
# <framework_display_name>

**框架ID**: <framework_id>                                            [G]
**来源**: <source_title>                                              [G]
**framework_type**: compiled                                           [G]
**compiled_source_id**: <source_id>                                   [G]
**compiled路径**: library/compiled/<source_id>/                       [G]
**建立时间**: <YYYY-MM>                                               [G]

---

## 短标签（用于index快速检索）

**SHORT_TRIGGER**: <SHORT_TRIGGER>                                    [G]
**SHORT_MECHANISM**: <SHORT_MECHANISM>                                [G]

---

## Hook注册

### 主触发 [G]
- <PRIMARY_HOOKS[0]>
- <PRIMARY_HOOKS[1]>
...

### 次触发 [G]
- <SECONDARY_HOOKS[0]>
...

### 训练补充触发 [M]

---

## 核心机制（一句话）[G]

<CORE_MECHANISM>

---

## 诊断入口 [G]

<DIAGNOSTIC_ENTRY（多行，行首→换-）>

---

## 一句话总结（第3层触发句）[G]

<TRIGGER_SENTENCE>

---

## 失效边界 [G]

- <FAILURE_BOUNDARIES[0]>
- <FAILURE_BOUNDARIES[1]>
...

---

## 配对框架 [G]

<PAIRED_FRAMEWORK>

---

## 内化状态 [C]

**当前等级**: L?
**sessions_total**: 0
**avg_level**: -
**complete_ratio**: -
**stuck_streak**: 0
**zero_level_event_types**: []
**最近训练**: -
**变化趋势**: 初始
```

模式B/C：文件标题行、[M] section、[C] section 全部从原文件原样复制，其余 [G] section 按模板替换。

---

## 执行步骤

1. 读取 meta.md，提取 YAML header 和 EXTRACTION_BLOCK
2. 判断运行模式：
   - `frameworks/<framework_id>.md` 不存在 → 模式A
   - 存在且 `framework_type: bootstrap` → 模式B
   - 存在且 `framework_type: compiled` → 模式C
3. 按字段映射表填入所有 [G] 字段
4. 按模式处理不可变区：
   - 模式A：[M] 空行，[C] 空值规则初始化，标题取 `framework_display_name`
   - 模式B：标题行、[M]、[C] 从原文件原样复制
   - 模式C：标题行、[M]、[C] 从原文件原样复制
5. 写入目标文件

---

## 验证清单（写入后核对）

```
模式A：
  ☐ 文件标题 = framework_display_name（不是 source_title）
  ☐ [C] section = 空值规则默认值
  ☐ [M]（训练补充触发）= 空白行

模式B：
  ☐ 文件标题与原 bootstrap 文件一致（未被替换）
  ☐ framework_type = compiled（已从 bootstrap 升级）
  ☐ [M]（训练补充触发）= 原值不变
  ☐ [C] section = 原值不变（训练数据连续）

模式C：
  ☐ 文件标题与原文件一致（未被替换）
  ☐ framework_type = compiled（保持不变）
  ☐ [M]（训练补充触发）= 原值不变
  ☐ [C] section = 原值不变

所有模式：
  ☐ 所有元数据行标注 [G]
  ☐ SHORT_TRIGGER / SHORT_MECHANISM 内容与 EXTRACTION 一致
  ☐ 主触发 / 次触发 条目数量与 EXTRACTION 一致
  ☐ DIAGNOSTIC_ENTRY 多行结构完整，行首→已换-
  ☐ FAILURE_BOUNDARIES 条目数量与 EXTRACTION 一致
```
