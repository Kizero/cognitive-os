# cognitive-os Plan v4 · 多引擎编译层

## 一、这份计划解决什么问题

当前系统已经有两层东西，但还没有被明确拆开：

- **编译引擎**：决定一本书/文章应该怎么读、怎么拆、怎么输出 step 文件
- **内容框架**：决定读完之后沉淀出什么分析框架，以及和现有框架如何互补/冲突/同构

现在生产侧实际上还是**单引擎**：

```
书/文章
  → shen-reading / shen-v3.4
  → compiled/<source_id>/step0-step1 ~ step5-final
  → EXTRACTION_BLOCK
  → frameworks/<framework_id>.md
```

这有两个限制：

1. 随着框架库增长，`compiled/` 结果不会自动变丰富，只有 Step 5 的“接入已有体系”会变厚
2. 混合型材料（既有可操作工具，又有话语结构；既要深度建模，又要快速提取）无法被清晰表达

**v4 的目标**：把生产侧升级为**多引擎、可组合、非单选**的编译层。

---

## 二、核心判断

### 2.1 引擎不是普通框架

不是所有 framework 都应该直接拿来当编译引擎。

目前最明显的**引擎型框架**有：

- `shen-reading`：深度建模引擎
- `solomon-tools`：工具书提取引擎
- `solomon-critical`：批判性阅读引擎

目前最明显的**内容型框架**有：

- `guns-germs`
- `wang-shiran`
- `finite-infinite`

区别：

- 引擎型框架回答：**这本材料应该怎么读？**
- 内容型框架回答：**这本材料最终沉淀出了什么判断工具？**

### 2.2 一本材料可以经过多个引擎

v4 的基本立场：

> 一本书/文章不是“只能使用某 1 种引擎”，而是可以有一个主引擎 + 若干辅助引擎。

常见情况：

- 以深度建模为主，同时做批判校验
- 以工具提取为主，同时做话语批判
- 以批判清场为主，之后再用深度建模重建结构

所以 v4 不采用“单选路由”，而采用**引擎栈（engine stack）**。

---

## 三、目标状态

### 3.1 编译侧升级为“主引擎 + 辅助引擎”

每个 `library/compiled/<source_id>/meta.md` 在 v4 目标态中，除了已有字段外，新增：

```yaml
primary_engine: shen-reading | solomon-tools | solomon-critical
auxiliary_engines:
  - <framework_id>
  - <framework_id>
engine_route_reason: "<为什么这样选>"
```

说明：

- `primary_engine`：决定主骨架
- `auxiliary_engines`：在主骨架之外追加校验/提取/批判 pass
- `engine_route_reason`：记录为什么这样编，不让路由选择变成隐性心证

### 3.2 engine_version 的语义收紧

当前 `engine_version: shen-v3.4` 混合了“引擎身份”和“版本号”。

v4 推荐拆成：

```yaml
primary_engine: shen-reading
primary_engine_version: shen-v3.4
auxiliary_engines:
  - solomon-critical
```

这样以后不会出现：

- 主引擎明明是 `solomon-tools`
- 结果 header 里还写 `shen-v3.4`

### 3.3 Step 文件仍保留统一落盘结构

v4 不推翻当前 `step0-step1 / step2 / step3-step4 / step5-final` 的四文件结构。

保留原因：

- 现有 compiled contract 已经稳定
- DISTILL 流水线已经基于当前结构运行
- 派生文件和人工审阅路径已经习惯这套形态

v4 只改变**每一步内部由哪些引擎参与**，不先改文件壳子。

---

## 四、引擎栈规则

### 4.1 主引擎只允许一个

主引擎负责主骨架，不允许多个主引擎并列。

原因：

- 不然 step 文件会失去主线
- 无法判断哪套方法拥有最终解释权

### 4.2 辅助引擎可以多个

辅助引擎可以 0 到多个，但必须写清楚它们分别作用在哪一段。

推荐写法：

```yaml
primary_engine: shen-reading
auxiliary_engines:
  - solomon-critical
engine_route_reason: "主体是理论框架书，需要深度建模；同时作者有强叙述位置，需在 Step 5 补一轮批判校验。"
```

### 4.3 组合不是任意乱叠，要有顺序

v4 暂定三条默认顺序规则：

1. `solomon-tools → solomon-critical`
   先提取武器，再批判话语  
   不能反过来，否则批判会先把工具夺走

2. `shen-reading → solomon-critical`
   先建立结构，再检验作者叙述、遮蔽和常数级别

3. `solomon-critical → shen-reading`
   只在“文本话语性极强，先清场才有可能重建”的材料上使用

---

## 五、三种典型路由

### 5.1 深度理论书

例：大部头理论书、哲学框架书、系统论著作

```yaml
primary_engine: shen-reading
auxiliary_engines:
  - solomon-critical
```

效果：

- `shen-reading` 负责 Step 0-4 的主骨架
- `solomon-critical` 主要在 Step 5 检查作者的立场、遮蔽和新问题

