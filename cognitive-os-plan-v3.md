# cognitive-os · 系统设计计划 v3.0

> 状态：contracts全部locked，可进入Phase 1建设
> 版本：经过6轮CodeX review后的最终收口版本
> 用途：新会话继续建设时的完整参考文档

---

## 一、项目定位

**cognitive-os** 是一个个人认知操作系统，解决一个具体问题：

> 读了很多书、建了很多分析框架，但现实中遇到问题时根本想不起来用。知识停留在文件里，不会在真实情境里自动浮现。

系统由两条流水线构成：

- **生产侧**：书/文章 → 沈老师引擎编译 → 结构化框架
- **训练侧**：真实事件触发 → 分层提示训练 → internalization_level提升

两条流水线共享同一个文件系统，不是两个独立工具。

---

## 二、目录结构

```
cognitive-os/
│
├── SOUL.md                         # 系统哲学
├── AGENTS.md                       # 项目规则（字段归属、落盘规则）
├── README.md                       # 导航入口
├── log.md                          # 操作审计日志（派生，从records+meta全量重建）
│
├── library/                        # 生产侧
│   ├── README.md
│   ├── index.md                    # [D]编译产物导航（派生）+ [M]待编译队列（人工维护）
│   ├── raw/                        # 原始书籍/文章
│   │   └── README.md
│   └── compiled/                   # 沈老师引擎输出，目录名用source_id
│       └── <source_id>/
│           ├── meta.md             # 元信息 + EXTRACTION_BLOCK（机器可读）
│           ├── step0-step1.md
│           ├── step2.md
│           ├── step3-step4.md
│           └── step5-final.md
│
├── frameworks/                     # 框架注册表
│   ├── README.md
│   ├── index.md                    # Mayor快速检索表（派生，从frameworks/*.md全量重建）
│   └── <framework_id>.md           # 每框架一文件
│
└── training/                       # 训练侧
    ├── MAYOR.md                    # 教练AI系统提示词
    ├── PROTOCOL.md                 # 训练协议
    └── records/                    # 训练记录（唯一权威来源）
        ├── README.md
        └── YYYY-MM.md
```

---

## 三、Source of Truth 级联规则

```
training/records/YYYY-MM.md
  ↓ 唯一权威来源，只追加，不修改
  ↓ 任何不一致时，records赢

library/compiled/*/meta.md
  ↓ 编译产物权威来源，驱动[G]字段生成

frameworks/<id>.md
  ├── [G/B]字段：由meta.md/EXTRACTION_BLOCK驱动
  └── [C]字段（内化状态）：由records全量重算，禁止手工写入

frameworks/index.md
  ↓ 由frameworks/*.md全量重建（触发条件见Contract 3）

library/index.md
  ├── [D]派生区：由library/compiled/*/meta.md全量重建
  └── [M]人工区：待编译队列，本身即权威来源，重建时原样保留

log.md
  ↓ 派生，从training/records/ + library/compiled/*/meta.md全量重建
  ↓ 不参与任何计算，只用于人类可读的操作审计
```

---

## 四、Contract 1：compiled output contract

### 4.1 source_id 与 framework_id 的关系

**source_id**：标识一份原始材料（一本书、一篇文章）。目录名用source_id。
**framework_id**：标识一个分析框架。

**v3当前设计：一个source对应一个framework**（schema为单framework形状）。
`meta.md`只有一个`framework_id`和一个`EXTRACTION_BLOCK`，
生产流水线只输出一个`frameworks/<framework_id>.md`。

一个source产出多个framework的场景暂不支持，待实际需求出现后再扩展。

### 4.2 目录规则

每个编译产物存于 `library/compiled/<source_id>/`，至少包含：

```
meta.md          必须
step0-step1.md   必须
step2.md         必须
step3-step4.md   必须
step5-final.md   必须
```

允许附加可选文件（`assets/`、`sources.md`等）。

### 4.3 meta.md 格式

