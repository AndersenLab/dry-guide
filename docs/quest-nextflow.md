[TOC]

# Installation

!!! Important
    If you haven't already, take a look at [Quest Setup](quest-setup) for more information on how to setup your environment on Quest. That page has a script which can automate much of what you see on this page, although I am leaving this page here as it has a lot of detail on configuring nextflow.

Nextflow can be installed with [linuxbrew](quest-linuxbrew). Use:

```
brew tap homebrew/science
brew install nextflow
```

# Quest cluster configuration

Configuration files allow you to define the way a pipeline is executed on Quest. 

[Read the quest documentation on configuration files](https://www.nextflow.io/docs/latest/config.html)

Configuration files are defined at a global level in `~/.nextflow/config` and on a per-pipeline basis within `<pipeline_directory>/nextflow.config`. Settings written in `<pipeline_directory>/nextflow.config` override settings written in `~/.nextflow/config`.

### Global Configuration: `~/.nextflow/config`

In order to use nextflow on quest you will need to define some global variables regarding the process. Our lab utilizies nodes and space dedicated to genomics projects. In order to access these resources your account will need to be granted access. Contact Quest and request access to the genomics nodes and project `b1042`. Once you have access you will need to modify your global configuration. Set your `~/.nextflow/config` file to be the following:

```
process {
    module='R/3.3.1'
    executor = 'pbs'
    queue = 'genomicsguest'
    clusterOptions = '-A b1042 -l walltime=24:00:00 -e errlog.txt'
}

workDir = "/projects/b1042/AndersenLab/work"
tmpDir = "/projects/b1042/AndersenLab/tmp"

```

This configuration file does the following:

* `module='R/3.3.1'` - automatically loads `R` for all processes.
* Sets the executor to `pbs` (which is what Quest uses)
* Sets the queue to `genomicsguest` which submits jobs to genomics nodes.
* `clusterOptions` - Sets the account to `b1042`; granting access to genomics-dedicated scratch space.

* `workDir` - Sets the working directory to scratch space on b1042.
* `tmpDir` - Creates a temporary working directory. This can be used within workflows when necessary.

### Pipeline Configuration

At the pipeline level you will want to define things like the number of processors for a given process, the analysis output directory, or the reference genome used. The following is an example from the `wi-nf` pipeline:

```
genome = "WS245"
reference = "/projects/b1059/data/genomes/c_elegans/${genome}/${genome}.fa.gz"
alignment_cores = 16
variant_cores = 6
compression_threads = 4
//date = new Date().format( 'yyyyMMdd' )
date = 20170531
analysis_dir = "/projects/b1059/analysis/WI-${date}"
SM_alignments_dir = "/projects/b1059/data/alignments"
beagle_location = "/projects/b1059/software/beagle/beagle.jar"

process {
    module='gcc/5.1.0:R/3.3.1'
    $perform_alignment {
        cpus = 4
    }
    $call_variants_individual {
        cpus = 6
        memory = '8G'
    }
    $call_variants_union {
        cpus = 6
        memory = '8G'
    }
    $merge_union_vcf {
        cpus = 20
        memory = '50G'
    }
    $filter_union_vcf {
        cpus = 20
        memory = '50G'
    }
    $annotate_vcf_snpeff {
        module='gcc/4.6.3'
    }
    $generate_isotype_vcf {
        errorStrategy='retry'
        maxRetries=20
        maxForks=10
    }

}
```

# Resources

* [Nextflow documentation](https://www.nextflow.io/docs/latest/) - 
* [Awesome Nextflow pipeline examples](https://github.com/nextflow-io/awesome-nextflow) - Repository of great nextflow pipelines.
