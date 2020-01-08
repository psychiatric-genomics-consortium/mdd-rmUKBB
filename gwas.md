---
title: MDD2 GWAS removing overlap with UK Biobank 
author: Mark Adams 
output:
  html_document:
    toc: TRUE
    number_sections: TRUE
    keep_md: true
  md_document:
    variant: markdown_github
---

Recent PGC MDD GWAS ([Wray...Sullivan 2018](https://www.nature.com/articles/s41588-018-0090-3?_ga=2.222013656.112907065.1541203200-2059058911.1541203200), [Howard...McIntosh 2019](https://www.nature.com/articles/s41593-018-0326-7)) contain summary statistics from [UK Biobank](https://www.ukbiobank.ac.uk/) (UKBB). Here we create summary statistics that have UK Biobank participants removed.

There are 959 participants that overlap between PGC MDD and UKBB cohorts. In previous analyses, this overlap was handled by removing overlapping individuals from the UKBB analysis. Thus, becuase these individuals were retained in the PGC cohorts, even versions of the Wray or Howard summary statistics that excluded the UKBB summary statistics from the meta-analysis (leave-one-out [LOO] with reference `noUKBB`) would overlap with the whole UKBB sample. For this analysis, we removed overlapping individuals from the PGC MDD cohorts before conducting the meta-analysis, and refer to these results as `rmUKBB` (**remove** UK Biobank).

Analysis performed on the [LISA cluster](http://geneticcluster.org/) using [Ricopili](https://sites.google.com/a/broadinstitute.org/ricopili/). The R code embedded in this document can be run while this document is rendered but bash code is set to not evaluated but can be submitted to the cluster to reproduce this analysis.

To get an interactive session on LISA and load required programs:


```bash

#Go into interactive mode
srun -n 16 -t 1:00:00 --pty bash -il

#Load R
module load pre2019
module load R/3.4.3

# download pandoc
curl -L -O https://github.com/jgm/pandoc/releases/download/2.9.1.1/pandoc-2.9.1.1-linux-amd64.tar.gz
tar xzf pandoc-2.9.1.1-linux-amd64.tar.gz

```

This document can be generate with


```bash

export PATH=$PATH:pandoc-2.9.1.1/bin
Rscript -e "rmarkdown::render('gwas.Rmd')"

```


# Directory setup

`DIR` is a stand-in for the location of each data set


```bash

mkdir data

# overlap data
ln -s /home/DIR/959_PGC_UKB_overlap.txt data/

# MDD Wave1 data
ln -s /home/DIR/v1/* data/

# BOMA data
ln -s /home/DIR/v1_boma/* data/

# GenRED
ln -s /home/DIR/v1_genred/* data/

```

# PGC MDD2-UKBB overlap

Overlap between PGC MDD and UKBB cohorts were matched by [genotype checksums](https://personal.broadinstitute.org/sripke/share_links/checksums_download/) (see also [Turchin and Hirschhorn 2012](https://academic.oup.com/bioinformatics/article/28/6/886/312495)).


```r
library(readr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(stringr)
library(tidyr)

overlap <- read_table2('data/959_PGC_UKB_overlap.txt', col_names=c('CS', 'FID', 'IID'))
```

```
## Parsed with column specification:
## cols(
##   CS = col_character(),
##   FID = col_character(),
##   IID = col_character()
## )
```

Tally cohorts with overlap


```r
cohorts_count <- 
overlap %>%
filter(CS != 'CheckSum.GenerationScotland.cs') %>%
select(FID) %>%
separate(FID, into=c('FID', 'IID'), sep='\\*') %>%
separate(FID, into=c('status', 'disorder', 'cohort', 'ancestry', 'sr', 'platform'), sep='_') %>% 
group_by(cohort) %>%
tally()

cohorts_count
```

```
## # A tibble: 13 x 2
##    cohort     n
##    <chr>  <int>
##  1 boma       2
##  2 col3       1
##  3 edi2      21
##  4 gep3     100
##  5 grnd       1
##  6 gsk2       2
##  7 i2b3       2
##  8 jjp2       4
##  9 rad3     191
## 10 rot4       1
## 11 shp0       2
## 12 stm2       1
## 13 twg2       9
```

# Phenotypes

For cohorts with overlap, create MDD case/control phenotype files where phenotype of overlapping participants is set to missing (`-9`).


```r
fams <- lapply(cohorts_count$cohort, function(cohort) {

        fam_file <- list.files('data', paste0('mdd_', cohort, '.+\\.fam$'), full.names=TRUE)

        if(length(fam_file > 0)) {
            fam <- read_table2(fam_file, col_names=c('FID', 'IID', 'father', 'mother', 'sex', 'pheno'), col_types='ccccii')
        } else {
           warning(paste('No .fam file for cohort', cohort))
           fam <- NULL
        }

        return(fam)
})
```

```
## Warning in FUN(X[[i]], ...): No .fam file for cohort jjp2
```

```
## Warning in FUN(X[[i]], ...): No .fam file for cohort shp0
```

```r
fams_pheno <- 
bind_rows(fams) %>%
select(FID, IID, pheno)

fams_pheno %>%
group_by(pheno) %>%
tally()
```

```
## # A tibble: 2 x 2
##   pheno     n
##   <int> <int>
## 1     1 15788
## 2     2  8774
```

Only retain phenotypes of participants from cohorts with overlap and set the phenotype of participants in the overlap file to `-9`


```r
rmUKBB_pheno <- 
fams_pheno %>%
left_join(overlap, by=c('FID', 'IID')) %>%
mutate(pheno=if_else(is.na(CS), true=pheno, false=-9L))

rmUKBB_pheno %>%
group_by(pheno) %>%
tally()
```

```
## # A tibble: 3 x 2
##   pheno     n
##   <int> <int>
## 1    -9   331
## 2     1 15608
## 3     2  8623
```

```r
write_tsv(rmUKBB_pheno, 'data/mdd_rmUKBB.pheno', col_names=F)
```

Write out `datasets_info` to list the cohorts to analyze


```r
datasets_info <- str_replace(unlist(sapply(cohorts_count$cohort, function(cohort) list.files('data', paste0('mdd_', cohort, '.+\\.ch\\.fl$')), simplify=TRUE, USE.NAMES=FALSE)), pattern='dasuqc1_', replacement='')

write(datasets_info, 'data/datasets_info', ncol=1) 
```

# GWAS

Run the Ricopoli pipeline


```bash

cd data
postimp_navi --out pgc_MDD13 --addout rmUKBB \
--mds MDD29.0515.nproj.menv.mds_cov \
--coco 1,2,3,4,5,6 --pheno mdd_rmUKBB.pheno \
--popname eur \
--onlymeta --noclump --noldsc --nolahunt

```

Copy anchor cohort single datasets that removed UKBB and link additional datasets for meta-analysis


```bash

mkdir -p data/summary_stats_0120_rmUKBB/single_dataset/additional_datasets

# single datasets
cp data/report_pgc_MDD13_rmUKBB/daner_mdd_*.gz data/summary_stats_0120_rmUKBB/single_dataset/

# additional datasets

# copy GenScot removing UKBB
cp DIR/daner_mdd_genscot_1119a_rmUKBB.gz data/summary_stats_0120_rmUKBB/single_dataset/additional_datasets/

# symlink other additional datasets except for GenScot and UKBB
ln -s DIR/summary_stats_0517/single_dataset/additional_datasets/daner_{GERA,mdd_decode,mddGWAS_new_ipsych}*.gz data/summary_stats_0120_rmUKBB/single_dataset/additional_datasets/

```

Link anchor cohort single datasets that did not need to remove UKBB.


```bash

for daner in $wave2/summary_stats_0517/single_dataset/daner_mdd_*.gz; do
        # get filename
        daner_file=$(basename $daner)
        # check if there is not already a daner file with this name
        if [ ! -f data/summary_stats_0120_rmUKBB/single_dataset/${daner_file} ]; then
                ln -s $daner data/summary_stats_0120_rmUKBB/single_dataset/
       fi
done

```

# Meta-analysis

Meta-analyze PGC MDD anchor cohorts


```bash

mkdir -p data/meta
cd data/meta

# symlink all sumstats into the meta analysis working directory
ln -s ../summary_stats_0120_rmUKBB/single_dataset/daner*.gz . 

# list of anchor cohort sumstats file
ls daner_mdd_*.gz > anchor_dataset_dir

# check that none of the symlinks are broken
for daner in daner_mdd_*.gz; do
        if [ -e $daner ]; then
          echo $daner does exist
        fi
done

# symlink the reference file
ln -s /home/DIR/v1/reference_info .

postimp_navi --result anchor_dataset_dir --onlymeta --out MDD29.0120a.rmUKBB 

```

Copy meta analysis file


```bash

cp data/meta/report_MDD29.0120a.rmUKBB/daner_MDD29.0120a.rmUKBB.gz data/summary_stats_0120_rmUKBB/single_dataset/

```

Set up full meta analysis


```bash

mkdir -p data/full_meta
cd data/full_meta

ln -s ../summary_stats_0120_rmUKBB/single_dataset/daner_MDD29.0120a.rmUKBB.gz .

ln -s ../summary_stats_0120_rmUKBB/single_dataset/additional_datasets/daner_*.gz .

ls daner_*.gz > full_dataset_dir

ln -s ../meta/reference_info .

postimp_navi --result full_dataset_dir --onlymeta --out pgc_mdd_meta_w2_no23andMe_rmUKBB

```