```markdown
---
source_id: guns-germs
source_title: 枪炮、病菌与钢铁
source_type: book
source_author: 贾雷德·戴蒙德
engine_version: shen-v3.4
compiled_date: 2026-04-16
framework_id: guns-germs
---

<!-- EXTRACTION_BLOCK: DO NOT EDIT FORMAT. Machine-readable only. -->
```yaml
CORE_MECHANISM: "地理决定起跑线——可驯化物种最多、传播轴线最优的大陆最先积累征服优势，与人种无关"

SHORT_MECHANISM: "差距根源在地图不在文化"

TRIGGER_SENTENCE: "当有人用文化或民族解释发展差距时，先追问起跑线在哪里——如果差距根源在5000年前，地图才是答案"

SHORT_TRIGGER: "发展差距叙事/文明优劣"

PRIMARY_HOOKS:
  - "任何关于'为什么A比B发达/强大/富裕'的叙事"
  - "有人用文化、民族、种族、勤奋解释历史差距"
  - "殖民历史讨论（被殖民方为何落后）"

SECONDARY_HOOKS:
  - "当代发展中国家与发达国家对比叙事"
  - "'为什么中国/印度/非洲……'类型的问题"

DIAGNOSTIC_ENTRY: |
  面对发展差距叙事，第一问：这个差距的根源在哪个时间尺度上？
  → 万年尺度（物种/轴线/地理）→ 本框架成立
  → 百年尺度（殖民/制度）→ 本框架+wang-shiran联用
  → 十年尺度（政策/偶然）→ 本框架基本无效

FAILURE_BOUNDARIES:
  - "公元1900年后的发展差距（韩朝对比，制度主导）"
  - "欧亚大陆内部差异（为何是西欧而非中国）"
  - "文化因素突破地理约束的案例（明治维新、韩国经济腾飞）"

PAIRED_FRAMEWORK: "wang-shiran（差距在百年尺度时切换）"
```
<!-- END_EXTRACTION_BLOCK -->
```

---

## 五、Contract 2：framework file contract

### 5.1 字段归属规则

```
[G] = Generated  由EXTRACTION_BLOCK生成，重编译时可覆盖
[M] = Manual     人工维护，编译流水线禁止修改
[C] = Computed   从records计算，任何人和流水线都不直接写
[B] = Bootstrap  bootstrap框架的手工填入值，无compiled artifact支撑
                 （bootstrap毕业后[B]→[G]）
```

### 5.2 完整文件格式

```markdown
# [框架名称]

**框架ID**: <framework_id>                                    [G/B]
**来源**: <source_title>                                      [G/B]
**framework_type**: compiled | bootstrap                       [G/B]
**compiled_source_id**: <source_id>（bootstrap时为null）       [G/B]
**compiled路径**: library/compiled/<source_id>/（bootstrap时为null）[G/B]
**建立时间**: YYYY-MM                                         [G/B]

---

## 短标签（用于index快速检索）

**SHORT_TRIGGER**: [触发域简述，10字以内]     [G/B]
（compiled：从EXTRACTION_BLOCK.SHORT_TRIGGER提取；bootstrap：人工填写）

**SHORT_MECHANISM**: [核心机制简述，10字以内]  [G/B]
（compiled：从EXTRACTION_BLOCK.SHORT_MECHANISM提取；bootstrap：人工填写）

---

## Hook注册

### 主触发 [G/B]
（compiled：从EXTRACTION_BLOCK.PRIMARY_HOOKS生成，重编译可覆盖）
（bootstrap：人工填写，毕业前不可被流水线覆盖）
- [填入]

### 次触发 [G/B]
（同上）
- [填入]

### 训练补充触发 [M]
（从实际训练记录中发现的新触发模式，人工积累，编译禁止修改）
- [人工添加]

---

## 核心机制（一句话）[G/B]
（compiled：从EXTRACTION_BLOCK.CORE_MECHANISM提取；bootstrap：人工填写）

---

## 诊断入口 [G/B]
（compiled：从EXTRACTION_BLOCK.DIAGNOSTIC_ENTRY提取；bootstrap：人工填写）

---

## 一句话总结（第3层触发句）[G/B]
（compiled：从EXTRACTION_BLOCK.TRIGGER_SENTENCE提取；bootstrap：人工填写）

---

## 失效边界 [G/B]
（compiled：从EXTRACTION_BLOCK.FAILURE_BOUNDARIES提取；bootstrap：人工填写）

---

## 配对框架 [G/B]
（compiled：从EXTRACTION_BLOCK.PAIRED_FRAMEWORK提取；bootstrap：人工填写）

---

## 内化状态 [C]
（从training/records/全量重算，禁止手工写入）

**当前等级**: L?
**sessions_total**: 0
**avg_level**: -
**complete_ratio**: -
**stuck_streak**: 0
**zero_level_event_types**: []
**最近训练**: -
**变化趋势**: -
```

