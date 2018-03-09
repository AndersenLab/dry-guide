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
    executor = 'pbs'
    queue = 'genomicsguest'
    clusterOptions = '-A b1042 -l walltime=24:00:00 -e errlog.txt'
    genome = "WS245"
    reference = "/projects/b1059/data/genomes/c_elegans/${genome}/${genome}.fa.gz"
}

workDir = "/projects/b1042/AndersenLab/work"
tmpDir = "/projects/b1042/AndersenLab/tmp"

```

This configuration file does the following:

* Sets the executor to `pbs` (which is what Quest uses)
* Sets the queue to `genomicsguest` which submits jobs to genomics nodes.
* `clusterOptions` - Sets the account to `b1042`; granting access to genomics-dedicated scratch space.

* `workDir` - Sets the working directory to scratch space on b1042.
* `tmpDir` - Creates a temporary working directory. This can be used within workflows when necessary.

# Resources

* [Nextflow documentation](https://www.nextflow.io/docs/latest/) - 
* [Awesome Nextflow pipeline examples](https://github.com/nextflow-io/awesome-nextflow) - Repository of great nextflow pipelines.
