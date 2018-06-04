# alignment-nf

The `alignment-nf` pipeline performs alignment for wild isolate sequence data on strain and isotype levels, and output BAMs and related information. Those BAMs can be used for downstream analysis of `concordance-nf`, `wi-nf` and variants calling.

[TOC]

## Usage
```
             ▗▖ ▝▜   ▝                       ▗      ▗▖ ▖▗▄▄▖
             ▐▌  ▐  ▗▄   ▄▄ ▗▗▖ ▗▄▄  ▄▖ ▗▗▖ ▗▟▄     ▐▚ ▌▐
             ▌▐  ▐   ▐  ▐▘▜ ▐▘▐ ▐▐▐ ▐▘▐ ▐▘▐  ▐      ▐▐▖▌▐▄▄▖
             ▙▟  ▐   ▐  ▐ ▐ ▐ ▐ ▐▐▐ ▐▀▀ ▐ ▐  ▐   ▀▘ ▐ ▌▌▐
            ▐  ▌ ▝▄ ▗▟▄ ▝▙▜ ▐ ▐ ▐▐▐ ▝▙▞ ▐ ▐  ▝▄     ▐ ▐▌▐
                         ▖▐
                         ▝▘
    parameters              description                    Set/Default
    ==========              ===========                    ===========
    --debug                 Set to 'true' to test          ${params.debug}
    --cores                 Regular job cores              ${params.cores}
    --goal                  for strain or isotype          ${params.goal}
    --out                   Directory to output results    ${params.out}
    --fqs                   fastq file (see help)          ${params.fqs}
    --fq_file_prefix        fastq prefix                   ${params.fq_file_prefix}
    --reference             Reference Genome (w/ .gz)      ${params.reference}
    --bamdir                Location for bams              ${params.bamdir}
    --tmpdir                A temporary directory          ${params.tmpdir}
    --email                 Email to be sent results       ${params.email}

    HELP: http://andersenlab.org/dry-guide/pipeline-alignment/
```

## Pipeline Overview

![Pipeline-overview](img/alignment-nf.png)

# Usage

## Docker File

