# cognitive-os · 系统哲学

## 核心问题

> 读了很多书、建了很多分析框架，但现实中遇到问题时根本想不起来用。知识停留在文件里，不会在真实情境里自动浮现。

## 这个系统在解决什么

不是知识管理，不是笔记系统。是一个**训练系统**：把分析框架从「我知道它」训练到「它会在对的时候自动浮现」。

## 两条流水线

**生产侧**：书/文章 → 沈老师引擎编译 → EXTRACTION_BLOCK → 结构化框架文件

**训练侧**：真实事件触发 → 分层提示训练 → internalization_level 提升

两条流水线共享同一个文件系统，不是两个独立工具。

## 成功长什么样

不是「我有50个框架」，而是「我在某个对话里，不自觉地用了 guns-germs，对方停了一下说'对，就是这个问题'」。

## AI Native 原则

这个系统的第一公民是 **AI 可操作的状态、任务和重建规则**：

- 状态存在结构块（EXTRACTION_BLOCK、RECORD_BLOCK、内化状态）里，不在正文
- 任务存在 `jobs/` 里，Mayor 读队列而不是靠临场发挥
- 重建规则存在 `rebuilders/` 里，AI 不需要翻 plan 就知道怎么重建任何派生文件
- 人类可读的正文是派生物；结构块才是权威来源

## 级联规则

```
training/records/YYYY-MM.md          唯一权威来源，只追加
library/compiled/*/meta.md           编译产物权威来源
frameworks/<id>.md [G/B/C字段]       由 meta.md 和 records 驱动
frameworks/index.md                  派生，从 frameworks/*.md 重建
library/index.md [D区]               派生，从 compiled/*/meta.md 重建
log.md                               派生，从 records + meta.md 重建
```
