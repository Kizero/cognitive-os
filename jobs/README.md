# jobs · 任务队列

**状态：占位（Phase 2 启用）**

## 用途

Mayor 读取此目录来调度任务，而不是每次临场决定训练什么。这是 AI Native 的核心：Mayor 消费显式的任务对象，而不是依赖提示词里的隐性规则。

## Phase 2 将支持的任务类型

```
compile <source_id>        编译新书：生成 step 文件 + meta.md + 框架文件
train <framework_id>       训练指定框架（跳过 Mayor 自动选择）
review <YYYY-MM>           生成月度复盘报告
rebuild <target>           强制重建派生文件（frameworks-index|library-index|log）
```

## 任务文件格式（Phase 2 设计，供参考）

```yaml
---
job_id: train-wang-shiran-20260417
job_type: train
target: wang-shiran
priority: high
created_at: 2026-04-17
status: pending
---
```

## 当前阶段

此目录为空。Mayor 直接从 `frameworks/index.md` 调度训练（选优先级最高的框架）。
