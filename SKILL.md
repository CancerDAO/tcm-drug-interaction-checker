---
name: cancerdao-tcm-drug-interaction-checker
description: "当患者询问中药、保健品、食物与西药（靶向药、免疫治疗、化疗）之间是否有冲突时使用。查询预加载高风险数据库 + 在线证据源，输出带溯源结论的交互报告（结论/机制/证据来源/严重程度/建议）。触发词：中药、虫草、灵芝、西洋参、姜黄、保健品、补品、冲突、相互作用、靶向药、免疫治疗、能不能吃、能不能喝。"
license: MIT
metadata:
  author: CancerDAO
  version: "2.0.0"
  tags: drug-interactions tcm supplements oncology
---

# CancerDAO TCM-Drug Interaction Checker

检查西药与中药/保健品/食物之间的相互作用，每条结论附证据来源。

## When to use

- 吃靶向药能喝西柚汁吗
- 虫草/灵芝/姜黄素和化疗药冲突吗
- 中药和免疫治疗一起吃有没有问题

## Inputs

**西药清单**（必填）：药物通用名或品牌名
**中药/保健品/食物清单**（必填）

## Outputs

每对交互：结论 | 机制说明 | 证据来源 | 严重程度 | 建议

## 高风险物品预加载

**严重（停用）**：

| 物品 | 机制 | 受影响药物 |
|---|---|---|
| 西柚汁 | 抑制 CYP3A4 → 增加靶向药血药浓度 | EGFR TKI |
| 圣约翰草 | 诱导 CYP3A4/P-gp → 降低药效 | 多种靶向药 |

**中等（谨慎使用）**：

| 物品 | 机制 | 受影响药物 |
|---|---|---|
| 姜黄素 | 影响 CYP 酶 + 肝脏代谢 | 化疗药 |
| 灵芝 | 调节免疫功能 | PD-1/PD-L1 |
| 甘草 | 假性醛固酮增多症 | 糖皮质激素 |
| 银杏叶 | 抗血小板聚集 | 抗凝药 |

**轻微/未知（可用）**：虫草、西洋参

## 溯源格式

```
[1] FDA Drug Interaction Database | 2026-05-14
[2] MSKCC Herbs Database | 2026-05-14
[3] Natural Medicines | 2026-05-14
```

## Safety disclaimer

> 本工具仅供参考，请与您的临床医生确认后再做用药决定。

## References

- `references/grapefruit-interactions.md`
- `references/st-johns-wort-interactions.md`
- `references/mskcc-herbs-database.md`
- `references/curcumin-interactions.md`
- `references/cordyceps-lingzhi-interactions.md`
- `references/interaction-severity-guide.md`

## File map

```
cancerdao-tcm-drug-interaction-checker/
├── SKILL.md
├── README.md
├── evals/
│   ├── evals.json
│   └── files/
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
