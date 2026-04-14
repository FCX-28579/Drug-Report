# Clinical Trial Matching Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A Claude Code / Anthropic Agent **Skill** for matching oncology (and other)
patients to clinical trials with **dual-source retrieval** across
[ClinicalTrials.gov](https://clinicaltrials.gov) and
[ChiCTR](https://www.chictr.org.cn) (中国临床试验注册中心).

Built on top of [NCBI TrialGPT](https://github.com/ncbi-nlp/TrialGPT) and
extended by [CancerDAO](https://github.com/CancerDAO) with a Chinese
clinical workflow, a criterion-level chain-of-thought evaluator, hard
grading rules, a three-stage verification pipeline, and a self-contained
HTML report.

> ⚠️ **This tool provides information matching only. It does not constitute
> medical advice or treatment recommendations.** All enrollment decisions
> must be reviewed by a qualified clinical research team.

---

## What's in this repo

```
clinical-trial-matching-skill/
├── SKILL.md                    # The skill definition (drop into ~/.claude/skills/)
├── LICENSE                     # MIT (CancerDAO additions)
├── NOTICE.md                   # Third-party attributions
└── repo/
    ├── LICENSE.NCBI-TrialGPT   # NCBI public-domain notice + required citation
    ├── README.md               # Original NCBI TrialGPT README
    ├── requirements.txt
    ├── trialgpt_retrieval/
    │   ├── dual_source_search.py      # ← CancerDAO: parallel NCT + ChiCTR search
    │   ├── hybrid_fusion_retrieval.py
    │   └── keyword_generation.py
    ├── trialgpt_matching/             # NCBI TrialGPT-Matching
    ├── trialgpt_ranking/              # NCBI TrialGPT-Ranking
    └── trialgpt_report/
        └── template.html              # ← CancerDAO: 8-section HTML report
```

The TREC-2021 / TREC-2022 / SIGIR evaluation datasets from upstream NCBI
TrialGPT are **not** included here — they're large and unnecessary for the
matching skill. Grab them from
[ncbi-nlp/TrialGPT](https://github.com/ncbi-nlp/TrialGPT) if you want to
reproduce benchmarks.

---

## Key enhancements over upstream TrialGPT

| | Upstream TrialGPT | This skill |
|---|---|---|
| **Data sources** | ClinicalTrials.gov | ClinicalTrials.gov **+ ChiCTR** |
| **Keyword strategy** | Single LLM pass | **8-dimension strategy**: disease-specific, generalized ("solid tumor"), cell-therapy-unconstrained, combo targets, pathway, Chinese keywords, **exhaustive drug-name enumeration**, **resistance-pathway keywords** |
| **Eligibility check** | Criterion-level matching | Same + **hard grading rules R1–R5** to prevent "high match" inflation (prior same-class drug, line-of-therapy mismatch, indication scope, organ-function borderline, ≥2 missing critical fields) |
| **Verification** | — | **3-stage post-hoc verification**: (a) NCT/ChiCTR ID validation via official APIs, (b) completeness review against the search plan, (c) patient-fit re-check |
| **Output** | JSON | **Self-contained HTML report** (8 sections: patient profile, treatment timeline, matching summary, ranked candidates, criterion-by-criterion assessment, excluded directions, referral centers, information gaps + action items) |
| **Compliance** | — | No scores, no "recommend", no priority ranking in patient-facing output |

---

## Installation

### 1. Install the skill

Drop the skill into your Claude Code skills directory:

```bash
git clone https://github.com/CancerDAO/clinical-trial-matching-skill.git
mkdir -p ~/.claude/skills/trialgpt-matching
cp -r clinical-trial-matching-skill/SKILL.md \
      clinical-trial-matching-skill/repo \
      ~/.claude/skills/trialgpt-matching/
```

### 2. Install Python dependencies

```bash
cd ~/.claude/skills/trialgpt-matching/repo
pip install -r requirements.txt
```

Requires Python 3.9+.

### 3. Install the ChiCTR MCP server (required for Chinese trials)

The ChiCTR data source is provided by the external
[chictr-mcp-server](https://github.com/PancrePal-xiaoyibao/chictr-mcp-server)
project. It is **not** vendored into this repo — install it separately.

**Option A — run via npx (recommended):**

```bash
npx -y chictr-mcp-server
```

**Option B — build from source:**

```bash
git clone https://github.com/PancrePal-xiaoyibao/chictr-mcp-server.git
cd chictr-mcp-server && npm install && npm run build && npm start
```

Then register it in Claude Code's MCP config (`~/.claude.json` or a
project-level `.mcp.json`):

```json
{
  "mcpServers": {
    "chictr": {
      "command": "npx",
      "args": ["-y", "chictr-mcp-server"]
    }
  }
}
```

Requires Node.js ≥ 18. Restart Claude Code and verify these tools show up:

- `mcp__chictr__search_trials`
- `mcp__chictr__get_trial_detail`

If the MCP server is unavailable, the skill will degrade gracefully and run
with ClinicalTrials.gov only.

### 4. Configure an LLM API key

The underlying TrialGPT modules call an LLM for criterion-level matching.
See `repo/README.md` for upstream configuration details (OpenAI / Azure /
compatible endpoints).

---

## Usage

Once installed, invoke the skill from any Claude Code conversation with a
patient summary:

```
诊断: 乙状结肠中分化腺癌 IV期, 双肺转移
分子特征: KRAS G12C, MSS, ATM 胚系突变
治疗线数: 5 线 (已用过化疗/靶向/免疫)
```

The skill will:

1. Extract the patient profile
2. Generate an 8-dimension search plan
3. Run parallel retrieval against ClinicalTrials.gov + ChiCTR
4. Score candidates with criterion-level CoT + hard rules R1–R5
5. Verify every NCT/ChiCTR ID against official APIs
6. Emit an HTML report to `~/Downloads/`

See `SKILL.md` for the full workflow and prompt contract.

---

## License & attribution

- **CancerDAO additions** (the skill definition, dual-source orchestrator,
  HTML report, workflow, and grading rules): [MIT](./LICENSE)
- **NCBI TrialGPT** (`repo/trialgpt_matching`, `repo/trialgpt_ranking`,
  parts of `repo/trialgpt_retrieval`): U.S. Government Work in the public
  domain. See `repo/LICENSE.NCBI-TrialGPT` and please cite:

  > Jin Q. et al. *Matching Patients to Clinical Trials with Large Language
  > Models.*

- **chictr-mcp-server** is an external dependency maintained by
  [PancrePal-xiaoyibao](https://github.com/PancrePal-xiaoyibao/chictr-mcp-server).

See [`NOTICE.md`](./NOTICE.md) for details.

---

## Contributing

Issues and PRs welcome at
<https://github.com/CancerDAO/clinical-trial-matching-skill>.

Built by [CancerDAO](https://github.com/CancerDAO) —
open-source AI for cancer patients.
