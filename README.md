# TCM-Drug Interaction Checker

Check interactions between Western oncology drugs (靶向药, 免疫治疗, 化疗) and TCM/supplements/foods.

## Skill Overview

| Item | Detail |
|------|--------|
| Name | tcm-drug-interaction-checker |
| Purpose | Identify drug-supplement interactions with evidence-based citations |
| Triggers | 中药, 虫草, 灵芝, 西洋参, 姜黄, 保健品, 补品, 冲突, 相互作用, 靶向药, 免疫治疗, 能不能吃, 能不能喝 |
| Output | Structured interaction report per item with conclusion, mechanism, evidence, severity, and recommendation |

## Quick Start

User describes their 西药 (Western drug) and what they are taking/eating from:
- TCM formulas (中药方子)
- Supplements (保健品/补品: 虫草, 灵芝, 姜黄素)
- Foods (食物: 西柚汁, 银杏叶)
- Food-herb dual-use items (药食同源: 甘草, 西洋参)

The skill returns a structured report with:
- 结论: 可用 / 谨慎使用 / 停用 / 未知
- 机制说明: Plain-language pharmacological explanation
- 证据来源: Inline citations
- 严重程度: 轻微 / 中等 / 严重 / 未知
- 建议: Specific action recommendation

## Pre-loaded High-Risk Items

| Item | Risk | Mechanism |
|------|------|-----------|
| 西柚汁 | 严重 | CYP3A4 inhibition → toxic TKI levels |
| 圣约翰草 | 严重 | CYP3A4/P-gp induction → reduced drug efficacy |
| 姜黄素 | 中等 | CYP enzyme effects on chemo metabolism |
| 灵芝 | 中等 | Immunomodulatory effects with ICI |
| 甘草 | 中等 | Pseudoaldosteronism with steroids |
| 银杏叶 | 中等 | Antiplatelet + anticoagulant/chemo = bleeding |
| 虫草 | 轻微 | Limited data, generally low concern |
| 西洋参 | 轻微-中等 | Hypoglycemia/hypertension risk |

## Safety Disclaimer

This tool provides reference information only. All medication adjustments must be made under clinical supervision. Data based on public databases and published literature; may not reflect latest research or individual variation.

## Directory Structure

```
tcm-drug-interaction-checker/
├── SKILL.md              # Main skill definition
├── README.md             # This file
└── evals/
    ├── evals.json        # Evaluation test cases
    └── files/
        └── interactions_reference.json  # Reference data for testing
```

## For Developers

See `evals/` for test cases and reference data. The skill parses user input to identify drug and supplement names, cross-references known interaction databases, and outputs structured reports.

To test the skill, run eval cases against your LLM/agent harness with the trigger phrases listed in SKILL.md.