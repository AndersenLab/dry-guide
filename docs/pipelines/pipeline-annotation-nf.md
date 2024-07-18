# annotation-nf

[TOC]

The [annotation-nf](https://github.com/AndersenLab/annotation-nf) pipeline performs variant annotation for the VCF at the isotype level using both [SnpEff](http://pcingola.github.io/SnpEff/) and [BCFtools/csq (BCSQ)](https://samtools.github.io/bcftools/howtos/csq-calling.html).

!!! Note
	Before running, make sure to check out the [genomes-nf](pipeline-genomes-nf.md) pipeline to ensure that the reference genome and annotation databases are set up properly.

# Pipeline overview

```
-------------    
ANNOTATION-NF
-------------

nextflow andersenlab/annotation-nf --debug

nextflow andersenlab/annotation-nf --vcf=hard-filtered.vcf --species=c_elegans --divergent_regions=divergent_regions_strain.bed

    parameters           description                                              Set/Default
    ==========           ===========                                              ========================
    --debug              Set to 'true' to test                                    ${params.debug}
    --species            Species: 'c_elegans', 'c_tropicalis' or 'c_briggsae'     ${params.species}
    --vcf                hard filtered vcf to calculate variant density           ${params.vcf}
    --divergent_regions  (Optional) Divergent region bed file                     ${params.divergent_regions}
    --reference          Reference used based on species and project              ${params.reference}
    --output             (Optional) output folder name                            ${params.output}
 
    username                                                                      ${"whoami".execute().in.text}

    HELP: http://andersenlab.org/dry-guide/pipelines/pipeline-annotation-nf
```

![Pipeline-overview](../img/annotation-nf.drawio.svg)

## Software Requirements

* The latest update requires Nextflow version 23+. On Rockfish, you can access this version by loading the `nf23_env` conda environment prior to running the pipeline command:

```
module load python/anaconda
source activate /data/eande106/software/conda_envs/nf23_env
```

### Relevant Docker Images

*Note: Before 20220301, this pipeline was run using existing conda environments on QUEST. However, these have since been migrated to docker imgaes to allow for better control and reproducibility across platforms. If you need to access the conda version, you can always run an old commit with `nextflow run andersenlab/annotation-nf -r 20220216-Release`*

* `andersenlab/annotation` ([link](https://hub.docker.com/r/andersenlab/annotation)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/annotation.Dockerfile` or `.github/workflows/build_docker.yml` GitHub actions will create a new docker image and push if successful
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
nextflow run -latest andersenlab/annotation-nf --debug
```

## Running on Rockfish

You should run this in a screen or tmux session.

```
nextflow run -latest andersenlab/annotation-nf --vcf <path_to_vcf> --species <species> --divergent_regions <path_to_file>
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
nextflow run -latest andersenlab/annotation-nf --debug
```

## --vcf

Path to the hard-filter, isotype VCF (output from `post-gatk-nf`)

## --species

Choose from c_elegans, c_briggsae, or c_tropicalis. Species will specifiy a default reference genome. You can select a different one if you prefer (see below)

## --divergent_regions

This is the `divergent_regions_strain.bed` file output from the `post-gatk-nf` pipeline. This file is used to add a column to the flat file if the variant is within a divergent region. Currently, *C. elegans* is the only species with divergent regions, if running for another species, do not provide a divergent_regions file and the pipeline will ignore it.

## --reference, --project, --ws_build (optional)

By default, the reference genome is set by the species parameter. If you don't want to use the default, you could change the project and/or ws_build. As long as the genome is in the proper location on quest (for more, see the [genomes-nf](pipeline-genomes-nf.md) pipeline), this will work. Alternatively, you could provide the path to a reference of your choice.

**Defaults:**
- *C. elegans* - `/vast/eande106/data/c_elegans/genomes/PRJNA13758/WS276/c_elegans.PRJNA13758.WS276.genome.fa.gz`
- *C. briggsae* - `/vast/eande106/data/c_briggsae/genomes/QX1410_nanopore/Feb2020/c_briggsae.QX1410_nanopore.Feb2020.genome.fa.gz`
- *C. tropicalis* - `/vast/eande106/data/c_tropicalis/genomes/NIC58_nanopore/June2021/c_tropicalis.NIC58_nanopore.June2021.genome.fa.gz`

## --ncsq_param (optional)

This parameter is necessary for correct annotation using BCSQ for variants with many different annotations (like found in divergent regions). In 20210121 we found that the default value of `224` was sufficient, but as more strains are added this number might need to increase. If there is an issue, you should see a warning error from BCFtools and they should suggest what to change this parameter to.

# CSQ compatible GFFs

There are a few specific change that need to be made to the GFF to use CSQ. These additions are made with the following script


```{}
library(tidyverse)


#gff_file <- "/vast/eande106/projects/Ryan/protein_structure/ben_1_convergence/annotate_cb/gffs/c_briggsae/test.gff"

fix_mRNA <- function(ID, Parent){
  transcript_id <- strsplit(ID, "=")[[1]][2]
  gene_id <- strsplit(Parent, "=")[[1]][2]
  
  new_at <- glue::glue("ID={transcript_id};Parent={gene_id};biotype=protein_coding")
  return(new_at)
} 


add_mrna_biotype <- function(gff_file, transcript_type = "mRNA"){
	
	gff_cols <- c("chrm_id", "source", "type", "start", "end", "score", "strand", "phase", "attributes") 
	
	gff <- data.table::fread(gff_file, header = FALSE, col.names = gff_cols)  #Separate the attribute column
	
	mrna_features <- gff  %>%
		dplyr::filter(type == transcript_type)  %>% 
		separate(attributes, sep=";", into = c("ID", "Parent")) %>% 
		dplyr::mutate(attributes = map2_chr(ID, Parent, fix_mRNA)) %>% 
		select(-ID, -Parent)

	#return(mrna_features)
	other_features <- gff  %>%
		dplyr::filter(type != transcript_type)  

	reformatted <- bind_rows(mrna_features, other_features)
	

	#name and save output file
	today <- format(Sys.time(), '%Y%m%d')
	file_id <- basename(gff_file)
	write_tsv(reformatted, glue::glue("{file_id}_reformatted_{today}.gff"), col_names = FALSE)
	 


}

ct_gff = "/vast/eande106/projects/Ryan/protein_structure/ben_1_convergence/annotate_cb/gffs/c_tropicalis/NIC58.final_annotation.fixed.CSQ.gff"
cb_gff = "/vast/eande106/projects/Ryan/protein_structure/ben_1_convergence/annotate_cb/gffs/c_briggsae/Curation-VF-230214.PC.clean.renamed.csq.gff3"

#Transcript type allows the "type" column in the gff to be dynamic 
add_mrna_biotype(ct_gff, transcript_type = "transcript")
add_mrna_biotype(cb_gff, transcript_type = "mRNA")
```
As of 05/02/23 these files are in the respective genomes folder on Rockfish

# Output

```
├── strain_vcf
│   ├── {strain}.{date}.vcf.gz
│   └── {strain}.{date}.vcf.gz.tbi
└── variation
    ├── WI.{date}.hard-filter.isotype.snpeff.vcf.gz
    ├── WI.{date}.hard-filter.isotype.snpeff.vcf.gz.tbi
    ├── snpeff.stats.csv
	├── WI.{date}.hard-filter.isotype.bcsq.vcf.gz
    ├── WI.{date}.hard-filter.isotype.bcsq.vcf.gz.tbi
    ├── WI.{date}.hard-filter.isotype.bcsq.vcf.gz.stats.txt 
	└── WI.{date}.strain-annotation.bcsq.tsv
 
```
# Data storage

## Cleanup

Once the pipeline has complete successfully and you are satisfied with the results, the final data can be moved to their final storage place on Rockfish accordingly:

* Both the `strain_vcf` and the `variation` folders can be moved to `/vast/eande106/data/{species}/WI/variation/{date}/vcf`
* If applicable, all snpeff `.bed` files (HIGH, LOW, MODERATE, etc.) can be moved to `/vast/eande106/data/{species}/WI/variation/{date}/tracks/` (*As of 20210901 this is no longer being produced for CaeNDR*)

## Updating CaeNDR and NemaScan

Check out the [CaeNDR](../other/caendr.md) page and the [WI protocol](https://katiesevans9.notion.site/Wild-isolate-sequence-analysis-protocol-00e76cc7f55f4bf6ab644dd99c883727) for more information about updating a new data release for CaeNDR.

