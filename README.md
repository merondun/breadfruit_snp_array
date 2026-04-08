

# Breadfruit SNP Array Development

The goal of this ~10 week project is to identify a set of candidate SNP markers for a 96-SNP breadfruit cultivar identification panel. We will [already have a cohort-level VCF ready](cohort_vcf_calling/README.md) for you encompassing 16 accessions, with the goals being to filter and evaluate SNPs for cultivar discrimination. You will assess SNP importance using, likely, both simple statistics and classification-based approaches, screen loci for technical suitability using flanking-sequence context, and produce a final annotated SNP set suitable for oligo ordering. But, please note that this outline is intended as a tentative roadmap! 

# Background Info 

## Breadfruit and SNP Array Info 

This [2023 paper](https://doi.org/10.1016/j.cub.2022.12.001) has a great overview of our current understanding of breadfruit genetic and geographic diversity. 

The goals will be similar to the fluidigm-scale components of this [lettuce paper](https://doi.org/10.1093/hr/uhac119), and the SNP components of this [melon paper](https://doi.org/10.1186/s12870-023-04056-7). There are also probably some useful components from our lab's recent [pineapple](https://doi.org/10.1007/s00438-025-02275-1) array paper.

Info on our germplasm collection of breadfruit near Hilo: [link](https://www.ars.usda.gov/pacific-west-area/hilo-hi/daniel-k-inouye-us-pacific-basin-agricultural-research-center/tropical-plant-genetic-resources-and-disease-research/docs/breadfruit-collection/) 

Info on breadfruit cultivar varieties from the national botanical garden: [link](https://ntbg.org/breadfruit/about-breadfruit/varieties/) 

## SciNet Overview

Nearly all of our work is done on the SciNet cluster [ceres](https://scinet.usda.gov/guides/use/) . I'm not sure how much background you have on HPC systems, but there's some good ceres-specific resources available, such as [here](https://datascience.101workbook.org/06-hpc/01-hpc-networks/02-scinet-usda-ars/03-scinet-ceres-cluster/#gsc.tab=0) .

I also highly recommend using ceres' [OnDemand](https://ceres-ondemand.scinet.usda.gov/pun/sys/dashboard)  tools: their VScode sessions allow you to request CPUs/RAM so you can troubleshoot / edit scripts directly in the session, save your workspace / folders / scripts for easy access at next log-in, and even easy figure downloading. I'd be happy to give you a quick screenshare overview in case you're curious. 

## Documentation & tools

Ceres has lots of tools available via typical `module load X`. If you are a conda/mamba user, you can access this via:

``` 
module load miniconda
mamba create -n env1 bcftools
source activate env1

#to deactivate
conda deactivate env1
```

You probably already have ways of documenting your code (Git / markdown), but in case not - I highly suggest a markdown editor for project-specific code management. These 'binders' can be great ways to organize your projects, particularly larger bioinfo projects where not everything will run in a single jupyter/rmarkdown doc. I forked over the one-time $15 for a [typora](https://typora.io/) license because I haven't found anything nearly as good for tracking code across dozens of projects, but there are free alternatives like [MarkText](https://marktext.me/) and [Obsidian](https://obsidian.md/). Not necessary, but potentially very useful! Also happy to give a quick markdown crash course if interested. 

## People



| Name                     | Function                             | Photo                                                        |
| ------------------------ | ------------------------------------ | ------------------------------------------------------------ |
| **Dr. Qingyi Yu**        | Primary advisor, Research Geneticist | ![Qingyi](imgs/Qingyi.png) |
| **Dr. Justin Merondun**  | Research advisor, Postdoc            | ![Justin](imgs/Justin.png) |
| **Amberly Buer**         | Lab support, Technician              | ![Amberly](imgs/Amberly.png) |
| **Dr. Zhikai Yang**      | Ancillary support, Postdoc           | ![Zhikai](imgs/Zhikai.png) |
| **Dr. Tracie Matsumoto** | USDA Hilo Plant Unit Research Leader | ![Tracie](imgs/Tracie.png) |

## Metadata

We have long read PacBio HiFi data for 14 breadfruit and 2 closely related species. Breadfruit has recently diversified into different ploidy levels including 3N and 4N. This elevation in ploidy was recent and results from self-duplication (autopolyploid), so we can use the haploid collapsed genome assembly for HART001 as a reference. 

| Accession | Cultivar      | Species                 | Ploidy          | Seeds    | Bases (Gb) | Read Length (Kb) |
| --------- | ------------- | ----------------------- | --------------- | -------- | ---------- | ---------------- |
| HART001   | Maafala       | Artocarpus altilis      | 2N              | Seedless | 59.32      | 11206.3          |
| HART050   | Kukumu tasi   | Artocarpus altilis      | 2N              | Seeded   | 38.96      | 17753.1          |
| HART069   | Ulu Fiti      | Artocarpus altilis      | 2N              | Seeded   | 72.95      | 12674.4          |
| HART030   | Huero         | Artocarpus altilis      | 3N              | Seedless | 96.64      | 18338.2          |
| HART032   | Hamoa         | Artocarpus altilis      | 3N              | Seedless | 88.25      | 15382.2          |
| HART033   | Patara        | Artocarpus altilis      | 3N              | Seedless | 139.76     | 14764.85         |
| HART038   | Lemai         | Artocarpus altilis      | 3N              | Seedless | 42.63      | 18956.6          |
| HART053   | Nahnmwal      | Artocarpus altilis      | 3N              | Seedless | 78.23      | 15006            |
| HART063   | Breadnut      | Artocarpus camansi      | A. camansi      | NA       | 37.9       | 9800.9           |
| HART067   | Dugdug        | Artocarpus mariannensis | A. mariannensis | NA       | 37.35      | 10214.9          |
| ZZ3       | Ulu hamoa     | Artocarpus sp.          | hybrid 2N       | Seeded   | 29.22      | 18949.4          |
| ZZ7       | Ulu afa elise | Artocarpus sp.          | hybrid 2N       | Seeded   | 28.6       | 20738.2          |
| ZZ9       | Ulu afa       | Artocarpus sp.          | hybrid 2N       | Seeded   | 30.02      | 19115.8          |
| HART046   | Midolab       | Artocarpus sp.          | hybrid 3N       | Seedless | 111.19     | 14132.65         |
| HART049   | Meinpohnsakar | Artocarpus sp.          | hybrid 3N       | Seedless | 92.85      | 8248.15          |
| H6        | Meikole       | Artocarpus sp.          | hybrid 4N       | Seeded   | 87.41      | 15742.8          |

We also have > 100 DNA extracts from lots of other cultivars that we will use to test the SNP array once we have the SNP panel. 



# Tentative Outline

Note that this is a just a potential roadmap, not a rigid action plan.

## Highest-priority goals

1. filtered SNP dataset
2. ranked SNP shortlist
3. flanking-sequence screen
4. final annotated 96-SNP panel

Note that typically, we would start with population-level sampling (ideally including a few replicates per cultivar, particularly for the seeded varieties), to identify broadly informative SNPs. Regardless, we should still be able to use this subset of cultivars to identify some useful SNPs.

Overall, the goal is to identify ~96 SNPs (and some back-ups) and their flanking sequences which can delineate our cultivars. We will order those oligos for scalable genotyping on our in-house [biomark x9](https://standardbio.com/products-services/biomark-x9/).

| Week | Overall objective                                            | Suggested tools                                              | Target outputs                                               |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | Familiarize on SciNet ceres (OnDemand VS Code / RStudio / Jupyter are good options) and **generate a filtered SNP dataset** from the provided normalized cohort VCF. Remove low-quality loci and SNPs overlapping repeats from the provided `.bed`. | `bcftools`, `bedtools`, R / Python                           | Clean filtered SNP set; summary of SNP counts and missingness by sample. |
| 2    | Recode SNPs for analysis using ALT allele presence/absence to simplify ploidy allele dosage complications. **Run basic exploratory analyses** to check sample/cultivar structure and identify outliers. | `bcftools query`,  for analysis Python (`pandas`, `scikit-allel`) or R (`vcfR`, `adegenet`) | Analysis-ready SNP matrix; PCA/clustering plots; note on sample outliers. |
| 3    | **Score SNPs for cultivar discrimination** using simple per-marker statistics such as between-cultivar differences, cultivar-specific ALT allele presence, and missingness. | Python (`pandas`, `numpy`) or R (`tidyverse`)                | Ranked SNP table based on diagnostic statistics.             |
| 4    | Run Random Forest **classification to predict cultivar identity** from SNPs and rank markers by feature importance. | Python ([`scikit-learn`](https://scikit-learn.org/stable/auto_examples/inspection/plot_permutation_importance.html?utm_source=openai)) or R ([`ranger`](https://parsnip.tidymodels.org/reference/details_rand_forest_ranger.html?utm_source=openai)) | Random Forest feature-importance table; classification summary/confusion matrix. |
| 5    | **Combine statistical ranking and Random Forest results** to generate a prioritized shortlist of candidate SNPs. Remove obvious redundant markers with highly similar genotype patterns or close physical proximity (e.g. 250 kb / 1 Mb) | Python (`pandas`, `numpy`) or R; optional correlation/LD scripts | Prioritized candidate SNP shortlist.                         |
| 6    | **Screen shortlisted SNPs** for assay suitability by checking flanking-region repeat overlap, nearby variants, and general flanking-sequence quality. Define a preliminary set of top candidate markers for the planned 96-SNP panel. | `bedtools`, `bcftools`, `samtools faidx`, Python or R for tables | Assay-screened candidate marker list; preliminary top panel set. |
| 7    | **Refine the candidate panel** by removing technically weak or redundant loci, confirm cultivar discrimination using in silico testing (for example leave-one-out cross-validation, confusion matrices, and clustering consistency), and prepare final marker annotations including flanking-sequence and QC/design notes. | Python or R; manual review                                   | Refined annotated 96-SNP panel plus backups.                 |
| 8    | **Extract final 96 SNPs** and flanking sequences into a table for ordering oligos, **double check all previous steps.** | `bcftools` / `bedtools` with R or python table checks        | Final annotated SNP table with flanks.                       |
| 9    | Edit, clean, and **finalize bioinformatic documentation, start final report.** | Markdown or similar                                          | Markdown notebook with code.                                 |
| 10   | **Synthesize final report** summarizing methods, filtering, ranking, in silico validation, and recommended 96-SNP panel. |                                                              | Final report; final 96-SNP panel; backup SNP list; some key figures / tables. |



# Potential Random Forest Approach

Just to give an idea of how one might approach the week 4 objectives of using random forest classifications to identify SNPs to predict cultivars, I have included a little snippet below. 

One route for SNP identification is multiclass classification using our SNP matrix against cultivar variety, so random forest is *one* good approach for this. While the ranked SNP table is very useful for telling us about which SNPs are great in isolation for predicting cultivar, we will also want to know which SNPs *together* are useful for predicting cultivars. 

In R, if we have a frame like this:

```R
dummy_data
sample_id   cultivar     snp_1   snp_2   snp_3   snp_4
HART001        Maafala      1       0       1       0
HART050        Kukumu_tasi         0       1       0       1
HART069        Ulu_Fiti        0       1       0       1
```

We can fit a random forest model like this:

```R
library(tidymodels)
library(ranger)

rf_spec <- rand_forest(
  mtry = 2,
  trees = 1000,
  min_n = 1
) %>%
  set_mode("classification") %>%
  set_engine("ranger", importance = "permutation")

rf_fit <- rf_spec %>%
  fit(cultivar ~ . - sample_id, data = dummy_data)
```

We would then assess prediction performance with confusion matrices and rank SNPs by feature importance to identify the most informative candidate markers.





