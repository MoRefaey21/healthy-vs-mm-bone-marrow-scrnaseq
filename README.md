# Single-Cell Dissection of Healthy vs Malignant Bone Marrow Microenvironments

A comparative single-cell RNA-seq analysis of healthy and multiple myeloma bone marrow, completed as a group project for the Single-Cell RNA-Seq course of the Postgraduate Bioinformatics Diploma at Nile University.

**Authors:** Yasmine M. Elgohary, Mohammed Refaey, Tuqa N. Mahmoud, Mohamed M. Wafy

## Question

Multiple myeloma (MM) plasma cells remodel the bone marrow niche by rewiring its intercellular signalling toward tumor support and immune evasion. This project asks how the malignant bone marrow microenvironment differs from a healthy one at single-cell resolution, across three layers: transcriptional state, cell-type composition, and inferred cell-cell communication. The comparison is anchored on cell types present in both samples so state changes are isolated within a stable cellular vocabulary rather than confounded with gross compositional shifts. Beyond the descriptive comparison, the analysis is framed around identifying immune-evasion signalling that could plausibly serve as a therapeutic target.

## Pipeline

The data are two 10X Genomics 3' GEX bone marrow samples from GEO accession GSE176131: GSM5356722 (healthy) and GSM5356726 (multiple myeloma). The Cell Ranger filtered matrices are merged into a single Seurat object with condition annotation and mitochondrial gene quantification. Cell QC uses MAD-based thresholds refined manually against the histograms (final ranges: 300 to 50,000 UMIs, 200 to 6,000 detected features, mitochondrial content below 15%), and gene QC drops features expressed in fewer than three cells. The retained data are normalized with both LogNormalize and SCTransform (3,000 variable features), then scored for cell-cycle phase using `cc.genes.updated.2019`. Dimensionality reduction runs PCA followed by UMAP on the top 20 PCs, with PCA-based diagnostics confirming that technical factors are not the dominant source of variation; cell-cycle variation is regressed out via SCTransform with `vars.to.regress = "Phase"`. Differential expression between malignant and healthy cells is computed with `FindMarkers` (Wilcoxon, log2FC > 0.25, FDR < 0.05) and visualized with EnhancedVolcano and a top-20 heatmap. The object is then split by condition for per-sample clustering: Louvain at resolutions 0 to 1 in steps of 0.1, evaluated with `clustree`, with a final resolution of 0.4 chosen for both. Each sample is annotated with `SingleR` against `HumanPrimaryCellAtlasData` from `celldex` at both cluster and single-cell granularity, then filtered to shared cell types so the downstream communication comparison is like-for-like. CellChat is applied to each sample using `CellChatDB.human`, with over-expressed gene and interaction identification, communication probability inference (`triMean`), pathway-level aggregation, and network centrality. The MIF, GALECTIN, and ANNEXIN pathways receive deeper inspection through bubble plots, hierarchy plots, and per-gene violin plots.

## Results

Quality filtering retained 12,761 of 13,685 input cells and 18,546 of 33,538 detected genes. Cell-cycle scoring assigned 7,929 cells to G1, 2,528 to S, and 2,304 to G2M. Differential expression returned 2,061 significant genes, 1,017 up-regulated in malignant and 1,044 down. The up-regulated set is dominated by immunoglobulin and plasma-cell markers (IGLV3-1, IGLC2, JCHAIN, MZB1, DERL3, CAMP), consistent with an expanded myeloma plasma-cell compartment; the down-regulated set is marked by loss of classical monocyte and myeloid receptors (CYP1B1, ALPL, CCR1, SIGLEC10, CD14, CREB5). Louvain clustering at resolution 0.4 yielded 16 clusters per condition, and SingleR identified 16 shared cell types spanning the hematopoietic hierarchy: HSC-G-CSF, CMP, GMP, MEP, Pro-B and Pre-B cells, B cells, T cells, NK cells, monocytes, myelocytes, pro-myelocytes, neutrophils, erythroblasts, platelets, and a BM compartment. CellChat recovered 454 highly variable ligand-receptor pairs and 36 signalling pathways in healthy versus 502 pairs and 22 pathways in malignant, and the aggregated networks show a 31% reduction in inferred interaction count (872 to 602) and a 41% reduction in cumulative interaction strength (20.142 to 11.876) in the malignant sample. Against this backdrop of overall signalling contraction, three immunomodulatory pathways were examined in detail. MIF (MIF–CD74–CXCR4/CD44) is clearly reduced in the malignant sample (about half the healthy communication probability; share of the network 13% to 11%), and MIF, CD74 and CD44 expression is lower in most cell types. ANNEXIN (ANXA1–FPR1/FPR2) is weaker in absolute terms but takes a larger share of the remaining network (12% to 16%), centred on myeloid cells. GALECTIN (LGALS9–CD44/CD45/P4HB), sent mainly by monocytes, reaches significance in more cell pairs in the malignant sample, including NK and T cells, while individual interactions are slightly weaker. ANNEXIN and GALECTIN are therefore relatively preserved within a suppressed network and are candidate immune-regulatory axes; with one sample per condition these patterns are descriptive.

## Tools

R with Seurat, SCpubr, and scater for the core single-cell workflow; SingleR with celldex (`HumanPrimaryCellAtlasData`) for reference-based annotation; CellChat with `CellChatDB.human` for cell-cell communication inference; EnhancedVolcano and ComplexHeatmap for visualization; clustree for clustering-resolution diagnostics; SCTransform for normalization and variance stabilization; tidyverse for general data wrangling; SeuratDisk and readr for I/O.

## Limitations

The comparison uses one healthy and one malignant sample, so differences in composition, expression and inferred communication are descriptive and cannot be tested statistically. CellChat probabilities were compared between two single-sample networks only.

## Data

The 10X matrices are not redistributed in this repository. They are publicly available from GEO under accession **GSE176131**, with samples **GSM5356722** (healthy bone marrow) and **GSM5356726** (multiple myeloma bone marrow). To reproduce, place the Cell Ranger filtered output under `mtx_data/control/` and `mtx_data/malignant/`, each containing `barcodes.tsv.gz`, `features.tsv.gz`, and `matrix.mtx.gz`.

## Reproducibility

Open `Single-Cell_Dissection_of_Healthy_vs_Malignant_Bone_Marrow_Microenvironments.Rmd` in RStudio with the working directory set to the project root. The script creates `write/figures/`, `write/tables/`, and `checkpoints/` on first run; these directories are not committed.

## Repository contents

- `Single-Cell_Dissection_of_Healthy_vs_Malignant_Bone_Marrow_Microenvironments.Rmd`: full analysis script
- `Single-Cell_Dissection_of_Healthy_vs_Malignant_Bone_Marrow_Microenvironments.pdf`: written report
- `_Presentation__Single-Cell_Dissection_of_Healthy_vs_Malignant_Bone_Marrow_Microenvironments.pdf`: project presentation
- `Markers.csv`: differential expression table, malignant vs healthy (`FindMarkers`, with an Up/Down annotation)
- `README.md`
- `LICENSE`
- `.gitignore`
