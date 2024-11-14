# Assignment 2

# Transcriptome Analysis of SARS-CoV-2 Infection in Human Respiratory Cells

## Overview

SARS-CoV-2, the virus responsible for COVID-19, has significant effects on the host transcriptome, particularly in respiratory cells. By comparing transcriptomic profiles between infected and non-infected samples at different time points, we can gain insights into the immune response and pathogenesis of the virus. This project focuses on analyzing miRNA expression levels in cultivated human respiratory cells exposed to SARS-CoV-2 versus mock-infected control cells, with samples collected at 24 and 72 hours post-infection.

---

## Goals

1. **Construct Transcriptome Profiles**: Align and quantify RNA-seq reads to the human reference genome.
2. **Differential Expression Analysis**: Identify differentially expressed genes (DEGs) between:
   - 24 H vs. 72 H time points
   - Mock (control) vs. SARS-CoV-2 infected conditions
3. **Gene Ontology (GO) Enrichment Analysis**: Discover biological processes, molecular functions, and cellular components associated with DEGs to infer potential functional changes in response to infection.

---

## Environment Setup

1. **Connect to HPC**: Log into the Quartz HPC using SSH.
   ```bash
   ssh asrashai@Quartz.uits.iu.edu
   ```

2. **Load Modules and Set Up Conda Environment**:
   Load the Anaconda module and create a dedicated environment for RNA-seq analysis with necessary packages.
   ```bash
   module load miniconda
   conda create -name A2
   conda activate A2
   
   ```

---

## Project Directory Structure

Organize the project files into a structured directory to simplify data management:
```bash
mkdir -p /N/u/asrashai/Quartz/A2/{aligned_reads,counts,fastqc_reports,genome,hisat2_index,Raw_data,trimmed_reads}
```

Define paths for each directory (for easy access in scripts):
```bash
DATA_DIR="/N/u/asrashai/Quartz/A2/Raw_data"
TRIM_DIR="/N/u/asrashai/Quartz/A2/trimmed_reads"
ALIGN_DIR="/N/u/asrashai/Quartz/A2/aligned_reads"
COUNTS_DIR="/N/u/asrashai/Quartz/A2/counts"
FASTQC_DIR="/N/u/asrashai/Quartz/A2/fastqc_reports"
GENOME_DIR="/N/u/asrashai/Quartz/A2/genome"
HISAT2_INDEX_DIR="/N/u/asrashai/Quartz/A2/hisat2_index"
```



## Data Download and Quality Control

1. **Download Data**: Use `prefetch` and `fasterq-dump` to download sequencing reads from NCBI’s SRA for each sample (mock and infected).
2. **Quality Control with FastQC**: Evaluate the quality of raw reads to identify potential issues like low-quality bases or adapter contamination.
3. **Aggregate QC Results with MultiQC**: Summarize FastQC reports for all samples in a single report using MultiQC for a comprehensive view.
4. **Read Trimming**: Use Trim Galore to remove adapters and low-quality regions, ensuring high-quality reads for alignment.

```bash
trim_galore --quality 20 --length 30 -o $TRIM_DIR $DATA_DIR/*.fastq
```


These steps ensure that only high-quality reads proceed to alignment, which is crucial for accurate downstream analysis.


## Reference Genome Preparation

1. **Download Reference Genome**: Obtain the human genome (hg38) and annotation files from UCSC or Ensembl.
2. **HISAT2 Indexing**: Index the genome with HISAT2, which optimizes it for fast and accurate read alignment.

---

## Alignment and Quantification

1. **Align Reads with HISAT2**: Use HISAT2 to align the trimmed reads to the hg38 genome, producing SAM files for each sample. SAM files are converted to BAM files (binary format) for efficient storage and processing.
2. **Quantify Gene Expression with FeatureCounts**: Use `featureCounts` to count the reads mapped to each gene, generating a raw gene count matrix. This count matrix serves as the input for differential expression analysis.

The gene_counts.txt file, generated by featureCounts, was transferred to the local system for further analysis.

## Data Transfer to Local Machine

Transfer result files from the HPC to your local machine for further analysis or sharing:
```bash
scp asrashai@Quartz.uits.iu.edu:/N/u/asrashai/Quartz/A2/counts/gene_counts.txt "C:\Users\asra tasneem\OneDrive\Desktop\gene_counts"
```
---


# Differential Expression Analysis with edgeR in RStudio

After transferring `gene_counts.txt` to the local system, differential expression analysis is conducted in R using edgeR.

1. **Loading Data**  
   Load `gene_counts.txt` (gene counts matrix) and `assignment_2_info.csv` (metadata) into R.

2. **Group and Sample Preparation**  
   Assign samples to experimental groups based on condition (Mock or SARS-CoV-2) and time points (24H or 72H).

3. **Data Normalization**  
   edgeR’s normalization accounts for sequencing depth differences, ensuring accurate comparison. A DGEList object is created, followed by normalization using `calcNormFactors`.

4. **Experimental Design and Model Fitting**  
   - **Design Matrix**: Defines comparisons for differential analysis.
   - **Model Fitting**: edgeR fits a generalized linear model (GLM) to model variability in gene expression across groups.

5. **Setting Contrasts and DEG Testing**  
   Contrasts are defined for:
   - **Condition (SARS-CoV-2 vs. Mock)**: `(SARS_24H + SARS_72H)/2 - (Mock_24H + Mock_72H)/2`
   - **Time Point (72H vs. 24H)**: `(Mock_24H + SARS_24H)/2 - (Mock_72H + SARS_72H)/2`
   
   edgeR’s `glmQLFTest` is used to identify DEGs, which are filtered based on p-values and fold changes, and saved as `DEG_SARS_vs_Mock.csv` and `DEG_72H_vs_24H.csv`.

## GO Enrichment Analysis

Gene Ontology (GO) enrichment analysis helps interpret DEGs by identifying enriched biological processes, molecular functions, and cellular components.

### Steps:

1. *Convert Gene IDs*: Map significant DEGs to Entrez IDs using ClusterProfiler’s bitr function.
2. *Perform GO Enrichment Analysis*: Use the enrichGO function in ClusterProfiler to identify enriched GO terms for DEGs.
3. *Visualization*: Generate bar plots to visualize the top enriched GO terms, providing insights into the biological functions associated with DEGs.

Output:
   - GO_enrichment_1.csv and GO_enrichment_2.csv for enriched terms
   - GO_enrichment_plots.pdf for visualized GO results

---


## Summary of Output Files

- **Differential Expression Results**:
   - `DESeq_results_72H_vs_24H.csv`
   - `DESeq_results_SARS_vs_Mock.csv`
- **Volcano Plots**:
   - `72H_vs_24H_volcanoplot.pdf`
   - `SARS-CoV-2_vs_Mock_volcanoplot.pdf`
- **GO Enrichment Results**:
   - `GO_enrichment_1.csv`
   - `GO_enrichment_2.csv`
   - `GO_enrichment_plots.pdf`

---

















