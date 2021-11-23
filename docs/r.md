# R

[TOC]

## General R resources

If you are looking for some help getting started with R or taking your R to the next level, check out these useful resources:

* [Swirl - interactive R learning](https://swirlstats.com/students.html)
* [Tidyverse workshop and resources](https://github.com/katiesevans/nuit_tidyverse)
* [Andersen Lab R Knowledge base & Cheatsheet](https://github.com/AndersenLab/code_club/blob/main/R_cheatsheet.md)
* [R-bloggers - tips and tricks](https://www.r-bloggers.com/)
* [Using Rprojects to organize scripts](https://www.r-bloggers.com/2020/01/rstudio-projects-and-working-directories-a-beginners-guide/)
* [Using workflowR for reproducible work](https://www.rdocumentation.org/packages/workflowr/versions/0.7.0)
* [Using Rmarkdown to generate reports](https://rmarkdown.rstudio.com/articles_intro.html)
* Also check out the `lab_code` slack channel for help/questions!

## Andersen Lab R Packages

The Andersen lab maintains several R packages useful for high-throughput data analysis.

### cegwas2

This package contains a set of functions to process phenotype data, perform GWAS, and perform post-mapping data processing for *C. elegans*. In 2019, the `cegwas2-nf` Nextflow pipeline was developed to perform GWA mapping on QUEST using this cegwas2 R package. However, mapping is rarely if never done with cegwas2 in R manually. To learn more about the `cegwas2` R package, see the [andersenlab/cegwas2](https://github.com/andersenlab/cegwas2) repo. For help running a GWA mapping using cegwas2, see [cegwas2-nf](https://github.com/andersenlab/cegwas2-nf) or [the dry guide](pipeline-cegwas.md)

!!! Note
	`cegwas2` was preceeded by `cegwas` and is now (as of 2021) superceeded by [`NemaScan`](https://github.com/AndersenLab/NemaScan)

### linkagemapping

This package includes all data and functions necessary to complete a mapping for the phenotype of your choice using the recombinant inbred lines from [Andersen, et al. 2015 (G3)](https://academic.oup.com/g3journal/article/5/5/911/6025542). Included with this package are the cross and map objects for this strain set as well a markers.rds file containing a lookup table for the physical positions of all markers used for mapping. To learn more about linkagemapping including how to install and use the package, check out the [andersenlab/linkagemapping](https://github.com/andersenlab/linkagemapping) repo.

!!! Note
	Also check out the [linkagemapping-nf](https://github.com/andersenlab/linkagemapping-nf) repo for a reproducible Nextflow pipeline for linkage mapping and two-dimensional genome scans (scan2) for one or several traits.

### COPASutils

The R package `COPASutils` provides a logical workflow for the reading, processing, and visualization of data obtained from the Union Biometrica Complex Object Parametric Analyzer and Sorter (COPAS) or the BioSorter large-particle flow cytometers. Data obtained from these powerful experimental platforms can be unwieldy, leading to difficulties in the ability to process and visualize the data using existing tools. Researchers studying small organisms, such as Caenorhabditis elegans, Anopheles gambiae, and Danio rerio, and using these devices will benefit from this streamlined and extensible R package. `COPASutils` offers a powerful suite of functions for the rapid processing and analysis of large high-throughput screening data sets. To learn more about COPASutils including how to install and use the package, check out the [andersenlab/COPASutils](https://github.com/andersenlab/copasutils) repo and the COPASutils [manuscript](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0111090)

### easysorter

This package is effectively version 2 of the `COPASutils` package. This package is specialized for use with worms and includes additional functionality on top of that provided by COPASutils, including division of recorded objects by larval stage and the ability to regress out control phenotypes from those recorded in experimental conditions To learn more about easysorter including how to install and use the package, check out the [andersenlab/easysorter](https://github.com/andersenlab/easysorter) repo. Here are some of the papers using `easysorter`:

- __The first easysorter paper__
A Powerful New Quantitative Genetics Platform, Combining *Caenorhabditis elegans* High-Throughput Fitness Assays with a Large Collection of Recombinant Strains ([Andersen et al. 2015](https://academic.oup.com/g3journal/article/5/5/911/6025542))

- __The first "V3" easysorter paper__
The Gene *scb-1* Underlies Variation in *Caenorhabditis elegans* Chemotherapeutic Responses ([Evans and Andersen 2020](https://academic.oup.com/g3journal/article/10/7/2353/6026329))

- __The first dominance/hemizygosity easysorter paper__
A Novel Gene Underlies Bleomycin-Response Variation in *Caenorhabditis elegans* ([Brady et al. 2019](https://academic.oup.com/genetics/article/212/4/1453/5931413))

*Almost every paper published from the lab has used easysorter, for more, check out [our lab papers](https://andersenlab.org/Publications/)*

!!! Note
	The `easysorter` package requires `COPASutils` installation as well.

### easyXpress

This package is designed for the reading, processing, and visualization of images obtained from the Molecular Devices ImageExpress Nano Imager, and processed with CellProfiler's WormToolbox. To learn more about easyXpress including how to install and use the package, check out the [andersenlab/easyXpress](https://github.com/andersenlab/easyXpress) repo and the [easyXpress manuscript](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0252000). 

### easyFulcrum

This package is designed for processing and analyzing ecological sampling data generated using the Fulcrum mobile application. To learn more about how to use easyFulcrum, check out the [andersenlab/easyFulcrum](https://github.com/AndersenLab/easyfulcrum) repo and the [easyFulcrum manuscript](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0254293).

## Using R on Quest

Unfortunately, R doesn't seem to work very well with conda environments, and making sure everyone has the same version of several different R packages (specifically Tidyverse) can be a nightmare for software development. Fortunately, there are several ways to get around this issue:

* __Docker container__ - instead of using conda environments (or maybe in addition to), a docker image can be generated with R packages installed. The benefit is that docker images can be used on Quest or locally and minimal set up is required. The downside is that on some occasions there can be errors generating the docker image with incompatible package versions.

* __Library Paths__ - For several recent pipelines on Quest, we have gotten around using a docker container for R packages by installing the proper versions of R packages to a shared location (`/projects/b1059/software/R_lib_3.6.0`). With the following code, any lab member can load the R package version from that location when running the pipeline, causing no errors, even if they don't have the package installed in their local R. 

```
# to manually install a package to this specific folder
# make sure to first load the correct R version, our pipelines are currently using 3.6.0
module load R/3.6.0

# open R
R

# install package
install.packages("tidyverse", lib = "/projects/b1059/software/R_lib_3.6.0")

# if you load the package normally, it won't work (unless you also have it installed in your path)
library(tidyverse) # won't work

# load the package from the specific folder
library(tidyverse, lib.loc = "/projects/b1059/software/R_lib_3.6.0")

# or add the path to your local R library to make for easier loading
.libPaths(c("/projects/b1059/software/R_lib_3.6.0", .libPaths() ))
library(tidyverse) # works

# you can add the following lines to a nextflow script to use these R packages
# set a parameter for R library path in case it updates in the future
params.R_libpath = "/projects/b1059/software/R_lib_3.6.0"

# add the .libPaths to the top of the script dynamically (we don't want it statically in case someone wants to use the pipeline outside of quest)
echo ".libPaths(c(\\"${params.R_libpath}\\", .libPaths() ))" | cat - ${workflow.projectDir}/bin/Script.R > Script.R
Rscript --vanilla Script.R

```

**Running Rstudio on QUEST**

The Quest Analytics Nodes allow users with active Quest allocations to use RStudio from a web browser. See [Research Computing: Quest Analytics Nodes](https://kb.northwestern.edu/quest-rstudio) for an overview of the system.

Go to the [Rstudio browser](https://rstudio.questanalytics.northwestern.edu) (*make sure you are connected with VPN if you are off campus*). Log in with your netid and password just like you would on QUEST.

!!! Note
	The version of R on the Rstudio browser is currently 4.1.1, which is likely different from the version of R you run on QUEST. Therefore, you will need to re-install any packages you want to use in the browser.

You can set the working directory with `setwd("path_to_directory")` and then open and save files and data in Rstudio just like you were using it locally on your computer -- but with data and files on Quest!!


