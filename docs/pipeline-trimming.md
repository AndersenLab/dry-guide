# trim-fq-nf

[TOC]

The [trim-fq-nf](https://github.com/AndersenLab/trim-fq-nf) workflow performs FASTQ trimming to remove poor quality sequences and technical sequences such as adapters. It should be used with high-coverage genomic DNA. You should not use the trim-fq-nf workflow on low-coverage NIL or RIL data. Recent updates in 2021 also include running a species check on the FASTQ and generating a sample sheet of high-quality, species-confirmed samples for [alignment](https://github.com/AndersenLab/alignment-nf).


# Pipeline overview

```

____           .__                     _____                            _____ 
_/  |_ _______ |__|  _____           _/ ____\\  ______           ____  _/ ____\\
\\   __\\\\_  __ \\|  | /     \\   ______ \\   __\\  / ____/  ______  /    \\ \\   __\\ 
 |  |   |  | \\/|  ||  Y Y  \\ /_____/  |  |   < <_|  | /_____/ |   |  \\ |  |   
 |__|   |__|   |__||__|_|  /          |__|    \\__   |         |___|  / |__|   
                         \\/                      |__|              \\/         
                                                                              


    parameters              description                                   Set/Default
    ==========              ===========                                   ========================
    --debug                 Use --debug to indicate debug mode            ${params.debug}
    --fastq_folder          Name of the raw fastq folder                  ${params.fastq_folder}
    --raw_path              Path to raw fastq folder                      ${params.raw_path}
    --processed_path        Path to processed fastq folder (output)       ${params.processed_path}
    --genome_sheet          File with fasta locations for species check   ${params.genome_sheet}
    --out                   Folder name to write results                  ${params.out}
    --subsample_read_count  How many reads to use for species check       ${params.subsample_read_count}

```

![Pipeline-overview](img/trim-fq-nf.drawio.svg)

* You have downloaded FASTQ Data to a subdirectory within a raw directory. For wild isolates this will be `/projects/b1059/data/transfer/raw/<folder_name>`
* FASTQs __must__ end in a `.fq.gz` extension for the pipeline to work.
* You have modified FASTQ names if necessary to add strain names or other identifying information.
* You have installed software-requirements (see below for more info)

## Software requirements

* Nextflow v20.01+ (see the dry guide on Nextflow [here](quest-nextflow.md) or the Nextflow documentation [here](https://www.nextflow.io/docs/latest/getstarted.html)). On QUEST, you can access this version by loading the `nf20` conda environment prior to running the pipeline command:

```
module load python/anaconda3.6
source activate /projects/b1059/software/conda_envs/nf20_env
```

* Currently only runs on Quest with conda environments installed at `/projects/b1059/software/conda_envs/`

!!! Note
    All FASTQs should end with a `_R1_001.fastq.gz` or a `_R2_001.fastq.gz`. You can rename FASTQs using the rename command:

    ```
    rename --dry-run --subst .fq.gz .fastq.gz --subst _1 _R1_001 --subst _2 _R2_001 *.fq.gz
    ```

    The `--dry-run` flag will output how files will be renamed. Review the output and remove
    the flag when you are ready. 

### Relevant Docker Images

*Note: Before 20220301, this pipeline was run using existing conda environments on QUEST. However, these have since been migrated to docker imgaes to allow for better control and reproducibility across platforms. If you need to access the conda version, you can always run an old commit with `nextflow run andersenlab/alignment-nf -r 20220216-Release`*

* `andersenlab/trim-fq` ([link](https://hub.docker.com/r/andersenlab/trim-fq)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/trim-fq.Dockerfile` or `.github/workflows/build_trimfq_docker.yml` GitHub actions will create a new docker image and push if successful
* `andersenlab/multiqc` ([link](https://hub.docker.com/r/andersenlab/multiqc)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/multiqc.Dockerfile` or `.github/workflows/build_multiqc_docker.yml` GitHub actions will create a new docker image and push if successful

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

## Testing the pipeline on QUEST

*This command uses a test dataset*

```
nextflow run andersenlab/trim-fq-nf --debug
```

## Running the pipeline on QUEST

*Note: if you are having issues running Nextflow or need reminders, check out the [Nextflow](quest-nextflow.md) page.*

```
nextflow run andersenlab/trim-fq-nf --fastq_folder <name_of_folder>
```

!!! Important
    The pipeline expects the folder containing raw fastq files to be located at `/projects/b1059/data/transfer/raw/`. And all processed fastq files will be output to `/projects/b1059/data/transfer/processed/`


# Profiles

## -profile standard (Default)

If no profile is designated, the default profile will run both fastq trimming AND species check

## -profile trim_only

Use this profile to only trim fastq files and not perform species check.

## -profile sp_check_only

Use this profile to only run species check and not fastq trimming. This is useful for running species checks on previously trimmed fastqs.


# Parameters

## --debug

You should use `--debug true` for testing/debugging purposes. This will run the debug test set (located in the `test_data/raw` folder).

For example:

```
nextflow run andersenlab/trim-fq-nf --debug -resume
```

Using `--debug` will automatically set the fastq_folder to `test_data/raw/20210406_test1`

## --fastq_folder

This should be the name of the folder containing all fastq files located at `/projects/b1059/data/transfer/raw/`. As long as there are no overlapping file names (be sure to check this first), you can combine multiple pools sequenced at the same time into one larger folder at this step.

### --raw_path (optional)

The path to the `fastq_folder` if not default (`/projects/b1059/data/transfer/raw/`)

### --processed_path (optional)

The path to output folder if not default (`/projects/b1059/data/transfer/processed/`)

### --genome_sheet (optional)

Path to a tsv file listing project IDs for species. Default is located in `bin/genome_sheet.tsv`

### --out (optional)

Name of output folder with results. Default is "processFQ-{fastq_folder}"

### --subsample_read_count (optional)

How many reads to use for species check. Default = 10,000


# Output

```
├── b1059/data/transfer/processed/
│   └── {strain}_{library}_{read}.fq.gz
- - - - - - - - - - - - - - - - - - - - - - - - - - - -
├── multi_QC
│   ├── multiqc_data
│   │   ├── *.txt
│   │   └── multiqc_data.json
│   ├── multiqc_report.html
│   └── {strain}_{library}_{read}_fastp.html
├── multiqc_data
│   └── multiqc_samtools_stats.txt
├── sample_sheet
│   ├── sample_sheet_{species}_{date}_ALL.tsv
│   └── sample_sheet_{species}_{date}_NEW.tsv
├── species_check
│   ├── species_check_{fastq_folder}.html
│   ├── {library}_multiple_librarires.tsv
│   ├── {library}_strains_most_likely_species.tsv
│   ├── {library}_strains_not_in_master_sheet.tsv
│   ├── {library}_strains_possibly_diff_species.tsv
│   └── WI_all_{date}.tsv
├── sample_sheet_{fastq_folder}_all_temp.tsv
└── log.txt
```

The resulting trimmed FASTQs will be output in the `b1059/data/transfer/processed` directory. The rest of the output files and reports will be generated in a new folder in the directory in which you ran the nextflow pipeline, labeled by `processFQ-{fastq_folder}`.

__MultiQC__

* `multi_QC/{strain}_{library}_fastp.html` - fastp html report detailing trimming and quality

* `multi_QC/multiqc_report.html` - aggregate multi QC report for all strains pre and post trimming

* `multi_QC/multiqc_data/*.txt or .json` - files used to make previous reports


__Species check__

If a species check is run, the `multiqc_data/multiqc_samtools_stats.txt` will contain the results of reads mapped to each species. Furthermore, several reports and sample sheets will be generated:
 
* `species_check/species_check_{date}_{library}.html` is an HTML report showing how many strains have issues (not in master sheet, possibly different species, etc.)

* `species_check/{library}_multiple_libraries.tsv` - strains sequenced in multiple libraries

* `species_check/{library}_strains_most_likely_species.tsv` - list of all strains in library labeled by (1) the species in record and (2) most likely species by sequencing

* `species_check/{library}_strains_not_in_master.tsv` - list of strains not found in Robyn's wild isolate master sheets for CE, CB, or CT

* `species_check/{library}_strains_possibly_diff_species.tsv` - list of strains whose species in record does not match the likely species by sequencing

* `species_check/WI_all_{date}.tsv` - copy of all strains (CE, CB, and CT) and species designation in record

* `sample_sheet_{date}_{library}_all_temp.tsv` - temporary sample sheet with all strains for all species combined. DO NOT USE THIS FOR ALIGNMENT.

__Sample sheets__

If a species check is run, the `species_check/sample_sheet` folder will also contain 6 sample sheets to be used for alignment:

* `sample_sheet/sample_sheet_{species}_{date}_ALL.tsv` - sample sheet for `alignment-nf` using ALL strains of a particular species (i.e. c_elegans). This is useful for species we have not performed any alignments for or when we update the reference genome and need to re-align all strains.

* `sample_sheet/sample_sheet_{species}_{date}_NEW.tsv` - sample sheet for `alignment-nf` using all fastq from any library for ONLY strains sequenced in this particular library of a particular species (i.e. c_elegans, RET63). This is useful when the reference genome does not change and there is no need to re-align thousands of strains to save on computational power.

!!! Note
    The "new" sample sheet will still contain old fastq sequenced in a previous pool (i.e. RET55) if that strain was re-sequenced in the current pool (i.e. RET63). After running `alignment-nf`, this will create a new BAM file incorporating all fastq for that strain.

# Data storage

## Backup

Once you have completed the trim-fq-nf pipeline you should backup the **raw** FASTQs. More information on this is available in the [backup](backup.md)

## Poor quality data

If you observe poor quality sequence data you should notify Robyn through the appropriate channels and then remove the data from further analysis.

## Cleanup

If you have triple-checked everything and are satisfied with the results, the original **raw** sequence data can be deleted. The **processed** sequence data (FASTQ files) should be moved to their appropriate location, split by species (`/projects/b1059/data/{species}/WI/fastq/dna/`). The following line can be used to move processed fastq prior to running `alignment-nf`:

```

# change directories into the folder containing the processed fastq files
cd /projects/b1059/data/transfer/processed/20210510_RET63/

# move files one species at a time (might be a more efficient line of code for this, but it works...)
# !!!! make sure to change the file name !!!!!

# file name ~ - ~ CHANGE THIS ~ - ~
file='/projects/b1059/Katie/trim-fq-nf/20210510_RET63/species_check/sample_sheet/sample_sheet_c_tropicalis_20201222a_NEW.tsv'

# species
sp="c_`echo $file | xargs -n1 basename | awk -F[__] '{print $4}'`"

# get list of files to move from file
awk NR\>1 $file > temp.tsv
cat temp.tsv | awk '{print $4}' > files_to_move.txt
cat temp.tsv | awk '{print $5}' >> files_to_move.txt

# move files
cat files_to_move.txt | while read line 
do
   mv $line /projects/b1059/data/$sp/WI/fastq/dna/
done

# remove temp file
rm files_to_move.txt
rm temp.tsv

```

!!! Note
    The sample sheets ONLY contain strains that species in record matches most likely species by sequencing. If, after moving all the FASTQ for each species to their proper folder, you have FASTQ remaining, these are likely to be found in `strains_possibly_diff_species.tsv`. You should notify Robyn and Erik about these strains through the appropriate channels and delete the FASTQ or move to another temporary location until it can be re-sequenced.



