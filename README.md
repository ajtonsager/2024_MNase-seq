# Workflow for producing profiles from MNase-seq bigWig data
Andrew J. Tonsager
## Introduction
This document contains a series of instructions to analyze published MNase-seq datasets starting with process bigWigs (.bw) obtained from the Gene Expression Omnibus (GEO) repository. These instructions will allow for recaptiulating the figures that I've generated for Chapter 3 of my Dissertation, "RNAPII-associated factors maintain nucleosome features over the yeast genome".
## 1. Setup
### Set up conda environments
For dependency reasons, I created two separate conda environments with different packages to perform this analysis. These analyses require both [WiggleTools](https://github.com/Ensembl/WiggleTools) and [deepTools2](https://deeptools.readthedocs.io/en/develop/).
Conda was installed in Linux using the Anaconda distribution, with instructions that can be found here: [Installing Conda in Linux](https://conda.io/projects/conda/en/latest/user-guide/install/linux.html)

To create an environment with the `deeptools` package installed with bioconda, run the following code:
```
conda create -n deeptools
source activate deeptools
conda install bioconda::deeptools
conda deactivate
```
To create an environment with the `wiggletools` package installed with bioconda, run the following code:
```
conda create -n wiggletools
source activate wiggletools
conda install bioconda::wiggletools
conda deactivate
```
### Obtain MNase-seq datasets from GEO
For the analyses conducted in Chapter 3 of my dissertation, I obtained data from the following published datasets:

| Chromatin Factor | Mutant Allele | Reference  | GEO Accession |
| :-------------: | :-------------: | :-------------: | :-------------: |
| Spn1 | *spn1<sup>141-305</sup>* | Tonsager et al., 2024 (preprint) | TBA |
| Spn1 | *spn1<sup>K192N</sup>* | Tonsager et al., 2024 (preprint) | TBA |
| Spn1 | *spn1<sup>K192N</sup>* 37C | Viktorovskaya et al., 2021 | [GSE160821](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE160821) |
| Spt6 | *spt6-YW* 30C, 37C | Viktorovskaya et al., 2021 | [GSE160821](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE160821) |
| Spt6 | *spt6-1004* 30C, 37C | Doris et al., 2018 | [GSE160821](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE115775) |
| Spt6 | *spt6-R,KK* | Connell et al., 2022 | [GSE184955](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE184955) |
| Spt4 | *spt4Δ* | Uzun et al., 2021 | [GSE159291](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE159291)
| Spt5 | *spt5-QS* 30C, 37C | Viktorovskaya et al., 2021 | [GSE160821](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE160821) |
| Spt5 | *spt5-AID* | Evrin et al., 2022 | [GSE181901](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE181901) |
| Spt5 | *spt5-3A* | Evrin et al., 2022 | [GSE181901](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE181901) |
| FACT | *spt16-G132D* 30C, 37C | Feng et al., 2016 | [GSE66215](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE66215) |
| FACT | *pob3-Q308K* | McCullough et al., 2019 | [GSE118332](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE118332) |
| RNAPII | *rpb1-1* 30C, 37C | Feng et al., 2016 | [GSE66215](https://www.ncbi.xyz/geo/query/acc.cgi?acc=GSE66215) |
| RNAPII | *rpb1-TPY,FSP | Connell et al., 2022 | [GSE184955](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE184955) |

When obtaining the data, obtain **bigWigs** (.bw) for this pipeline. If the file type for a dataset of interest is in the **wiggle** (.wig) format, it can still be incorporated into the analysis by first compressing it into a bigWig using the `wigToBigWig` program from the [binary utilities directory from UCSC](https://hgdownload.soe.ucsc.edu/admin/exe/). 

For this analysis, I obtained each dataset for all replicate samples from mutant and control strains. This is because I needed to compare against the authors' internal controls for the analysis.

### Obtain TSS annotatations for master gene list and lists ranked by RNAPII occupancy and length
The list of yeast transcription start sites used for Chapter 3 was derived from a published dataset which determined transcript ends [Pelechano et al., 2013](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3705217/). This list also contains many transcription start sites which produce untranslated transcripts like CUTs, SUTs, and XUTs. These were removed from the list of mapped TSSs, and the resulting 4928 transcription start sites were stored in a new GTF annotation file provided here: `Pelechano.2013_mRNA.gtf`.

This gene list was ranked by Rpb3 occupancy as described in Chapter 3 of my dissertation. Rpb3 ChIP-seq data was kindly provided by Dr. Sarah Swygert from our department, published in [Swygert et al., 2019](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6368455/). Genes were ranked by Rpb3 occupancy in a region spanning  -500 to 0 relative to the mapped TSS locations in the `Pelechano.2013_mRNA.gtf` master gene list. The result is stored in the `Rpb3_quartiles.txt` file, where Quartile Q1 represented the lowest Rpb3 and Q4 represented the highest Rpb3 occupancy. Notably, the column header contains the label `deepTools_group`, which is required for recognition of the different quartiles when plotting the data.

Genes were separately ranked into a list by length, defined as the length between the mapped TSS and TES genomic locations contained within the annotation file.

## 2. Generate median coverage tracks for replicate samples using WiggleTools
### Generate average bigWig tracks 

### Remove 2-micron sequence from wiggle file if present

### Use wigToBigWig to compress average wiggle files



