# Per-task detail and failure modes

Depth behind the Step 3 routing in `SKILL.md`. This file explains the metric each
task uses, the ranking outcome, and the way each tool tends to fail. It gives rank
order and qualitative direction only. For the ranking tables see `rankings.md`.

The organizing finding is task dissociation. CNV profile accuracy, subclone
reconstruction, and tumor-spot identification do not move together across tools.
A strong profile caller is not automatically a strong subclone caller, and the
reverse holds too. Never carry a ranking from one task into another.

## Task 1. CNV profile accuracy

Metric: max-F1 of predicted CNV against matched WES calls.

Outcome. On synthetic ground truth InferCNV HMM mode leads, then CONICSmat. On
real FFPE tissue CONICSmat leads, with both InferCNV modes next, then SCEVAN and
TUSCAN. On fresh-frozen tissue without a BAM, SCEVAN and CopyKAT lead, with
CONICSmat and TUSCAN close behind.

Failure modes.

- Focal events are recovered poorly by every tool. Broad arm-level CNVs are
  reliable, focal CNVs are not. Do not present focal calls as trustworthy.
- The fresh-frozen with BAM split is not a real ranking. It has only 2 samples,
  and nearly every tool sits at chance level. Adding a BAM lets the allele-based
  tools run, but it does not make their profile accuracy interpretable on this
  cohort. This is why the skill does not promote allele tools on Task 1 just
  because a BAM is present.
- CopyVAE is the weakest CNV-profile caller across splits.
- Below roughly 0.5 tumor purity, treat all calls cautiously. InferCNV is the
  most stable across the purity range. Numbat needs higher purity to compete.

## Task 2. Subclone reconstruction

Metric: subclone ARI on synthetic multiclonal sections, spatial assortativity on
real sections. Monoclonal synthetic sections are scored by the predicted clone
count, where the ideal is one and lower is better.

Outcome. CalicoST is clearly the most accurate. It leads on synthetic accuracy,
on real fresh-frozen spatial coherence, and it alone reliably returns a single
clone when the truth is one. It is allele-based and fresh-frozen only.

Failure modes.

- InferCNV recovers CNV profiles well yet strongly over-segments, returning far
  more clones than the truth when only one is present. Treat its clone count as
  inflated. This tool is the clearest single case of task dissociation.
- SCEVAN is near chance on synthetic subclone accuracy.
- Numbat is next-best on synthetic accuracy when it runs, but it can return no
  clones on a section. These are method-level, missing-not-at-random failures.
  Report the failure rate over the full set of sections, and report accuracy only
  on the sections where the tool returned a result.
- TUSCAN gives the highest spatial coherence on FFPE, but this can be an
  artifact. It tends to group clones into concentric bands from tumor core to
  border, driven by H&E intensity, which need not reflect true subclonal
  structure. Spatial coherence cannot verify clone number, so co-report the clone
  count and verify against tissue.
- CalicoST and STARCH both take the clone number as a parameter. CalicoST is
  robust to it. STARCH is sensitive to it, so its output shifts with the setting.
- Manual refinement of subclones is advisable, especially on FFPE.

## Task 3. Tumor-spot identification

Metric: balanced accuracy against histology annotation. Balanced accuracy is the
primary metric because F1 is confounded by tumor prevalence. There is no synthetic
arm for this task.

Outcome. On FFPE, CopyKAT is the most accurate on mixed sections. On fresh-frozen,
Numbat is the most accurate and is the only tool that reliably reports no CNV on
tumor-free tissue.

Failure modes.

- CopyKAT has the lowest specificity of all tools on tumor-free tissue. It
  over-calls when tumor is sparse or absent. Do not trust it when tumor may be
  absent.
- Expression-based callers in general over-call on tumor-free or low-tumor tissue.
  They tend to label all epithelium as tumor, or to invert the labels.
- TUSCAN is mid-tier and can invert the call on all-tumor sections.
- CopyVAE is weak on mixed sections, though acceptable on specificity.
- Numbat is conservative. It under-calls, which is why it has the highest
  specificity on tumor-free tissue. Its negative-control rank rests on a single
  imputed sample.
