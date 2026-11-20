# TCGA-BRCA mRNA data dictionary

## Why the dataset was created

The Cancer Genome Atlas (TCGA) measured several molecular layers on the same tumors to describe the
molecular diversity of cancers. Its breast invasive carcinoma project (TCGA-BRCA) covers mRNA, miRNA,
DNA methylation, copy number and protein abundance of primary breast tumors.

The mixOmics team built a normalised and filtered version of TCGA-BRCA for DIABLO, a method that
integrates several molecular layers to discriminate breast cancer subtypes (Singh et al., 2019). This
version keeps four blocks per tumor (mRNA, miRNA, methylation, protein), the clinical data and the
PAM50 subtype, and splits the tumors into a training set of 379 samples and a test set of 610 samples.

The files of this repository only contain the mRNA block of the training set. They were not collected
to build a gene co-expression network: the gene selection and the normalisation were made for subtype
classification.

## From patient to table

1. a primary breast tumor is sampled and its mRNA is measured by TCGA;
2. the mixOmics team normalises the measurements for technical biases and keeps 2000 genes;
3. the mRNA block of the 379 training tumors is saved in `TCGA.normalised.mixDIABLO.RData`;
4. each value `x` is replaced by `ln(1 + x)`;
5. the first 50, 175, 300 and 379 tumors give the four files of `data/tests/`.

Exact provenance, transformation and checksums are in `data/MANIFEST.json`.

## What the source does not establish

The files do not state the measurement platform, the normalisation method, nor the rule used to keep
2000 genes. They hold no clinical data and no subtype: the subtype counts in `data/MANIFEST.json` come
from the source file, matched on the row identifiers. A module found in these files therefore describes
the co-expression of 2000 genes chosen for subtype classification, not of the whole transcriptome.

## Input files

| File | Tumors | Genes |
|---|---|---|
| `tests/mrna2_small.csv` | 50 | 2000 |
| `tests/mrna2_medium.csv` | 175 | 2000 |
| `tests/mrna2_large.csv` | 300 | 2000 |
| `tests/mrna2_full.csv` | 379 | 2000 |

| Element | Meaning |
|---|---|
| First row | Empty cell, then one gene symbol per column (`STC2`, `APOD`, ...) |
| First column | Last four characters of the TCGA participant barcode (`A0SH` for `TCGA-XX-A0SH`) |
| Other cells | `ln(1 + x)` of the normalised expression `x` of the gene in the tumor |

## Reference outputs

Each folder of `data/tests/` holds the outputs of the four steps for the four input files, with
K in {50, 100, 250, 500, 750, 1000}, tau in {1, 15, 30} and l in {0.6, 0.9}. Each folder also holds a
`tests_<date>` file listing the commands that produced it and the beta printed by the `POW` option.

| Folder | Content |
|---|---|
| `190727_current/` | Reference used by `tests/run_tests.py`, with copies of the four input files |
| `230626_old/` | Previous reference, output names contain `_groups_` |
| `120626_old/` | First reference, output names contain `_groups_` |

| Output | Format |
|---|---|
| `<dataname>_cleaned_<K>.csv` | Same format as the input file, restricted to the K genes of highest variance |
| `<dataname>_TOM_<beta>.csv` | TOM matrix of the K genes, values only, no header |
| `<dataname>_<option>_[<l>_]<tau>.csv` | Module number of each gene, one line per gene |
| `<groupname>_eigen.csv` | One line per module: module number, then the eigengene value of each tumor |

## References

- Singh A, Shannon CP, Gautier B, Rohart F, Vacher M, Tebbutt SJ, Le Cao KA (2019). DIABLO: an
  integrative approach for identifying key molecular drivers from multi-omics assays. *Bioinformatics*,
  35(17), 3055-3062. https://doi.org/10.1093/bioinformatics/bty1054
- The Cancer Genome Atlas Network (2012). Comprehensive molecular portraits of human breast tumours.
  *Nature*, 490, 61-70. https://doi.org/10.1038/nature11412
