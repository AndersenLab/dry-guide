# Nextflow

Nextflow is an awesome program that allows you to write a computational pipeline by making it simpler to put together many different tasks, maybe even using different programming languages. Nextflow makes parallelism easy and works with SLURM to schedule jobs so you don't have to! Nextflow also supports docker containers and conda enviornments to streamline reproducible pipelines and can be easily adapted to run on different systems - including Google Cloud! Not convinced? Check out this [intro](https://www.nextflow.io/).

[TOC]

# Instalation

Nextflow can be installed with two easy steps:

```
# download with wget or curl
wget -qO- https://get.nextflow.io | bash

# or #

curl -s https://get.nextflow.io | bash

# make binary executable on your system
chmod +x nextflow

# Optional - move nextflow to directory accessible by your $PATH to avoid having to type full path to nextflow each time
# you can check your $PATH with:
echo $PATH

# it likely contains `/usr/local/bin` so you could move nextflow there
mv nextflow /usr/local/bin/

```

## Nextflow versions

You might also want to update nextflow or be able to run different versions. This can be done in several different ways

1. Update Nextflow

```
nextflow self-update
```

2. Use a specific version of Nextflow

```
NXF_VER=20.04.0 nextflow run main.nf
```

3. Use the `nf20` conda environment built for AndersenLab pipelines

```
module load python/anaconda3.6
source activate /projects/b1059/software/conda_envs/nf20_env
```

(You can check your Nextflow version with `nextflow -v` or `nextflow --version`)

!!! Note
	If you load this conda environment, it is not necessary to have Nextflow version 20 installed on your system -- you don't even need to have Nextflow installed at all! Because different versions might have updates that affect the running of Nextflow, it is important to keep track of the version of Nextflow you are using, as well as all software packages.

!!! Important
	Many of the Andersen Lab pipelines (if not all) are written with the new `DSL2` (see below) which requires Nextflow-v20.0+. Loading the `nf20` conda environment is a great way to run these pipelines

# Quest cluster configuration

Configuration files allow you to define the way a pipeline is executed on Quest. 

