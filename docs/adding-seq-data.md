# Sample Sheets

The `wi-nf`, `concordance-nf`, and `nil-ril-nf` pipelines all make use of sample sheets. Sample sheets specify which fastqs belong to a given strain or isotype.

!!! note

    The nomenclature regarding sample sheets has changed slightly. To make things clearer these filename changes
    were applied in some of the repos.

    SM_sample_sheet --> sample_sheet.tsv
    construct_SM_sheet.sh --> construct_sample_sheet.tsv


# Creating sample sheets

For the `wi-nf` and `concordance-nf` pipelines, sample-sheets are generated using the file located (in each of these repos) in the `scripts/construct_sample_sheet.sh`. Importantly, these scripts are almost identical except that the `concordance-nf` pipeline constructs a sample sheet for __strains__ whereas the `wi-nf` sample sheet is for __isotypes__.

When adding new sequence data you need to update these scripts. See [Adding Sequencing Data](adding-seq-data.md) for more information.

# Sample-Sheet Format

The sample sheet defines which FASTQs belong to which isotypes, set FASTQ IDs, library, and more. The sample sheet is constructed using the `scripts/construct_SM_sheet.sh` script. When new sequence data is added, this needs to be modified to add information about the new FASTQs. Updates to this script should be committed to the repo.

!!! Important
    You have to assign isotypes using the concordance script before they can be processed using wi-nf.

When you run the `scripts/construct_SM_sheet.sh`, it will output the `SM_Sample_sheet.tsv` file in the base of the repo. You should carefully examine the diff of this file using Git to ensure it is modifying the sample sheet correctly. __Errors can be disasterous__. Note that the output uses absolute paths to FASTQ files.

__Sample sheet structure__

```
AB1 BGI1-RET2-AB1   RET2    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-1P.fq.gz /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-2P.fq.gz original_wi_set
AB1 BGI2-RET2-AB1   RET2    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-1P.fq.gz /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-2P.fq.gz original_wi_set
AB1 BGI3-RET2b-AB1  RET2b   /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-1P.fq.gz    /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-2P.fq.gz    original_wi_set
```

Notice that the file does not include a header. The table with corresponding columns looks like this.

| isotype   | fastq_pair_id   | library   | fastq-1-path   | fastq-2-path   | sequencing_folder |
|:----|:---------------|:------|:-----------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------|:----------------|
| AB1 | BGI1-RET2-AB1  | RET2  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-1P.fq.gz  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI1-RET2-AB1-trim-2P.fq.gz  | original_wi_set |
| AB1 | BGI2-RET2-AB1  | RET2  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-1P.fq.gz  | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI2-RET2-AB1-trim-2P.fq.gz  | original_wi_set |
| AB1 | BGI3-RET2b-AB1 | RET2b | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-1P.fq.gz | /projects/b1059/data/fastq/WI/dna/processed/original_wi_set/BGI3-RET2b-AB1-trim-2P.fq.gz | original_wi_set |

The columns are detailed below:

* __isotype__ - The name of the isotype. It is used to group FASTQ-pairs into BAMs which are treated as individuals.
* __fastq_pair_id__ - This must be unique identifier for all individual FASTQ pairs. There is (unfortunately) no standard for this.
* __library__ - A name identifying the DNA library. If the FASTQ-pairs for a strain were sequenced using different library preps they should be assigned different library names. Likewise, if they were the same DNA library they should have the same library name. Keep in mind that within an isotype the library names for each strain must be independent.
* __fastq-1-path__ - The __absolute__ path of the first fastq.
* __fastq-2-path__ - The __absolute__ path of the second fastq.
