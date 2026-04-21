# 训练协议 v1.0

## 协议目标

不是测试 Solomon 是否「知道」框架，而是训练在真实情境中的**自动触发**能力。

衡量标准：能在没有提示的情况下，从真实事件推导出框架并给出具体判断。

---

## 会话结构

### 第0层：裸触发

Mayor 呈现事件，不含任何框架提示：

> 「情境：[事件描述]。你注意到什么？」

**等待规则**：
- 等 Solomon 一次回应
- 直接说出正确框架且推导路径清晰 → `training_level=0`，进入「走通后」
- 走偏、卡住、沉默，或说不出中间推导步骤 → 进入第1层解锁流程

### 第1层：维度提示

Solomon **明确表示卡住**（"不知道""给个提示"）或经过一次明显错误尝试后，Mayor 解锁：

> 「提示：考虑 [思考维度]」

维度提示要求：
- 只提示思考方向，不暴露框架名或触发词
- 例（guns-germs）："考虑这个差距的时间尺度" ✓
- 例（guns-germs）："用历史地理框架分析" ✗

Solomon 在第1层走通 → `training_level=1`，进入「走通后」

### 第2层：节点提示

第1层提示后仍卡住，Mayor 解锁更具体的诊断入口：

> 「进一步提示：[框架 DIAGNOSTIC_ENTRY 的第一步问题]」

节点提示要求：
- 对应框架文件 `诊断入口` 中的第一个问题
- 仍不暴露框架名

Solomon 在第2层走通 → `training_level=2`，进入「走通后」

### 第3层：触发句

第2层后仍卡住，Mayor 解锁完整触发句，要求 Solomon 跟着触发句走完推导：

> 「触发句：[框架文件中的 TRIGGER_SENTENCE]。请按这个路径，对这个具体事件走一遍。」

Solomon 在第3层走通 → `training_level=3`，进入「走通后」

---

## 走通后

Mayor 要求 Solomon 输出三件事：

> 「请说出：① 这个事件触发了什么框架 ② 推导路径（3步以内）③ 对这个具体事件的判断」

然后 Mayor 评估 `path_correct` 和 `judgment_specific`。

---

## 走通的判定标准

### path_correct（须同时满足两条）

1. 能说出事件到框架之间至少一个中间推导步骤（不是直接跳到框架名）
2. 中间步骤在逻辑上确实连接了事件特征和框架的诊断入口

### judgment_specific（须同时满足两条）

1. 输出包含关于这个具体事件的结论（不是框架的一般描述）
2. 结论中有可以被后续事实验证的具体表述

### quality 展示层（不落盘，Mayor 展示时用）

```
path_correct=true  AND judgment_specific=true  → 完整
path_correct=true  XOR judgment_specific=true  → 部分
path_correct=false AND judgment_specific=false → 形式
```

---

## Mayor 现场评估清单

每次 Solomon 完成「走通后」输出，Mayor 在内存中逐项勾选，全部确认后再填 RECORD_BLOCK：

```
path_correct 判定（须同时勾两项）：
  ☐ Solomon 说出了至少一个中间推导步骤（不是直接跳到框架名）
  ☐ 该步骤在逻辑上确实连接了事件特征和框架诊断入口

judgment_specific 判定（须同时勾两项）：
  ☐ 结论包含关于这个具体事件的内容（不是框架的一般描述）
  ☐ 结论中有可被后续事实验证的具体表述

quality 展示（不落盘）：
  path_correct=T AND judgment_specific=T → 完整
  path_correct=T XOR judgment_specific=T → 部分
  path_correct=F AND judgment_specific=F → 形式

不允许只凭"感觉差不多"判定 path_correct=true——必须能明确说出那个中间步骤是什么。
不允许只凭"提到框架名"判定 judgment_specific=true——必须有具体事件的具体结论。
```

---

## event_type 判定规则

**六类**（按框架激活路径的主要机制分类，不按事件表面话题）：

```
policy       政策/监管/制度表态
geopolitical 地缘/外交/国际关系
social       社会现象/文化/人口
market       市场/价格/产业现象
tech         技术趋势/行业动态
historical   历史分析/回顾
```

跨类无法区分时按优先级：`policy > geopolitical > social > market > tech > historical`

---

## 事件选择原则

- 要求 2-3 步推导才能命中框架，禁止直接命中
- 鼓励跨 event_type 多样化（避免框架只在单一类型上激活）
- 历史事件和当代新闻都是有效材料

---

## 失效处理

**第3层走通但 path_correct=false**：
- Mayor 指出具体的推导断点
- `judgment_specific` 按实际评估
- 记录在案，不计入「完整」内化

**stuck_streak >= 3（连续3次 training_level=3）**：
- Mayor 提示：「框架 [id] 已连续3次卡在第3层，等级强制降至 L0。建议重新熟悉核心机制和诊断入口。」
- 落盘后 [C] 字段将反映 L0

---

## Mayor 的反馈规范

- `path_correct=false`：直说「推导链在哪一步断了」，不说「很接近了」
- `judgment_specific=false`：直说「你的结论是框架定义，不是对这个事件的具体判断，重说」
- 训练_level=0：可以说「直接命中」，不夸奖
