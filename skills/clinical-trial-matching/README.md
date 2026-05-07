# clinical-trial-matching

Parent orchestrator skill for end-to-end oncology clinical trial matching with dual-source retrieval (ClinicalTrials.gov + ChiCTR), per-trial LLM analysis, and a self-contained HTML decision report.

## Install

```bash
npx skills add CancerDAO/clinical-trial-matching-skill
```

This installs the parent skill plus 4 subskills it dispatches:
- [`trial-gater`](../trial-gater/) — criterion eligibility + R1-R5 hard rules
- [`trial-risk-annotator`](../trial-risk-annotator/) — per (mechanism × cancer) risk narratives
- [`trial-efficacy-contextualizer`](../trial-efficacy-contextualizer/) — efficacy + per-line SoC
- [`decision-synthesizer`](../decision-synthesizer/) — Top-N decision paths + GoC trigger

Then register the ChiCTR MCP server (one-time):

```bash
bash scripts/setup-chictr-mcp.sh
```

## Structure

- `SKILL.md` — full workflow + prompt contract
- `data/clinical_ontology.json` — cancer aliases, chemo regimens, therapy classes (NO efficacy / risk / SoC — those live in subskill rules)
- `scripts/` — deterministic Python mechanism (parallel HTTP retrieval, NCT live verification, 5-dim feasibility scoring, HTML template fill)
- `examples/` — worked patient + search_plan JSON examples

## Usage

Trigger from any conversation by describing the patient in natural language. See top-level [README](../../README.md) for examples.