### 5.3 Bootstrap框架规则

**定义**：bootstrap框架是没有对应compiled artifact的框架，[G]字段由人工填写标注为[B]。

**适用场景**：wang-shiran、shen-reading等在系统建立前已存在的框架，或永久无法对应书籍的经验型框架。

**毕业规则**：当bootstrap框架对应的材料被正式编译后，按 `library/DISTILL.md` **模式B** 执行（输入 meta.md 全文）：
1. [G] 字段覆盖原 [B] 值，`framework_type: bootstrap → compiled`
2. [M] 字段（训练补充触发）原样保留
3. [C] 字段（内化状态）原样保留，训练连续性不中断

**bootstrap不是临时状态**：没有对应原始材料的框架可以永久保持bootstrap状态。

---

## 六、Contract 3：frameworks/index.md contract

### 6.1 文件定位

Mayor的路由表。从frameworks/*.md全量重建，不增量维护。

### 6.2 列定义与来源

| 列名 | 来源字段 | 派生规则 |
|------|---------|---------|
| 框架ID | framework_id | 直接取值 |
| 触发域 | SHORT_TRIGGER [G/B] | 直接取值（bootstrap和compiled均有此字段） |
| 一句话 | SHORT_MECHANISM [G/B] | 直接取值（bootstrap和compiled均有此字段） |
| 等级 | 内化状态.当前等级 [C] | 直接取值 |
| 最近训练 | 内化状态.最近训练 [C] | 直接取值 |
| 优先级 | 等级 | L?或L0→★★★；L1→★★；L2或L3→★ |

### 6.3 重建触发条件

以下任一情况发生时触发全量重建：
- 任何frameworks/*.md的内化状态[C]发生变化（训练落盘后）
- 任何frameworks/*.md的SHORT_TRIGGER或SHORT_MECHANISM发生变化（包括：bootstrap初次填写、重编译覆盖[G]字段、bootstrap毕业）
- 新框架文件被创建或删除

### 6.4 重建方式

扫描所有frameworks/*.md，提取上表对应字段，生成完整表格，全量替换index.md内容。

### 6.5 文件格式

```markdown
# 框架快速检索表
更新时间：[时间戳]
来源：frameworks/*.md全量重建

| 框架ID | 触发域 | 一句话 | 等级 | 最近训练 | 优先级 |
|--------|--------|--------|------|---------|--------|
| guns-germs | 发展差距叙事/文明优劣 | 差距根源在地图不在文化 | L1 | 2026-04-16 | ★★ |
| wang-shiran | 政策表态/制度现象 | 穿透话语看真实行动者 | L? | - | ★★★ |
| shen-reading | 知识工作/学习方法 | 画不出来的地方就是漏洞 | L? | - | ★★★ |

优先级含义：
  ★★★ L?或L0（优先安排训练）
  ★★  L1（维持训练频率）
  ★   L2-L3（偶尔激活即可）
```

---

## 七、Contract 4：library/index.md contract

### 7.1 文件定位

编译产物导航与待编译队列。含两个分区，权威来源不同。

### 7.2 分区定义

```
[D]派生区：## 已编译 节
  来源：library/compiled/*/meta.md全量重建
  规则：重建时全量替换，不保留任何手工编辑

[M]人工区：## 待编译队列 节
  来源：本区内容本身即权威来源
  规则：重建时原样保留，不覆盖
