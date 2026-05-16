---
name: tcm-drug-interaction-checker
description: "当患者询问中药、保健品、食物与西药（靶向药、免疫治疗、化疗）之间是否有冲突时使用。系统查询预加载的高风险物品数据库 + 在线证据源，输出带溯源结论的交互报告（结论/机制/证据来源/严重程度/建议）。触发词：中药、虫草、灵芝、西洋参、姜黄、保健品、补品、冲突、相互作用、靶向药、免疫治疗、能不能吃、能不能喝、忌口。"
license: MIT
metadata:
  author: CancerDAO
  version: "2.0.0"
  tags: drug-interactions tcm supplements oncology
---

# CancerDAO TCM-Drug Interaction Checker

检查西药与中药/保健品/食物之间的相互作用，每条结论附证据来源。

> ⚠️ **免责声明**：本工具仅供参考，不能替代医生、药师或中医师的临床判断。所有用药调整应在主管医生指导下进行。

## When to use

触发以下场景时激活本技能：

- 询问"吃靶向药能喝西柚汁吗"
- 询问"虫草/灵芝/姜黄素和化疗药冲突吗"
- 询问"中药和免疫治疗一起吃有没有问题"
- 提到正在用某种西药，同时提到某种中药/补品/食物

## Inputs

**西药清单**（必填）：
- 药物通用名或品牌名
- 示例：奥希替尼、帕博利珠单抗、吉西他滨、顺铂

**中药/保健品/食物清单**（必填）：
- 示例：虫草花、灵芝孢子粉、西柚汁、姜黄素胶囊、甘草

## Outputs — 强制 5 字段结构化模板（每对交互一份）

每对"<西药> × <中药/保健品/食物>" 输出**必须**完整包含下列 5 个字段，**顺序固定**，**字段名不可缩写**。缺任一字段 = bug。

```
### <西药> × <中药/保健品/食物>

**1. 结论**：[可用 / 谨慎使用 / 停用 / 证据不足无法判断]
（注意：未直接给"可用"——证据不足时必须用"证据不足无法判断"，禁止默认放行）

**2. 机制**：[具体酶/受体/通路名 + 通俗解释]
- 命中酶/受体/通路：[例如：抑制 CYP3A4 / 诱导 P-gp / 拮抗 PAF 受体 / 影响血小板聚集]
- 通俗解释：[一句话患者能懂的版本]

**3. 严重程度**：[轻微 / 中等 / 严重 / 证据不足]
- 理由：[1 句话，引用具体临床后果]

**4. 证据来源**（必须从下列 5 项 ENUM 中选一项，不允许自由文本）：
   - `FDA`：FDA Drug Interaction Database / FDA 药品说明书 / FDA 在线检索
   - `MSKCC`：Memorial Sloan Kettering About Herbs Database
   - `PubMed`：PubMed 检索得到的具体临床/药理学研究（必须附 PMID 或 DOI）
   - `药理学推断`：无人体证据，仅基于已知药代/药效机制的合理推断
   - `未知`：以上四项都没有命中

**5. 建议**：
   - 行动：[继续使用 / 间隔 X 小时 / 停用 N 天 / 立即停药并联系医生]
   - 替代方案：[如果停用，是否有更安全的替代品]
   - 交叉风险：[这位患者还在用什么药，触发了什么风险]
```

### 关键规则：当证据来源 ∈ {`药理学推断`, `未知`} 时强制降级

这是 skill 的核心安全契约——不要让模型在没有证据的情况下自信地说"会冲突"或"没事"。

**降级触发条件**：第 4 字段选了 `药理学推断` 或 `未知`。

**触发后的强制输出**：

- 第 1 字段（结论）**不允许**写 `可用` 或 `停用`，**只能**写 `证据不足无法判断` 或 `谨慎使用`。
- 第 3 字段（严重程度）**不允许**写 `轻微` 或 `严重`，**只能**写 `证据不足` 或 `中等`。
- 输出末尾**必须**追加一段固定的"证据有限声明"块：

  ```
  ⚠️ 证据有限声明
  这一对相互作用在权威临床数据库（FDA / MSKCC / 已发表 PubMed 研究）中
  没有找到直接证据。上面的判断是 [药理学推断 / 完全未知]，不是临床证据。
  请把这一项明确告诉你的主治医生或临床药师——
  他们能在你的具体用药方案里判断是否值得保留这个组合。
  ```

### 为什么这一规则存在（不要删）

2026-05 agent-skills-eval 显示：tcm-drug-interaction-checker 加 skill 后，5/5 失败案例（甘草+地高辛 / 银杏+阿司匹林 / 灵芝+华法林 / 当归+他莫昔芬等）的共同模式是——**模型在 ground truth 缺失时自信编造结论**。中药互作的真实情况就是大部分组合没有人体证据。**逼模型说"不知道"比逼它说"可以/不行"更安全**。

这一规则不是为了规避责任——是因为对一个拿着化疗药和保健品的患者来说，"证据有限，请咨询医生"是诚实且能保护他的；"基于药理学这两个应该没事"是一个假阳性的安全感。

