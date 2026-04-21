# library · 生产侧

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
