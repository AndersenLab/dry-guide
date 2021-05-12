# post-gatk-nf


The [post-gatk-nf](https://github.com/AndersenLab/post-gatk-nf) pipeline performs two main tasks: (1) variant annotation for the VCF at the isotype level and (2) population genetics analyses (such as identifying shared haplotypes and divergent regions) at the isotype level. The VCFs output from this pipeline are used within the lab and also released to the world via CeNDR.

This page details how to run the pipeline.

[TOC]

## Usage
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
    --vcf                   Hard filtered vcf to calculate variant density         (required)
    --sample_sheet          TSV with column iso-ref strain, bam, bai (no header)   (required)
    --species               Species: 'c_elegans', 'c_tropicalis' or 'c_briggsae'   c_elegans
    --project               Project name for species reference                     PRJNA13758
    --ws_build              WormBase version for species reference                 WS276
    --output                Output folder name.                                    popgen-date (in current folder)

    HELP: http://andersenlab.org/dry-guide/pipeline-postGATK/


```

<small>The logo above looks better in your terminal!</small>

## Pipeline Overview

![Pipeline-overview](img/post-gatk-nf-flow.png)

# Usage

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

!!! Important
    This pipeline currently only supports analysis on Quest, cannot be run locally


## Quick Start

__Testing on Quest__

*This command uses a test dataset*

```
nextflow run main.nf --debug
```

__Running on Quest__

You should run this in a screen session.

```
nextflow run main.nf --vcf <path_to_vcf> --sample_sheet <path_to_sample_sheet>
```


## Sample Sheet

The `sample sheet` is generated from the sample sheet used as input for [`wi-gatk-nf`](https://github.com/AndersenLab/wi-gatk) with only columns for strain, bam, and bai subsetted. Make sure to remove any strains that you do not want to include in this analysis.

Remember that in `--debug` mode the pipeline will use the sample sheet located in `test_data/sample_sheet.tsv`.

!!! Important
    There is no header for the sample sheet!

The `sample sheet` has the following columns:

* __strain__ - the name of the strain
* __bam__ - name of the bam alignment file
* __bai__ - name of the bam alignment index file

!!! Note
    As of 20210501, bam and bam.bai files for all strains of a particular species can be found in one singular location: `/projects/b1059/data/{species}/WI/alignments/` so there is no longer need to provide the location of the bam files.


# Parameters

## --debug

You should use `--debug true` for testing/debugging purposes. This will run the debug test set (located in the `test_data` folder).

For example:

```
nextflow run main.nf --debug -resume
```

Using `--debug` will automatically set the sample sheet to `test_data/sample_sheet.tsv`

## --sample_sheet

A custom sample sheet can be specified using `--sample_sheet`. The sample sheet format is described above in [sample sheet](#sample_sheet)

When using `--debug true`, the `test_data/sample_sheet.tsv` file is used.

## --vcf

Path to the hard-filtered vcf output from [`wi-gatk-nf`](https://github.com/AndersenLab/wi-gatk). VCF should contain **ALL** strains, the first step will be to subset isotype reference strains for further analysis.

!!! Note
    This should be the **hard-filtered** VCF

## --species

__default__ = c_elegans

Options: c_elegans, c_briggsae, or c_tropicalis

## --project

__default__ = PRJNA13758

WormBase project ID for selected species. Choose from some examples [here](https://github.com/AndersenLab/genomes-nf/blob/master/bin/project_species.tsv)

## --ws_build

__default__ = WS276

WormBase version to use for reference genome.

## --output

__default__ - `popgen-YYYYMMDD`

A directory in which to output results. If you have set `--debug true`, the default output directory will be `popgen-YYYYMMDD-debug`.

# Output

```
├── bcsq
│   ├── WI.{date}.hard-filter.isotype.bcsq.vcf.gz
│   ├── WI.{date}.hard-filter.isotype.bcsq.vcf.gz.tbi
│   └── WI.{date}.hard-filter.isotype.bcsq.vcf.gz.stats.txt
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
├── snpeff
│   ├── WI.{date}.hard-filter.isotype.snpeff.vcf.gz
│   ├── WI.{date}.hard-filter.isotype.snpeff.vcf.gz.tbi
│   └── snpeff.stats.csv
├── tree
│   ├── WI.{date}.hard-filter.isotype.min4.tree
│   └── WI.{date}.hard-filter.min4.tree
├── WI.{date}.hard-filter.isotype.vcf.gz
└── WI.{date}.hard-filter.isotype.vcf.gz.tbi
```