```

### 7.3 列定义与来源（[D]派生区）

| 列名 | 来源 | 派生规则 |
|------|------|---------|
| source_id | meta.md YAML.source_id | 直接取值 |
| 书名/文章 | meta.md YAML.source_title | 直接取值 |
| 类型 | meta.md YAML.source_type | 直接取值 |
| 编译日期 | meta.md YAML.compiled_date | 直接取值 |
| 框架ID | meta.md YAML.framework_id | 直接取值 |
| 核心机制 | EXTRACTION_BLOCK.SHORT_MECHANISM | 直接取值 |
| 路径 | `library/compiled/<source_id>/` | 拼接source_id |

### 7.4 重建算法

```
1. 读取当前library/index.md，提取[M]区域内容
   （## 待编译队列标题行到文件末尾，含注释标记）
2. 扫描所有library/compiled/*/meta.md，生成新的[D]区域内容
3. 输出 = [D]区域（新生成）+ [M]区域（原样保留）
4. 全量写入library/index.md
```

### 7.5 文件格式

```markdown
# 编译产物索引
更新时间：[时间戳]

<!-- [D] 以下由重建程序全量替换，请勿手动编辑 -->
## 已编译

| source_id | 书名/文章 | 类型 | 编译日期 | 框架ID | 核心机制 | 路径 |
|-----------|---------|------|---------|--------|---------|------|
| guns-germs | 枪炮、病菌与钢铁 | book | 2026-04-16 | guns-germs | 差距根源在地图不在文化 | library/compiled/guns-germs/ |
<!-- [D] 结束 -->

<!-- [M] 以下为人工维护区，重建程序不修改 -->
## 待编译队列

| 书名 | 优先级 | 预计framework_id | 备注 |
|-----|--------|----------------|------|
| 国家为什么会失败 | ★★★ | acemoglu-institutions | 与guns-germs互补，填补百年尺度 |
<!-- [M] 结束 -->
```

---

## 八、Contract 5：training record contract

### 8.1 有效记录集（effective records）定义

```python
# 伪代码
voided_ids = {
    r.supersedes_training_id
    for r in all_records
    if r.supersedes_training_id is not None
}
effective = [
    r for r in all_records
    if r.record_status == "committed"
    and r.training_id not in voided_ids
]
```

### 8.2 record_status合法输入值

```
committed   正常记录，参与统计
voided      作废，不参与统计
```

`superseded`是派生状态（training_id被后续记录的supersedes_training_id指向），不落盘。

### 8.3 纠错机制（append-only）

发现错误记录时追加新记录，不修改原记录：
- 完全错误替换：新建`record_status: committed` + `supersedes_training_id: <旧ID>`
- 纯作废：新建`record_status: voided` + `supersedes_training_id: <旧ID>`

### 8.4 记录格式

`quality`字段不落盘，在展示层由规则现算（见8.6）。

```markdown
## [YYYY-MM-DD HH:MM] | [framework_id] | L[training_level]

<!-- RECORD_BLOCK
training_id: guns-germs-20260416-143015
framework_id: guns-germs
event_type: geopolitical
source_ref: news
training_level: 1
hint_level1_used: true
hint_level2_used: false
hint_level3_used: false
path_correct: true
judgment_specific: true
record_status: committed
supersedes_training_id: null
-->

**事件**：[一句话描述真实事件]
**卡住节点**：[training_level > 0时必填]
**推导路径**：[事件 → 中间步骤 → 框架，3步以内]
**输出判断**：[具体结论，不是框架名或定义]
```

### 8.5 字段规范

**training_id格式**：`<framework_id>-<YYYYMMDD>-<HHMMSS>`（秒级，个人单线程使用零碰撞）

**event_type固定六类**（按框架激活路径的主要机制分类，不按事件表面话题）：

```
policy       政策/监管/制度表态
geopolitical 地缘/外交/国际关系
social       社会现象/文化/人口
market       市场/价格/产业现象
tech         技术趋势/行业动态
historical   历史分析/回顾
```

跨类无法区分时优先级：`policy > geopolitical > social > market > tech > historical`

### 8.6 quality展示层计算（不持久化）

```
path_correct=true  AND judgment_specific=true  → 完整
path_correct=true  XOR judgment_specific=true  → 部分
path_correct=false AND judgment_specific=false → 形式
```

**path_correct判定（两条标准）：**
1. 能说出事件到框架之间至少一个中间推导步骤（不是直接跳到框架名）
2. 中间步骤在逻辑上确实连接了事件特征和框架的诊断入口

**judgment_specific判定（两条标准）：**
1. 输出包含关于这个具体事件的结论（不是框架的一般描述）
2. 结论中有可以被后续事实验证的具体表述

---

## 九、Contract 6：log.md contract

### 9.1 文件定位

派生文件。全量重建，不增量维护。用于人类可读的操作审计，不参与任何计算。

### 9.2 时间源规则

```
训练事件时间戳：从training_id解出（秒级）
  training_id格式：<framework_id>-<YYYYMMDD>-<HHMMSS>
  解析：取后两段拼成 YYYY-MM-DD HH:MM:SS

编译事件时间戳：从meta.md的compiled_date读取（日级，只到日）

同时间戳排序规则（保证重建幂等）：
  同日期内：compile事件排在train事件之前
  同类型同时间戳：按framework_id字母升序
```

### 9.3 行格式

```
训练事件（秒级）：
## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality>

作废事件（秒级，来自record_status=voided的记录）：
## [YYYY-MM-DD HH:MM:SS] void | <training_id>

替换事件（秒级，新committed记录有supersedes_training_id时）：
## [YYYY-MM-DD HH:MM:SS] train | <framework_id> | L<N>·<quality> | supersedes:<old_training_id>

编译事件（日级）：
## [YYYY-MM-DD] compile | <source_id>（<source_title>）→ <framework_id>
```

**void的精确定义**：
- void行仅来自record_status=voided的记录，时间戳从该记录training_id解出
- 不为"被superseded的committed记录"单独生成void行
- 被取代信息通过新committed记录行尾的`supersedes:<old_id>`表达

### 9.4 重建算法

```
1. 收集训练/作废事件：
   扫描training/records/YYYY-MM.md，解析所有RECORD_BLOCK
   - record_status=committed，supersedes_training_id=null
     → type=train，时间戳从training_id解出
   - record_status=committed，supersedes_training_id≠null
     → type=train（带supersedes后缀），时间戳从training_id解出
   - record_status=voided
     → type=void，时间戳从training_id解出

2. 收集编译事件：
   扫描library/compiled/*/meta.md
   读取compiled_date/source_id/source_title/framework_id
   → type=compile，时间戳=compiled_date（日级）

