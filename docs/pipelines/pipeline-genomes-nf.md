# genomes-nf

[TOC]

# Overview

This repo contains a nextflow pipeline that downloads, indexes, and builds annotation databases for reference genomes from wormbase. The following outputs are created:

1. A BWA Index
2. SNPeff annotation database
3. CSQ annotation database
4. Samtools faidx index
5. A GATK Sequence dictionary file

## Software Requirements

* The latest update requires Nextflow version 23+. On Rockfish, you can access this version by loading the `nf23_env` conda environment prior to running the pipeline command:

```
module load python/anaconda
source activate /data/eande106/software/conda_envs/nf23_env
```

### Relevant Docker Images

* `andersenlab/genomes` ([link](https://hub.docker.com/r/andersenlab/genomes-nf)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/genomes.Dockerfile` or `.github/workflows/build.yml` GitHub actions will create a new docker image and push if successful.

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
nextflow run -latest andersenlab/genomes-nf --debug
```

## Running on Rockfish

You should run this in a screen or tmux session.

```
nextflow run -latest andersenlab/genomes-nf -resume --wb_version=WS276 --projects=c_elegans/PRJNA13758
```

# Parameters

## `-profile`

Can be set to 'rockfish' (default), `local`, or `quest`.

## `-wb_version`

The wormbase version to build. For example, `WS276`.

## `--projects`

A comma-delimited list of `species/project_id` identifiers. A table below lists the current projects that can be downloaded. This table is regenerated as the first step of the pipeline, and stored as a file called `project_species.tsv` in the `params.output` folder (`./genomes` if working locally).

The current set of available species/projects that can be built are:

| species         | project     |
|:----------------|:------------|
| b_xylophilus    | PRJEA64437  |
| c_briggsae      | PRJNA10731  |
| c_angaria       | PRJNA51225  |
| a_ceylanicum    | PRJNA231479 |
| a_suum          | PRJNA62057  |
| a_suum          | PRJNA80881  |
| b_malayi        | PRJNA10729  |
| c_brenneri      | PRJNA20035  |
| c_elegans       | PRJEB28388  |
| c_elegans       | PRJNA13758  |
| c_elegans       | PRJNA275000 |
| c_latens        | PRJNA248912 |
| c_remanei       | PRJNA248909 |
| c_remanei       | PRJNA248911 |
| c_remanei       | PRJNA53967  |
| c_inopinata     | PRJDB5687   |
| c_japonica      | PRJNA12591  |
| c_sp11          | PRJNA53597  |
| c_sp5           | PRJNA194557 |
| c_nigoni        | PRJNA384657 |
| c_sinica        | PRJNA194557 |
| c_tropicalis    | PRJNA53597  |
| d_immitis       | PRJEB1797   |
| h_bacteriophora | PRJNA13977  |
| l_loa           | PRJNA60051  |
| m_hapla         | PRJNA29083  |
| m_incognita     | PRJEA28837  |
| h_contortus     | PRJEB506    |
| h_contortus     | PRJNA205202 |
| n_americanus    | PRJNA72135  |
| p_exspectatus   | PRJEB6009   |
| o_tipulae       | PRJEB15512  |
| p_redivivus     | PRJNA186477 |
| s_ratti         | PRJEA62033  |
| s_ratti         | PRJEB125    |
| o_volvulus      | PRJEB513    |
| p_pacificus     | PRJNA12644  |
| t_muris         | PRJEB126    |
| t_spiralis      | PRJNA12603  |
| t_suis          | PRJNA208415 |
| t_suis          | PRJNA208416 |


# Output

Outputs are nested under `params.output` with the following structure:

```
c_elegans                                                                   (species)
└── genomes
	└── PRJNA13758                                                          (project)
		└── WS276                                                           (build)
			├── c_elegans.PRJNA13758.WS276.genome.dict                      (dict file)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz                     (fasta)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.amb                 (bwa index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.ann                 (bwa index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.bwt                 (bwa index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.fai                 (samtools faidx index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.gzi                 (bwa index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.pac                 (bwa index)
			├── c_elegans.PRJNA13758.WS276.genome.fa.gz.sa                  (bwa index)
			├── csq
			│   ├── c_elegans.PRJNA13758.WS276.csq.gff3.gz                  (CSQ annotation GFF3)
			│   ├── c_elegans.PRJNA13758.WS276.csq.gff3.gz.tbi              (tabix index)
			│   ├── c_elegans.PRJNA13758.WS276.AA_Length.tsv                (protein lengths)
			│   └── c_elegans.PRJNA13758.WS276.AA_Scores.tsv                (blosum and grantham scores)
			├── lcr
			│   ├── c_elegans.PRJNA13758.WS276.repeat_masker.bed.gz         (low complexity regions)
			│   ├── c_elegans.PRJNA13758.WS276.repeat_masker.bed.gz.tbi     (tabix index)
			│   ├── c_elegans.PRJNA13758.WS276.dust.bed.gz                  (low complexity regions)
			│   └── c_elegans.PRJNA13758.WS276.dust.bed.gz.tbi              (tabix index)
			└── snpeff
				├── c_elegans.PRJNA13758.WS276                              (tabix index)
				│   ├── genes.gtf.gz                                        (Reference GTF)
				│   ├── sequences.fa                                        (fasta genome (unzipped))
				│   └── snpEffectPredictor.bin                              (snpEff annotation db)
				└── snpEff.config                                           (snpEff configuration file)

```

## Notes

* The SNPeff databases are not collected together in one location as is often the case. Instead, they are stored individually with their own configuration files.
* The GFF3 files for some species are not as developed as _C. elegans_. As a consequence, the biotype is inferred from the Attributes column of the GFF. See `bin/format_csq.R` for more details.

!!! Warning
	The updated csq-formated gff script needs to be updated for other species besides *C. elegans* (if running the default mode)
