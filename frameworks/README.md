# frameworks · 框架注册表

每个分析框架一个文件：`<framework_id>.md`

## 快速检索

见 `index.md`（派生文件，从所有框架文件全量重建，触发条件见 `rebuilders/rebuild-frameworks-index.md`）。

## 框架类型

| 类型 | 含义 |
|------|------|
| `compiled` | 有对应 compiled artifact，[G] 字段由框架蒸馏流水线生成 |
| `bootstrap` | 无对应 compiled artifact，[B] 字段人工填写，可永久保持此状态 |

## 字段规则

见 `AGENTS.md` 第一节（字段归属规则）。

## 当前框架

| 框架ID | 类型 | 状态 |
|--------|------|------|
| guns-germs | compiled | L? |
| wang-shiran | bootstrap | L? |
| shen-reading | bootstrap | L? |