3. 合并排序：
   先按时间戳升序
   同时间戳：compile先于train，同类按framework_id字母序

4. 全量写入log.md
```

### 9.5 可验证性

相同的records数据集 + 相同的meta.md数据集，重建结果完全相同。任何工具可随时从records重建log.md并验证一致性。

---

## 十、internalization_level判定链

```
统计窗口：从effective records中取最近5次
          sessions_total < 3时 → L?（默认优先级★★★）

计算变量：
  avg_level        = 最近5次effective records的training_level平均值
  complete_ratio   = 最近5次中(path_correct AND judgment_specific)的比例
  stuck_streak     = 当前连续training_level==3的次数（从末尾算）
  zero_event_types = 所有历史effective records中training_level==0的不同event_type数量

判定链（严格按此顺序）：
  if sessions_total < 3:           → L?
  elif stuck_streak >= 3:          → L0（强制降级）
  elif avg_level < 0.5
       and complete_ratio >= 0.8
       and zero_event_types >= 3:  → L3（身体记忆形成）
  elif avg_level < 1.5
       and complete_ratio >= 0.6:  → L2
  elif avg_level <= 2.5
       and complete_ratio >= 0.3:  → L1
  else:                            → L0
```

---

## 十一、Mayor设计（两段式）

### 第一段：调度 + 训练

```
1. 读 frameworks/index.md，找优先级★★★的框架
2. 选择一个真实事件（历史/新闻/文章/书籍均可，不限当天）
   要求：能命中目标框架，但需要2-3步推导（不允许直接命中）
3. 执行训练协议（PROTOCOL.md）
   第0层：裸触发，等待，不给提示
   第1层：维度提示（Solomon明确卡住或请求时解锁）
   第2层：节点提示（仍卡住时解锁）
   第3层：触发句（仍卡住时解锁，要求走完flowchart）
