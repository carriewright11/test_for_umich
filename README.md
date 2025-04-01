# README for Acute Myeloid Leukemia Heatmap Analysis
Overview
This repository contains an analysis of RNA-seq data related to Acute Myeloid Leukemia (AML). The analysis utilizes a sample dataset from Shih et al., 2017 and is designed to create clustered heatmaps to visualize gene expression patterns in AML.

**Purpose of the Analysis**
The primary goal of this analysis is to explore how mutations in IDH2 and TET2 affect gene expression in AML. We use RNA-sequencing results from 19 acute myeloid leukemia model mice, processed and normalized through refine.bio.

Key Features:
Utilizes a harmonized dataset from refine.bio.
Implements the pheatmap package for generating heatmaps.
Filters genes based on variance to focus on the most informative genes for clustering.
Getting Started
Prerequisites
To run this analysis, ensure you have the following R packages installed:

pheatmap
DESeq2
magrittr
dplyr
readr
tibble
sessioninfo
Installation
To install the required packages, run:

r

Copy
if (!("pheatmap" %in% installed.packages())) {
  install.packages("pheatmap", update = FALSE)
}

if (!("DESeq2" %in% installed.packages())) {
  BiocManager::install("DESeq2", update = FALSE)
}
Setting Up the Analysis Environment
Create necessary directories:
data/ for input data
plots/ for output heatmaps
results/ for filtered gene data
r

Copy
# Create directories
if (!dir.exists("data")) dir.create("data")
if (!dir.exists("plots")) dir.create("plots")
if (!dir.exists("results")) dir.create("results")
Download the Dataset:
Obtain the dataset from refine.bio and place it in the data/SRP070849/ directory.
Analysis Steps
1. Importing Data
Load the metadata and gene expression data into R:

r

Copy
metadata <- readr::read_tsv("data/SRP070849/metadata_SRP070849.tsv")
expression_df <- readr::read_tsv("data/SRP070849/SRP070849.tsv") %>%
  tibble::column_to_rownames("Gene")
2. Filtering Genes by Variance
Select genes with variance in the upper quartile:

r

Copy
variances <- apply(expression_df, 1, var)
upper_var <- quantile(variances, 0.75)
df_by_var <- dplyr::filter(data.frame(expression_df), variances > upper_var)
3. Preparing Metadata for Annotation
Add mutation information based on sample titles to the metadata:

r

Copy
annotation_df <- metadata %>%
  dplyr::mutate(mutation = dplyr::case_when(
    startsWith(refinebio_title, "TET2") ~ "TET2",
    startsWith(refinebio_title, "IDH2") ~ "IDH2",
    TRUE ~ "unknown"
  )) %>%
  dplyr::select(refinebio_accession_code, mutation, refinebio_treatment) %>%
  tibble::column_to_rownames("refinebio_accession_code")
4. Creating and Saving the Heatmap
Generate and save the annotated heatmap:

r

Copy
heatmap_annotated <- pheatmap(df_by_var, 
                               cluster_rows = TRUE, 
                               cluster_cols = TRUE, 
                               show_rownames = FALSE, 
                               annotation_col = annotation_df, 
                               main = "Annotated Heatmap",
                               scale = "row")

# Save as PNG
png("plots/aml_heatmap.png")
heatmap_annotated
dev.off()
5. Session Info
At the end of your analysis, print your session info for reproducibility:

r

Copy
sessioninfo::session_info()
Conclusion
This analysis provides insights into gene expression related to IDH2 and TET2 mutations in AML. The generated heatmaps can help visualize the effects of targeted therapies on gene expression patterns.

For further details and to replicate the analysis, refer to the notebook provided in this repository.