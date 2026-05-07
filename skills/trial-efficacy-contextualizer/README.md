# trial-efficacy-contextualizer

LLM subskill that produces (a) the trial's expected efficacy snapshot for the patient's cancer + mutation context, and (b) a vs-Standard-of-Care head-to-head comparison.

## Install

```bash
# Standalone
npx skills add CancerDAO/clinical-trial-matching-skill --skill trial-efficacy-contextualizer

# With parent skill
npx skills add CancerDAO/clinical-trial-matching-skill
```

## What it does

For each trial, this subskill:

1. Searches for trial-specific published data (CT.gov citations, recent ASCO / ESMO readouts)
2. Falls back through evidence tiers (trial_specific_phase_3 → phase_2 → mutation_class_baseline → drug_class_baseline → no_data) and emits the highest-tier match
3. Lists the patient's SoC options at their current treatment line (using `rules/soc-{cancer}-by-line.md`)
4. Composes a head-to-head comparison narrative

Replaces v1.7.x `efficacy_database.json` + `soc_benchmarks.json`, which had only 1 CRC SoC entry (BRAF V600E 1L) and applied class baselines without mutation-matching (e.g. KRAS G12D drug class baseline applied to a KRAS G12C patient).

## Mandatory grounding

Every efficacy claim must declare:
- `evidence_source.tier` (one of the enumerated tiers)
- `applies_because` — explicit justification linking trial drug class + cancer + mutation to the patient
- `caveats` — list important deviations from the patient context

If no defensible evidence is found, emit `match_type: no_data` rather than fabricating numbers.

## Rules

- [SoC for CRC by line](rules/soc-crc-by-line.md) — 1L through 4L+, including KRAS G12C-specific options
- [SoC for NSCLC by line](rules/soc-nsclc-by-line.md) — EGFR, ALK, KRAS G12C, no-driver subgroups
- [SoC for PDAC by line](rules/soc-pdac-by-line.md) — FOLFIRINOX/AG paradigm + KRAS G12D emerging
- [Output schema](rules/output-efficacy-context-schema.md)

For cancers without a dedicated SoC file, the LLM generates SoC + efficacy from training knowledge.

## See also

- Parent: [`clinical-trial-matching`](../clinical-trial-matching/)
- Sibling subskills: [`trial-gater`](../trial-gater/), [`trial-risk-annotator`](../trial-risk-annotator/), [`decision-synthesizer`](../decision-synthesizer/)