4. 走通后要求输出具体判断
5. 在内存中生成结构化RECORD_BLOCK（准备落盘）
```

### 第二段：落盘事务

```
"训练完成。以下是本次记录，确认后落盘（或说'自动'跳过确认）："
[展示RECORD_BLOCK内容]

幂等检查：
  若records中已存在相同training_id：
    → 跳过步骤1（不重复追加records）
    → 直接执行步骤2、3、4（重新计算并重建）
    → 跳过步骤5（records未变化，log无需重建）
  若不存在：
    → 正常执行全部步骤

步骤1：追加 training/records/YYYY-MM.md（一条记录）
步骤2：从该框架所有effective records重新计算internalization_level
步骤3：全量更新 frameworks/<id>.md 的[C]字段（内化状态section）
步骤4：全量重建 frameworks/index.md
步骤5：从所有training/records/ + 所有library/compiled/*/meta.md 全量重建log.md
```

---

## 十二、生产侧流水线（书→框架）

```
Step 1: 书/文章放入 library/raw/

Step 2: 用沈老师引擎 v3.4 完整跑一遍
        输出：library/compiled/<source_id>/ 下5个step文件

Step 3: 撰写 meta.md
        必须包含：YAML头（source_id/source_title/source_type/source_author/
                          engine_version/compiled_date/framework_id/
                          framework_display_name）
                  framework_display_name = 框架文件标题行，可与source_title不同
                  （例：source_title="国家为什么会失败"，framework_display_name="制度决定论"）
        必须包含：完整EXTRACTION_BLOCK（含SHORT_MECHANISM和SHORT_TRIGGER）

Step 4: 框架蒸馏（按 library/DISTILL.md 执行）
        输入：meta.md全文（YAML头 + EXTRACTION_BLOCK）
        输出：frameworks/<framework_id>.md
        [G]字段从EXTRACTION_BLOCK填入
        [M]字段（训练补充触发）留空
        [C]字段（内化状态）初始化为L?、sessions_total=0

Step 5: 全量重建索引
        重建 frameworks/index.md（扫描所有frameworks/*.md）
        重建 library/index.md（[D]区扫描所有compiled/*/meta.md，[M]区原样保留）

