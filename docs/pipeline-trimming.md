# trimmomatic-nf

The trimmomatic workflow should be used to initially process sequence data. You should not use the trimmomatic workflow on low-coverage NIL or RIL data.

### Docker

The trimmomatic pipeline is very simple and only requires trimmomatic to be installed. However, a docker image is available for trimmomatic here:

[cyverseuk/trimmomatic/](https://hub.docker.com/r/cyverseuk/trimmomatic/)

### Usage

New sequence data should be stored in the appropriate path on the cluster in the `raw` directory. For example, when adding new sequence data for wild isolates, data should be added to a new folder here:

`/projects/b1059/data/fastq/WI/dna/raw/<folder_name>`

!!! Important
    All FASTQs should end with a `_1.fq.gz` or a `_2.fq.gz`. To rename FASTQs you can use:
    
    ```
    rename --dry-run --subst .fastq.gz .fq.gz --subst _R1_001 _1 --subst _R2_001 *.fastq.gz
    ```

Outside of these simple changes to the filenames, no further changes should be made. The original filenames are potentially useful when tracing issues. Downstream steps use a file to connect the filenames with the appropriate strain or isotype.

To begin running the pipeline you will `cd` to the directory containing the raw FASTQs and run the pipeline:

```
# cd to directory of fastqs
nextflow run Andersenlab/trimmomatic-nf
```

The resulting trimmed FASTQs will be output in the `processed` directory located up one level from the current directory. For example:

__FASTQs are deposited in this directory__

`/projects/b1059/data/fastq/WI/dna/raw/new_wi_seq`

__You run the pipeline while sitting in the same directory:__

`/projects/b1059/data/fastq/WI/dna/raw/new_wi_seq`

__And results are output in the following directory:__

`/projects/b1059/data/fastq/WI/dna/raw/new_wi_seq`

