# Full per-task rankings and compute cost

Detailed backup for the routing calls in `SKILL.md`. Read this when a
recommendation needs the underlying ordering rather than a one-line answer.

## How to read these tables

- Each cell is a rank. Rank uses competition ranking, so ties share the smaller
  rank.
- This file gives rank order only. The underlying median values are held out on
  purpose, because they still need source verification and this file ships
  publicly. Rank order is stable and safe to publish.
- `—` means the tool was not evaluable in that split. It does not mean zero.
  Allele-based tools (Numbat, CalicoST, XClone) run only where a BAM exists, so
  they are absent from FFPE and FF-no-BAM splits.
- `InferCNV (CNV)` is HMM mode. `InferCNV (Expr)` is expression mode. The two
  Numbat rows are the same run scored in HMM and expression mode.
- Aggregation is the per-tool median across samples and replicates, matching the
  boxplot figures.

**Provenance.** Ranks trace to the benchmark summary workbook, overall ranking
sheet, generated 2026-07-19.

**Sample counts (median taken over).** Task 1 FFPE n=8, FF-no-BAM n=12, FF-BAM
n=2. Task 2 real FFPE n=16, FF-BAM n=8. Task 3 FFPE n=10, FF-BAM n=5. Synthetic
Task 1 and Task 2 use 20 sections, 15 multiclonal and 5 monoclonal.

---

## Task 1. CNV profile accuracy

Metric: max-F1 against matched WES CNV calls. Higher is better. Neg-ctrl is the
fraction of the 2 CNV-negative samples correctly called CNV-free.

| Tool | Synthetic | Real FFPE | FF-no-BAM | FF-BAM | Neg-ctrl (frac of 2) |
|---|---|---|---|---|---|
| CONICSmat | 2 | 1 | 4 | 6 | 0 |
| CopyKAT | 3 | 4 | 2 | 12 | 0 |
| InferCNV (CNV) | 1 | 2 | 6 | 8 | 0 |
| TUSCAN | — | 6 | 3 | 5 | 0 |
| InferCNV (Expr) | 4 | 3 | 7 | 7 | 0 |
| SCEVAN | 9 | 5 | 1 | 3 | 0 |
| infercnvpy | 5 | 7 | 5 | 1 | 0 |
| Numbat (CNV) | 6 | — | — | 13 | 1/2 |
| Numbat (Expr) | 6 | — | — | 2 | 1/2 |
| STARCH | 8 | 9 | 8 | 9 | 0 |
| CopyVAE | 12 | 8 | 9 | 11 | 0 |
| XClone | 10 | — | — | 10 | 0 |
| CalicoST | 11 | — | — | 4 | 0 |

Reading notes.

- Synthetic ground truth: InferCNV HMM mode leads, then CONICSmat. This drives
  the synthetic routing in `SKILL.md`.
- Real FFPE: CONICSmat first, both InferCNV modes next, then SCEVAN and TUSCAN.
- FF-no-BAM: SCEVAN and CopyKAT lead, CONICSmat and TUSCAN close behind.
- **FF-BAM is noise, not signal.** Only 2 samples (skin_P6_V3/V4), and nearly
  every tool sits at chance level. Do not read the FF-BAM column as a real
  ordering. It is excluded from any overall ranking. This is why the skill
  de-ranks allele-based tools here rather than promoting them just because BAM
  finally lets them run.
- Neg-ctrl = fraction of the 2 CNV-negative samples (skin_P4_V1/V2) correctly
  called CNV-free. Only Numbat scored above zero, at 1 of 2. Not ranked.

## Task 2. Subclone reconstruction

Metric: subclone ARI on synthetic multiclonal sections; spatial assortativity on
real sections. The monoclone column ranks the predicted subclone count when the
truth is one clone, where the ideal is one and lower is better.

| Tool | Syn multiclonal ARI | Real FFPE (assort.) | Real FF-BAM (assort.) | Syn monoclone (pred #, ideal 1) |
|---|---|---|---|---|
| CalicoST | 1 | — | 1 | 1 |
| TUSCAN | — | 1 | 2 | — |
| STARCH | 3 | 3 | 3 | 2 |
| SCEVAN | 5 | 2 | 4 | 4 |
| Numbat | 2 | — | 6 | 2 |
| InferCNV | 4 | 4 | 5 | 5 |

Reading notes.

- CalicoST leads on synthetic accuracy, on real FF spatial coherence, and it
  alone reliably returns one clone when the truth is one. It is allele-based and
  FF-only.
- Numbat is next on synthetic accuracy when it runs, but it can fail to return
  clones on a section.
- InferCNV recovers profiles well yet strongly over-segments, returning far more
  clones than the truth. Treat its clone count as inflated.
- TUSCAN gives the highest FFPE assortativity, but this can be an H&E-driven
  banding artifact rather than true subclonal structure. Assortativity cannot
  verify clone number.
- STARCH and Numbat sit at the same predicted-clone rank on monoclonal sections.

## Task 3. Tumor-spot identification

Metric: balanced accuracy against histology annotation. No synthetic arm. Higher
is better, except the neg-ctrl column, which ranks specificity on tumor-free
tissue.

| Tool | Real FFPE | Real FF-BAM | Neg-ctrl specificity |
|---|---|---|---|
| Numbat | — | 1 | 1 |
| CopyKAT | 1 | 2 | 5 |
| TUSCAN | 3 | 3 | 4 |
| SCEVAN | 2 | 5 | 3 |
| CopyVAE | 4 | 4 | 2 |

Reading notes.

- CopyKAT is most accurate on mixed FFPE sections but has the worst specificity
  of all tools on tumor-free tissue. It over-calls when tumor is sparse or absent.
- Numbat is most accurate on FF and the only tool that reliably reports no CNV on
  tumor-free tissue, so it ranks first on specificity. It is conservative. Its
  neg-ctrl rank rests on 1 imputed sample (V1_2).
- SCEVAN is second on FFPE. TUSCAN is mid. CopyVAE is weak on mixed sections but
  acceptable on specificity.

## Compute cost

Real-data runs only. Rank 1 is best, meaning fastest wall-time or least memory.
Rank order only. Absolute figures are held out for the same reason as the accuracy
medians. Use the ordering, and see the qualitative bands in the `SKILL.md` compute
section.

| Tool | Wall-time | Peak RAM |
|---|---|---|
| CopyVAE | 1 | 1 |
| SCEVAN | 2 | 9 |
| CopyKAT | 3 | 8 |
| InferCNV (Expr) | 4 | 3 |
| CONICSmat | 5 | 4 |
| infercnvpy | 6 | 7 |
| STARCH | 7 | 12 |
| InferCNV (CNV) | 8 | 6 |
| TUSCAN | 9 | 11 |
| CalicoST | 10 | 2 |
| XClone | 11 | 10 |
| Numbat | 12 | 5 |

Reading notes.

- CopyVAE is cheapest on both axes but weak on accuracy for every task.
- InferCNV expression mode and CONICSmat are the good balance. CopyKAT and SCEVAN
  are fast but memory-heavy.
- Numbat is by far the slowest in wall-time. STARCH has by far the largest memory
  footprint.
- InferCNV HMM mode is much slower and heavier than its expression mode, which is
  the accuracy-for-speed tradeoff inside one tool.
- Numbat is a single run, so its HMM and expression scoring share one cost. All
  12 tools have real timing, including TUSCAN.
