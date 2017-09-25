# nils-nf

The nil-nf pipeline will align, call variants, and generate datasets for NIL sequence data. It runs a hidden-markov-model to fill in missing genotypes from low-coverage sequence data.

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


Before you begin, you will need access to a VCF with high-coverage data from the parental strains. In general, this can be obtained using the latest release of the wild-isolate data.

```
# cd to directory of fastqs
nextflow run main.nf -resume --fq_folder data/test_fq<fq_directory>
```

The __NIL__ workflow works a little differently than the RIL and WI workflows. NIL sequence sets generally only need to be run once, and the resultant datasets are not mixed together as they are with wild isolate and RIAIL sequence data.

In order to process NIL data, you need to move the sequence data to a folder and create a `fq_sheet.tsv`. This file defines the fastqs that should be processed. __Note__: Unlike other workflows the fastq path provided is *relative* and not *absolute* for each fastq. This is because the NIL workflow processes fastqs located within a folder rather.

## fq_sheet.tsv

The `fq_sheet.tsv` defines the fastqs to be processed as part of the NIL workflow. It comes in the following format:

| strain   | fastq_pair_id   | library   | fastq-1-path   | fastq-2-path   |
|:-------|:-----------------------|:------------------|:-------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------|
| QX98   | QX98_GCTACGCTGCTCGAA   | GCTACGCTGCTCGAA   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX98_GCTACGCT-GCTCGAA_L003_R1_001.fq.gz   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX98_GCTACGCT-GCTCGAA_L003_R2_001.fq.gz   |
| QX99   | QX99_AGGCAGAAGCTAATC   | AGGCAGAAGCTAATC   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_AGGCAGAA-GCTAATC_L005_R1_001.fq.gz   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_AGGCAGAA-GCTAATC_L005_R2_001.fq.gz   |
| QX99   | QX99_AGGCAGAAGCTAATC   | AGGCAGAAGCTAATC   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_AGGCAGAA-GCTAATC_L006_R1_001.fq.gz   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_AGGCAGAA-GCTAATC_L006_R2_001.fq.gz   |
| QX99   | QX99_CGAGGCTGTGGCAAT   | CGAGGCTGTGGCAAT   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_CGAGGCTG-TGGCAAT_L003_R1_001.fq.gz   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_CGAGGCTG-TGGCAAT_L003_R2_001.fq.gz   |
| QX99   | QX99_CGAGGCTGTGGCAAT   | CGAGGCTGTGGCAAT   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_CGAGGCTG-TGGCAAT_L004_R1_001.fq.gz   | /projects/b1059/data/fastq/RIL/dna/processed/151009_D00422_0262_BC7NJ0ANXX-ECA/QX99_CGAGGCTG-TGGCAAT_L004_R2_001.fq.gz   |

This file can be generated using a script that *looks* like this, but not that you may have to modify it depending on the naming convention used for the provided fastqs.

```
ls -1 *.gz | grep 'R1' | awk -v pwd=`pwd` '{
                            split($0, a, "_");
                            SM = a[1]; 
                            gsub("-","",a[2]); // LB
                            ID = a[1] "_" a[2];
                            fq2 = $1; gsub("_R1", "_R2", fq2);
                            print SM "\t" ID "\t" a[2] "\t" pwd "/" $1 "\t" pwd "/" fq2;
                            }' > fq_sheet.tsv
```

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
