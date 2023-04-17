## Users’ Manual of rvTWAS
### rvTWAS: 

![My Image](Fig1.PNG)

### Installation
rvTWAS is a batteries-included JAR executable. All needed external jar packages are included in the downloadable, rvTWAS.jar. To download all necessary files, users can use the command 
`git clone XXXX`

As we used an R package "susieR" and "SKAT", the users have to install "R", "susieR"(https://cran.r-project.org/web/packages/susieR/susieR.pdf),and "SKAT"(https://cran.r-project.org/web/packages/SKAT/index.html). The versions of "R", "SuSiE", and "SKAT" packages that we have used on our platform are: version 3.6.1 for "R", version 0.12.35 for "susieR", and version 2.0.0 for "SKAT" Users are also expected to have java (version: 1.8) and Plink (version: 1.9) installed on their platform.

Usage:
The rvTWAS model includes two steps. First, rvTWAS selects genetic variants (and their weights) using the GTEx cohort that contains genotype and the gene expression. Second, rvTWAS aggregates the selected (weighted) variants to associate them to the disease phenotype. For each step, rvTWAS provides two alternative methods (1.a) Elastic Net, (1.b) Linear Mixed Model, and (2.a) Linear Combination, (2.b) Weighted Kernel, potentially four configurations. However, the recommended default protocol is (1.b) + (2.b), i.e., linear mixed model plus kernel method.

### 1. Feature selection using SuSiE[REF]:
####1.1 Prepare input data:\
**1.1.1	Gene expression file:** \
Take the whole blood as an example. The fully processed, filtered and normalized gene expression matrices in bed format ("Whole_Blood.v8.normalized_expression.bed") for whole blood was downloaded from GTEx portal (https://gtexportal.org/home/datasets). We included 221 samples in our analysis and removed sex chromosomes. The covariates used in eQTL analysis, including top five genotyping principal components (PCs), were obtained from GTEx_Analysis_v8_eQTL_covariates.tar.gz, which was downloaded from GTEx portal (https://gtexportal.org/home/datasets). Then, we further performed a probabilistic estimation of expression residuals (PEER) analysis to adjust for top five genotyping PCs, age, and other potential confounding factors (PEERs)[2] for downstream prediction model building. There is a description of how to download and use the PEER tool here: https://github.com/PMBio/peer/wiki/Tutorial. The command that we used is shown as below: 

`Rscript ./code/Peer_Script.R`

According to the GTEx protocol, if the number of samples is between 150 and 250, 30 PEER factors should be used. For our study, the number of samples is 221, so we used 60 PEER factors. 

**1.1.2	genotype file:**  
The whole genome sequencing file, GTEx_Analysis_2017-06-05_v8_WholeGenomeSeq_866Indiv.vcf, was downloaded from dbGaP (https://www.ncbi.nlm.nih.gov/projects/gap/cgi-bin/study.cgi?study_id=phs000424.v8.p2). The genotype dataset is quality controlled using the tool PLINK [3] (https://zzz.bwh.harvard.edu/plink/ ). Multiple QC steps were applied by excluding variants with missingness rate > 0.1, high deviations from Hardy-Weinberg equilibrium at p<10-6, and removing samples with missingness rate > 0.1. To be noticed that, we include all common variants, low frequency variants and rare variants. 

The input genotype file ("genotype_file") with the format as below:

 CHR,LOC,GTEX-111CU,GTEX-111FC,GTEX-111YS,GTEX-117YW,GTEX-117YX, … … \
 1,933303,0,0,0,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0, … … \
 1,933411,1,2,2,2,2,2,2,2,2,2,2,2,1,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2, … … \
 1,933653,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,0,0,0,0,0,1,0,0,0, … … \
 … …

**1.1.3	SNP annotation file:** \
The input snp annotation file ("snp_annot_file"), contains annotations for all variants with the format as below (the same format as bim file): 
1       chr1_10291_C_T_b38      0       10291   T       C \
1       chr1_10291_C_T_b38      0       10291   T       C \
1       chr1_10419_T_C_b38      0       10419   C       T \
1       chr1_10423_C_G_b38      0       10423   G       C \
… …


**1.1.4	Gene annotation file:** \
The input gene annot file ("gene_annot_file") is downloaded from GENCODE: https://www.gencodegenes.org/human/release_26.html, in the GTF format and build in GRCh38.


**1.1.5	GWAS file:** \
The input GWAS file ("gwas_file") contains "CHR" and "POS" columns, we just need to make sure that all the SNPs being trained in the SuSiE model can be found in the GWAS dataset.

#### 2. Training SuSiE model:
We processed one chromosome at a time by executing this code, take chromsome 1 as an example:\
`Rscript ./code/Susie_Gene_Chr.R  1`

### 2. Feature aggregation using SKAT[REF]:

