# st-cnv-skill

A Claude Agent Skill that recommends copy-number variation (CNV) inference tools
and settings for spatial transcriptomics data. It is derived from a benchmark of
11 CNV callers across synthetic and real tumor datasets.

## What it does

This skill is a decision layer, not a ranking of a single best tool. The benchmark
finding it encodes is task dissociation: CNV profile accuracy, subclone
reconstruction, and tumor-spot identification do not move together across tools,
so the right tool depends on the task, the tissue, and the goal. The skill asks
for the setup, then routes to a tool, a reference strategy, and a workflow. It
advises only. It does not run the tools.

## What it covers

- Tool routing for three tasks: CNV profile accuracy, subclone reconstruction, and
  tumor-spot identification.
- A capability matrix for all 11 tools (language, input signal, reference mode,
  BAM and H&E needs).
- Reference strategy, including per-tool observation-and-reference setups.
- A platform and sparsity workflow, with meta-cell aggregation for sparse data.
- A compute-aware feasibility filter that narrows candidates by RAM ceiling,
  wall-time budget, and dataset scale.

## Structure

```
st-cnv-skill/
  SKILL.md
  references/
    tool_matrix.md
    by_task.md
    reference_strategy.md
    platform_sparsity.md
    rankings.md
```

Rankings are given as relative order only. Absolute accuracy and cost values are
intentionally withheld pending source verification.

## Install

- Claude Code or Cowork: place the `st-cnv-skill/` folder in your skills
  directory, or install the packaged `.skill` file.
- The skill triggers when a user has spatial transcriptomics data and is choosing
  or running a CNV caller.

## Companion R package

The meta-cell aggregation pipeline referenced for sparse and high-resolution data
(cell segmentation, RCTD deconvolution, meta-cell aggregation, InferCNV):
https://github.com/jintong-shi/stMetaCell

## Citation

Benchmark paper: DOI to be added on acceptance.

## License

MIT. See [LICENSE](LICENSE).
