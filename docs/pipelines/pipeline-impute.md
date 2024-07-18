# VCF Imputation


[TOC]

The [impute-nf](https://github.com/AndersenLab/impute-nf) pipeline subsets isotype reference strains from a hard-filter vcf file, creates a SNV-only VCF, and imputes a new VCF. This step is required for fine-mapping with NemaScan.

This page details how to run the pipeline.

# Pipeline overview

```

        ##########                     #                    ##
            ##                       #####                 #  #
            ##                         #                   #
            ##    # ## ##  ###  #   #  #   ##       # ##  ####
            ##     #  #  # #  # #   #  #  #  #  ###  #  #  #
            ##     #  #  # #  # #   #  #  ###        #  #  #
            ##     #  #  # #  # #   #  #  #          #  #  #
        ########## #  #  # ###   ###   #   ###       #  #  #
                           #
                           #
                           #

    parameters                 description                                              Set/Default
    ==========                 ===========                                              ========================
    --debug                    Set to 'true' to test                                    (optional)
    --species                  Species: 'c_elegans', 'c_tropicalis' or 'c_briggsae'     (required)
    --vcf                      hard filtered vcf to calculate variant density           (required)
    --out                      output folder name                                       (optional)
    --chrI | chrII | chrIII... Window and overlap for each chromosome                   (optional)

```

## Software Requirements

* The latest update requires Nextflow version 23+. On Rockfish, you can access this version by loading the `nf23_env` conda environment prior to running the pipeline command:

```
module load python/anaconda
source activate /data/eande106/software/conda_envs/nf23_env
```

### Relevant Docker Images

*Note: Before 20220301, this pipeline was run using existing conda environments on QUEST. However, these have since been migrated to docker imgaes to allow for better control and reproducibility across platforms. If you need to access the conda version, you can always run an old commit with `nextflow run andersenlab/post-gatk-nf -r 20220216-Release`*

* `andersenlab/beagle` ([link](https://hub.docker.com/r/andersenlab/beagle)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/beagle.Dockerfile` or `.github/workflows/build_beagle_docker.yml` GitHub actions will create a new docker image and push if successful

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
nextflow run -latest andersenlab/impute-nf --debug
```

## Running on Rockfish

You should run this in a screen or tmux session.

```
nextflow run -latest andersenlab/impute-nf --vcf <path_to_vcf> --species <species>
```

# Parameters

## -profile

There are three configuration profiles for this pipeline.

* `rockfish` - Used for running on Rockfish (default).
* `quest`    - Used for running on Quest.
* `local`    - Used for local development.

!!! Note
	If you forget to add a `-profile`, the `rockfish` profile will be chosen as default

## --debug

You should use `--debug` for testing/debugging purposes. This will run the debug test set (located in the `test_data` folder).

For example:

```
nextflow run -latest andersenlab/impute-nf --debug
```

## --species

Options: c_elegans, c_briggsae, or c_tropicalis

## --vcf

Path to the hard-filtered vcf output from [`wi-gatk`](https://github.com/AndersenLab/wi-gatk). VCF should contain **ALL** strains.

## --chrI|chrII|chrIII|chrIV|chrV|chrX|MtDNA (optional)

The window size and overlap to use as inputs to Beagle. These parameters have been checked and decided on by previous lab members and Erik. Some chromosomes might require a window size of 3 and an overlap of 1. In recent conversation with the person who manages Beagle, they mentioned we should probably use default values unless we have done simulations to show these values are better. Note for the future maybe.

## --out (optional)

__default__ - `impute-YYYYMMDD`

A directory in which to output results. If you have set `--debug`, the default output directory will be `impute-YYYYMMDD-debug`.

# Output

```
└── variation
    ├── WI.20240718.hard-filter.isotype.SNV.vcf.gz
    ├── WI.20240718.hard-filter.isotype.SNV.vcf.gz.tbi
    ├── WI.20240718.impute.isotype.SNV.vcf.gz
    └── WI.20240718.impute.isotype.SNV.vcf.gz.tbi

```

