# cognitive-os · 个人认知操作系统

## 是什么

把分析框架从「我知道它」训练到「它会在对的时候自动浮现」的训练系统。

## 快速入口

| 目标 | 去哪里 |
|------|--------|
| 开始训练 | 用 `training/MAYOR.md` 作为 AI 系统提示，新建会话 |
| 查看框架列表 | `frameworks/index.md` |
| 查看已编译书目 | `library/index.md` |
| 查看操作历史 | `log.md` |
| 了解系统规则 | `AGENTS.md` |
| 了解系统哲学 | `SOUL.md` |
| 重建派生文件 | `rebuilders/` |

## 目录结构

```
cognitive-os/
├── SOUL.md                    系统哲学
├── AGENTS.md                  字段归属、落盘规则、幂等规则
├── README.md                  本文件
├── log.md                     操作审计日志（派生）
│
├── library/                   生产侧
│   ├── index.md               编译产物导航（[D]派生）+ 待编译队列（[M]人工维护）
│   ├── raw/                   原始书籍/文章
│   └── compiled/              编译产物（按 source_id 组织）
│
├── frameworks/                框架注册表
│   ├── index.md               Mayor 快速检索表（派生）
│   ├── finite-infinite.md
│   ├── guns-germs.md
│   ├── shen-reading.md
│   ├── solomon-critical.md
│   ├── solomon-tools.md
│   └── wang-shiran.md
│
├── training/                  训练侧
│   ├── MAYOR.md               教练 AI 系统提示词
│   ├── PROTOCOL.md            训练协议
│   └── records/               训练记录（唯一权威来源）
│
├── rebuilders/                派生文件重建规则（AI Native 核心）
│   ├── rebuild-frameworks-index.md
│   ├── rebuild-library-index.md
│   └── rebuild-log.md
│
└── jobs/                      任务队列（Mayor 调度用，Phase 2 启用）
```

## 当前框架

6 个框架，guns-germs 已完成第 1 次训练（L?），其余 L?（未训练）：

| 框架ID | 触发域 | 优先级 |
|--------|--------|--------|
| finite-infinite | 竞争/胜负/策略框架下的选择 | ★★★ |
| guns-germs | 发展差距叙事/文明优劣 | ★★★ |
| shen-reading | 知识工作/学习方法 | ★★★ |
| solomon-critical | 话语体系书/权力-知识文本 | ★★★ |
| solomon-tools | 工具书/可操作方法 | ★★★ |
| wang-shiran | 制度/社会行为可重复出现 | ★★★ |

## 开始训练

1. 打开 `training/MAYOR.md`，将全文作为 AI 系统提示
2. 新建对话，说「开始训练」
3. Mayor 会按三级 tie-break 选框架（sessions_total → 最近训练 → 字母序），选一个真实事件，执行分层提示协议
4. 训练结束后确认落盘，检查 `frameworks/index.md` 是否正确更新