[Read the quest documentation on configuration files](https://www.nextflow.io/docs/latest/config.html)

Configuration files are defined at a global level in `~/.nextflow/config` and on a per-pipeline basis within `<pipeline_directory>/nextflow.config`. Settings written in `<pipeline_directory>/nextflow.config` override settings written in `~/.nextflow/config`.

## Global Configuration: `~/.nextflow/config`

In order to use nextflow on quest you will need to define some global variables regarding the process. Our lab utilizies nodes and space dedicated to genomics projects. In order to access these resources your account will need to be granted access. Contact Quest and request access to the genomics nodes and project `b1042`. Once you have access you will need to modify your global configuration. Set your `~/.nextflow/config` file to be the following:

```
process {
    executor = 'slurm'
    queue = 'genomicsguestA'
    clusterOptions = '-A b1042 -t 24:00:00 -e errlog.txt'
}

workDir = "/projects/b1042/AndersenLab/work/<your folder>"
tmpDir = "/projects/b1042/AndersenLab/tmp"

```

This configuration file does the following:

* Sets the executor to `slurm` (which is what Quest uses)
* Sets the queue to `genomicsguestA` which submits jobs to genomics nodes. The `genomicsguestA` will submit jobs to our dedicated nodes first, which we have high priority. If our dedicated nodes are full, it will submit to other nodes we don't have priority. So far, our lab have 2 dedicated nodes, with 28 cores and related memory (close to 1:5) for each dedicated node. We will have more in the future.  
* `clusterOptions` - Sets the account to `b1042`; granting access to genomics-dedicated scratch space.
* `workDir` - Sets the working directory to scratch space on b1042. To better organization, Please build your own folder under `/projects/b1042/AndersenLab/work/`, and define it here.
* `tmpDir` - Creates a temporary working directory. This can be used within workflows when necessary.


# Running Nextflow

Theoretically, running a Nextflow pipeline should be very straightforward (although with any developing pipelines there are always bugs to work out).

## Running Nextflow from a local directory

The most common way our lab runs Nextflow is by first cloning the git repo to your directory and then running:

```
nextflow run <path_to_git_repo>/main.nf --in test.tsv

# or if the script you want to run is called main.nf, you don't need to specify it
nextflow run <path_to_git_repo> --in test.tsv
```

In this example above, you are located in a different directory than the nextflow `main.nf` script. For most pipelines, you can be in the same or a different directory and it will run just fine.

!!! Note
	Parameters or arguments to Nextflow scripts are designated with the `--`. In the case above, there is a parameter called "in" which we are setting to be the `test.tsv` file.

!!! Note
	The standard name of a nextflow script is `main.nf` but it doesn't have to be! If you just call `nextflow run cegwas2-nf` it will automatically choose the `main.nf` script. It is best practice to always write out the script name though

When Nextflow is running, it will print to the console the name of each process in the pipeline as well as update on the progress of the script:

![nextflow example](img/nextflow_example.png)

For example, in the above screenshot from a NemaScan run, there are 14 different processes in the pipeline. Notice that the `fix_strain_names_bulk` process has already completed! Meanwhile, the `vcf_t_geno_matrix` process is still running. You can also see that the `prepare_gcta_files` is actually run 4 times (in this case, because it is run once per 4 traits in my dataset).

Another important piece of information from this Nextflow run is the hash that designates the working directory of each process. the `[ad/eea615]` next to the `fix_strain_names_bulk` process indicates that the working directory is located at `>path_to_nextflow_working_directory>/ad/eea615...`. This can be helpful if you want to go into that directory to see what is actually happening or trouble shoot errors.

!!! Note
	I highly recommend adding this function to your `~/.bash_profile` to easily access the Nexflow working directory: `gw() {cd /projects/b1042/AndersenLab/work/<your_name>/$1*}` so that when you type `gw 3f/6a21a5` (the hash Nextflow shows that indicates the specific working directory for a process) you will go to that folder automatically.

## Running Nextflow from a remote directory

Nextflow can also be run without first cloning the git repo. You can just tell Nextflow which git repo to use and it will do the rest! This can be helpful to reduce clutter and avoid making changes to the actual pipeline.

```
nextflow run AndersenLab/cegwas2-nf --traitfile test.tsv --annotation bcsq --vcf 20210121
```

The above command *should* pull the most recent commit from the **master** branch for the `cegwas2-nf` repo. Note, sometimes Nextflow does not seem to pull the most recent changes and I am not sure why.


## Resume

Another great thing about Nextflow is that it caches all the processes that completed successfully, so if you have an error at the middle or end of your pipeline, you can re-run the same Nextflow command with `-resume` and it will start where it finished last time!

```
nextflow run AndersenLab/cegwas2-nf --traitfile test.tsv --annotation bcsq --vcf 20210121 -resume
```

There is usually no downside to adding `-resume`, so you can get into the habit of always adding it if you want.

!!! Important
	There is some confusion with how the `-resume` works and sometimes it doesn't work as expected. Check out [this](https://www.nextflow.io/blog/2019/demystifying-nextflow-resume.html) and [this other](https://www.nextflow.io/blog/2019/troubleshooting-nextflow-resume.html) guide for more help. One thing I've learned is there is a difference between `process.cache = 'deep' vs. 'lenient'`.


# Writing Nextflow Pipelines

!!! Note
	Learning to script with Nextflow definitely has a high learning curve. Don't get discouraged! Start with something small and simple. Maybe convert a current script you have that uses a large for loop into a nextflow pipeline to start getting the hang of things!

## The working directory
- **Each execution of a process happens in its own temporary working directory.** 
- Specify the location of working directory with `workDir = '/path_to_tmp/'` in nextflow.config, or with `-w` option when running `nextflow main.nf`.
- The working directory is the folder named like `/path_to_tmp/4d9c3b333734a5b63d66f0bc0cfcdc` that Nextflow points you to when there is an error in execution. This folder contains the error log that could be useful for debugging. One can find the folder path in the .nextflow.log or in the report.html. 
- This folder only contains files (usually in form of symlinks, see below) from the input channel, so it's isolated from the rest of the file system. 
- This folder will also contain all output files (unless specifically directed elsewhere), and only those specified in the output channels and `publishDir` will be moved or copied to the `publishDir`. 
- Note that with `publishDir "path", mode: 'move'`, the output file will be moved outside of the working directory and Nextflow will not be able to use it as input for another process, so only use it when there is not a following process that uses the output file. 
- Be mindful that if the `""" (script section) """` involves changing directory, such as `cd` or `rmarkdown::render( knit_root_dir = "folder/" )`, Nextflow will still only search the working directory for output files. 
- Run `nextflow clean -f` in the excecution folder to clean up the working directories.


## Where am I?
- In Nextflow scripts (.nf files), one can use 
  - `${workflow.projectDir}` to refer where the project locates (usually the folder of main.nf). For example: `publishDir "${workflow.projectDir}/output", mode: 'copy'` or `Rscript ${workflow.projectDir}/bin/task.R`.
  - `${workflow.launchDir}` to refer to where the script is called from. 
- `$baseDir` usually refers to the same folder as `${workflow.projectDir}` but it can also be used in the config file, where `${workflow.projectDir}` and `${workflow.launchDir}` are not accessible.   
- They are much more reiable than `$PWD` or `$pwd`.


## Print - debugger's best friend
- To print a channel, use `.view()`. It's especially useful to resolve `WARN: Input tuple does not match input set cardinality declared by process`. (Don't forget to remove `.view()` after debugging) 
```
  channel_vcf
    .combine(channel_index)
    .combine(channel_chr)
    .view()
```
- To print from the script section inside the processes, add `echo true`.
```
  process test {
    echo true    // this will print the stdout from the script section on Terminal
    input: path(vcf)
    """
    head $vcf
    """
  }
```

