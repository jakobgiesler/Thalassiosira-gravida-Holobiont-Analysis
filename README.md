Analysis code for the manuscript *Microbiome-mediated enhancement of Arctic diatom abiotic acclimation* (Giesler et al., *in prep.*).

The project compares a diatom grown **with its associated bacteria (xenic)** and **without bacteria (axenic)** under two temperatures (9 °C and 13.5 °C) and two photoperiods (16 h and 24 h light). It combines:

- **growth rates** of xenic and axenic cultures,
- **host transcriptomics** (poly-A selected libraries) to find diatom genes that respond to the presence of bacteria,
- **bacterial metatranscriptomics** (rRNA-depleted libraries) to find bacterial genes that respond to the treatments,
- **host–microbiome co-expression** analysis linking diatom and bacterial gene clusters.

## Repository contents

| File | Description |
|---|---|
| `01_growthrates.Rmd` | Specific growth rates (µ) per treatment, three-way ANOVA (bacteria × temperature × photoperiod) with Tukey HSD, and effect sizes (Cohen's d) for temperature and photoperiod within axenic and xenic cultures. |
| `02_diff_expression_polyA.Rmd` | Differential expression of diatom genes (DESeq2) between xenic and axenic cultures, separately for each temperature × photoperiod combination. PCA, sample-distance heatmap, UpSet plot of shared DE genes, and annotated tables of up- and down-regulated genes. |
| `03_diff_expression_rRNAdepleted.Rmd` | Differential expression of bacterial genes across treatments (DESeq2 likelihood-ratio test); exports VST-normalised counts of significant genes (`DE_vst_bacteria.txt`). |
| `04_enrichment_analysis_polyA.Rmd` | COG category composition of genes consistently up- or down-regulated in xenic cultures compared with the reference transcriptome (radar plots), hypergeometric enrichment tests, and selection of significantly affected COG categories. |
| `05_host_microbiome_coexpression.Rmd` | k-means clustering of DE diatom and bacterial genes, correlation of cluster centroids, global host–microbiome network, co-expression modules, COG composition and enrichment of clusters, taxonomic composition of bacterial clusters, and PFAM odds ratios. |

## Workflow

```
01_growthrates.Rmd                      (independent)

02_diff_expression_polyA.Rmd ──► 04_enrichment_analysis_polyA.Rmd ──┐
                                                                    ├──► 05_host_microbiome_coexpression.Rmd
03_diff_expression_rRNAdepleted.Rmd ────────────────────────────────┘
```

1. `01_growthrates.Rmd` compares growth of xenic and axenic cultures across treatments.
2. `02_diff_expression_polyA.Rmd` identifies diatom genes that are differentially expressed between xenic and axenic cultures in all four treatment combinations.
3. `03_diff_expression_rRNAdepleted.Rmd` produces the normalised expression table of differentially expressed bacterial genes.
4. `04_enrichment_analysis_polyA.Rmd` tests which COG categories are over- or under-represented among the diatom genes from script 02. The significant categories (M, O, W, Q, F, I, P, A, B, J, L) define the diatom gene set used for co-expression.
5. `05_host_microbiome_coexpression.Rmd` clusters the diatom and bacterial gene sets and analyses how host and bacterial clusters co-vary.

## Data

Raw RNA sequencing reads are available at the European nucleotide archive *PRJEB106470*. Growth data are available at PANGAEA: *https://doi.org/10.1594/PANGAEA.992592*.

The scripts expect the following inputs:

| Script | Input files |
|---|---|
| `01_growthrates.Rmd` | Growth data table from PANGAEA |
| `02_diff_expression_polyA.Rmd` | featureCounts tables in `count_tables/` (`*.counts.short.txt`), sample metadata `wp2_meta.xlsx`, eggNOG-mapper annotation of the diatom reference transcriptome |
| `03_diff_expression_rRNAdepleted.Rmd` | Cleaned bacterial featureCounts tables in `cleaned_bac_counts/` (`*.counts_cleaned.txt`), sample metadata `DE_bac_meta.txt` |
| `04_enrichment_analysis_polyA.Rmd` | eggNOG-mapper annotation of the diatom reference transcriptome, `upregulated_in_xe.xlsx` and `downregulated_in_xe.xlsx` (from script 02), `cog_categories.xlsx` |
| `05_host_microbiome_coexpression.Rmd` | Files in `data/`, listed at the top of the document |

## Running the analyses

1. Clone the repository and place the input data as described above.
2. In scripts `01`–`04`, replace `setwd("/PATH")` with the folder that contains your data. `05_host_microbiome_coexpression.Rmd` uses relative paths and expects a `data/` folder next to it; outputs are written to `output/`.
3. Knit the documents in numerical order, e.g. in RStudio or with

   ```r
   rmarkdown::render("02_diff_expression_polyA.Rmd")
   ```

## Requirements

Analyses were run in R (≥ 4.3). Required packages:

```r
# CRAN
install.packages(c(
  "tidyverse", "readxl", "writexl", "knitr", "rmarkdown", "ggthemes", "ggforce",
  "RColorBrewer", "pheatmap", "UpSetR", "qualpalr", "viridis", "reshape2", "igraph",
  "growthrates", "multcompView", "emmeans", "effectsize", "BiocManager", "remotes"
))

# Bioconductor
BiocManager::install("DESeq2")

# GitHub
remotes::install_github("ricardo-bion/ggradar")
```

The exact package versions used are printed by `sessionInfo()` at the end of `05_host_microbiome_coexpression.Rmd`.

