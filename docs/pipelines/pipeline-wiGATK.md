# wi-gatk

[TOC]

The `wi-gatk` pipeline filters and calls variants from wild isolate sequence data.


# Pipeline Overview

```

	_______ _______ _______ __  __      _______ _______ 
	|     __|   _   |_     _|  |/  |    |    |  |    ___|
	|    |  |       | |   | |     <     |       |    ___|
	|_______|___|___| |___| |__|\__|    |__|____|___|    
											  

	parameters                 description                           Set/Default
	==========                 ===========                           ========================
	--debug                    Use --debug to indicate debug mode    null
	--output                   Release Directory                     WI-{date}
	--sample_sheet             Sample sheet                          null
	--bam_location             Directory of bam files                /projects/b1059/data/{species}/WI/alignments/
	--mito_name                Contig not to polarize hetero sites   MtDNA

	Reference Genome
	--------------- 
	--reference_base           Location of ref genomes               /projects/b1059/data/{species}/genomes/
	--species/project/build    These 4 params form --reference       {species} / {project} / {ws_build}

	Variant Filters         
	---------------           
	--min_depth                Minimum variant depth                 5
	--qual                     Variant QUAL score                    30
	--strand_odds_ratio        SOR_strand_odds_ratio                 5
	--quality_by_depth         QD_quality_by_depth                   20
	--fisherstrand             FS_fisher_strand                      100
	--high_missing             Max % missing genotypes               0.95
	--high_heterozygosity      Max % max heterozygosity              0.10


```

![Pipeline-overview](../img/wi-gatk.drawio.svg)

## Software Requirements

* The latest update requires Nextflow version 23+. On Rockfish, you can access this version by loading the `nf23_env` conda environment prior to running the pipeline command:

```
module load python/anaconda
source activate /data/eande106/software/conda_envs/nf23_env
```

## Relevant Docker Images

* `andersenlab/gatk4` ([link](https://hub.docker.com/r/andersenlab/gatk4)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/gatk4.Dockerfile` or `.github/workflows/build_docker.yml` GitHub actions will create a new docker image and push if successful
* `andersenlab/r_packages` ([link](https://hub.docker.com/r/andersenlab/r_packages)): Docker image is created manually, code can be found in the [dockerfile](https://github.com/AndersenLab/dockerfile/tree/master/r_packages) repo.

Make sure that you add the following code to your `~/.bash_profile`. This line makes sure that any singularity images you download will go to a shared location on `/vast/eande106` for other users to take advantage of (without them also having to download the same image).

```
# add singularity cache
export SINGULARITY_CACHEDIR='/vast/eande106/singularity/'
```

!!! Note
	If you need to work with the docker container, you will need to create an interactive session as singularity can't be run on Rockfish login nodes.
	
	```
	interact -n1 -pexpress
	module load singularity
	singularity shell [--bind local_dir:container_dir] /vast/eande106/singularity/<image_name>
	```

# Usage

*Note: if you are having issues running Nextflow or need reminders, check out the [Nextflow](../rockfish/rf-nextflow.md) page.*

## Testing on Rockfish

*This command uses a test dataset*

```
nextflow run -latest andersenlab/wi-gatk --debug
```

## Running on Rockfish

You should run this in a screen or tmux session.

*Note: if you are having issues running Nextflow or need reminders, check out the [Nextflow](../rockfish/rf-nextflow.md) page.*

```
nextflow run -latest andersenlab/wi-gatk --sample_sheet <path_to_sample_sheet> --species c_elegans
```

# Parameters

## -profile

There are three configuration profiles for this pipeline.

* `rockfish` - Used for running on Rockfish (default).
* `quest`    - Used for running on Quest.
* `local`    - Used for local development.

!!! Note
	If you forget to add a `-profile`, the `rockfish` profile will be chosen as default

## --sample_sheet

The sample sheet is automatically generated from `alignment-nf`. The sample sheet contains 5 columns as detailed below:

| strain   | bam   | bai   | coverage  | percent_mapped   | 
|:----|:-------|:------|:-----------|:-------------|
| AB1 | AB1.bam  | AB1.bam.bai  | 64  | 99.4 | 
| AB4 | AB4.bam  | AB4.bam.bai  | 52  | 99.2 | 
| BRC20067 | BRC20067.bam  | BRC20067.bam.bai  | 30  | 92.5 | 

!!! Important
	It is essential that you always use the pipelines and scripts to generate this sample sheet and **NEVER** manually. There are lots of strains and we want to make sure the entire process can be reproduced.

!!! Note
	The sample sheet produced from `alignment-nf` is only for strains that you ran in the alignment pipeline most recently. If you want to combine old strains with new strains, you will have to combine two or more sample sheets. **If you are running a species-wide analysis for CaeNDR, please follow the notes in the full WI protocol [here](https://katiesevans9.notion.site/Wild-isolate-sequence-analysis-protocol-00e76cc7f55f4bf6ab644dd99c883727)**

## --bam_location (optional)

Path to directory holding all the alignment files for strains in the analysis. Defaults to `/vast/eande106/data/{species}/WI/alignments/`

!!! Important
	Remember to move your bam files output from [alignment-nf](https://github.com/andersenlab/alignment-nf) to this location prior to running `wi-gatk`. In **most** cases, you will want to run `wi-gatk` on all samples, new and old combined.

## --species (optional)

__default__ = c_elegans

Options: c_elegans, c_briggsae, or c_tropicalis

## --project (optional)

__default__ = PRJNA13758

WormBase project ID for selected species. Choose from some examples [here](https://github.com/AndersenLab/genomes-nf/blob/master/bin/project_species.tsv)

## --ws_build (optional)

__default__ = WS283

WormBase version to use for reference genome.

## --reference (optional)

A fasta reference indexed with BWA. On Rockfish, the reference is available here:

```
/vast/eande106/data/c_elegans/genomes/PRJNA13758/WS283/c_elegans.PRJNA13758.WS283.genome.fa.gz
```

!!! Note
	If running on Rockfish, instead of changing the `reference` parameter, opt to change the `species`, `project`, and `ws_build` for other species like c_briggsae (and then the reference will change automatically) 

## --mito_name (optional)

Name of contig to skip het polarization. Might need to change for other species besides c_elegans if the mitochondria contig is named differently. Defaults to `MtDNA`.

## --output (optional)

A directory in which to output results. By default it will be `WI-YYYYMMDD` where YYYYMMDD is todays date.


# Output

The final output directory looks like this:

```

├── variation
│   ├── *.hard-filter.vcf.gz
│   ├── *.hard-filter.vcf.tbi
│   ├── *.hard-filter.stats.txt
│   ├── *.hard-filter.filter_stats.txt
│   ├── *.soft-filter.vcf.gz
│   ├── *.soft-filter.vcf.tbi
│   ├── *.soft-filter.stats.txt
│   └── *.soft-filter.filter_stats.txt
└── report
    ├── multiqc.html
	└── multiqc_data
        └── multiqc_*.json
   
```

# Data Storage

## Cleanup

The `hard-filter.vcf` is the input for both the [`concordance-nf`](https://github.com/AndersenLab/concordance-nf) pipeline and the [`post-gatk-nf`](https://github.com/AndersenLab/post-gatk-nf) pipeline. Once both pipelines have been completed successfully, the hard and soft filter vcf and index files (everything output in the `variation` folder) can be moved to `/vast/eande106/data/{species}/WI/variation/{date}/vcf`.