[andersenlab/wi-nf](https://hub.docker.com/r/andersenlab/wi-nf/) is the docker image used within the alignment-nf pipeline. If Quest ever supports singularity, it can be converted to a singularity image and used with Nextflow.

## Environments

If Docker is not enable or need to run this pipeline on `Quest`, environments should be set up before running this pipeline. The [Andersen-Lab-Env](http://andersenlab.org/dry-guide/quest-andersen-lab-env/) satisfy all the requirments of this pipeline, and highly recommend for the Quest users.

## --goal

As this pipeline can perform both strain and isotype alignments, a params `--goal` should be specified each time when you running this pipeline. Only `strain` and `isotype` are acceptable for this params. Usually, __strain__ should be performed before __isotype__. __isotype__ should not be performed until we update the isotype information.

## Profiles and Running the Pipeline

The `nextflow.config` file in this pipeline contains three profiles, including local, quest_debug and quest.

* `local` - Used for local development and enable running with docker.
* `quest_debug` - Runs a small subset of available test data. Should complete within a couple of hours. For testing/diagnosing issues on Quest.
* `quest` - Runs the entire dataset.

### Running the pipeline locally

When running locally, the pipeline will run using the `andersenlab/wi-nf` docker image. You must have docker installed.

```
nextflow run main.nf -profile local -resume -with-docker --goal "strain"
```

### Debugging the pipeline on Quest

Before you run this pipeline, a debug process is needed and recommended. The debug profile use a smaller test data, which running much faster and will encounter errors much sooner should they need to be fixed. If the debug dataset runs to completion it is likely that the full dataset will as well.

```
nextflow run main.nf -profile quest_debug -resume --goal "strain"
```

### Running the pipeline on Quest

The pipeline can be run on Quest using the following command:

```
nextflow run main.nf -profile quest -resume --goal "strain"
```

# Configuration

Most configuration is handled using the `-profile` flag and `nextflow.config`; If you want to fine tune things you can use the options below.

## --cores

The number of cores to use during alignments.

## --out

A directory in which to output results. By default it will be `WI-YYYY-MM-DD` where YYYY-MM-DD is todays date.

## --fqs (FASTQs)

When running the `alignment-nf` pipeline you must provide a sample sheet that tells it where fastqs are and which samples group into strains/isotypes. By default, this is the sample sheet in the base of the alignment-nf repo , but can be specified using `--fqs` if an alternative is required. The sample sheet provides information on the strain/isotype, fastq_pairs, library, location of fastqs, and sequencing folder. See [Sample Sheet](# Sample Sheet) for more details.

## --fqs_file_prefix

A prefix path for FASTQs defined in a sample sheet. The sample sheet designed for usage on Quest (`SM_sample_sheet.tsv`) uses absolute paths so no FASTQ prefix is necessary. Additionally, there is no need to set this option as it is set for you when using the `-profile` flag. This option is only necessary (maybe) with a custom dataset where you are not using absolute paths to reference FASTQs.

## --reference

A fasta reference indexed with BWA. On Quest, the reference is available here:

```
/projects/b1059/data/genomes/c_elegans/WS245/WS245.fa.gz
```

## --email

Enable a email notification when you use this params.

## --tmpdir

A directory for storing temporary data.

# Sample Sheet

A sample sheet is the only acceptable input style for sequences in this pipeline. Sample sheet should be constructed by using the scripts under `scripts` folder, or follow the format in [Sample Sheets](http://andersenlab.org/dry-guide/sample-sheets/).

!!! note
       
       If you don't use `--fqs` to specify the directory of sample sheet, you should put sample sheet in the root of this repo. The name structure is restricted as `SM_strain_sheet.tsv` or `SM_isotype_sheet.tsv`.
   
# Output

## Strain

When you use `--goal "strain"`

```
├── BAM
│   ├── WN2065.bam
│   └── WN2065.bam.bai
├── fq
│   ├── fq_bam_idxstats.tsv
│   ├── fq_bam_stats.tsv
│   ├── fq_coverage.full.tsv
│   └── fq_coverage.tsv
├── log.txt
└── SM
    ├── bam_duplicates.tsv
    ├── SM_bam_idxstats.tsv
    ├── SM_bam_stats.tsv
    ├── SM_coverage.full.tsv
    ├── SM_coverage.mb.tsv.gz
    └── SM_coverage.tsv
```

* __log.txt__ - A summary of nextflow run.
* __BAM__ - BAMs for strains are included in this folder.
* __fq__ - The coverage and BAMs information for each sample were included here.
* __SM__ - The coverage and BAMs information for each strain, samples come from the same strain were merged and presented here.

## Isotype

When you use `--goal "isotype"`

```
├── BAM
│   ├── RC301.bam
│   └── RC301.bam.bai
├── log.txt
├── phenotype
│   ├── MT_content.tsv
│   └── telseq.tsv
├── report
│   ├── multiqc_data
│   │   ├── multiqc_data.json
│   │   ├── multiqc_fastqc.json
│   │   ├── multiqc_general_stats.json
│   │   ├── multiqc_picard_AlignmentSummaryMetrics.json
│   │   ├── multiqc_picard_insertSize.json
│   │   ├── multiqc_samtools_idxstats.json
│   │   ├── multiqc_samtools_stats.json
│   │   └── multiqc_sources.json
│   └── multiqc.html
└── SM
    ├── isotype_bam_idxstats.tsv
    ├── isotype_bam_stats.tsv
    ├── isotype_coverage.full.tsv
    └── isotype_coverage.tsv
```

* __log.txt__ - A summary of nextflow run.
* __BAM__ - BAMs for isotype are included in this folder.
* __phenotype__ - Mitochondrial content and Telomere length for each isotype.
* __SM__ - Alignment statistics by isotype, samples come from the same isotype were merged.
* __report__ - A comprehensive report of isotype BAMs.

