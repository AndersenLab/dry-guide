# post-gatk-nf

[TOC]

The [post-gatk-nf](https://github.com/AndersenLab/post-gatk-nf) pipeline performs population genetics analyses (such as identifying shared haplotypes and divergent regions) at the isotype level. The VCFs output from this pipeline are used within the lab and also released to the world via CeNDR.

This page details how to run the pipeline.

# Pipeline overview
```


      * * * *                    **           * * * *    * * *    * * * *    *   *                         *
     *       *                * * * * *     *        *  *     *      *       *  *                         * *
    *        *                   **         *           *     *      *       * *                         * *
   *        *   * * * * * *      **    ***  *           * * * *      *       * *      ***      *          *
  * * * * *    *   * *   * *     **         *    * * *  *     *      *       *  *             * * *      *
 *            *     *   *   *   *  *        *        *  *     *      *       *   *           *     *    *   *
*              * * *   * * * * *    *        * * * * *  *     *      *       *    *         *      * * * * *  
                                                                                                      **
                                                                                                     * * 
                                                                                                    *  *
                                                                                                   *  *
                                                                                                    *
                         
    parameters              description                                            Set/Default
    ==========              ===========                                            ========================
    --debug                 Use --debug to indicate debug mode                     (optional)
    --vcf_folder            Folder to hard and soft filtered vcf                   (required)
    --sample_sheet          TSV with column iso-ref strain, bam, bai (no header)   (required)
    --species               Species: 'c_elegans', 'c_tropicalis' or 'c_briggsae'   c_elegans
    --output                Output folder name.                                    popgen-date (in current folder)


```

![Pipeline-overview](img/drawio/post-gatk.drawio.svg)

## Software Requirements

* The latest update requires Nextflow version 20.0+. On QUEST, you can access this version by loading the `nf20` conda environment prior to running the pipeline command:

```
module load python/anaconda3.6
source activate /projects/b1059/software/conda_envs/nf20_env
```

Alternatively you can update Nextflow by running:

```
nextflow self-update
```

### Relevant Docker Images

*Note: Before 20220301, this pipeline was run using existing conda environments on QUEST. However, these have since been migrated to docker imgaes to allow for better control and reproducibility across platforms. If you need to access the conda version, you can always run an old commit with `nextflow run andersenlab/post-gatk-nf -r 20220216-Release`*

* `andersenlab/postgatk` ([link](https://hub.docker.com/r/andersenlab/postgatk)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/postgatk.Dockerfile` or `.github/workflows/build_postgatk_docker.yml` GitHub actions will create a new docker image and push if successful
* `andersenlab/tree` ([link](https://hub.docker.com/r/andersenlab/tree)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/tree.Dockerfile` or `.github/workflows/build_tree_docker.yml` GitHub actions will create a new docker image and push if successful
* `andersenlab/pca` ([link](https://hub.docker.com/r/andersenlab/pca)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/pca.Dockerfile` or `.github/workflows/build_pca_docker.yml` GitHub actions will create a new docker image and push if successful
* `andersenlab/r_packages` ([link](https://hub.docker.com/r/andersenlab/r_packages)): Docker image is created manually, code can be found in the [dockerfile](https://github.com/AndersenLab/dockerfile/tree/master/r_packages) repo.

To access these docker images, first load the `singularity` module on QUEST.

```
module load singularity
```

Also, make sure that you add the following code to your `~/.bash_profile`. This line makes sure that any singularity images you download will go to a shared location on `b1059` for other users to take advantage of (without them also having to download the same image).

```
# add singularity cache
export SINGULARITY_CACHEDIR='/projects/b1059/singularity/'
```


# Usage

!!! Note
  For more info about running Nextflow pipelines in the Andersen Lab, check out [this page](quest-nextflow.md)

## Testing on Quest

*This command uses a test dataset*

```
nextflow run andersenlab/post-gatk-nf --debug
```

## Running on Quest

*Note: if you are having issues running Nextflow or need reminders, check out the [Nextflow](quest-nextflow.md) page.*


You should run this in a screen session.

### Profiles

There are now three ways to run this pipeline:

1. `-profile standard` (default): runs original processes including subseting VCF and divergent and haplotype calls. 
    - sample_sheet, vcf_folder, (species) 
2. `-profile pca`: does not run the original post-gatk processes, only the PCA analysis. *Note: requires different parameters*
    - snv_vcf, species, anc, eigen_ld, pops
3. `-profile standard --pca`: runs all processes including subseting VCF, divergent and haplotype calls, PCA analysis of isotypes. *Requires additional parameters relating to PCA*
    - sample_sheet, vcf_folder, species, anc, eigen_ld
    - Note: the `-profile standard` is optional, just adding the `--pca` param is enough.

```
nextflow run andersenlab/post-gatk-nf --vcf <path_to_vcf> --sample_sheet <path_to_sample_sheet>
```

# Parameters

## --debug

You should use `--debug true` for testing/debugging purposes. This will run the debug test set (located in the `test_data` folder).

For example:

```
nextflow run andersenlab/post-gatk-nf --debug
```

Using `--debug` will automatically set the sample sheet to `test_data/sample_sheet.tsv`

## --sample_sheet

A custom sample sheet can be specified using `--sample_sheet`. The `sample sheet` is generated from the sample sheet used as input for [`wi-gatk-nf`](https://github.com/AndersenLab/wi-gatk) with only columns for strain, bam, and bai subsetted. **Make sure to remove any strains that you do not want to include in this analysis.** (*i.e. subset to keep only ISOTYPE strains*)

Remember that in `--debug` mode the pipeline will use the sample sheet located in `test_data/sample_sheet.tsv`.

!!! Important
    There is no header for the sample sheet!

The `sample sheet` has the following columns:

* __strain__ - the name of the strain
* __bam__ - name of the bam alignment file
* __bai__ - name of the bam alignment index file

!!! Note
    As of 20210501, bam and bam.bai files for all strains of a particular species can be found in one singular location: `/projects/b1059/data/{species}/WI/alignments/` so there is no longer need to provide the location of the bam files.


## --vcf_folder

Path to the **folder** containing both the hard-filtered and soft-filtered vcf outputs from [`wi-gatk`](https://github.com/AndersenLab/wi-gatk). VCF should contain **ALL** strains, the first step will be to subset isotype reference strains for further analysis.

!!! Note
    This should be the **path to the folder**, we want to isotype-subset both hard and soft filtered VCFs. For example: `--vcf_folder /projects/b1059/projects/Katie/wi-gatk/WI-20210121/variation/` or `--vcf_folder /projects/b1059/data/c_elegans/WI/variation/20210121/vcf/`

### --species (optional)

__default__ = c_elegans

Options: c_elegans, c_briggsae, or c_tropicalis

### --snv_vcf (pca profile)

File path to SNV-filtered VCF

### --pops (pca profile)

Strain list to filter VCF for PCA analysis. No header:

| AB1 |
| --- |
| CB4856 |
| ECA788 |

!!!Note
    If you run the standard profile with pca this file will be automatically generated to include all isotypes.

### --eigen_ld (pca)

LD thresholds to test for PCA. Can provide multiple with `--eigen_ld 0.8,0.6,0.4`

### --anc (pca)

Ancestor strain to use for PCA. 

!!! Note 
    Make sure this strain is in your VCF

### --output (optional)

__default__ - `popgen-YYYYMMDD`

A directory in which to output results. If you have set `--debug true`, the default output directory will be `popgen-YYYYMMDD-debug`.

# Output

```
├── ANNOTATE_VCF
│   ├── ANC.bed.gz
│   ├── ANC.bed.gz.tbi
│   ├── Ce330_annotated.vcf.gz
|   └── Ce330_annotated.vcf.tbi
├── EIGESTRAT
│   └── LD_{eigen_ld}
│       ├── INPUT_FILES
│       │   └── *
│       ├── OUTLIER_REMOVAL
│       │   ├── eigenstrat_outliers_removed_relatedness
│       │   ├── eigenstrat_outliers_removed_relatedness.id
│       │   ├── eigenstrat_outliers_removed.evac
│       │   ├── eigenstrat_outliers_removed.eval
│       │   ├── logfile_outlier.txt
│       │   └── TracyWidom_statistics_outlier_removal.tsv
│       └── NO_REMOVAL
│           └── same as outlier_removal
├── pca_report.html
├── divergent_regions
│   ├── Mask_DF
│   │   └── [strain]_Mask_DF.tsv
|   └── divergent_regions_strain.bed
├── haplotype
│   ├── haplotype_length.pdf
│   ├── sweep_summary.tsv
│   ├── max_haplotype_genome_wide.pdf
│   ├── haplotype.pdf
│   ├── haplotype.tsv
│   ├── [chr].ibd
│   └── haplotype_plot_df.Rda
├── tree
│   ├── WI.{date}.hard-filter.isotype.min4.tree
│   ├── WI.{date}.hard-filter.isotype.min4.tree.pdf
│   ├── WI.{date}.hard-filter.min4.tree
│   └── WI.{date}.hard-filter.min4.tree.pdf
├── NemaScan
│   ├── strain_isotype_lookup.tsv
│   ├── div_isotype_list.txt
│   ├── haplotype_df_isotype.bed
│   ├── divergent_bins.bed
│   └── divergent_df_isotype.bed
└── variation
    ├── WI.{date}.small.hard-filter.isotype.vcf.gz
    ├── WI.{date}.small.hard-filter.isotype.vcf.gz.tbi
    ├── WI.{date}.hard-filter.isotype.SNV.vcf.gz
    ├── WI.{date}.hard-filter.isotype.SNV.vcf.gz.tbi
    ├── WI.{date}.soft-filter.isotype.vcf.gz
    ├── WI.{date}.soft-filter.isotype.vcf.gz.tbi
    ├── WI.{date}.hard-filter.isotype.vcf.gz
    └── WI.{date}.hard-filter.isotype.vcf.gz.tbi

```

# Data storage

## Cleanup

Once the pipeline has complete successfully and you are satisfied with the results, the final data can be moved to their final storage place on Quest accordingly:

* Everything in the `haplotype` foder can be moved to `/projects/b1059/data/{species}/WI/haplotype/{date}`
* Everything in the `divergent_regions` folder can be moved to `/projects/b1059/data/{species}/WI/divergent_regions/{date}`
* Everything in the `tree` folder can be moved to `/projects/b1059/data/{species}/WI/tree/{date}`
* Everything in the `variation` folder can be moved to `/projects/b1059/data/{species}/WI/variation/{date}/vcf`
* Everything in the `NemaScan` folder can replace the old verions in NemaScan when ready `NemaScan/input_data/{species}/isotypes`
* Everything in the `EIGENSTRAT` folder can be moved to `/projects/b1059/data/{species}/WI/pca/{date}`


## Updating CeNDR

Check out the [CeNDR](cendr.md) page for more information about updating a new data release for CeNDR.

## Updating `NemaScan`

Once a new CeNDR release is ready, it is important to also update the genome-wide association mapping packages to ensure users can appropriately analyze data from new strains as well as old strains. Here is a list of things that need to be updated:

* The default vcf should be changed to the newest release date (i.e. from 20200815 to 20210121). Users will still have the option to use an earlier vcf.
* Everything in the `NemaScan` folder can replace the old verions in NemaScan when ready `NemaScan/input_data/{species}/isotypes`
* Be sure that the small-vcf is stored in the proper file location to be accessed, both on QUEST and on GCP for CeNDR/local users.

!!! Note
    Although users will have the option to use an older vcf, the divergent region data will always be pulled from the most recent release. There could be minor changes from release to release. If this is a concern, we could switch to pulling the divergent data directly from `b1059` instead of including it in the `bin/` of the mapping pipeline.



