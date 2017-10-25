# wi-nf

The `wi-nf` pipeline aligns, calls variants, and performs analysis from wild isolate sequence data.

[TOC]

## Usage

```

     ▄         ▄  ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄        ▄  ▄▄▄▄▄▄▄▄▄▄▄ 
    ▐░▌       ▐░▌▐░░░░░░░░░░░▌                       ▐░░▌      ▐░▌▐░░░░░░░░░░░▌
    ▐░▌       ▐░▌ ▀▀▀▀█░█▀▀▀▀                        ▐░▌░▌     ▐░▌▐░█▀▀▀▀▀▀▀▀▀ 
    ▐░▌       ▐░▌     ▐░▌                            ▐░▌▐░▌    ▐░▌▐░▌          
    ▐░▌   ▄   ▐░▌     ▐░▌           ▄▄▄▄▄▄▄▄▄▄▄      ▐░▌ ▐░▌   ▐░▌▐░█▄▄▄▄▄▄▄▄▄ 
    ▐░▌  ▐░▌  ▐░▌     ▐░▌          ▐░░░░░░░░░░░▌     ▐░▌  ▐░▌  ▐░▌▐░░░░░░░░░░░▌
    ▐░▌ ▐░▌░▌ ▐░▌     ▐░▌           ▀▀▀▀▀▀▀▀▀▀▀      ▐░▌   ▐░▌ ▐░▌▐░█▀▀▀▀▀▀▀▀▀ 
    ▐░▌▐░▌ ▐░▌▐░▌     ▐░▌                            ▐░▌    ▐░▌▐░▌▐░▌          
    ▐░▌░▌   ▐░▐░▌ ▄▄▄▄█░█▄▄▄▄                        ▐░▌     ▐░▐░▌▐░▌          
    ▐░░▌     ▐░░▌▐░░░░░░░░░░░▌                       ▐░▌      ▐░░▌▐░▌          
     ▀▀       ▀▀  ▀▀▀▀▀▀▀▀▀▀▀                         ▀        ▀▀  ▀           

    parameters              description                    Set/Default
    ==========              ===========                    =======
       
    --debug                 Set to 'true' to test          ${params.debug}
    --cores                 Regular job cores              ${params.cores}
    --out                   Directory to output results    ${params.out}
    --fqs                   fastq file (see help)          ${params.fqs}
    --fq_file_prefix        fastq prefix                   ${params.fq_file_prefix}
    --reference             Reference Genome               ${params.reference}
    --annotation_reference  SnpEff annotation              ${params.annotation_reference}
    --bamdir                Location for bams              ${params.bamdir}
    --tmpdir                A temporary directory          ${params.tmpdir}

    HELP: http://andersenlab.org/dry-guide/pipeline-wi/

```

# Parameters

## --debug

The pipeline comes pre-packed with fastq's and a VCF that can be used to debug. You can use the following command to debug:

```
nextflow run main.nf --debug --reference=<path to reference>
```

## --cores

The number of cores to use during alignments and variant calling.

## --out

A directory in which to output results. By default it will be `WI-YYYY-MM-DD` where YYYY-MM-DD is todays date.

## --fqs (FASTQs)

In order to process NIL data, you need to move the sequence data to a folder and create a `fq_sheet.tsv`. This file defines the fastqs that should be processed. The fastq files are *relative* to that file. The fastq sheet details the FASTQ files and their associated strains. It should be tab-delimited and look like this:

```
ECA551  ECA551_S2_L004  S2  ECA551_S2_L004_1P.fq.gz ECA551_S2_L004_2P.fq.gz seq_folder_1
ECA552  ECA552_S16_L004 S16 ECA552_S16_L004_1P.fq.gz    ECA552_S16_L004_2P.fq.gz    seq_folder_1
ECA571  ECA571_S5_L004  S5  ECA571_S5_L004_1P.fq.gz ECA571_S5_L004_2P.fq.gz seq_folder_1
ECA571  ECA572_S6_L004  S6  ECA572_S6_L004_1P.fq.gz ECA572_S6_L004_2P.fq.gz seq_folder_1
```

Notice that the file does not include a header. The table with corresponding columns looks like this.

| isotype   | fastq_pair_id   | library   | fastq-1-path   | fastq-2-path   | sequencing_folder |
|:-------|:----------------|:----|:-------------------------|:-------------------------|:-------------|
| ECA551 | ECA551_S2_L004  | S2  | ECA551_S2_L004_1P.fq.gz  | ECA551_S2_L004_2P.fq.gz  | seq_folder_1 |
| ECA552 | ECA552_S16_L004 | S16 | ECA552_S16_L004_1P.fq.gz | ECA552_S16_L004_2P.fq.gz | seq_folder_1 |

The columns are detailed below:

* __strain__ - The name of the strain. If a strain was sequenced multiple times this file is used to identify that fact and merge those fastq-pairs together following alignment.
* __fastq_pair_id__ - This must be unique identifier for all individual FASTQ pairs.
* __library__ - A string identifying the DNA library. If you sequenced a strain from different library preps it can be beneficial when calling variants. The string can be arbitrary (e.g. LIB1) as well if only one library prep was used.
* __fastq-1-path__ - The __relative__ path of the first fastq.
* __fastq-2-path__ - The __relative__ path of the second fastq.

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

