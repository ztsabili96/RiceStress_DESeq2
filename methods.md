This document outlines the full analytical workflow used to identify differentially expressed genes in rice under drought stress, starting from raw RNA-seq data downloaded from NCBI SRA and processed entirely within the Galaxy platform.
# 1. Data Acquisition
- Raw RNA-seq data were downloaded from the NCBI Sequence Read Archive (SRA).  
- Species: Oryza sativa
- Tissue: Leaf  
- Conditions: Three biological replicates for control and three for drought stress  
# 2. Preprocessing in Galaxy
All steps were performed on (https://usegalaxy.org):
##Quality Control
- Tool: FastQC  
- Purpose: Assess read quality, GC content, and adapter contamination
## Trimming
- Tool: Trimmomatic
- Purpose: Remove low-quality bases and adapter sequences
## Alignment
- Tool: HISAT2  
- Reference genome: Oryza sativa IRGSP-1.0 (Ensembl Plants)  
- Output: BAM files for each sample  
- Reference: Kim et al., 2015 ([DOI](https://doi.org/10.1038/nmeth.3317))
## Read Counting
- Tool: featureCounts  
- Annotation: GTF file from Ensembl Plants  
- Output: Raw count matrix for each sample  
- Reference: Liao et al., 2014 ([DOI](https://doi.org/10.1093/bioinformatics/btt656))
# 3. Differential Expression Analysis in Galaxy
- Tool: DESeq2 
- Input: Combined count matrix from featureCounts  
- Metadata: Sample condition defined as “control” or “stress”  
- Parameters:
- Design formula: `~ condition`
- Significance thresholds: `padj < 0.05`, `|log2FC| > 1`  
- Output: DESeq2 results table including log2 fold change, p-value, and adjusted p-value  
- Reference: Love et al., 2014 ([DOI](https://doi.org/10.1186/s13059-014-0550-8))
# 4. Post-Processing and Visualization in R
- DESeq2 results were downloaded from Galaxy and imported into R.  
- Genes were categorized based on significance:
  - Upregulated: log2FC > 1 and padj < 0.05
  - Downregulated: log2FC < -1 and padj < 0.05
  - Not Significant: all others
- A volcano plot was generated using “ggplot2” in R:
results <- read.table("DESeq2_result.tabular", header=TRUE, sep="\t", row.names=1)
results$significance <- "Not Significant"
results$significance[results$padj < 0.05 & results$log2FoldChange > 1] <- "Upregulated"
results$significance[results$padj < 0.05 & results$log2FoldChange < -1] <- "Downregulated"
library(ggplot2)
ggplot(results, aes(x=log2FoldChange, y=-log10(padj), color=significance)) +
  geom_point(alpha=0.6, size=1.5) +
  scale_color_manual(values=c("red", "gray", "blue")) +
  theme_minimal() +
  labs(title="Volcano Plot: Stress vs Control",
       x="log2 Fold Change",
       y="-log10 Adjusted p-value")

## 5. Output Files
- `DESeq2_results.tabular`: full results table with log2FC and padj  
- `volcano_plot.png`: visualization of differential expression  
- `volcano_plot.R`: R script for categorization and plotting

## 6. Reproducibility Notes
- All preprocessing and DE analysis steps were performed on Galaxy with default or specified parameters.  
- Visualization and post-processing were performed in R version 4.5.1 using `ggplot2`.  
- The project is structured for clarity and reproducibility in GitHub.
## References
- Kim D, Langmead B, Salzberg SL. HISAT: a fast spliced aligner with low memory requirements. *Nature Methods*. 2015. [https://doi.org/10.1038/nmeth.3317] 
- Liao Y, Smyth GK, Shi W. featureCounts: an efficient general-purpose program for assigning sequence reads to genomic features. *Bioinformatics*. 2014. [https://doi.org/10.1093/bioinformatics/btt656]
- Love MI, Huber W, Anders S. Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biology*. 2014. [https://doi.org/10.1186/s13059-014-0550-8] 
Note: This project was developed as part of a personal learning journey and academic portfolio building. It is intended for educational purposes and demonstration of reproducible bioinformatics practices.
