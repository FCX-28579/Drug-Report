# Clinical Trial Matching Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A Claude Code skill for matching oncology patients (especially Chinese patients) to clinical trials. Dual-source retrieval across [ClinicalTrials.gov](https://clinicaltrials.gov) + [ChiCTR](https://www.chictr.org.cn), criterion-by-criterion eligibility evaluation, mechanism-aware risk and efficacy contextualization, and a self-contained HTML report.

> ⚠️ Information matching only. Not medical advice. All enrollment decisions must be reviewed by a qualified clinical research team.

---

## Quick start

### Install

```bash
git clone https://github.com/CancerDAO/clinical-trial-matching-skill.git
cp -r clinical-trial-matching-skill/skills/* ~/.claude/skills/
bash ~/.claude/skills/clinical-trial-matching/scripts/setup-chictr-mcp.sh
```

Restart Claude Code.

### Use

In any conversation, describe the patient in natural language:

```
帮我做临床试验匹配:
诊断: 乙状结肠中分化腺癌 IV 期, 双肺/肝转移
分子特征: KRAS G12C, MSS, TMB 7.7
治疗线数: 已完成 2 线 (mFOLFOX6 PR; KELOX+卡瑞利珠+阿帕替尼 PD), 当前三线 KELOX+卡瑞利珠+贝伐进行中
合并症: HTN3 + CAD + 支架术后, 近期肾功能异常
```

Or in English:

```
Shortlist trials for: 69M sigmoid colon adenocarcinoma stage IV (rpT4aN2aM1),
KRAS G12C MSS, post-FOLFOX/KELOX+ICI, currently on 3L KELOX+camrelizumab+bevacizumab.
Severe CV comorbidity. Patient in Shanxi, treated in Beijing.
```

The skill produces `~/Downloads/临床试验匹配报告_{patient_id}_{date}.html` — a self-contained file with patient profile, treatment timeline, top-3 decision paths (each with feasibility, expected efficacy, vs-SoC comparison, risk profile, alternatives, timeline), Goals-of-Care section when triggered, full match inventory, and information-gap action items.

A worked example is at [`skills/clinical-trial-matching/examples/PT-17CE02BC33-*.json`](skills/clinical-trial-matching/examples/).

---

## How it works

1 parent skill orchestrates 4 LLM subskills + a thin Python mechanism layer:

```
┌─ clinical-trial-matching (parent) ─────────────────────────┐
│   Python: dual_source_search → nct_verifier → feasibility  │
│                                                             │
│   For each candidate trial (subagent-dispatched):           │
│     → trial-gater                  (R1–R5 eligibility)     │
│     → trial-risk-annotator         (mechanism × cancer)    │
│     → trial-efficacy-contextualizer (efficacy + per-line SoC)│
│                                                             │
│     → decision-synthesizer          (Top-N + GoC + diversity)│
│                                                             │
│   Python: html_renderer → report.html                       │
└─────────────────────────────────────────────────────────────┘
```

Mechanism stays in Python (deterministic, fast, stdlib-only). Clinical knowledge lives in the LLM subskills as markdown rule files.

---

## Repository layout

Following the [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) convention:

```
skills/
  clinical-trial-matching/         # parent orchestrator + scripts/ + data/ + examples/
  trial-gater/                     # criterion eligibility + R1-R5 hard rules
  trial-risk-annotator/            # per (mechanism × cancer) risk narratives
  trial-efficacy-contextualizer/   # efficacy + per-line SoC comparison
  decision-synthesizer/            # Top-N decision paths + GoC trigger + diversity
```

Each skill has its own `SKILL.md` and a `rules/` directory with prefix-named markdown files (e.g. `R1-prior-same-class-drug.md`, `risk-kras-g12c-by-cancer.md`, `soc-crc-by-line.md`).

For agents working on the codebase, see [`AGENTS.md`](./AGENTS.md).

---

## Adding a new cancer type

> **Code is mechanism. Knowledge is in subskills.**

You should never need to edit Python or extend a JSON lookup table to add clinical knowledge. To support a new cancer:

1. Add the cancer's aliases / chemo regimens to [`skills/clinical-trial-matching/data/clinical_ontology.json`](skills/clinical-trial-matching/data/clinical_ontology.json)
2. Add `skills/trial-efficacy-contextualizer/rules/soc-{cancer}-by-line.md` with the standard-of-care benchmarks per line
3. (Optional) Add `skills/trial-risk-annotator/rules/risk-{mechanism}-{cancer}.md` for cancer-specific mechanism risks
4. Add a worked example to `skills/clinical-trial-matching/examples/`

No code change required. The LLM subskills consume the new rule files automatically on the next invocation.

---

## Data sources

| Source | Coverage | Access |
|---|---|---|
| [ClinicalTrials.gov](https://clinicaltrials.gov) | Global trials registry | API v2 (no key) |
| [ChiCTR](https://www.chictr.org.cn) | Chinese registered trials | [chictr-mcp-server](https://github.com/PancrePal-xiaoyibao/chictr-mcp-server) (Puppeteer; one-line install) |

ChiCTR's site requires browser automation (no JSON API, anti-scraping). The bundled `setup-chictr-mcp.sh` registers the MCP server in your Claude config; if it's unavailable, the skill degrades to ClinicalTrials.gov only and annotates the report.

---

## Compliance posture (patient-facing output)

- No numerical scores shown
- No "推荐" / "recommend" wording — uses "匹配理由" / "match rationale"
- No priority ranking
- Investigator + center info provided so patients can self-contact, but no specific contact directed
- Disclaimer in footer

Internal scoring (feasibility composite, evidence tier, confidence penalty) is preserved for debugging but not rendered.

---

## License & attribution

- **CancerDAO additions**: [MIT](./LICENSE)
- Inspired by [NCBI TrialGPT](https://github.com/ncbi-nlp/TrialGPT) (8-dimension keyword strategy + criterion-level CoT pattern). The original Python package is not vendored. See [`NOTICE.md`](./NOTICE.md).
- ChiCTR access via [chictr-mcp-server](https://github.com/PancrePal-xiaoyibao/chictr-mcp-server) by [PancrePal-xiaoyibao](https://github.com/PancrePal-xiaoyibao).

---

## Contributing

PRs welcome at <https://github.com/CancerDAO/clinical-trial-matching-skill>.

The high-leverage contributions are **rule files for new cancers / drug classes / risk narratives**, not Python changes. See [`AGENTS.md`](./AGENTS.md) for the decision tree on what belongs in code vs rules. See [`CHANGELOG.md`](./CHANGELOG.md) for version history.

Built by [CancerDAO](https://github.com/CancerDAO) — open-source AI for cancer patients.
