---
name: st-cnv-skill
description: Recommends copy-number variation (CNV) inference tools and settings for spatial transcriptomics data, based on a benchmark of 11 CNV callers across synthetic and real tumor datasets. Use whenever a user has spatial transcriptomics data (any platform, including 10x Visium, VisiumHD, and imaging-based whole-transcriptome assays) and is choosing or running a CNV caller, even if they do not name a specific task. Trigger for questions about which CNV tool to use, how to set a reference, subclone reconstruction, tumor-spot identification, CNV profile accuracy, tool runtime or memory cost, or CNV inference on FFPE versus fresh-frozen sections.
license: MIT
metadata:
  author: Jintong Shi
  version: "0.3.0"
  paper: "TODO: add DOI on acceptance"
  source_repo: https://github.com/jintong-shi/stMetaCell
---

# Spatial transcriptomics CNV tool selector

Guidance for choosing a CNV inference tool and setup for spatial transcriptomics
(ST) data. It is derived from a benchmark of 11 CNV callers on synthetic ST data
with known ground truth and on real tumor sections with matched DNA sequencing or
pathology annotations. This skill advises. It does not run the tools.

The single most important rule: no tool wins across all tasks. Match the tool to
the task, the tissue, and the goal, not to a global best.

## Step 1. Establish the setup

Gather whatever the user has not already stated. Do not assume.

1. Task: CNV profile inference, subclone reconstruction, or tumor-spot identification.
2. Platform and sparsity. Any platform can work. If the data are not standard 10x Visium, first check the gene and UMI counts per spatial unit. Sparsity, not the platform name, decides the workflow (see Platform and sparsity).
3. Tissue and BAM. Fresh-frozen (FF) or FFPE, and whether matched BAM files exist. Allele-based tools need FF tissue with BAM. FFPE probe-based reads carry no allele information.
4. Tumor purity, per sample or per spot, if known.
5. Goal: more accurate or more complete. Precision (few false calls, high confidence) points to a conservative caller. Coverage (catch as many events as possible) points to an expression-based caller.
6. Compute budget and acceptable runtime, scaled to dataset size (number of spots or cells and genes). Allele-based tools scale worst.
7. Language preference: R, Python, or no preference.
8. Whether a matched H&E image is available. TUSCAN requires one.

## Step 2. Tool capability matrix

Eleven tools. "Allele" tools use SNP allele frequencies and need FF tissue with
matched BAM. "H&E" means a paired histology image is required.

| Tool | Language | Input | Needs BAM | Needs H&E | CNV profile | Subclone | Tumor-spot |
|---|---|---|---|---|---|---|---|
| InferCNV | R | expression | no | no | yes | yes | no |
| infercnvpy | Python | expression | no | no | yes | no | no |
| CopyKAT | R | expression | no | no | yes | no | yes |
| SCEVAN | R | expression | no | no | yes | yes | yes |
| CopyVAE | Python | expression | no | no | yes | no | yes |
| CONICSmat | R | expression | no | no | yes | no | no |
| XClone | Python | allele | yes | no | yes | no | no |
| Numbat | R | allele | yes | no | yes | yes | yes |
| STARCH | Python | expression | no | no | yes | yes | no |
| CalicoST | Python | allele | yes | no | yes | yes | no |
| TUSCAN | R | expression | no | yes | yes | yes | yes |

CalicoST and STARCH take the clone number as an input parameter. It acts as an
initial estimate, not a fixed output. infercnvpy is a Python reimplementation of
InferCNV, so R and Python users can both reach InferCNV-style output.

## Step 3. Route by task

