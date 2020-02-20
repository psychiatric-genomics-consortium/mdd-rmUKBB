# mdd-rmUKBB
Psychiatric Genomics Consortium (PGC) Major Depressive Disorder (MDD) genome-wide association study meta-analysis removing individual overlap with UK Biobank **(Work in progress)**.

## Project overview

Many uses of genome-wide summary statistics require that there be no sample overlap between the discovery and testing datasets. [UK Biobank](https://www.ukbiobank.ac.uk/) is an open health data set which has been included in previous PGC Major Depressive Disorder GWAS (Wray et al 2018, Howard et al 2019). Because UK Biobank (UKBB) is used by many researchers, we have conducted and released GWAS summary stastics where overlap with UKBB has been excluded.

Datasets used are individual level data from the MDD Wave2 cohorts and summary statistics from the additional MDD cohorts (deCODE, GenScot, GERA, iPsych, 23andMe).

Data for this project are held on [LISA](http://geneticcluster.org/) in the directories listed in the `README.mddw2sum` and `README.mdd00001` files in your LISA home directory. Preimputation QC and imputation was performed previously using the [RICOPILI](https://sites.google.com/a/broadinstitute.org/ricopili) modules.

This project is viewable at [https://psychiatric-genomics-consortium.github.io/mdd-rmUKBB/](https://psychiatric-genomics-consortium.github.io/mdd-rmUKBB/).


### Setup and render notebook

To get an interactive session on LISA and load required programs:

```{bash, eval=FALSE}

#Go into interactive mode
srun -n 16 -t 1:00:00 --pty bash -il

#Load R
module load pre2019
module load R/3.4.3

# download pandoc
curl -L -O https://github.com/jgm/pandoc/releases/download/2.9.1.1/pandoc-2.9.1.1-linux-amd64.tar.gz
tar xzf pandoc-2.9.1.1-linux-amd64.tar.gz


```

Generate main GWAS notebook

```{bash, eval=FALSE}

export PATH=$PATH:pandoc-2.9.1.1/bin
Rscript -e "rmarkdown::render('gwas.Rmd')"

```


## Data Availability

Meta-analyzed summary statistics excluding 23andMe will be available for [download from the PGC](https://www.med.unc.edu/pgc/results-and-downloads/mdd/) as "TBD" (checksum: TBD). Results including 23andMe will be available by contacting the PGC [Data Access Committee](https://www.med.unc.edu/pgc/shared-methods/open-source-philosophy/)

## Requirements

* [Ricopili](https://sites.google.com/a/broadinstitute.org/ricopili/) - Rapid Imputation and COmputational PIpeLIne for conducting GWAS and meta-analysis
* [R project](https://www.r-project.org/) - For preparing phenotype files
* R packages: `readr`, `dplyr`, `stringr`, `tidyr`
* Individual-level data access: Data is held securely on the [LISA server](http://geneticcluster.org/)

## Analysts

### Major Depressive Disorder Working Group of the Psychiatric Genomics Consortium

* **Mark James Adams** - *analyst* - [Edinburgh](https://mhdss.ac.uk)
* **Swapnil Awasthi** - *analyst* - [Broad](https://www.broadinstitute.org/)
* **David Howard** - *analyst* - [KCL](https://www.kcl.ac.uk/)
* **Naomi Wray** - *analytical group director* - [Queensland](https://cnsgenomics.com/)
* **Stephan Ripke** - *analytical group director* - [Broad](https://www.broadinstitute.org/)
* **Andrew McIntosh** - *workgroup chair* - [Edinburgh](https://mhdss.ac.uk)
* **Cathryn Lewis** - *workgroup chair* - [KCL](https://www.kcl.ac.uk/)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* [Major Depressive Disorder Workgroup of the Psychiatric Genomics Consortium](https://www.med.unc.edu/pgc/pgc-workgroups/major-depressive-disorder/)
* The PGC has received major funding from the US National Institute of Mental Health (5 U01MH109528-03).

## Contact

- [Mark James Adams](mailto:mark.adams@ed.ac.uk), [University of Edinburgh](https://www.ed.ac.uk/profile/dr-mark-james-adams).

## References

* Wray, N.R., Ripke, S., Mattheisen, M. et al. Genome-wide association analyses identify 44 risk variants and refine the genetic architecture of major depression. Nat Genet 50, 668–681 (2018). DOI:[10.1038/s41588-018-0090-3](https://doi.org/10.1038/s41588-018-0090-3)
* Howard, D.M., Adams, M.J., Clarke, T. et al. Genome-wide meta-analysis of depression identifies 102 independent variants and highlights the importance of the prefrontal brain regions. Nat Neurosci 22, 343–352 (2019). DOI:[10.1038/s41593-018-0326-7](https://doi.org/10.1038/s41593-018-0326-7)