## `Channel.fromPath("A.txt")` in channel creation
- `Channel.from( "A.txt" )` will put `A.txt` as is into the channel 
- `Channel.fromPath( "A.txt" )` will add a full path (usually current directory) and put `/path/A.txt` into the channel. 
- `Channel.fromPath( "folder/A.txt" )` will add a full path (usually current directory) and put `/path/folder/A.txt` into the channel. 
- `Channel.fromPath( "/path/A.txt" )` will put `/path/A.txt` into the channel. 
- In other words, `Channel.fromPath` will only add a full path if there isn't already one and ensure there is always a full path in the resulting channel.
- This goes hand in hand with `input: path("A.txt")` inside the process, where **Nextflow actually creates a symlink named `A.txt`** (note the path from first `/` to last `/` is stripped) **linking to `/path/A.txt` in the working directory**, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.


## `input: path("A.txt")` in the process section 
- With `input: path("A.txt")` one can refer to the file in the script as `A.txt`. Side note `A.txt` doesn't have to be the same name as in channel creation, it can be anything, `input: path("B.txt")`, `input: path("n")` etc. 
- With `input: path(A)` one can refer to the file in the script as `$A`, and the value of `$A` will be the original file name (without path, see section above). 
- `input: path("A.txt")` and `input: path "A.txt"` generally both work. Occasionally had errors that required the following (tip from [@danielecook](https://github.com/danielecook)): 
  - If not in a tuple, use `input: path "A.txt"` 
  - If in a tuple, use `input: tuple path("A.txt"), path("B.txt")`
  - This goes the same for `output`.
- From [@pditommaso](https://github.com/pditommaso): `path(A)` is almost the same as `file(A)`, however the first interprets a value of type string as the input file path (ie the location in the file system where it's stored), the latter interprets a value of type string and materialise it to a temporary files. It's recommended the use of `path` since it's less ambiguous and fits better in most use-cases.


## DSL2
- Moving to DSL2 is a one-way street. It's so intuitive with clean and readable code.
- In DSL1, each queue channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes
- In DSL2, each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module. (One may be able to call a process in different workflows, haven't tested yet).
- DSL2 also enforces that all inputs needs to be combined into 1 channel before it goes into a process. See the [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) for useful operators. 
- [Simple steps to convert from original syntax to DSL2](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_convert_DSL2.pdf)
- [Deprecated operators](https://www.nextflow.io/docs/latest/dsl2.html#dsl2-migration-notes).


## Run reports
- `nextflow main.nf -with-report -with-timeline -with-dag`
- `-with-report` Nextflow html report contains resource usage for each process, and details (most useful being the status and working directory) for each process 
- `-with-timeline` How much wait time and run time each process took for the run. Very useful reference for optimizing resource allocation and improving run time.
- `-with-dag` Make a flowchart to show the relationship of channels and processes. 
- [Software dependencies](https://www.nextflow.io/docs/latest/tracing.html#execution-report) to use these features. Note the differences on Mac and Linux.
- How to set them up in the [nextflow.config](https://github.com/AndersenLab/wi-gatk/blob/master/nextflow.config) so they are automatically generated for each run. Credit [@danielecook](https://github.com/danielecook) 

## Require users to sepcify a parameter value
- There are 2 types of paramters: (a) one with no actual value (b) one with actual values. 
- **(a)** If a parameter is specified but no value is given, it is implicitly considered `true`. So one can use this to run debug mode `nextflow main.nf --debug`
```
    if (params.debug) {
        ... (set parameters for debug mode)
    } else {
        ... (set parameters for normal use)
    }
```
   - or to print help message `nextflow main.nf --help`
```
    if (params.help) {
        println """
        ... (help msg here)
        """
        exit 0
    }
```

- **(b)** For parameters that need to contain a value, Nextflow recommends to set a default and let users to overwrite it as needed. However, if you want to require it to be specified by the user:
```
    params.reference = null   // no quotes. this line is optional, since without initialising the parameter it will default to null. 
    if (params.reference == null) error "Please specify a reference genome with --reference"
```  

- Below works as long as the user always append a value: `--reference=something`. It will not print the error message with: `nextflow main.nf --reference` (without specifying a value) because this will set `params.reference` to `true` (see point **(a)**) and `!params.reference` will be `false`. 
```
    if (!params.reference) error "Please specify a reference genome with --reference"
```

# Resources

* [Nextflow documentation](https://www.nextflow.io/docs/latest/)
* [Nextflow cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf)
* [Nextflow gitter](https://gitter.im/nextflow-io/nextflow)
* [Awesome Nextflow pipeline examples](https://github.com/nextflow-io/awesome-nextflow) - Repository of great nextflow pipelines.
* [Official Nextflow patterns](https://nextflow-io.github.io/patterns/index.html)
* [Google group](https://groups.google.com/forum/#!forum/nextflow)


