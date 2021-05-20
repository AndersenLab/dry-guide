# Sample Sheets

[TOC]

The `wi-nf`, `concordance-nf`, and `nil-ril-nf` pipelines all make use of sample sheets. Sample sheets specify which fastqs belong to a given strain or isotype.

# Creating sample sheets

### wi-nf and concordance-nf pipelines

For the `wi-nf` and `concordance-nf` pipelines, sample-sheets are generated using the file located (in each of these repos) in the `scripts/construct_sample_sheet.sh`. Importantly, these scripts are almost identical except that the `concordance-nf` pipeline constructs a sample sheet for __strains__ whereas the `wi-nf` sample sheet is for __isotypes__.

When adding new sequence data you need to update these scripts.

!!! note

    The nomenclature regarding sample sheets and scripts was changed in March of 2018 to make it clearer. You may encounter older files with the following names that correspond to the newer names

    * `SM_sample_sheet` --> `sample_sheet.tsv`
    * `construct_SM_sheet.sh` --> `construct_sample_sheet.tsv`

### nil-ril-nf

For the nil-ril-nf pipelines you must manually create the sample sheets according to the format below.

# Sample-Sheet Format

The sample sheet defines which FASTQs belong to which strain/isotype and specifies additional information regarding a sample. Additional information specfieid are the FASTQ ID (a unique identifier for a FASTQ-pair), Sequencing POOL (which defines the group of samples that were sequenced together), the locations of the FASTQs, and the sequencing folder.

!!! note
    
    Internally, the 'sequencing pool' information as treated as the DNA-library identifier by BWA (`LB`).
    Our lab processes sequence data such that the pool name uniquely identifies DNA-libraries for each sample.

__Sample sheet structure__

All columns are required.

* __Sample Identifier__ - How FASTQs should be grouped in the pipeline. Usually this is by strain or isotype.
* __FASTQ ID__ - A unique ID for the FASTQ pair. It must be unique for all sequencing runs defined in the sample sheet.
* __Sequencing pool__ - The sequencing pool is often defined arbitrarily. It refers to the set of strains that were sequenced together. It acts as an identifer of the DNA library within the pipelines.
* __FASTQ1__ - A relative or absolute path to the first FASTQ.
* __FASTQ2__ - A relative or absolute path to the second FASTQ.
* __Sequencing Folder__ - This column is provided for informational purposes. It generally refers to the name of the folder containing the FASTQs.


__Example__ 
```
AB1 BGI1-RET2-AB1   RET2    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-1P.fq.gz /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-2P.fq.gz original_wi_set
AB1 BGI2-RET2-AB1   RET2    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-1P.fq.gz /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-2P.fq.gz original_wi_set
AB1 BGI3-RET2b-AB1  RET2b   /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-1P.fq.gz    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-2P.fq.gz    original_wi_set
```

Notice that the file __does not include a header__. The table with corresponding header included below look like this:

| Sample Identifeir   | FASTQ ID | Sequencing Pool | fastq-1-path   | fastq-2-path   | sequencing_folder |
|:----|:---------------|:------|:-----------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------|:----------------|
| AB1 | BGI1-RET2-AB1  | RET2  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-1P.fq.gz  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-2P.fq.gz  | original_wi_set |
| AB1 | BGI2-RET2-AB1  | RET2  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-1P.fq.gz  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-2P.fq.gz  | original_wi_set |
| AB1 | BGI3-RET2b-AB1 | RET2b | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-1P.fq.gz | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-2P.fq.gz | original_wi_set |


### Absolute vs. relative paths

When constructing the sample sheet for the `wi-nf` and `concordance-nf` pipelines you are required to use the absolute paths to each FASTQ. The `nil-ril-nf` pipeline can use relative paths to FASTQs by specifying the `--fq_file_prefix` option to the parent directory containing FASTQs. 