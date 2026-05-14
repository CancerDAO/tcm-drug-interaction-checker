---
name: tcm-drug-interaction-checker
description: "查你正在吃的西药和家里炖的汤、中药方子、保健品之间有没有冲突，每条结论都附上指南或数据库来源。Triggers on: 中药, 虫草, 灵芝, 西洋参, 姜黄, 保健品, 补品, 冲突, 相互作用, 靶向药, 免疫治疗, 能不能吃, 能不能喝"
license: MIT
metadata:
  author: CancerDAO
  version: "1.0.0"
---

# TCM-Drug Interaction Checker

## When to Use

This skill activates when the user asks about interactions between:
- 西药 (Western oncology drugs: 靶向药, 免疫治疗, 化疗药)
- 中药/中成药 (TCM formulas: 汤剂, 中成药)
- 补品/保健品 (supplements: 虫草, 灵芝, 姜黄素, 灵芝孢子粉)
- 药食同源 (food-herb dual-use: 西柚汁, 甘草, 西洋参)
- 饮食禁忌 (dietary items: 西柚汁, 绿茶, 银杏)

Trigger phrases: 能不能吃, 能不能喝, 有没有冲突, 相互作用, 影响药效吗, 和靶向药冲突吗

## Inputs

1. **西药清单** (required): Drug name(s) — can be brand name or generic name
   - Examples: 奥希替尼, Keytruda (帕博利珠单抗), 吉西他滨, 顺铂

2. **中药/补品/食物清单** (required): TCM, supplements, or foods being consumed
   - Examples: 虫草花, 灵芝孢子粉, 西柚汁, 姜黄素胶囊, 甘草

## Outputs

For each supplement/drug pair, output:

| 字段 | 说明 |
|------|------|
| 结论 | 可用 / 谨慎使用 / 停用 / 未知 |
| 机制说明 | 药理学机制，用通俗中文解释 |
| 证据来源 | citation (NCI, FDA, Natural Medicines, Chinese herbal refs) |
| 严重程度 | 轻微 / 中等 / 严重 / 未知 |
| 建议 | 具体行动建议 |

## Workflow

1. **Parse input**: Identify 西药 names and supplement/food names from user query
2. **Match known high-risk items**: Check against pre-loaded interaction database
3. **Cross-reference databases**:
   - NCI Cancer Information Database
   - FDA Drug Interaction Database
   - Natural Medicines Comprehensive Database
   - Memorial Sloan Kettering Cancer Center (MSKCC) Herbs & Supplements Database
   - Chinese herbal medicine interaction references (中药药理学)
4. **Explain mechanism**: Translate pharmacological interaction into plain Chinese
5. **Rate severity**: 轻微 / 中等 / 严重 / 未知
6. **Cite sources**: Inline citation with database and access date
7. **Safety disclaimer**: Always recommend clinician verification

## Pre-loaded High-Risk Interaction Database

### 严重 (Avoid / Significant Interaction)

**西柚汁 (Grapefruit Juice)**
- 机制: 抑制 CYP3A4 酶 → 增加靶向药血药浓度 → 可能出现严重不良反应
- 常见受影响的药: EGFR TKI (奥希替尼, 吉非替尼, 厄洛替尼), 多种靶向药
- 证据: FDA Drug Interaction Database, NCI
- 建议: 避免食用

**圣约翰草 (St. John's Wort / 贯叶连翘)**
- 机制: 诱导 CYP3A4/P-gp → 降低靶向药血药浓度 → 药效下降
- 证据: Natural Medicines Comprehensive Database
- 建议: 停用

### 中等 (Use with Caution)

**姜黄素 (Curcumin)**
- 机制: 可能影响 CYP 酶活性和肝脏代谢；高剂量可能增强化疗效果或产生拮抗
- 证据: NCI Cancer Information Database, published literature
- 建议: 化疗期间慎用，与主管医生确认剂量

**灵芝 (Lingzhi / Ganoderma lucidum)**
- 机制: 可能调节免疫功能，与免疫治疗产生协同或拮抗效应
- 证据: Memorial Sloan Kettering Cancer Center Herbs Database
- 建议: 免疫治疗期间慎用，监测免疫相关指标

**甘草 (Licorice / 甘草)**
- 机制: 假性醛固酮增多症风险，与糖皮质激素联用可能加重水钠潴留
- 证据: Natural Medicines Comprehensive Database
- 建议: 糖皮质激素使用者避免长期大剂量

**银杏叶 (Ginkgo Biloba)**
- 机制: 抑制血小板聚集，与抗凝药/化疗可能导致出血风险
- 证据: NCI, Natural Medicines
- 建议: 抗凝治疗或出血风险患者避免

### 轻微 / 未知 (Low Risk or Limited Data)

**虫草 (Cordyceps / 虫草花)**
- 机制: 有限的临床数据，未发现明确相互作用
- 证据: 有限队列研究，无严重不良事件报告
- 建议: 可用，但注意质量问题，化疗期间优先通过食物补充而非高剂量补剂

**西洋参 (American Ginseng / 西洋参)**
- 机制: 可能调节血糖，与降糖药协同；可能影响血压
- 证据: Natural Medicines
- 建议: 血糖异常或血压异常患者监测

## Safety Disclaimer

> 本工具仅供参考，不能替代医生、药师或中医师的临床判断。
> 所有用药调整应在主管医生指导下进行。
> 本工具基于公开数据库和文献，可能不覆盖最新研究或个体差异。

## Evidence Sources Format

```
[来源] 数据库/文献名 | 访问日期
示例:
[1] FDA Drug Interaction Database | 2026-05-14
[2] Memorial Sloan Kettering Cancer Center Herbs Database | 2026-05-14
[3] Natural Medicines Comprehensive Database | 2026-05-14
[4] NCI Cancer Information Database | 2026-05-14
```