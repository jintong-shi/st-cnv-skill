# Platform and sparsity

The workflow is decided by data sparsity, not by the platform name. Any spatial
platform can work. The question that matters is how many genes and UMIs each
spatial unit carries, because a CNV caller needs enough genes per unit to estimate
a copy-number profile.

## Sparsity check first

Before choosing a workflow, measure genes and UMIs per spatial unit.

- **10x Visium** is the well-characterized case. Units are dense enough. Run tools
  directly.
- **Any other platform** can work, but check sparsity before running anything. Do
  not assume a platform is dense or sparse from its name alone.

## If the data are sparse

A single spatial unit may carry too few genes for a stable profile. This is the
typical case for VisiumHD at the 8-um bin or at single-cell resolution, and for
imaging-based assays at single-cell resolution.

Workflow: aggregate pure epithelial cells within a fixed radius into meta-cells,
then run the caller on the meta-cells. Aggregating only within one cell type
restores gene coverage without mixing cell types, which would otherwise blur the
copy-number signal. The companion R package implements this end to end, from cell
segmentation through RCTD deconvolution and radius-based meta-cell aggregation to
InferCNV. See the repo at https://github.com/jintong-shi/stMetaCell.

Key point: aggregate within a single, purity-filtered cell type. Mixing tumor and
normal cells into one meta-cell defeats the purpose.

## If the data are dense

No aggregation is needed. For example, the Xenium whole-transcriptome assay is
dense enough to run a strong caller such as InferCNV directly on epithelial cells.

## Summary rule

Sparse, whatever the technology, means aggregate pure epithelial cells into
meta-cells first. Dense means run the caller directly. Decide by the per-unit gene
and UMI counts, not by the platform label.
