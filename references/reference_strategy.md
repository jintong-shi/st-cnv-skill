# Reference strategy

How to choose the normal reference, and which tools need one. The reference is the
population a caller treats as copy-number neutral. A poor reference is a common
cause of false positives, so this choice matters as much as the tool choice.

## Which reference to use

1. **Benign epithelium, if you can identify it.** From histology annotation or
   from deconvolution, use benign epithelial spots or cells as the reference. This
   is the preferred setting, because it matches the tumor cells' own cell type and
   isolates the copy-number signal.
2. **Tumor microenvironment, if no benign epithelium is available.** Use stromal
   and immune spots, or another available normal population, as the reference.
   This works but is less matched to the tumor cell type, so read the calls with
   more caution on tumor-free or low-tumor tissue, where over-calling is more
   likely.

Whatever the reference, sanity-check the output against tissue histology. Calls
can invert or over-call when the reference is a poor match.

## Reference mode by tool

Source-verified from the benchmark tool index.

- **Reference required.** CONICSmat, XClone, Numbat. These must be given a
  reference population. They will not run sensibly without one.
- **Reference optional.** InferCNV, CopyKAT, SCEVAN, infercnvpy. These can run
  with or without a reference. Supplying a good benign reference generally
  improves the calls.
- **Reference free.** CopyVAE, STARCH, TUSCAN, CalicoST. These take no reference.
  CalicoST is reference-free even though it is allele-based.

## Tool-specific reference setups

**InferCNV and other observation-versus-reference tools (Task 1).** These tools
score an observation group against a reference group.

- With histology, set benign epithelium as the reference and cancer epithelium as
  the observation. Matching epithelium to epithelium isolates the copy-number
  signal best.
- Without histology, run a deconvolution first, then set TME-enriched spots
  (stroma and immune) as the reference and epithelium-enriched spots as the
  observation.

**CopyKAT for tumor-spot classification on FFPE (Task 3).**

- Try a reference-free run first. On FFPE, CopyKAT with no reference is often more
  accurate than the same tool given a TME reference.
- The reference-free run is stochastic. Run it twice. If the two runs return
  nearly opposite labels, that instability is itself a signal that tumor content
  is very low, and the classification should not be trusted.

## Practical notes

- Reference-free does not mean reference-independent in effect. These tools infer
  a baseline internally, so their calls still depend on the mix of cell types in
  the section.
- On tumor-free or low-tumor sections, the reference choice interacts with the
  over-calling failure mode described in `by_task.md`. A reference drawn from the
  same tissue can help hold down false positives, but no reference choice fully
  removes the risk. Confirm against histology.
- For allele-based tools that require a reference (XClone, Numbat), the reference
  also anchors the allele baseline, so it must come from the same sample and assay
  wherever possible.