Step 6: 全量重建log.md
        从所有training/records/ + 所有library/compiled/*/meta.md重建，全量替换
```

---

## 十三、实施阶段

### Phase 1：建立目录和初始文件（第1天）

**注意事项：**
- wang-shiran.md和shen-reading.md作为bootstrap框架处理
- bootstrap框架的[G]字段标注[B]，由人工填写
- guns-germs需补meta.md（含完整EXTRACTION_BLOCK，包括SHORT_MECHANISM和SHORT_TRIGGER）

任务清单：
- [ ] 建立完整目录树
- [ ] 创建SOUL.md
- [ ] 创建AGENTS.md（含字段归属规则、落盘规则、幂等规则、bootstrap规则）
- [ ] 创建README.md
- [ ] 把《枪炮》4个step文件放入library/compiled/guns-germs/
- [ ] 创建library/compiled/guns-germs/meta.md（含完整EXTRACTION_BLOCK）
- [ ] 按Contract 2重建frameworks/guns-germs.md（framework_type: compiled）
- [ ] 创建frameworks/wang-shiran.md（framework_type: bootstrap，[B]字段人工填写）
- [ ] 创建frameworks/shen-reading.md（framework_type: bootstrap，[B]字段人工填写）
- [ ] 全量重建frameworks/index.md（3行）
- [ ] 全量重建library/index.md（[D]1行 + [M]待编译队列）
- [ ] 创建training/records/2026-04.md（空文件+schema说明注释）
- [ ] 全量重建log.md（仅1条compile事件）

**Phase 1完成标志：**
- 新读者能从README找到入口
- frameworks/index.md列出3个框架，所有列有确定值
- library/index.md的[D]/[M]分区标注正确
- 能完成一次 raw→compiled→framework→training record 的完整闭环演练

---

### Phase 2：升级MAYOR.md和PROTOCOL.md（第2天）

任务清单：
- [ ] 重写MAYOR.md（两段式：调度训练/落盘事务，含幂等检查）
- [ ] 重写PROTOCOL.md（严格分层提示协议，含event_type规则和rubric）
- [ ] 测试：走一次完整训练会话，验证落盘后frameworks/index.md正确重建
- [ ] 测试：重复落盘相同training_id，验证log.md不重建，数据不变
- [ ] 测试：从records重建log.md，验证结果与当前log.md完全一致

**Phase 2完成标志：**
- Mayor说出当前最高优先级框架及等级原因
- 重试落盘不产生重复数据
- log.md可从records机械重建，结果一致

---

### Phase 3：框架蒸馏模板（第3天）

任务清单：
- [ ] 编写框架蒸馏prompt（输入EXTRACTION_BLOCK→输出frameworks/*.md的[G]字段）
- [ ] 验证：用guns-germs的EXTRACTION_BLOCK跑一遍，格式完全匹配
- [ ] 把蒸馏流程和bootstrap毕业规则写入AGENTS.md

**Phase 3完成标志：**
- 输入EXTRACTION_BLOCK，10分钟内输出合格框架文件
- [G]字段全部正确，[M]和[C]字段保持空/初始值
- 可重复执行，输出稳定

---

### Phase 4：运行验证（第一周）

任务清单：
- [ ] 生产侧：选下一本书跑完整流水线（建议《国家为什么会失败》）
- [ ] 训练侧：每天至少一次训练会话，记录触发层级
- [ ] 验证：从records重建log.md，与当前log.md完全一致
- [ ] 验证：effective records计算在真实数据上正确运行

**Phase 4完成标志：**
- log.md有≥5条train事件和≥1条compile事件
- 有完整闭环（raw→compiled→framework→training record）至少一次
- log.md可从records机械重建

---

### Phase 5：月度复盘（第一个月末）

月度复盘报告格式：
```
## YYYY-MM 月度认知训练报告

框架内化进度：
| 框架 | 月初等级 | 月末等级 | 有效训练次数 | avg_level | complete_ratio | 趋势 |

本月有效训练：N次（跳过：M次）
本月命中的event_type分布：[统计]
avg_level变化最大的框架：[分析]
下月重点训练框架：[基于L?/L0框架]
待编译书籍建议：[基于现有框架空缺]
```

---

## 十四、成功标准

| 时间点 | 可观察的成功证据 |
|--------|----------------|
| Phase 1完成 | 新读者能从README找到入口；所有index.md列有确定来源；能完成完整闭环演练 |
| Phase 2完成 | Mayor说出最高优先级框架；重试落盘不重复；log.md可从records重建且一致 |
| Phase 3完成 | 输入EXTRACTION_BLOCK→10分钟内输出合格框架文件，可重复 |
| Phase 4第一周 | log.md≥5 train事件，≥1 compile事件，有完整闭环，effective records计算正确 |
| Phase 5完成 | 月度报告有至少一个框架显示等级变化 |
| 3个月后 | 至少一个框架达到L2（avg_level < 1.5，complete_ratio ≥ 60%） |

---

## 十五、已有素材清单

### 已编译（需补meta.md后注册）
- 《枪炮、病菌与钢铁》→ 4个step文件已存在，需补meta.md

### Bootstrap框架（人工填写[B]字段）
- wang-shiran（实然拆解法）
- shen-reading（读书建模法）

### 待编译优先队列
1. 《国家为什么会失败》（Acemoglu）— 填补百年尺度，与guns-germs直接互补

---

## 十六、版本变更历史摘要

| 版本 | 核心变更 |
|------|---------|
| v1.0 | 初始计划，方向正确，contracts尚未成型 |
| v2.0 | 三大contract建立；training_id幂等；EXTRACTION_BLOCK严格YAML；bootstrap框架；library/index.md补contract |
| v2.1（草案） | framework文件加SHORT字段；library/index.md [D]/[M]分区；收回一对多承诺；log.md补最小contract |
| v3.0 | log.md时间源锁定（training_id解出秒级，compiled_date日级）；void定义精确化；log追加→全量重建统一；index重建触发条件扩展；library/index.md描述与SOT对齐；contracts全部locked |

---

*v3.0 · 经6轮CodeX review收口 · contracts状态：全部locked · 可进入Phase 1*
