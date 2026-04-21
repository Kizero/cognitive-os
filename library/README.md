# library · 生产侧

## 编译引擎选择

**默认引擎**：首次编译书籍/文章时，优先使用 `shen-reading`（沈老师读书建模法）作为引擎。

**原因**：shen-reading 是方法论框架（bootstrap 类型），提供了完整的结构化分析流程（Pre-Step → Step 0-5），能系统性地从材料中提取可复用的框架。

**引擎扩展原则**（这块是活的）：

| 场景 | 推荐引擎 | 说明 |
|------|---------|------|
| 首次编译任何材料 | shen-reading | 默认选择，结构化流程最完整 |
| 管理学/组织行为书籍 | wang-shiran | 制度分析视角，找根规则 |
| 话语体系/权力-知识文本 | solomon-critical | 话语解构视角 |
| 工具书/方法论书 | solomon-tools | 操作性提取视角 |
| 竞争/策略叙事 | finite-infinite | 游戏框架视角 |

**扩展触发条件**：
- 当框架库足够丰富时（≥5 个 compiled 框架且至少 L1 等级）
- 可根据材料类型选择已有框架作为解读引擎
- meta.md 的 `engine_version` 字段记录实际使用的引擎及版本

**多引擎联用**：同一材料可用多个引擎分别编译，产出多个视角的 EXTRACTION_BLOCK（存放在不同的 compiled 目录，如 `<source_id>-wang`、`<source_id>-solomon`）。

---

## 目录结构

```
library/
├── README.md         本文件
├── index.md          编译产物导航（[D]派生）+ 待编译队列（[M]人工维护）
├── raw/              原始书籍/文章存档
└── compiled/         沈老师引擎编译输出（按 source_id 组织）
    └── <source_id>/
        ├── meta.md             必须（含 EXTRACTION_BLOCK）
        ├── step0-step1.md      必须
        ├── step2.md            必须
        ├── step3-step4.md      必须
        └── step5-final.md      必须
```

## 添加新书的流程

1. 将原始文件放入 `raw/`
2. 用沈老师引擎 v3.4 完整跑一遍
3. 输出 5 个 step 文件到 `compiled/<source_id>/`
4. 撰写 `compiled/<source_id>/meta.md`（含完整 EXTRACTION_BLOCK，包括 SHORT_MECHANISM 和 SHORT_TRIGGER；YAML header 必须含 `framework_display_name`——框架文件标题行，可与 `source_title` 不同）
5. 按 `library/DISTILL.md` 执行框架蒸馏（输入 meta.md 全文）→ `frameworks/<framework_id>.md`
6. 重建 `frameworks/index.md`（见 `rebuilders/rebuild-frameworks-index.md`）
7. 重建 `library/index.md`（见 `rebuilders/rebuild-library-index.md`）
8. 重建 `log.md`（见 `rebuilders/rebuild-log.md`）