## --vcf (Parental VCF)

Before you begin, you will need access to a VCF with high-coverage data from the parental strains. In general, this can be obtained using the latest release of the wild-isolate data which is usually located in the b1059 analysis folder. For example, you would likely want to use:

`/projects/b1059/analysis/WI-20170531/vcf/WI.20170531.hard-filter.vcf.gz`

This is the __hard-filtered__ VCF, meaning that poor quality variants have been stripped. Use hard-filtered VCFs for this pipeline.

Set the parental VCF as `--vcf=/the/path/to/WI.20170531.hard-filter.vcf.gz`

## --reference

A fasta reference indexed with BWA. On Quest, the reference is available here:

```
/projects/b1059/data/genomes/c_elegans/WS245/WS245.fa.gz
```

## --tmpdir

A directory for storing temporary data.

# Output

The final output directory looks like this:

```
.
├── log.txt
├── fq
│   ├── fq_bam_idxstats.tsv
│   ├── fq_bam_stats.tsv
│   ├── fq_coverage.full.tsv
│   └── fq_coverage.tsv
├── SM
│   ├── SM_bam_idxstats.tsv
│   ├── SM_bam_stats.tsv
│   ├── SM_coverage.full.tsv
│   └── SM_coverage.tsv
├── hmm
│   ├── gt_hmm.(png/svg)
│   └── gt_hmm.tsv
├── bam
│   └── <BAMS + indices>
├── duplicates
│   └── bam_duplicates.tsv
├── sitelist
│   ├── N2.CB4856.sitelist.tsv.gz
│   └── N2.CB4856.sitelist.tsv.gz.tbi
└── vcf
    ├── NIL.filtered.stats.txt
    ├── NIL.filtered.vcf.gz
    ├── NIL.filtered.vcf.gz.csi
    ├── NIL.hmm.vcf.gz
    ├── NIL.hmm.vcf.gz.csi
    ├── gt_hmm.tsv
    ├── gt_hmm_fill.tsv
    └── union_vcfs.txt
```

### log.txt

A summary of the nextflow run.

### duplicates/

__bam_duplicates.tsv__ - A summary of duplicate reads from aligned bams.

### fq/

* __fq_bam_idxstats.tsv__ - A summary of mapped and unmapped reads by fastq pair.
* __fq_bam_stats.tsv__ - BAM summary by fastq pair.
* __fq_coverage.full.tsv__ - Coverage summary by chromosome
* __fq_coverage.tsv__ - Simple coverage file by fastq

### SM/

If you have multiple fastq pairs per sample, their alignments will be combined into a strain or sample-level BAM and the results will be output to this directory.

* __SM_bam_idxstats.tsv__ - A summary of mapped and unmapped reads by sample.
* __SM_bam_stats.tsv__ - BAM summary at the sample level
* __SM_coverage.full.tsv__ - Coverage at the sample level
* __SM_coverage.tsv__ - Simple coverage at the sample level.

### hmm/

* __gt_hmm.(png/svg)__ - Haplotype plot for NILs.
* __gt_hmm.tsv__ - Long form genotypes file.

### plots/

* __coverage_comparison.(png/svg/pdf)__ - Compares FASTQ and Sample-level coverage. Note that coverage is not simply cumulative. Only uniquely mapped reads count towards coverage, so it is possible that the sample-level coverage will not equal to the cumulative sum of the coverages of individual FASTQ pairs.
* __duplicates.(png/svg/pdf)__ - Coverage vs. percent duplicated.
* __unmapped_reads.(png/svg/pdf)__ - Coverage vs. unmapped read percent.

### sitelist/

* `<A>.<B>.sitelist.tsv.gz[+.tbi]` - A tabix-indexed list of sites found to be different between both parental strains.

### vcf/

* __gt_hmm.tsv__ - Haplotypes defined by region with associated information.
* __gt_hmm_fill.tsv__ - Same as above, but using `--infill` and `--endfill` with VCF-Kit. For more information, see [VCF-Kit Documentation](http://vcf-kit.readthedocs.io/en/latest/)
* __NIL.filtered.vcf.gz__ - A VCF genotypes including the NILs and parental genotypes.
* __NIL.filtered.stats.txt__ - Summary of filtered genotypes. Generated by `bcftools stats NIL.filtered.vcf.gz`
* __NIL.hmm.vcf.gz__ - The RIL VCF as output by VCF-Kit; HMM applied to determine genotypes.
* __union_vcfs.txt__ - A list of VCFs that were merged to generate RIL.filter.vcf.gz



## Loading BigQuery

```
release_date=20170312
bq load --field_delimiter "\t" \
        --skip_leading_rows 1  \
        --ignore_unknown_values \
        andersen-lab:WI.${release_date} \
        gs://elegansvariation.org/releases/${release_date}/WI.${release_date}.tsv.gz \
        CHROM:STRING,POS:INTEGER,SAMPLE:STRING,REF:STRING,ALT:STRING,FILTER:STRING,FT:STRING,GT:STRING
```