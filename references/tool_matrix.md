# Full tool capability matrix

Complete capability, mode, and language table for the eleven CNV callers. Every
column here is source-verified against the benchmark tool index. Use it when the
one-line matrix in `SKILL.md` is not enough, for example when you need to know a
tool's native data model, its reference mode, or whether it was eligible for the
synthetic benchmark at all.

## Columns

- **Native model.** Whether the method was designed for dissociated scRNA-seq or
  natively for spatial transcriptomics. scRNA-based tools are applied to ST by
  treating each spatial unit as a cell.
- **Input signal.** Expression means the tool reads relative expression only.
  Allele means it also uses SNP allele frequencies, which requires fresh-frozen
  tissue with a matched BAM.
- **Reference mode.** `required` means the tool must be given a normal reference,
  `optional` means it can run with or without one, `free` means it takes no
  reference.
- **Synthetic-eligible.** Whether the tool could be scored on the synthetic
  benchmark. TUSCAN is the sole exception because it requires an H&E image, which
  the synthetic sections do not have.
- **CNV / Subclone / Tumor-spot.** Which of the three benchmark tasks the tool
  can perform.

## Matrix

| Tool | Language | Native model | Input signal | Reference mode | Needs BAM | Needs H&E | Synthetic-eligible | CNV profile | Subclone | Tumor-spot |
|---|---|---|---|---|---|---|---|---|---|---|
| InferCNV | R | scRNA-based | expression | optional | no | no | yes | yes | yes | no |
| infercnvpy | Python | scRNA-based | expression | optional | no | no | yes | yes | no | no |
| CopyKAT | R | scRNA-based | expression | optional | no | no | yes | yes | no | yes |
| SCEVAN | R | scRNA-based | expression | optional | no | no | yes | yes | yes | yes |
| CopyVAE | Python | scRNA-based | expression | free | no | no | yes | yes | no | yes |
| CONICSmat | R | scRNA-based | expression | required | no | no | yes | yes | no | no |
| XClone | Python | scRNA-based | allele | required | yes | no | yes | yes | no | no |
| Numbat | R | scRNA-based | allele | required | yes | no | yes | yes | yes | yes |
| STARCH | Python | ST-native | expression | free | no | no | yes | yes | yes | no |
| CalicoST | Python | ST-native | allele | free | yes | no | yes | yes | yes | no |
| TUSCAN | R | ST-native | expression | free | no | yes | no | yes | yes | yes |

## Per-tool notes

- **InferCNV** has two modes. HMM mode is the more accurate CNV-profile setting.
  Expression mode is faster and lighter but less accurate. It over-segments badly
  in subclone reconstruction.
- **infercnvpy** is a Python reimplementation of InferCNV, so R and Python users
  can both reach InferCNV-style output. It is scored on the CNV profile task only.
- **CopyKAT** is strong for tumor-spot calling on mixed sections but over-calls on
  tumor-free tissue.
- **SCEVAN** is the only scRNA-based tool that covers all three tasks. It is
  mid-tier on accuracy.
- **CopyVAE** is the cheapest tool on both wall-time and memory, but weak on
  accuracy for every task.
- **CONICSmat** needs a reference and is a top-tier CNV-profile caller on real
  FFPE tissue.
- **XClone**, **Numbat**, and **CalicoST** are the three allele-based tools. They
  run only where fresh-frozen tissue with a matched BAM exists. CalicoST takes no
  reference; XClone and Numbat require one.
- **STARCH** and **CalicoST** take the clone number as an input parameter. It is
  an initial estimate, not a fixed output. CalicoST is robust to this parameter;
  STARCH is sensitive to it.
- **TUSCAN** requires a paired H&E image, which excludes it from the synthetic
  benchmark. Its subclone output tends to follow H&E intensity bands.

See `rankings.md` for the numeric per-task ranking and compute cost.