## Outputs (legacy 简表，仍保留用于汇总)

| 字段 | 说明 |
|---|---|
| 结论 | 可用 / 谨慎使用 / 停用 / 证据不足无法判断 |
| 机制说明 | 药理学机制，用通俗中文解释 |
| 证据来源 | ENUM {FDA / MSKCC / PubMed / 药理学推断 / 未知} |
| 严重程度 | 轻微 / 中等 / 严重 / 证据不足 |
| 建议 | 具体行动建议 |

## Workflow

### Step 1 — 解析输入

识别：
- 西药名称（通用名/品牌名）
- 中药/保健品/食物名称

### Step 2 — 匹配高风险物品数据库

查询预加载数据库（按严重程度排序）：

**严重（停用）**：

| 物品 | 机制 | 受影响药物 | 证据 |
|---|---|---|---|
| 西柚汁 | 抑制 CYP3A4 → 增加靶向药血药浓度 | EGFR TKI、多种靶向药 | FDA Drug Interaction Database |
| 圣约翰草 | 诱导 CYP3A4/P-gp → 降低药效 | 多种靶向药、免疫抑制剂 | Natural Medicines |

**中等（谨慎使用）**：

| 物品 | 机制 | 受影响药物 | 证据 |
|---|---|---|---|
| 姜黄素 | 影响 CYP 酶 + 肝脏代谢 | 化疗药（Doxorubicin, Irinotecan 等）| NCI, MSKCC Herbs |
| 灵芝 | 调节免疫功能 | 免疫治疗（PD-1/PD-L1）| MSKCC Herbs |
| 甘草 | 假性醛固酮增多症 | 糖皮质激素 | Natural Medicines |
| 银杏叶 | 抗血小板聚集 | 抗凝药、出血风险患者 | NCI |

**轻微/未知（可用，但注意质量）**：

| 物品 | 机制 | 证据 |
|---|---|---|
| 虫草 | 有限临床数据，未发现明确相互作用 | 有限队列研究 |
| 西洋参 | 可能调节血糖/血压 | Natural Medicines |

详见：`references/cordyceps-lingzhi-interactions.md`、`references/curcumin-interactions.md`、`references/grapefruit-interactions.md`、`references/st-johns-wort-interactions.md`

### Step 3 — 溯源

每个结论必须附证据来源，格式：
```
[1] FDA Drug Interaction Database | 2026-05-14
[2] Memorial Sloan Kettering Cancer Center Herbs Database | 2026-05-14
[3] Natural Medicines Comprehensive Database | 2026-05-14
[4] NCI Cancer Information Database | 2026-05-14
```

### Step 4 — 输出安全免责声明

每次输出必须附：
> 本工具仅供参考，请与您的临床医生确认后再做用药决定。

## 严重程度分级标准

| 等级 | 定义 | 行动 |
|---|---|---|
| 严重 | 可能导致严重不良反应或药效丧失 | 停用，联系医生 |
| 中等 | 可能影响药效或增加副作用 | 谨慎使用，咨询医生 |
| 轻微 | 有限证据，可能有轻微影响 | 可用，监测 |
| 未知 | 数据不足，无法评估 | 咨询医生 |

详见：`references/interaction-severity-guide.md`

## 常见错误

| 错误 | 正确做法 |
|---|---|
| 说"可以吃"但不解释机制 | 必须说明机制 + 证据 |
| 不区分西柚汁 vs 橙汁 | 橙汁 CYP3A4 抑制弱，可作为替代 |
| 忽略时间窗（西柚汁效应持续 24h+）| 说明效应持续时间 |
| 不提质量差异（虫草市场混乱）| 提醒质量问题，建议可靠渠道 |

## References

- `references/grapefruit-interactions.md` — CYP3A4 机制 + 受影响药物完整列表
- `references/st-johns-wort-interactions.md` — CYP450 诱导 +  serotonin syndrome 风险
- `references/mskcc-herbs-database.md` — MSKCC About Herbs 数据库使用指南
- `references/curcumin-interactions.md` — 姜黄素与化疗药相互作用
- `references/cordyceps-lingzhi-interactions.md` — 虫草/灵芝相互作用
- `references/licorice-interactions.md` — 甘草与激素相互作用
- `references/ginkgo-biloba-interactions.md` — 银杏与抗凝药相互作用
- `references/interaction-severity-guide.md` — 严重程度分级标准
- `references/fda-drug-interactions.md` — FDA 药物相互作用资源

## File map

```
cancerdao-tcm-drug-interaction-checker/
├── SKILL.md
├── README.md
├── evals/
│   ├── evals.json
│   └── files/
│       └── interactions_reference.json
└── references/
    ├── grapefruit-interactions.md
    ├── st-johns-wort-interactions.md
    ├── mskcc-herbs-database.md
    ├── curcumin-interactions.md
    ├── cordyceps-lingzhi-interactions.md
    ├── licorice-interactions.md
    ├── ginkgo-biloba-interactions.md
    ├── interaction-severity-guide.md
    └── fda-drug-interactions.md
```