### 5.2 工具书 / 方法书

例：管理工具、流程方法、操作手册、实践指南

```yaml
primary_engine: solomon-tools
auxiliary_engines:
  - solomon-critical
```

效果：

- `solomon-tools` 负责把书变成可测试工具
- `solomon-critical` 负责检查这些工具服务于什么叙事和权力结构

### 5.3 混合型材料

例：既提供方法，又自带强烈世界观的书

```yaml
primary_engine: shen-reading
auxiliary_engines:
  - solomon-tools
  - solomon-critical
```

效果：

- `shen-reading` 保证主结构
- `solomon-tools` 提取可部署规则
- `solomon-critical` 检查叙述位置和遮蔽

---

## 六、Step 文件如何变得“更丰富”

这正是 v4 想解决的核心。

当前 compiled 的丰富度，主要靠：

- 书本身内容
- Step 5 对现有框架的接入

v4 之后，compiled 的丰富度多出一个来源：

- **同一材料被多个引擎加工**

具体表现：

### Step 0–1

- `shen-reading`：负责骨架提取、概念速览
- `solomon-tools`：补“哪部分是可部署工具”
- `solomon-critical`：补“哪些定义是叙述者预设”

### Step 2

- `shen-reading`：边界例/反例伪装的结构判断
- `solomon-tools`：工具是否真能部署
- `solomon-critical`：哪些“成功案例”其实是话语包装

### Step 3–4

- `shen-reading`：结构图和可执行模型主线
- `solomon-tools`：把规则写成测试计划
- `solomon-critical`：补“这套规则对谁有利/遮蔽谁”

### Step 5

Step 5 将成为引擎层与框架层的汇合点：

- 接入已有内容框架
- 记录引擎间冲突与顺序理由
- 给蒸馏 prompt 提供更厚的判断依据

---

## 七、v4 不做什么

为了避免范围爆炸，v4 明确**不做**以下事情：

1. 不改 `frameworks/*.md` 的字段结构
2. 不改 `DISTILL.md` 的输出 contract
3. 不改 `frameworks/index.md / library/index.md / log.md` 的重建规则
4. 不把所有 framework 都升级成 engine
5. 不在这一阶段引入“自动路由器”

v4 是**先把多引擎 contract 写清楚，再手动执行几次验证**，不是一步到位自动化。

---

## 八、实施阶段

### Phase 6：引擎层显式化

任务清单：

- [ ] 新建 `library/ENGINES.md`
- [ ] 明确定义 engine 与 framework 的区别
- [ ] 为 `shen-reading / solomon-tools / solomon-critical` 写一张引擎路由表
- [ ] 定义主引擎 / 辅助引擎 / 顺序规则
- [ ] 在生产侧文档中把“单一沈老师引擎”改写为“默认引擎 + 可组合引擎栈”

完成标志：

- 新读者能回答“哪些是编译引擎，哪些只是内容框架”
- 文档中不再把所有 compiled 都默认为单一 `shen-v3.4`

### Phase 7：meta.md 升级为多引擎 header

任务清单：

- [ ] 为 `meta.md` 增加 `primary_engine`
- [ ] 增加 `auxiliary_engines`
- [ ] 增加 `engine_route_reason`
- [ ] 规定 `primary_engine_version` 的写法
- [ ] 用现有两个 compiled 样本各补一次 header 演练

完成标志：

- 任一 compiled 目录都能从 `meta.md` 看出“这本材料为什么这样编”
- 引擎选择不再藏在 step 正文里

### Phase 8：做一次真正的混合编译验证

建议材料：

- 《国家为什么会失败》
- 或一本文字里同时有工具、理论、叙事的材料

任务清单：

- [ ] 选一份“不能只用一个引擎讲清楚”的材料
- [ ] 明确主引擎与辅助引擎
- [ ] 跑一遍完整 compiled
- [ ] 在 Step 5 明确写出“如果只用单引擎，会漏掉什么”
- [ ] 蒸馏为新框架，并重建所有派生文件

完成标志：

- 有至少一个 compiled 样本能清楚证明“多引擎 > 单引擎”
- engine stack 的顺序和作用范围可复述、可重复

---

## 九、成功标准

| 时间点 | 可观察的成功证据 |
|--------|----------------|
| Phase 6完成 | 文档里清楚区分编译引擎与内容框架 |
| Phase 7完成 | meta.md 明确记录主引擎、辅助引擎和路由理由 |
| Phase 8完成 | 至少一个 compiled 样本证明多引擎编译有额外信息增益 |

---

## 十、最小落地建议

如果只做最小版本，不一次铺太开，建议顺序是：

1. 先写 `library/ENGINES.md`
2. 再升级 `meta.md` header
3. 再挑一本真正混合型材料做验证

不要一开始就做自动路由。

先把“多引擎是怎么回事”写清楚，再让系统学会自动选。
