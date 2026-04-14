# Third-Party Attributions

This repository builds on the following upstream projects. We gratefully
acknowledge their authors.

## NCBI TrialGPT

- Upstream: https://github.com/ncbi-nlp/TrialGPT
- License: U.S. Government Work / Public Domain (see `repo/LICENSE.NCBI-TrialGPT`)
- Scope: the retrieval / matching / ranking modules under `repo/trialgpt_*`
  are derived from NCBI TrialGPT.
- Citation (required by NCBI notice):

  > Qiao Jin, Zifeng Wang, Charalampos S. Floudas, Fangyuan Chen, Changlin
  > Gong, Dara Bracken-Clarke, Elisabetta Xue, Yifan Yang, Jimeng Sun,
  > Zhiyong Lu. *Matching Patients to Clinical Trials with Large Language
  > Models.*

## chictr-mcp-server

- Upstream: https://github.com/PancrePal-xiaoyibao/chictr-mcp-server
- Role: external runtime dependency that provides MCP tools for querying
  ChiCTR (中国临床试验注册中心). Installed separately via `npx`; source is **not**
  vendored into this repository.

## CancerDAO Enhancements

The following additions are contributed by CancerDAO and released under the
MIT license (see `LICENSE`):

- Dual-source search orchestration (`repo/trialgpt_retrieval/dual_source_search.py`)
- HTML report template (`repo/trialgpt_report/template.html`)
- The `SKILL.md` skill definition: 8-dimension keyword strategy,
  criterion-level chain-of-thought evaluation, hard grading rules (R1–R5),
  three-stage verification pipeline, compliance guardrails, and the Chinese
  clinical workflow.
