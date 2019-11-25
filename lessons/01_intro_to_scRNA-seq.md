---
title: "Introduction to single-cell RNA-seq"
author: "Mary Piper, Lorena Pantano, Meeta Mistry, Radhika Khetani, Rory Kirchner"
date: Thursday, October 31st, 2019
---

Approximate time: 45 minutes

## Learning Objectives:

* Understand the general steps leading to generation of the count matrix

# Single-cell RNA-seq

## Why single-cell RNA-seq

Across human tissues there is an incredible diversity of cell types, states, and interactions. To better understand these tissues and the cell types present, single-cell RNA-seq (scRNA-seq) offers a glimpse into what genes are being expressed at the level of individual cells. 


<p align="center">
<img src="../img/scRNA-seq_cell_diversity.png" width="600">
</p>


_**Image credit:** courtesy of Dr. Ayshwarya Subramanian_


This exciting and cutting-edge method can be used to:

- explore which cell types are present in a tissue
- identify unknown/rare cell types or states
- elucidate the changes in gene expression during differentiation processes or across time or states
- identify genes that are differentially expressed in particular cell types between conditions (e.g. treatment or disease)
- explore changes in expression among a cell type while incorporating spatial, regulatory, and/or protein information

Popular methods to address some of the more common investigations include:

<p align="center">
<img src="../img/sc_analyses.png" width="600">
</p>

## Challenges of scRNA-seq analysis

Prior to scRNA-seq, transcriptome analysis was performed using bulk RNA-seq, which is straight-forward method for comparing the **averages of cellular expression**. This can be the best choice of method if looking for disease biomarkers or not expecting or not concerned with a lot of cellular heterogeneity in the sample.

While bulk RNA-seq can explore differences in gene expression between conditions (e.g. treatment or disease), the differences at the cellular level are not adequately captured. For instance, in the images below, if analyzed in bulk (left) we would not detect the correct association between the expression of gene A and gene B. However, if we properly group the cells by cell type or cell state, we can see the correct correlation between the genes.

<p align="center">
<img src="../img/sc_vs_bulk_cells.png" width="500">
</p>

_**Image credit:** Trapnell, C. Defining cell types and states with single-cell genomics, Genome Research 2015 (doi:
https://dx.doi.org/10.1101/gr.190595.115)_

Despite scRNA-seq being able to capture expression at the cellular level, sample generation and library preparation is more expensive and the analysis is much **more complicated and more difficult to interpret**. The complexity of analysis of scRNA-seq data involves:

- Large volume of data
- Low depth of sequencing per cell
- Technical variability across cells/samples
- Biological variability across cells/samples

### Large volume of data

Expression data from scRNA-seq experiments represent ten or hundreds of thousands of reads for thousands of cells. The data output is much larger, requiring higher amounts of memory to analyze, larger storage requirements, and more time to run the analyses.

### Low depth of sequencing per cell

For the droplet-based methods of scRNA-seq, the depth of sequencing is shallow, often detecting only 10-50% of the transcriptome per cell. This results in cells showing zero counts for many of the genes. However, in a particular cell, a zero count for a gene could either mean that the gene was **not being expressed** or the transcripts were just **not detected**. Across cells, genes with higher levels of expression tend to have fewer zeros. Due to this feature, many genes will not be detected in any cell and gene expression will be highly variable between cells.

> **Zero-inflated?** scRNA-seq data is often referred to as zero-inflated; however, recent analyses suggest that it does not contain more zeros than what would be expected given the sequencing depth [[Valentine Svensson's blog post](http://www.nxn.se/valent/2017/11/16/droplet-scrna-seq-is-not-zero-inflated)].

### Biological variability across cells/samples

Uninteresting sources of biological variation can result in gene expression between cells being more similar/different than the actual biological cell types/states, which can obscure the cell type identities. Uninteresting sources of biological variation (unless part of the experiment's study) include:

- **Transcriptional bursting:** Gene transcription is not turned on all of the time for all genes. Time of harvest will determine whether gene is on or off in each cell.
- **Varying rates of RNA processing:** Different RNAs are processed at different rates.
- **Continuous or discrete cell identities (e.g. the pro-inflammatory potential of each individual T cell):** Continuous phenotypes are by definitition variable in gene expression, and separating the continuous from the discrete can sometimes be difficult.
- **Environmental stimuli:** The local environment of the cell can influence the gene expression depending on spatial position, signaling molecules, etc.
- **Temporal changes:** Fundamental fluxuating cellular processes, such as cell cycle, can affect the gene expression profiles of individual cells.

<p align="center">
<img src="../img/sc_biol_variability.png" width="500">
</p>

_**Image credit:** Wagner, A, et al. Revealing the vectors of cellular identity with single-cell genomics, Nat Biotechnol. 2016 (doi:https://dx.doi.org/10.1038%2Fnbt.3711)_

### Technical variability across cells/samples

Technical sources of variation can result in gene expression between cells being more similar/different based on technical sources instead of biological cell types/states, which can obscure the cell type identities. Technical sources of variation include:

- **Cell-specific capture efficiency:** Different cells will have differing numbers of transcripts captured resulting in differences in sequencing depth (e.g. 10-50% of transcriptome).
- **Library quality:** Degraded RNA, low viability/dying cells, lots of free floating RNA, poorly dissociated cells, and inaccurate quantitation of cells can result in low quality metrics
- **Amplification bias:** During the amplification step of library preparation, not all transcripts are amplified to the same level.
- **Batch effects:** Batch effects are a significant issue for scRNA-Seq analyses, since you can see significant differences in expression due solely to the batch effect. 

	<img src="../img/batch_effect_pca.png" width="600">
	
	_**Image credit:** [Hicks SC, et al., bioRxiv (2015)](https://www.biorxiv.org/content/early/2015/08/25/025528)_

	To explore the issues generated by poor batch study design, they are highlighted nicely in [this paper](https://f1000research.com/articles/4-121/v1).
	
	**How to know whether you have batches?**
	
	- Were all RNA isolations performed on the same day?
	
	- Were all library preparations performed on the same day?
	
	- Did the same person perform the RNA isolation/library preparation for all samples?
	
	- Did you use the same reagents for all samples?
	
	- Did you perform the RNA isolation/library preparation in the same location?
	
	If *any* of the answers is **‘No’**, then you have batches.
	
	**Best practices regarding batches:**
	
	- Design the experiment in a way to **avoid batches**, if possible.
	
	- If unable to avoid batches:
	
	  - **Do NOT confound** your experiment by batch:
	
	  	<img src="../img/confounded_batch.png" width="300">
	    
	  	_**Image credit:** [Hicks SC, et al., bioRxiv (2015)](https://www.biorxiv.org/content/early/2015/08/25/025528)_
	  
	  - **DO** split replicates of the different sample groups across batches. The more replicates the better (definitely more than 2).
	  
	  	<img src="../img/batch_effect.png" width="300">
	
	  	_**Image credit:** [Hicks SC, et al., bioRxiv (2015)](https://www.biorxiv.org/content/early/2015/08/25/025528)_
	    
	  - **DO** include batch information in your **experimental metadata**. During the analysis, we can regress out the variation due to batch so it doesn’t affect our results if we have that information.

These characteristics make the **analysis of the data more difficult** than bulk RNA-seq. In addition, the analyses can vary depending whether the goal is marker identification, lineage tracing, or some other custom analysis. Therefore, tools specific for scRNA-seq and the different methods of library preparation are needed. 

***

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*