# NIL-NF

The nil-nf pipeline will align, call variants, and generate datasets for NIL sequence data. It runs a hidden-markov-model to fill in missing genotypes from low-coverage sequence data.

__TOC__

## Usage

```

    ███╗   ██╗██╗██╗      ███╗   ██╗███████╗
    ████╗  ██║██║██║      ████╗  ██║██╔════╝
    ██╔██╗ ██║██║██║█████╗██╔██╗ ██║█████╗
    ██║╚██╗██║██║██║╚════╝██║╚██╗██║██╔══╝
    ██║ ╚████║██║███████╗ ██║ ╚████║██║
    ╚═╝  ╚═══╝╚═╝╚══════╝ ╚═╝  ╚═══╝╚═╝


    parameters           description                    Set/Default
    ==========           ===========                    =======

    --debug              Set to 'true' to test          false
    --cores              Number of cores                4
    --A                  Parent A                       N2
    --B                  Parent B                       CB4856
    --cA                 Parent A color (for plots)     #0080FF
    --cB                 Parent B color (for plots)     #FF8000
    --analysis_folder    Folder for results             results
    --analysis_dir       Sub-Folder                     results/NIL-N2-CB4856-2017-09-25
    --fqs                fastq file (see help)          (required)
    --reference          Reference Genome               /path/to/WS245.fa.gz
    --vcf                VCF to fetch parents from      (required)
    --tmpdir             A temporary directory          tmp/


    The Set/Default column shows what the value is currently set to
    or would be set to if it is not specified (it's default).


```

## Setting up sequence data (`--fqs`)

In order to process NIL data, you need to move the sequence data to a folder and create a `fq_sheet.tsv`. This file defines the fastqs that should be processed. The fastq files are *relative* to that file. `The fq_sheet.tsv` file should be tab-delimited and look like this:

```
NIL_01   NIL_01_ID    S16 NIL_01_1.fq.gz   NIL_01_2.fq.gz
NIL_02   NIL_02_ID    S1  NIL_02_1.fq.gz   NIL_02_2.fq.gz
```

Notice that the file does not include a header. The table with corresponding columns looks like this.

| strain   | fastq_pair_id   | library   | fastq-1-path   | fastq-2-path   |
|:-------|:-----------------------|:------------------|:-------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------|
| NIL_01 | NIL_01_ID | S16 | NIL_01_1.fq.gz | NIL_01_2.fq.gz |
| NIL_02 | NIL_02_ID | S1  | NIL_02_1.fq.gz | NIL_02_2.fq.gz |

This file needs to be placed along with the sequence data into a folder. The tree will look like this:

```
NIL_SEQ_DATA/
├── NIL_01_1.fq.gz
├── NIL_01_2.fq.gz
├── NIL_02_1.fq.gz
├── NIL_02_2.fq.gz
└── fq_sheet.tsv
```

Set `--fqs` as `--fqs=/the/path/to/fq_sheet.tsv`.

!!! Important
    Do not perform any pre-processing on NIL data. NIL-data is low-coverage by design and you want to retain as much sequence data (however poor) as possible.

## Parental VCF (`--vcf`)

Before you begin, you will need access to a VCF with high-coverage data from the parental strains. In general, this can be obtained using the latest release of the wild-isolate data which is usually located in the b1059 analysis folder. For example, you would likely want to use:

`/projects/b1059/analysis/WI-20170531/vcf/WI.20170531.hard-filter.vcf.gz`

This is the __hard-filtered__ VCF, meaning that poor quality variants have been stripped. Use hard-filtered VCFs for this pipeline.

Set the parental VCF as `--vcf=/the/path/to/WI.20170531.hard-filter.vcf.gz`

## Important Notes

* NIL sequencing uses low coverage by design. No pre-processing (ie trimming) takes place because variants will be called at specific positions and low quality/adapter contamination are unlikely to be problematic.

## Output

The output directory looks like this:

```
├── SM
│   ├── SM_bam_idxstats.tsv
│   ├── SM_bam_stats.tsv
│   ├── SM_coverage.full.tsv
│   └── SM_coverage.tsv
├── bam
│   ├── <bam_files + indices>
├── duplicates
│   └── bam_duplicates.tsv
├── fq
│   ├── fq_bam_idxstats.tsv
│   ├── fq_bam_stats.tsv
│   ├── fq_coverage.full.tsv
│   └── fq_coverage.tsv
├── hmm
│   ├── gt_hmm.png
│   ├── gt_hmm.svg
│   └── gt_hmm.tsv
├── plots
│   ├── coverage_comparison.png
│   ├── coverage_comparison.svg
│   ├── duplicates.png
│   ├── duplicates.svg
│   ├── unmapped_reads.png
│   └── unmapped_reads.svg
└── vcf
    ├── NIL.filtered.stats.txt
    ├── NIL.hmm.vcf.gz
    ├── NIL.hmm.vcf.gz.csi
    ├── gt_hmm.tsv
    ├── gt_hmm_fill.tsv
    ├── merged.raw.vcf.gz
    ├── merged.raw.vcf.gz.csi
    └── union_vcfs.txt
```

### duplicates/

__bam_duplicates.tsv__ - A summary of duplicate reads from aligned bams.

### fq/

* __fq_bam_idxstats.tsv__
* __fq_bam_stats.tsv__
* __fq_coverage.full.tsv__
* __fq_coverage.tsv__

### SM/

* __SM_bam_idxstats.tsv__
* __SM_bam_stats.tsv__
* __SM_coverage.full.tsv__
* __SM_coverage.tsv__

### hmm/

* __gt_hmm.(png/svg/pdf)__ - Haplotypes for NILs.
* __gt_hmm.tsv__ - Long form genotypes file.

### plots/

__coverage_comparison.(png/svg/pdf)__

__duplicates.(png/svg/pdf)__

__unmapped_reads.(png/svg/pdf)__

### vcf/

* __gt_hmm.tsv__ - Haplotypes defined by region with associated information. 
* __gt_hmm_fill.tsv__ - Same as above, but using `--infill` and `--endfill` with VCF-Kit. For more information, see [VCF-Kit Documentation](http://vcf-kit.readthedocs.io/en/latest/)
* __NIL.filter.vcf.gz__ - A VCF of filtered genotypes. 
* __NIL.filtered.stats.txt__ - Summary of filtered genotypes. Generated by `bcftools stats RIL.filter.vcf.gz`
* __NIL.hmm.vcf.gz__ - The RIL VCF as output by VCF-Kit; HMM applied to determine genotypes.
* __union_vcfs.txt__ - A list of VCFs that were merged to generate RIL.filter.vcf.gz
