# `repo/` — runtime assets

This directory holds the executable assets used by `SKILL.md` at the
project root.

```
repo/
├── trialgpt_retrieval/
│   └── dual_source_search.py   # Parallel NCT + ChiCTR HTTP retriever (stdlib only)
└── trialgpt_report/
    └── template.html           # 8-section self-contained HTML report
```

## Notes

- **No Python dependencies.** `dual_source_search.py` uses only
  `urllib`, `concurrent.futures`, and `json`. Python 3.9+ on PATH is
  enough.
- **No LLM client here.** All keyword generation, criterion-level
  evaluation, and ranking is performed by Claude in the conversation,
  driven by the prompts in `../SKILL.md`. There is no `OPENAI_API_KEY`
  step.
- **NCBI lineage.** Earlier releases vendored the
  [NCBI TrialGPT](https://github.com/ncbi-nlp/TrialGPT) Python package
  (`trialgpt_matching/`, `trialgpt_ranking/`, and parts of
  `trialgpt_retrieval/`). Those modules called Azure OpenAI directly
  and were never invoked by this skill's workflow, so they have been
  removed. The keyword strategy and criterion-level evaluation pattern
  are conceptually inspired by the NCBI paper — see `../NOTICE.md` for
  citation.
