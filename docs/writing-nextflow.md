# Writing Nextflow pipelines

Check out the [Nextflow documentation](https://www.nextflow.io/docs/latest/) for help getting started!

!!! Note
	Learning to script with Nextflow definitely has a high learning curve. Don't get discouraged! Start with something small and simple. Maybe convert a current script you have that uses a large `for` loop into a nextflow pipeline to start getting the hang of things!

## When should my pipeline be a Nextflow script?

Not every analysis needs to be a Nextflow pipeline. For smaller analyses (especially those with few inputs) it might be easier to write a bash/shell script. However, there are many advantages to Nextflow:

1. When you are running many parallel tasks
    * Nextflow takes care of all the job submissions and is really good at running the same basic script across 6 chromosomes, 500 strains, 1000 permutations, whatever you need!
2. When your analysis consists of several different steps that are either sequential and/or can be run simultaneously. 
    * Because Nextflow is great at parallelization, it knows which steps rely on other steps and can speed up your script by running independent steps in parallel.
    * Also, you can take advantage of the `-resume` function for scripts that take a long time to run because Nextflow caches results (which means if there is an error you can fix it but you don't have to start over from the begining!)
3. You want to be able to easily run your script on different computing platforms (i.e. QUEST, local machine, GCP...)
    * This can be done by creating different profiles for each platform. Check out the [nextflow documentation](https://www.nextflow.io/docs/latest/config.html#config-profiles) on profiles.

## The basics

**Channels**

All input and output files in Nextflow are piped through "channels". You can create a channel with an input file or parameter and then feed these channels to a "process" (or step). Channels can be merged, split, or rearranged. Check out the Nextflow documentation for [channels](https://www.nextflow.io/docs/latest/channel.html) to learn more. Also check out this [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) for useful operators.

*Miscellaneous channels tips:*

- `Channel.from("A.txt")` will put `A.txt` as is into the channel 
- `Channel.fromPath("A.txt")` will add a full path (usually current directory) and put `/path/A.txt` into the channel. 
- `Channel.fromPath("folder/A.txt")` will add a full path (usually current directory) and put `/path/folder/A.txt` into the channel. 
- `Channel.fromPath("/path/A.txt")` will put `/path/A.txt` into the channel. 
- In other words, `Channel.fromPath` will only add a full path if there isn't already one and ensure there is always a full path in the resulting channel.
- This goes hand in hand with `input: path("A.txt")` inside the process, where **Nextflow actually creates a symlink named `A.txt`** (note the path from first `/` to last `/` is stripped) **linking to `/path/A.txt` in the working directory**, so it can be accessed within the working directory by the script `cat A.txt` without specifying a path.

**Processes**

Each chunk of code in a nextflow script is broken up into distinct "processes" that you can think of as "steps" or even small/large "functions". A process can have one line of code or hundreds. It can be done in bash or by running an R or python script. An example process is shown below:

```
# first process in linkagemapping-nf
process split_pheno {
	input:
		file('infile')

	output:
        file('*.tsv')

    """
    Rscript --vanilla ${workflow.projectDir}/bin/split_pheno.R ${infile} ${params.thresh}

    """
}
```

Each process is defined by `process <name> {}` and has three basic parts: 1) input, 2) output, 3) script. You can dictate the pipeline by generating a **workflow** that acts like a protocol or recipe for Nextflow: which inputs go to which process and what order to do things in. *Note, this is different than in DSL1, which is not actively used anymore*. To learn more about processes, check out the [nextflow docs](https://www.nextflow.io/docs/latest/process.html).

```
# example of a simple workflow (at the top of a nextflow script)
workflow {
    # create a channel from the input file and give it to the split_pheno process
    Channel.fromPath(params.in) | split_pheno
	
    # take the output from split_pheno and "flatten it" and send to the mapping process
    split_pheno.out.flatten() | mapping
}
```

*Miscellaneous notes on input paths for process:*

- With `input: path("A.txt")` one can refer to the file in the script as `A.txt`. Side note `A.txt` doesn't have to be the same name as in channel creation, it can be anything, `input: path("B.txt")`, `input: path("n")` etc. 
- With `input: path(A)` one can refer to the file in the script as `$A`, and the value of `$A` will be the original file name (without path, see section above). 
- `input: path("A.txt")` and `input: path "A.txt"` generally both work. Occasionally had errors that required the following (tip from [@danielecook](https://github.com/danielecook)): 
  - If not in a tuple, use `input: path "A.txt"` 
  - If in a tuple, use `input: tuple path("A.txt"), path("B.txt")`
  - This goes the same for `output`.
- From [@pditommaso](https://github.com/pditommaso): `path(A)` is almost the same as `file(A)`, however the first interprets a value of type string as the input file path (ie the location in the file system where it's stored), the latter interprets a value of type string and materialise it to a temporary files. It's recommended the use of `path` since it's less ambiguous and fits better in most use-cases.

**The working directory**

One of the main distinctions of Nextflow is that each execution of a process happens in its own temporary working directory. This is important for several reasons:

1. You do not need to name temporary files dynamically (i.e. with strain or trait name) to avoid overwriting files, because you can repeat the same process with a different trait in a different directory.
    * This means you can call a temporary file `strain.bam` instead of `DL238.bam` and `CB4856.bam`. This can make for simpler coding
    * However, if you provide strain name as a value in the input channel, it is easy to name files dynamically with `${strain}.bam` which will output `DL238.bam` or `CB4856.bam`
2. If there is an error, you can go into the working directory to see all input and output files (sometimes in the form of symlinks) for that specific process. You can also find the script (`.command.sh`) that was run and try to reproduce the error manually. If there was an error, the message is recorded in `errlog.txt`
    * If there is an error, the Nextflow error output will point you to the working directory for that specific process and might look something like `/projects/b1042/AndersenLab/work/katie/4c/4d9c3b333734a5b63d66f0bc0cfcdc`
    * You can also find the working directory from the hash shown next to a running/completed process. For example `[4c/4d9c3b]` corresponds to the working directory above.
    * See the [running nextflow](quest-nextflow.md) page for creating a function to automatically `cd` into the working directory given that hash.
    * You can also find the working directory in the `.nextflow.log` file or in the `report.html` if one is generated.

*Miscellaneous tips on the working directory:*

- Note that with `publishDir "path", mode: 'move'`, the output file will be moved outside of the working directory and Nextflow will not be able to use it as input for another process, so only use it when there is not a following process that uses the output file. 
- Be mindful that if the `""" (script section) """` involves changing directory, such as `cd` or `rmarkdown::render( knit_root_dir = "folder/" )`, Nextflow will still only search the working directory for output files. 
- Run `nextflow clean -f` in the excecution folder to clean up the working directories.
- In Nextflow scripts (.nf files), one can use: 
  - `${workflow.projectDir}` to refer where the project locates (usually the folder of main.nf). For example: `publishDir "${workflow.projectDir}/output", mode: 'copy'` or `Rscript ${workflow.projectDir}/bin/task.R`.
  - `${workflow.launchDir}` to refer to where the script is called from. 
    - `$baseDir` usually refers to the same folder as `${workflow.projectDir}` but it can also be used in the config file, where `${workflow.projectDir}` and `${workflow.launchDir}` are not accessible.   
    - They are much more reliable than `$PWD` or `$pwd`.

!!! Note
	The standard name of a nextflow script is `main.nf` but it doesn't have to be! If you just call `nextflow run andersenlab/nemascan` it will automatically choose the `main.nf` script. It is best practice to always write out the script name though

**Debugging with print**
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

**Notes on transition to DSL2**

If you are new to nextflow or don't know anything about DSL1 or DSL2, you can disregard this section and use DSL2 syntax!
- Moving to DSL2 is a one-way street. It's so intuitive with clean and readable code.
- In DSL1, each queue channel can only be used once. 
- In DSL2, a channel can be fed into multiple processes
- In DSL2, each process can only be called once. The solution is either `.concat()` the input channels so they run as parallel processes, or put the process in a module and import multiple times from the module. (One may be able to call a process in different workflows, haven't tested yet).
- DSL2 also enforces that all inputs needs to be combined into 1 channel before it goes into a process. See the [cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf) for useful operators. 
- [Simple steps to convert from original syntax to DSL2](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_convert_DSL2.pdf)
- [Deprecated operators](https://www.nextflow.io/docs/latest/dsl2.html#dsl2-migration-notes).

**Run reports**

- `nextflow main.nf -with-report -with-timeline -with-dag`
- `-with-report` Nextflow html report contains resource usage for each process, and details (most useful being the status and working directory) for each process 
- `-with-timeline` How much wait time and run time each process took for the run. Very useful reference for optimizing resource allocation and improving run time.
- `-with-dag` Make a flowchart to show the relationship of channels and processes. 
- [Software dependencies](https://www.nextflow.io/docs/latest/tracing.html#execution-report) to use these features. Note the differences on Mac and Linux.
- Or, set this up in the `nextflow.config` file for a pipeline to ensure they are generated each time the script is run:

```
import java.time.*
Date now = new Date()

params {
    tracedir = "pipeline_info"
    timestamp = now.format("yyyyMMdd-HH-mm-ss")
}

timeline {
    enabled = true
    file = "${params.tracedir}/${params.timestamp}_timeline.html"
}
report {
    enabled = true
    file = "${params.tracedir}/${params.timestamp}_report.html"
}

```

**How to require users to sepcify a parameter value**

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

## Resources

* [Nextflow documentation](https://www.nextflow.io/docs/latest/)
* [Nextflow cheatsheet](https://github.com/danrlu/Nextflow_cheatsheet/blob/main/nextflow_cheatsheet.pdf)
* [Nextflow gitter](https://gitter.im/nextflow-io/nextflow)
* [Awesome Nextflow pipeline examples](https://github.com/nextflow-io/awesome-nextflow) - Repository of great nextflow pipelines.
* [Official Nextflow patterns](https://nextflow-io.github.io/patterns/index.html)
* [Google group](https://groups.google.com/forum/#!forum/nextflow)
