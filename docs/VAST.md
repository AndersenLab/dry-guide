# Data storage on Rockfish

[TOC]

In 2023, the Andersen lab began moving its computing resources from QUEST at Northwestern to Rockfish at Johns Hopkins. The goal was to have a seamless transition by maintaining the file system and structure in Rockfish as similar as possible to QUEST.

The primary file system that the Andersen lab uses in Rockfish is called VAST. The VAST partition purchased by the lab has a file quota of 120TB. In addition, the lab has access to two secondary partitions: `data_eande106` and `scr4_eande106`, with file quotas of 10TB and 1TB respectively. Once granted access to the Andersen lab security group and allocation, you will find symbolic links for `vast`, `data`, and `scr4` partitions attached to your home directory (e.g. `/home/<user>/vast`). Symbolic links act like shortcuts that will allow you to access all partitions without having to use intricate file paths that point to the true location of the partitions in the Rockfish file system.

`vast` is where the *vast* (hehe, get it?) majority of files will exist. A breakdown of the directories of this partition is described below. Files stored in `vast` cannot have special characters (e.g. !@#$^&*()?:;) and have a few protected directory names that are not allowed (e.g. `aux`, `prn`, `con`)

`data_eande106` is where software and pipelines will live. Software and pipeline files often contain special characters (e.g. packages with `::` delimiters) or directories with protected names. Any future software must be installed here. A breakdown of the directories of this partition is described below.

`scr4_eande106` is a small partition primarily used for scratch space. It's currently empty, and it's supposed to be used only to hold small test files and scripts, with the aim to eventually delete such test files and reuse this space.**

### analysis (eande106_data)

This directory contains non-data output and analyses from general lab pipelines (i.e. `wi-gatk` or `alignment-nf`). It is organized by pipeline and then by analysis type-date. **If you are running these pipelines (including `nil-ril-nf`) it is important that you move your analysis folder here once complete (and out of your personal folder) so that everyone has access to the results.**

### software (eande106_data)

**General**
This is a great location to download any software tools or packages that you need to use that you cannot access with Quest or create a conda environment for. Especially important if they are software packages that other people in the lab might also use, as it is a shared space.

**Conda environments**

Inside the directory `conda_envs` you can find all the shared lab conda environments necessary for running certain nextflow pipelines. If you create a shared conda environment, **it is important that you update the `README.md` with the code you used to create the environment for reproducible purposes**. You can create a conda environment in this directory with:

```
conda create -p /home/<user>/data_eande106/software/conda_envs/<name_of_env>
```

You can also update your `~/.condarc` file to point to this directory so that you can easily load conda environments just by the name (i.e. `source activate nf20_env` instead of `source activate /home/<user>/data_eande106/software/conda_envs/nf20_env`):

```
channels:
  - conda-forge
  - bioconda
  - defaults

auto_activate_base: false

envs_dirs:
  - ~/.conda/envs/
  - /home/<user>/data_eande106/software/conda_envs/
```

!!! Important
    It is very important that you do **not** update any packages or software while running a shared conda environment. This is especially an issue with Nextflow and the `nf20_env` environment. Updating nextflow whilie running this environment **will update the version in the environment -- and it needs to be `v20.0` for many of the pipelines to run successfully**

If you want to see what all is loaded in a particular conda environment, or re-create an environment, you can use:

```
# make an exact copy 
conda create --clone py35 --name py35-2

# list all packages and versions
conda list --explicit > bio-env.txt

# list a history of revisions
conda list --revisions

# go back to a previous revision
conda install --revision 2

# create environment from file
conda env create --file bio-env.txt

```

Also, check out this [cheat sheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf) or our [conda page](quest-conda.md) for more.

!!! Note
    Most of the nextflow pipelines are written using `module load anaconda3/2022.05; source activate nf20_env`, so if you are having trouble running with conda, try loading the right environment first.


**R libraries**

Unfortunately, R doesn't seem to work very well with conda environments, and making sure everyone has the same version of several different R packages (specifically Tidyverse) can be a nightmare for software development. One way we have gotten around this is by installing the proper versions of R packages to a shared location (`/projects/b1059/software/R_lib_3.6.0`). Check out the **Using R on Quest** section of [the R page](r.md) to learn more about installing a package to this folder and using it in a script.

!!! Important
    It is very important that you do **not** update any packages in this location unless absolutely necessary and proceed with extreme caution (especially if it is `tidyverse`!!!). Updating packages could break pipelines that rely on them.

**Docker images**

This is not available yet, but might be a good idea to store shared docker images here. *Note: can change the `singularity_cache` parameter in nextflow scripts to store images here in the future*.

### data (vast)

This directory contains all the lab data -- split by species. The goal is to create the same file structure across all species. This not only makes it easy to find the file you are looking for, but also makes it easy to script file locations. A general example of a species' file structure is below:

```
├── genomes
│   ├── genetic_map
│   │   └──  {chr}.map
│   ├── {project_ID}
│   │   └──  {WS_build}
│   │       ├──  *.genome.fa*
│   │       ├──  csq
│   │       │    ├──  *.AA_Length.tsv
│   │       │    ├──  *.AA_Scores.tsv
│   │       │    └──  *.csq.gff3.gz
│   │       ├──  lcr
│   │       │    ├──  *.dust.bed.gz
│   │       │    └──  *.repeat_masker.bed.gz
│   │       └──  snpeff
│   │            ├──  {species}.{project}.{ws_build}
│   │            │   ├──  genes.gtf.gz
│   │            │   ├──  sequences.fa
│   │            │   └──  snpEffectPredictor.bin
│   │            └──  snpEff.config
│   └──  WI_PacBio_assemblies
├── RIL
│   ├── alignments
│   │   └──  {strain}.bam
│   ├── fastq 
│   │   └──  {strain}.*.fastq.gz
│   └── variation
│       └──  {project_analysis}
├── NIL
│   ├── alignments
│   │   └──  {strain}.bam
│   ├── fastq 
│   │   └──  {strain}.*.fastq.gz
│   └── variation
│       └──  {project_analysis}
├── WI
│   ├── alignments
│   │   ├──  _bam_not_for_cendr
│   │   └──  {strain}.bam
│   ├── concordance
│   │   └──  {cendr_release_date}
│   ├── divergent_regions
│   │   └──  {cendr_release_date}
│   ├── fastq 
│   │   ├──  dna
│   │   │   ├──  _fastq_not_for_cendr   
│   │   │   └──  {strain}.*.fastq.gz
│   │   └──  rna
│   │       └──  {project}
│   ├── haplotype
│   │   └──  {cendr_release_date}
│   ├── tree
│   │   └──  {cendr_release_date}
│   └── variation
│       └──  {cendr_release_date}
│           ├──  tracks   
│           └──  vcf
│               ├──  strain_vcf
│               │   └──  {strain}.{date}.vcf.gz
│               └──  WI.{date}.*.vcf.gz
└── {other - i.e. BSA, MUTANT, HiC, PacBio}
    └── *currently could look like anything in here - organized by project*

```

!!! Note
    This restructure is still not 100% complete and things might continue to change as we expand our genomes and data types. If you see something that is not organized well, let Erik or Nic know.

### docs (vast)

This directory should probably be removed and the docs inside stored elsewhere. Currently, shows the process for fastq SRA submission from 2019 and then again from 2021.

### projects (vast)

Projects is where most people will spend most of their time. Each user with access to quest should create a user-specific folder (named with their name) in the `projects` directory. Inside your folder, you can do all your work: analyzing data, writing new scripts, temporary storage, etc.

!!! Note
    Please be aware of how much space you are using in your personal folder. Of course some projects might require more space than others, and some projects require a lot of temporary space that can be deleted once completed. However, if you find that you have > 500-1000 GB of used space in your folder, please take a look if there is any unused data or scripts. **Either way, it is good practice to clean out your folder every few months to avoid storage pile-up.** You can check how much space you are using with `du -hs *` or ask Katie.

### to_be_deleted (vast)

This directory is a temporary holding place for large files/data that can be deleted. Think of it as a soft-trash bin. If, after a few months of living in `to_be_deleted`, it is likely the data in fact can be deleted without being missed. This was a temporary solution for the large restructure in 2021 and can likely be removed in the future.

### workflows (vast)

This directory is being phased out. Workflows will now be hosted on GitHub and any lab members who wish to run a shared workflow should run remotely (if nextflow script) or clone the repo into their personal folders.