### CNV profile inference
- Best on synthetic ground truth: InferCNV in HMM mode, then CONICSmat.
- FFPE real tissue: CONICSmat and InferCNV (both HMM and expression mode) are the top tier, with SCEVAN and TUSCAN following.
- FF without BAM: SCEVAN and CopyKAT lead, with CONICSmat and TUSCAN close behind.
- FF with BAM: this additionally opens XClone and Numbat, but treat any accuracy edge cautiously. On the small BAM cohort most tools sat near chance, so ranking there is noisy.
- Purity: roughly 0.5 tumor purity is a reasonable minimum for most tools. Below it, treat calls cautiously. InferCNV is stable across the purity range. Numbat needs higher purity to become competitive.
- Scale: broad CNVs are recovered better than focal ones. Do not expect reliable focal detection from any tool.
- InferCNV has a faster expression mode without the HMM. It is quicker and lighter but less accurate than the HMM mode.

### Subclone reconstruction
- FF with BAM: CalicoST is clearly the most accurate. It leads on synthetic accuracy, on monoclonal specificity (it alone reliably returns one clone when the truth is one), and on real FF spatial coherence. It is allele-based and FF-only.
- Numbat is the next-best on synthetic accuracy when it runs, but it can fail to return clones on a section.
- InferCNV recovers CNV profiles well but strongly over-segments. It often returns tens of clones when the truth is one. Treat its clone count as inflated.
- FFPE: TUSCAN gives the highest spatial coherence, but this can be an artifact. It tends to group clones into concentric bands from tumor core to border, driven by H&E intensity, which need not reflect true subclonal structure. Verify against tissue and do not trust clone identity on its own.
- SCEVAN and STARCH are mid-to-weak on accuracy.
- Manual refinement of subclones is advisable, especially on FFPE.
- Profile accuracy and subclone accuracy are dissociable. A strong profile caller is not automatically a strong subclone caller, and the reverse holds too.

### Tumor-spot identification
- FFPE: CopyKAT is the most accurate on mixed sections, but it over-calls badly on tumor-free tissue, where its specificity is the lowest of all tools. Do not trust it when tumor is sparse or absent. Before giving it a TME reference, try CopyKAT with no reference first, since the reference-free run is often more accurate than the TME-reference run. The reference-free run is stochastic, so run it twice. If the two runs return nearly opposite labels, tumor content is very low and the classification is unreliable.
- FF: Numbat is the most accurate and the only tool that reliably reports no CNV on tumor-free tissue. It is conservative.
- SCEVAN is second on FFPE. TUSCAN is mid and can invert the call on all-tumor sections. CopyVAE is weak on mixed sections, though acceptable on specificity.
- On tumor-free or low-tumor tissue, expression-based callers over-call. They tend to label all epithelium as tumor, or invert the labels.

### Precision versus coverage
- If the goal is precision, and FF plus BAM are available, Numbat is the conservative choice. It under-calls, so use it to confirm candidate events rather than to maximize coverage.
- If the goal is coverage, expression-based callers such as InferCNV recover more events, at the cost of more false positives.

## Reference strategy

- With histology available, use benign epithelium as the reference and cancer epithelium as the observation. Matching epithelium to epithelium is the preferred setting.
- Without histology, run a deconvolution first, then use TME-enriched spots (stroma and immune) as the reference and epithelium-enriched spots as the observation.
- See references/reference_strategy.md for the per-tool observation-and-reference setups.
- Reference-required tools (CONICSmat, XClone, Numbat) must be given a reference. Reference-free tools (CopyVAE, STARCH, TUSCAN, CalicoST) take none. InferCNV, CopyKAT, SCEVAN, and infercnvpy can run either way.

## Compute and time

Relative cost from the benchmark's real-data runs. Absolute figures depend on
dataset size, so use these as an ordering, not a promise.

- Cheapest: CopyVAE is the fastest and lightest, but weak on accuracy for every task.
- Good balance: InferCNV expression mode and CONICSmat. CopyKAT and SCEVAN are fast but use more memory.
- Slow in wall-time: InferCNV HMM mode, TUSCAN, CalicoST, XClone, and Numbat, with Numbat by far the slowest.
- Memory-heavy: STARCH has by far the largest RAM footprint. TUSCAN and XClone are also high.
- InferCNV's HMM mode is substantially slower and heavier than its expression mode. That is the accuracy-for-speed tradeoff inside one tool.
- Estimate from dataset size before committing to an allele-based tool on a large section.

