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
| RNAPII | *rpb1-TPY,FSP* | Connell et al., 2022 | [GSE184955](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE184955) |

When obtaining the data, obtain **bigWigs** (.bw) for this pipeline. If the file type for a dataset of interest is in the **wiggle** (.wig) format, it can still be incorporated into the analysis by first compressing it into a bigWig using the `wigToBigWig` program from the [binary utilities directory from UCSC](https://hgdownload.soe.ucsc.edu/admin/exe/). 

For this analysis, I obtained each dataset for all replicate samples from mutant and control strains. This is because I needed to compare against the authors' internal controls for the analysis.

### Obtain TSS annotatations for master gene list and lists ranked by RNAPII occupancy and length
The list of yeast transcription start sites used for Chapter 3 was derived from a published dataset which determined transcript ends [Pelechano et al., 2013](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3705217/). This list also contains many transcription start sites which produce untranslated transcripts like CUTs, SUTs, and XUTs. These were removed from the list of mapped TSSs, and the resulting 4928 transcription start sites were stored in a new GTF annotation file provided here: `Pelechano.2013_mRNA.gtf`.

This gene list was ranked by Rpb3 occupancy as described in Chapter 3 of my dissertation. Rpb3 ChIP-seq data was kindly provided by Dr. Sarah Swygert from our department, published in [Swygert et al., 2019](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6368455/). Genes were ranked by Rpb3 occupancy in a region spanning  -500 to 0 relative to the mapped TSS locations in the `Pelechano.2013_mRNA.gtf` master gene list. The result is stored in the `Rpb3_quartiles.txt` file, where Quartile Q1 represented the lowest Rpb3 and Q4 represented the highest Rpb3 occupancy. Notably, the column header contains the label `deepTools_group`, which is required for recognition of the different quartiles when plotting the data.

Genes were separately ranked into a list by length, defined as the length between the mapped TSS and TES genomic locations contained within the annotation file. As with the annotation ranked by Rpb3 occupancy, the final column has the `deepTools_group` header, which needs to be unchanged for recognition of the quartiles as separate lists.

## 2. Generate median coverage tracks for replicate samples using WiggleTools
### Generate average bigWig tracks 
WiggleTools was used to calculate average scores from bigWigs coming from replicate datasets from both mutant and control samples. This step was performed for Figures 3.3 - 3.5 of the dissertation. The `mean` function was used to calculate the average score and the `write` function was used to write the scores to a new wiggle file.
```
source activate wiggletools
wiggletools write [sample]-avg.wig mean [sample]-1.bw [sample]-2.bw ... [sample]-n.bw
```

### Remove 2-micron sequence from wiggle file if present
Some of the wiggle files contain scores over the 2-micron plasmid (files from *pob3-Q308K* dataset in this analysis) and they were removed from the wiggle file prior to compression into bigWigs using `sed`.
```
sed -i '/2-micron/d' WT-avg.wig
```

### Use wigToBigWig to compress average wiggle files
The average wiggle files produced with WiggleTools was compressed using `wigToBigWig` tool from the [binary utilities directory from UCSC](https://hgdownload.soe.ucsc.edu/admin/exe/). As stated on the directory, if this utility is freshly downloaded, you will need to update the file system permissions to allow your operating system to run the program.
```
chmod +x ./filePath/wigToBigWig
```
To compress the wiggle file into a bigWig, you need a file containing the size of each chromosome. This is provided in the `sacCer3.chrom.sizes` file, which was obtained from UCSC [here:](https://hgdownload.soe.ucsc.edu/goldenPath/sacCer3/bigZips/).
```
./wigToBigWig [sample]-avg.wig sacCer3.chrom.sizes [sample]-avg.bw
```

## 3. Calculate scores at target genomic regions with computeMatrix
The `computeMatrix` python tool is part of the deepTools2 list of tools. [computeMatrix](https://deeptools.readthedocs.io/en/develop/content/tools/computeMatrix.html) calculates the scores from input files (.bw) over genomic regions specified within the annotation (.gtf or .txt). For this analysis, the `reference-point` option was used, which aligns the reads to provided TSSs and over a desired region upstream and downstream of the TSS without any scaling by gene length. The output is a matrix containing the scores (.gz) which can be used for plotting profiles as performed in this study, or as a heatmap if desired.

To prepare matricies from input averaged bigwigs over the master TSS list, the following general command was used for each mutant and its corresponding wild-type control:
```
source activate deeptools
computeMatrix reference-point -S [WT]-avg.bw [mutant]-avg.bw -R Pelechano.2013_mRNA.gtf -a 1500 -b 500 --missingDataAsZero -o [mutant]-avg.gz
```
The matrix is generated with scores over the inputed region in bp downstream of (after) the TSS `-a` and upstream of (before) the TSS `-b` with these commands. The ``--missingDataAsZero`` option is used to remove missing data where there is no coverage. The data over these regions is treated as zeros. The default bin size `-bs` is 10bp and was used for all analyses in Chapter 3.

For more information on the various options used with computeMatrix, run the following command:
```
computeMatrix reference-point –help
```

To prepare matricies from individual replicates over TSS lists ranked by Rpb3 occupancy or gene length, the following command was used for mutant datasets with their wild-type control samples:
```
computeMatrix reference-point -S [WT]-1.bw [WT]-2.bw ... [WT]-n.bw [mutant]-1.bw [mutant]-2.bw ... [mutant]-n.bw -R Length_quartiles.txt -a 1500 -b 500 --missingDataAsZero -o [mutant]-len.gz
computeMatrix reference-point -S [WT]-1.bw [WT]-2.bw ... [WT]-n.bw [mutant]-1.bw [mutant]-2.bw ... [mutant]-n.bw -R Rpb3_quartiles.txt -a 1500 -b 500 --missingDataAsZero -o [mutant]-expr.gz
```
## 4. Visualize metagene analysis with a profile using plotProfile
The `plotProfile` python tool is also from the deepTools2 list of tools. [plotProfile](https://deeptools.readthedocs.io/en/develop/content/tools/plotProfile.html) creates a plot using the scores and genomic region contained within the matrix outputted from `computeMatrix`. These can be outputted as images or as PDFs (I used .png and .pdf for each).

To generate the images used for average profiles over the master TSS list (Figures 3.3A, 3.4A, 3.5A), the following code was run for each corresponding matrix file:
```
plotProfile -m [mutant]-avg.gz -o [mutant]-avg.pdf --colors black blue --perGroup --samplesLabel WT [mutant]
```
The `--perGroup` option is used to plot each profile onto the same figure. Colors and labels were adjusted using Adobe Illustrator 2024. The colors used is specified with `--colors` and be inputed with either a hexcode (e.g. #eeff22) or by name using [supported colors](https://matplotlib.org/stable/gallery/color/named_colors.html).
To generate the images used for profiles ranked by Rpb3 occupancy or gene length (Figures B.5 - B.10), the following code was run for each corresponding matrix file:
```
plotProfile -m [mutant]-len.gz -o [mutant]-len.png --perGroup --samplesLabel WT-1 WT-2 ... WT-n [mutant]-1 [mutant]-2 ... [mutant]-n
```
For this command, `--perGroup` is used to ensure that the profile from each sample is plotted on the same figure, and each quartile is plotted as a separate figure.
