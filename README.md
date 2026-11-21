# WGCNA-BRCA

Weighted gene co-expression network analysis (WGCNA, Zhang and Horvath, 2005) groups genes whose
expression rises and falls together across samples into modules. This repository implements WGCNA in
four steps and applies it to the mRNA expression of 379 TCGA breast tumors (TCGA-BRCA).

## Requirements

- uv 0.12.23
- Python 3.13.13 or later
- numpy 2.2.6 or later
- scipy 1.17.1 or later

## Input format

The input CSV holds the gene names on the first row, a sample identifier in the first column, and the
numeric expression values in the other columns:

```
,GENE_A,GENE_B
SAMPLE_1,1.2,3.4
```

## Run the steps

Run the commands from the project root. Each step reads its input from `<datadir>` and writes its
output in `<datadir>`. `<dataname>` is the CSV file name, extension included. The examples use
`results/`, which git ignores:

```
mkdir -p results
cp data/tests/mrna2_full.csv results/
```

### Step 1 - keep the K genes of highest variance

```
uv run src/clean_dataset.py <datadir> <dataname> <K>
```

Output: `<datadir>/<dataname>_cleaned_<K>.csv`

### Step 2 - build the gene network

```
uv run src/build_graph.py <datadir> <dataname> POW
uv run src/build_graph.py <datadir> <dataname> TOM <beta>
```

`POW` prints the soft-thresholding power beta: the smallest integer from 1 to 30 whose scale-free fit
reaches `R^2 > 0.8`, or 30 when none does. `TOM` writes the topological overlap matrix
`<datadir>/<dataname>_TOM_<beta>.csv`.

### Step 3 - find the modules

```
uv run src/find_modules.py <datadir> <dataname> static <tau> <l>
uv run src/find_modules.py <datadir> <dataname> adaptive <tau>
uv run src/find_modules.py <datadir> <dataname> dynamic <tau>
```

`tau` is an integer and `l` a real number.
Output: `<datadir>/<dataname>_static_<l>_<tau>.csv`, `<datadir>/<dataname>_adaptive_<tau>.csv` or
`<datadir>/<dataname>_dynamic_<tau>.csv`, holding one module number per gene, in the order of the rows
of the TOM matrix.

### Step 4 - compute the module eigengenes

```
uv run src/extract_eigen.py <datadir> <dataname> <groupname>
```

`<dataname>` is the output of step 1 and `<groupname>` the output of step 3.
Output: `<datadir>/<groupname>_eigen.csv`, with eigengene values rounded to three decimals.

### Full example

```
uv run src/clean_dataset.py results mrna2_full.csv 100
uv run src/build_graph.py results mrna2_full.csv_cleaned_100.csv POW
uv run src/build_graph.py results mrna2_full.csv_cleaned_100.csv TOM 9
uv run src/find_modules.py results mrna2_full.csv_cleaned_100.csv_TOM_9.csv dynamic 15
uv run src/extract_eigen.py results mrna2_full.csv_cleaned_100.csv mrna2_full.csv_cleaned_100.csv_TOM_9.csv_dynamic_15.csv
```

## Run the tests

```
uv run tests/run_tests.py
```

The suite runs every step on the four input files and compares each output with the reference outputs
of `data/tests/190727_current/`. `--quick` limits it to `mrna2_small.csv` with K = 50, `--palier <n>`
to one step, and `--help` lists the other options.

## Project structure

```
WGCNA-BRCA/
|-- data/
|   |-- MANIFEST.json (provenance, transformation and checksums of the input files)
|   |-- DATA_DICTIONARY.md (meaning and format of the input and reference files)
|   |-- pdfs/ (articles of the method)
|   `-- tests/ (input files and reference outputs)
|-- src/
|   |-- csv_utils.py (reading and writing of the CSV files)
|   |-- clean_dataset.py (step 1)
|   |-- build_graph.py (step 2)
|   |-- find_modules.py (step 3)
|   `-- extract_eigen.py (step 4)
|-- tests/
|   `-- run_tests.py (test entry point)
`-- results/ (outputs of the examples)
```

## Data

The input files are the mRNA block of the 379 training tumors of the normalised TCGA-BRCA data built by
the mixOmics team for DIABLO, with each value `x` replaced by `ln(1 + x)`. Their provenance is detailed
in `data/MANIFEST.json`, their meaning and limits in `data/DATA_DICTIONARY.md`.

## Authors

Thomas Morin and Clovis Thoreton.

## References

- Zhang B, Horvath S (2005). A general framework for weighted gene co-expression network analysis.
  *Statistical Applications in Genetics and Molecular Biology*, 4(1), 17. https://doi.org/10.2202/1544-6115.1128
- Langfelder P, Zhang B, Horvath S (2008). Defining clusters from a hierarchical cluster tree: the
  Dynamic Tree Cut package for R. *Bioinformatics*, 24(5), 719-720. https://doi.org/10.1093/bioinformatics/btm563
- Morabito S, Reese F, Rahimzadeh N, Miyoshi E, Swarup V (2023). hdWGCNA identifies co-expression
  networks in high-dimensional transcriptomics data. *Cell Reports Methods*, 3(6), 100498.
  https://doi.org/10.1016/j.crmeth.2023.100498
- Singh A, Shannon CP, Gautier B, Rohart F, Vacher M, Tebbutt SJ, Le Cao KA (2019). DIABLO: an
  integrative approach for identifying key molecular drivers from multi-omics assays. *Bioinformatics*,
  35(17), 3055-3062. https://doi.org/10.1093/bioinformatics/bty1054
- The Cancer Genome Atlas Network (2012). Comprehensive molecular portraits of human breast tumours.
  *Nature*, 490, 61-70. https://doi.org/10.1038/nature11412
