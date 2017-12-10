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

# Usage

## Docker File

[andersenlab/wi-nf](https://hub.docker.com/r/andersenlab/wi-nf/) is the docker file for the wi-nf pipeline. It can be converted to a singularity image for use later.

## Running the pipeline on Quest

__Typical usage:__

```
nextflow run main.nf -profile quest -resume -with-report report.html
```

## --debug

The pipeline comes pre-packed with fastq's and a VCF that can be used to debug. You can use the following command to debug:

```
nextflow run main.nf --debug --reference=<path to reference> -with-docker andersenlab/wi-nf
```

Note the use of the `-with-docker` flag.

## --cores

The number of cores to use during alignments and variant calling.

## --out

A directory in which to output results. By default it will be `WI-YYYY-MM-DD` where YYYY-MM-DD is todays date.

## --fqs (FASTQs)

When running the __wi-nf__ pipeline you must provide a sample sheet that tells it where fastqs are and which samples group into isotypes. By default, this is the sample sheet in the base of the wi-nf repo (`SM_sample_sheet.tsv`), but can be specified using `--fqs` if an alternative is required. The sample sheet provides information on the isotype, fastq_pairs, library, location of fastqs, and sequencing folder.

More information on the sample sheet and adding new sequence data in the [adding new sequence data](adding new sequence data) section.

## --fqs_file_prefix
----

The sample sheet is constructed using the `scripts/construct_SM_sheet.sh` script. When new sequence data is added, this needs to be modified to add information about the new FASTQs. Changes should be committed to the repo. More information on this in the [adding new sequence data](adding new sequence data) section. Unfortunately, no sequencing center can agree on how to name FASTQs, so you'll have to come up with a way to pull out the appropriate 

!!! Important
    You have to call new isotypes using the concordance script before they can be processed using wi-nf.

When you run the script, it will output the `SM_Sample_sheet.tsv` using the location of FASTQs for Quest. You The script will output a sample sheet that defines information about FASTQs. By default absolute or relative mannerThis file defines the fastqs that should be processed. The fastq files are *relative* to that file. The fastq sheet details the FASTQ files and their associated strains. It should be tab-delimited and look like this:

__Sample sheet structure__

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

## --reference

A fasta reference indexed with BWA. On Quest, the reference is available here:

```
/projects/b1059/data/genomes/c_elegans/WS245/WS245.fa.gz
```

## --tmpdir

A directory for storing temporary data.

# Adding new sequence data



# Output

The output from the pipeline is structured to enable it to easily be integrated with CeNDR. The final output directory looks like this:

```
├── log.txt
├── alignment
│   ├── isotype_bam_idxstats.tsv
│   ├── isotype_bam_stats.tsv
│   ├── isotype_coverage.full.tsv
│   └── isotype_coverage.tsv
├── cegwas
│   ├── kinship.Rda
│   └── snps.Rda
├── isotype
│   ├── tsv
│   │   ├── <isotype>.(date).tsv.gz
│   │   └── <isotype>.(date).tsv.gz
│   └── vcf
│       ├── <isotype>.(date).vcf.gz
│       └── <isotype>.(date).vcf.gz.tbi
├── phenotype
│   ├── MT_content.tsv
│   ├── kmers.tsv
│   └── telseq.tsv
├── popgen
│   ├── WI.(date).tajima.bed.gz
│   ├── WI.(date).tajima.bed.gz.tbi
│   └── trees
│       ├── I.pdf
│       ├── I.png
│       ├── I.tree
│       ├── ...
│       ├── genome.pdf
│       ├── genome.png
│       └── genome.tree
├── report
│   ├── multiqc.html
│   └── multiqc_data
│       ├── multiqc_bcftools_stats.json
│       ├── multiqc_data.json
│       ├── multiqc_fastqc.json
│       ├── multiqc_general_stats.json
│       ├── multiqc_picard_AlignmentSummaryMetrics.json
│       ├── multiqc_picard_dups.json
│       ├── multiqc_picard_insertSize.json
│       ├── multiqc_samtools_idxstats.json
│       ├── multiqc_samtools_stats.json
│       ├── multiqc_snpeff.json
│       └── multiqc_sources.json
├── tracks
│   ├── (date).HIGH.bed.gz
│   ├── (date).HIGH.bed.gz.tbi
│   ├── (date).LOW.bed.gz
│   ├── (date).LOW.bed.gz.tbi
│   ├── (date).MODERATE.bed.gz
│   ├── (date).MODERATE.bed.gz.tbi
│   ├── (date).MODIFIER.bed.gz
│   ├── (date).MODIFIER.bed.gz.tbi
│   ├── phastcons.bed.gz
│   ├── phastcons.bed.gz.tbi
│   ├── phylop.bed.gz
│   └── phylop.bed.gz.tbi
└── variation
    ├── WI.(date).soft-filter.vcf.gz
    ├── WI.(date).soft-filter.vcf.gz.csi
    ├── WI.(date).soft-filter.vcf.gz.tbi
    ├── WI.(date).soft-filter.stats.txt
    ├── WI.(date).hard-filter.vcf.gz
    ├── WI.(date).hard-filter.vcf.gz.csi
    ├── WI.(date).hard-filter.vcf.gz.tbi
    ├── WI.(date).hard-filter.stats.txt
    ├── WI.(date).hard-filter.genotypes.tsv
    ├── WI.(date).hard-filter.genotypes.frequency.tsv
    ├── WI.(date).impute.vcf.gz
    ├── WI.(date).impute.vcf.gz.csi
    ├── WI.(date).impute.vcf.gz.tbi
    ├── WI.(date).impute.stats.txt
    ├── sitelist.tsv.gz
    └── sitelist.tsv.gz.tbi
```

### log.txt

A summary of the nextflow run.

### alignment/

Alignment statistics by isotype

* __isotype_bam_idxstats.tsv__ - A summary of mapped and unmapped reads by sample.
* __isotype_bam_stats.tsv__ - BAM summary at the sample level
* __isotype_coverage.full.tsv__ - Coverage at the sample level
* __isotype_coverage.tsv__ - Simple coverage at the sample level.

### cegwas/

* __kinship.Rda__ - A kinship matrix constructed from `WI.(date).impute.vcf.gz`
* __snps.Rda__ - A mapping snp set generated from `WI.(date).hard-filter.vcf.gz`

### isotype

This directory contains files that integrate with the genome browser on CeDNR.

#### isotype/tsv/

This directory contains tsv's that are used to show where variants are in CeNDR.

#### isotype/vcf/

This direcoty contains the VCFs of isotypes.

#### phenotype/

* __MT_content.tsv__ - Mitochondrial content (MtDNA cov / Nuclear cov).
* __kmers.tsv__ - 6-mers for each isotype.
* __telseq.tsv__ - Telomere length for each isotype ~ split out by read group.

#### popgen/

* __WI.(date).tajima.bed.gz__ - Tajima's D bedfile for use on the report viewer of CeNDR.
* __WI.(date).tajima.bed.gz.tbi__ - Tajima's D index.
* __trees/__ - Phylogenetic trees for each chromosome and the entire genome.
    * I.(pdf|png|tree)
    * ...
    * genome.(pdf|png|tree)

#### report/

* multiqc.html - A comprehensive report of the sequencing run.
* __multiqc\_data/__
    * multiqc_bcftools_stats.json
    * multiqc_data.json
    * multiqc_fastqc.json
    * multiqc_general_stats.json
    * multiqc_picard_AlignmentSummaryMetrics.json
    * multiqc_picard_dups.json
    * multiqc_picard_insertSize.json
    * multiqc_samtools_idxstats.json
    * multiqc_samtools_stats.json
    * multiqc_snpeff.json
    * multiqc_sources.json

#### track/

* __(date).LOW.bed.gz(+.tbi)__ - LOW effect mutations bed track and index.
* __(date).MODERATE.bed.gz(+.tbi)__ - MODERATE effect mutations bed track and index.
* __(date).HIGH.bed.gz(+.tbi)__ - HIGH effect mutations bed track and index.
* __(date).MODIFIER.bed.gz(+.tbi)__ - MODERATE effect mutations bed track and index.
* __phastcons.bed.gz(+.tbi)__ - Phastcons bed track and index.
* __phylop.bed.gz(+.tbi)__ - PhyloP bed track and index

#### Variation/

* __WI.(date).soft-filter.vcf.gz(+.csi|+.tbi)__ -
* __WI.(date).soft-filter.stats.txt__ -
* __WI.(date).hard-filter.vcf.gz(+.csi|+.tbi)__ -
* __WI.(date).hard-filter.stats.txt__ - 
* __WI.(date).hard-filter.genotypes.tsv__ - 
* __WI.(date).hard-filter.genotypes.frequency.tsv__ - 
* __WI.(date).impute.vcf.gz(+.csi|+.tbi)__ - 
* __WI.(date).impute.stats.txt__ - 
* __sitelist.tsv.gz(+.tbi)__ - Union of all sites identified in original SNV variant calling round.