### Compute-aware feasibility filter

Use this after task routing to drop candidates that will not fit the machine. Ask
the user for three things: the RAM ceiling of the server, the acceptable
wall-time, and the dataset scale, meaning the order of magnitude of spatial units
and genes and whether the data are sparse. Then filter using the relative cost
ordering in `references/rankings.md`, not absolute figures.

- **Tight time budget or large dataset.** Prefer the cheap wall-time tools:
  CopyVAE, SCEVAN, CopyKAT, InferCNV expression mode, CONICSmat. Treat the slowest
  as high risk: the allele-based tools, with Numbat slowest, then XClone and
  CalicoST, followed by InferCNV HMM mode and TUSCAN. Allele-based tools also
  scale worst as the section grows, so their runtime rises faster than the rest.
- **Tight RAM or large dataset.** Avoid the memory-heavy tools first. STARCH is
  the most likely to run out of memory, followed by TUSCAN and XClone. RAM-light
  options are CopyVAE, CalicoST, InferCNV expression mode, CONICSmat, and Numbat.
  CopyKAT and SCEVAN are fast in wall-time but carry a higher memory footprint, so
  on a large dataset with a low RAM ceiling they can still fail.
- **Accuracy for cost inside InferCNV.** HMM mode is the more accurate CNV-profile
  setting but the heavier one. When the budget is tight, fall back to expression
  mode, which is faster and lighter with some accuracy loss.
- **Reduce the problem before switching tools.** If the data are sparse, meta-cell
  aggregation (see Platform and sparsity) lowers the number of units the caller
  sees, which cuts both time and memory directly. Try this before abandoning a
  preferred tool on cost grounds.
- **Probe allele-based tools first.** They scale worst. On a large section,
  estimate from a small subset or a trial run before committing to a full run.

Report the surviving candidates in three bands, recommended, use with caution, and
likely infeasible, and say whether the binding limit is time or memory. The
ordering is relative. Absolute runtime and memory always depend on dataset size,
so present this as guidance, not a guarantee.

## Platform and sparsity

- 10x Visium is the well-characterized case. Run tools directly.
- Any other platform can work, but check sparsity first (genes and UMIs per unit).
- If the data are sparse, whatever the technology, aggregate pure epithelial cells within a fixed radius into meta-cells before running the caller. This restores gene coverage without mixing cell types. VisiumHD at the 8-um bin or at single-cell resolution is the typical sparse case. See the companion R package at https://github.com/jintong-shi/stMetaCell.
- If the data are not sparse, for example the Xenium whole-transcriptome assay, run a strong caller such as InferCNV on epithelial cells directly. No aggregation is needed.

## Scope and caveats

- This guidance comes from one benchmark. Know its limits. The real cohort is FFPE-dominant with few FF sections, so allele-based tools were tested less comprehensively. Every caller was run under default settings with no per-tool tuning. Real-data subclone accuracy was assessed only indirectly through spatial coherence, which cannot verify clone number.
- Always sanity-check automatic CNV output against tissue histology. Calls can invert or over-call.
- When a recommendation rests on few sections or is exploratory, say so rather than presenting it as firm.

## Further detail

Read these only when the question needs them:
- references/tool_matrix.md — full capability, mode, and language table.
- references/by_task.md — per-task ranking detail and failure modes.
- references/reference_strategy.md — reference-mode handling per tool.
- references/platform_sparsity.md — sparsity check and meta-cell workflow.
- references/rankings.md — full per-task ranking and compute-cost tables.

Companion resources:
- Paper: TODO citation and DOI.
- R package (meta-cell aggregation pipeline): https://github.com/jintong-shi/stMetaCell
