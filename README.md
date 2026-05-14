# CancerDAO TCM-Drug Interaction Checker

中药/保健品与西药相互作用查询 skill。每条结论附证据来源。

## Install

```bash
npx skills add CancerDAO/tcm-drug-interaction-checker
```

## When to use

- 吃靶向药能喝西柚汁吗
- 虫草/灵芝和免疫治疗冲突吗
- 化疗期间能吃姜黄素吗

## Key features

- 预加载高风险物品数据库（CYP3A4 抑制/诱导）
- 证据溯源（FDA、MSKCC、Natural Medicines）
- 严重程度分级（轻微/中等/严重/未知）
- 机制说明（通俗中文）

## References

- `references/grapefruit-interactions.md` — CYP3A4 机制
- `references/st-johns-wort-interactions.md` — CYP450 诱导
- `references/mskcc-herbs-database.md` — MSKCC 数据库使用

MIT — CancerDAO
