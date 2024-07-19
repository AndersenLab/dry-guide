# NemaScan

GWA Mapping and Simulation with _C. elegans, C. tropicalis, and C. briggsae_

# Pipeline overview

```
O~~~     O~~                                      O~ O~~
O~ O~~   O~~                                    O~~    O~~
O~~ O~~  O~~    O~~  O~~~ O~~ O~~     O~~       O~~          O~~~    O~~    O~~~ O~~
O~~  O~~ O~~  O~   O~ O~~  O~  O~~  O~~  O~~      O~~      O~~     O~~  O~~  O~~  O~~
O~~   O~ O~~ O~~~~~O~ O~~  O~  O~~ O~~   O~~         O~~  O~~     O~~   O~~  O~~  O~~
O~~    O~O~~ O~       O~~  O~  O~~ O~~   O~~   O~~    O~~  O~~    O~~   O~~  O~~  O~~
O~~      O~~   O~~~~  O~~  O~  O~~   O~~ O~~~    O~ O~~      O~~~   O~~ O~~~ O~~  O~~

parameters              description                                            Set/Default
==========              ===========                                            ========================
--traitfile             Name of file containing strain and phenotype           (required)
--vcf                   Generally a CaeNDR release date or path to vcf         (optional - 20231213)
--download_vcf          Fetch VCF files from CaeNDR                            (optional - false)
--species               c_elegans, c_briggsae, or c_tropicalis                 (optional - c_elegans)
--mapping               Run GWAS mapping                                       (optional - true)
--matrix                Create genotype matrix                                 (optional - false)
--simulation            Run GWAS mapping                                       (optional - false)
--out                   Name of folder that will contain the results           (optional - Analysis_Results-{date})

Optional arguments (Mapping):
--pca                   Use PCA as a covariate for mapping                     (optional - false)
--finemap               Perform fine-mapping                                   (optional - true)
--mediation             Run mediation analysis                                 (optional - false)
--fix                   Filter and prune trait values                          (optional - true)

Optional arguments (Mapping & Matrix):
--strains               File with set of strains to use                        (optional - strain_file.tsv)

Optional arguments (Simulation):
--simulate_nqtl         Number of QTL to simulate per phenotype                (optional - simulate_nqtl.csv)
--simulate_h2           File with phenotype heritability                       (optional - simulate_h2.csv)
--simulate_reps         Number of replicates to simulate                       (optional - 2)
--simulate_maf          File of minor allele frequency thresholds              (optional - simulate_maf.csv)
--simulate_eff          File of effect size range (e.g. 0.2-0.3)               (optional - simulate_effect_sizes.csv)
--simulate_strains      File of strain names and strain lists                  (optional - simulate_strains.csv)
--simulate_qtlloc       File with genomic range where markers are pulled from  (optional - whole genome)

Optional arguments (Mapping & Simulation):
--sthresh               Significance threshold for QTL                         (optional - 'BF')
--group_qtl             QTL distance to combine the QTL into one               (optional - 1000)
--ci_size               Number of SNVs used to define the QTL CI               (optional - 150)
--maf                   Minimum minor allele frequency to use                  (optional - 0.05)
--sparse_cut            Off-diagonal relatedness matrix value cutoff           (optional - 0.05)

```

![](../img/nemascan.drawio.svg)

## Software Requirements

* The latest update requires Nextflow version 23+. On Rockfish, you can access this version by loading the `nf23_env` conda environment prior to running the pipeline command:

```
module load python/anaconda
source activate /data/eande106/software/conda_envs/nf23_env
```

### Relevant Docker Images

* `andersenlab/nemascan` ([link](https://hub.docker.com/r/andersenlab/nemascan)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/nemascan.Dockerfile` or `.github/workflows/build_nemascan_docker.yml` GitHub actions will create a new docker image and push if successful.
* `andersenlab/mediation` ([link](https://hub.docker.com/r/andersenlab/mediation)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/mediation.Dockerfile` or `.github/workflows/build_med_docker.yml` GitHub actions will create a new docker image and push if successful.
* `andersenlab/gcta` ([link](https://hub.docker.com/r/andersenlab/gcta)): Docker image is created within this pipeline using GitHub actions. Whenever a change is made to `env/gcta.Dockerfile` or `.github/workflows/build_gcta_docker.yml` GitHub actions will create a new docker image and push if successful.
* `andersenlab/r_packages` ([link](https://hub.docker.com/r/andersenlab/r_packages)): Docker image is created manually, code can be found in the [dockerfile](https://github.com/AndersenLab/dockerfile/tree/master/r_packages) repo.
* `andersenlab/prep_sims` ([link](https://hub.docker.com/r/mckeowr1/prep_sims)): Docker image is created manually.
* `andersenlab/assess_sims` ([link](https://hub.docker.com/r/mckeowr1/assess_sims)): Docker image is created manually.

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
nextflow run -latest andersenlab/nemascan --debug
```

## Running on Rockfish

You should run this in a screen or tmux session.

```
nextflow run -latest andersenlab/nemascan -profile mappings --vcf 20210121 --traitfile input_data/c_elegans/phenotypes/PC1.tsv
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
nextflow run -latest andersenlab/nemascan --debug
```

## --traitfile

A tab-delimited formatted (.tsv) file that contains trait information.  Each phenotype file should be in the following format (replace trait_name with the phenotype of interest):

| strain | trait_name_1 | trait_name_2 |
| --- | --- | --- |
| JU258 | 32.73 | 19.34 |
| ECA640 | 34.065378 | 12.32 |
| ... | ... | ... | 124.33 |
| ECA250 | 34.096 | 23.1 |

## --vcf (default: 20231213)

CaeNDR release date for the VCF file with variant data (i.e. "20210121") Hard-filter VCF will be used for the GWA mapping and imputed VCF will be used for fine mapping. If this flag is not used, the most recent VCF for the _C. elegans_ species will be downloaded from [CaeNDR](https://caendr.org).

!!! Note
	If you want to use a custom VCF, you may provide the full path to the vcf in place of the CaeNDR release date. This custom VCF will be used for BOTH GWA mapping and fine-mapping steps (instead of the imputed vcf).

## --download_vcf (default: false)

Fetch VCF files from CaeNDR site rather than using local built-in file paths.

## --species (default: c_elegans)

Choose between `c_elegans`, `c_tropicalis` or `c_briggsae`

## --mapping (default: true)

Indicates to perform a genome-wide analysis of your trait of interest.

## --matrix (default: false)

Indicates to run the matrix production step.

This takes a list of strains and outputs the genotype matrix.

## --simulation (default: false)

Indicates to run QTL simulations. This mode uses simulations to establish GWA performance benchmarks. Users can specify the heritability of simulated traits, the number of QTL underlying simulated traits of interest, the strains the user intends to use in a prospective GWA mapping experiment, or the location of previously detected QTL. Understanding the null expectations of GWA mappings within given parameter spaces may provide experimenters with additional guidance before initiating an experiment, or serve as a validation tool for previous mappings.

## --out (default: Analysis_Results-{date})

A user-specified output directory name. (Default: `Analysis_Results-{date}`)

## Optional Mapping Parameters

### --pca

Use 1st principle component as a covariate for mapping. (Default: false)

### --finemap

Perform fine-mapping of QTL intervals. (Default: true)

### --mediation

'true' or 'false' input for whether or not to run the mediation analysis (overlapping expression variation with phenotypic variation to drive a QTL). (Default: false)

### --fix

Filter trait values by combining strain replicates and pruning extreme values. (Default: true)

## Optional Mapping & Matrix Parameter

### --strains

A file (.tsv) that contains a list of strains used for generating the genotype matrix. There is no header:

```
JU258
ECA640
...
ECA250
```

(Default: `input_data/${params.species}/phenotypes/strain_file.tsv`)

## Optional Simulation Parameters

### --simulate_nqtl 

A single column CSV file that defines the number of QTL to simulate (format: one number per line, no column header) (Default is provided: `input_data/all_species/simulate_nqtl.csv`).

### --simulate_h2 

A CSV file with phenotype heritability. (format: one value per line, no column header) (Default is located: `input_data/all_species/simulate_h2.csv`).

### --simulate_reps

The number of replicates to simulate per number of QTL and heritability (Default: 2).

### --simulate_maf

A single column CSV file that defines the minor allele frequency threshold used to filter the VCF prior to simulations (Default: 0.05).

### --simulate_eff

A CSV file specifying a range of causal QTL effects. QTL effects will be drawn from a uniform distribution bound by these two values. If the user wants to specify _Gamma_ distributed effects, the value in this file can be simply specified as "gamma". (format: one value per line, no column header) (Default is located: input_data/all_species/simulate_effect_sizes.csv).

### --simulate_strains

A TSV file specifying the population in which to simulate GWA mappings. Multiple populations can be simulated at once, but causal QTL will be drawn independently for each population as a result of minor allele frequency and LD pruning prior to mapping. (format: one line per population; supplied population name and a comma-separated list of each strain in the population) (Default is located: input_data/all_species/simulate_strains.tsv).

### --simulate_qtlloc

A .bed file specifying genomic regions from which causal QTL are to be drawn after MAF filtering and LD pruning. (format: CHROM START END for each genomic region, with no header. NOTE: CHROM is specified as NUMERIC, not roman numerals as is convention in _C. elegans_) (Default is located: input_data/all_species/simulate_locations.bed).

## Optional Mapping and Simulation Parameters

### --sthresh

This determines the signficance threshold required for performing post-mapping analysis of a QTL. `BF` corresponds to Bonferroni correction, `EIGEN` corresponds to correcting for the number of independent markers in your data set, and `user-specified` corresponds to a user-defined threshold, where you replace user-specified with a number. For example `--sthresh=4` will set the threshold to a `-log10(p)` value of 4. We recommend using the strict `BF` correction as a first pass to see what the resulting data looks like. If the pipeline stops at the `summarize_maps` process, no significant QTL were discovered with the input threshold. You might want to consider lowering the threshold if this occurs. (Default: `BF`)

### --group_qtl

QTL within this number of markers from each other will be grouped as a single QTL by `Find_GCTA_Intervals_*.R`. (Default: 1000)

### --ci_size

The number of markers for which the detection interval will be extended past the last significant marker in the interval. (Default: 150)

### --maf

The minor allele frequency for filtering variants to use for gwas mapping (default 0.05)

```
nextflow run -latest andersenlab/nemascan --matrix --vcf 20210121 --strains input_data/elegans/phenotypes/strain_file.tsv
```

### -sparse_cut

Any off-diagonal value in the genetic relatedness matrix greater than this is set to 0 (Default: 0.05)


# Input Data Folder Structure (`NemaScan/input_data`)

```
all_species
	├── rename_chromosomes
	├── simulate_effect_sizes.csv
	├── simulate_h2.csv
	├── simulate_maf.csv
	├── simulate_nqtl.csv
	├── simulate_strains.tsv
	└── simulate_locations.bed
c_elegans (repeated for c_tropicalis and c_briggsae)
	└── genotypes  
			├── test_vcf
			├── test_vcf_index
			└── test_bcsq_annotation
	└── phenotypes
			├── PC1.tsv
			├── strain_file.tsv
			└── test_pheno.tsv
	└── annotations
			├── GTF file
			└── refFlat file
	└── isotypes
			├── div_isotype_list.txt
			├── divergent_bins.bed
			├── divergent_df_isotype.bed
			├── haplotype_df_isotype.bed
			└── strain_isotype_lookup.tsv
```

# Mapping Output Folder Structure

```
Phenotypes
	├── strain_issues.txt
	└── pr_traitname.tsv
Genotype_Matrix
	├── Genotype_Matrix.tsv
	└── total_independent_tests.txt
 Mapping
	└── Raw
			├── traitname_lmm-exact_inbred.fastGWA
			└── traitname_lmm-exact.loco.mlma
	└── Processed
			├── traitname_AGGREGATE_qtl_region.tsv
			├── processed_traitname_AGGREGATE_mapping.tsv
			└── QTL_peaks.tsv
Plots
	└── ManhattanPlots
			└── traitname_manhattan.plot.png
	└── LDPlots
			└── traitname_LD.plot.png (if > 1 QTL detected)
	└── EffectPlots
			├── traitname_[QTL.INFO]_LOCO_effect.plot.png (if detected)
			└── traitname_[QTL.INFO]_INBRED_effect.plot.png (if detected)
Fine_Mappings
	└── Data             
			├── traitname_[QTL.INFO]_bcsq_genes.tsv
			├── traitname_[QTL.INFO]_ROI_Genotype_Matrix.tsv
			├── traitname_[QTL.INFO]_finemap_inbred.fastGWA
			└── traitname_[QTL.INFO]_LD.tsv
	└── Plots   
			├── traitname_[QTL.INFO]_finemap_plot.pdf
			└── traitname_[QTL.INFO]_gene_plot_bcsq.pdf
Divergent_and_haplotype
	├── all_QTL_bins.bed
	├── all_QTL_div.bed
	├── div_isotype_list.txt
	└── haplotype_in_QTL_region.txt
Reports
	├── NemaScan_Report_traitname_main.html
	└── NemaScan_Report_traitname_main.Rmd
```

### Phenotypes folder
* `strain_issues.txt` - Output of any strain names that were changed to match vcf (i.e. isotypes that are not reference strains)
* `pr_traitname.tsv` - Processed phenotype file for each trait. This is the file that goes into the mapping

### Genotype_Matrix folder
* `Genotype_Matrix.tsv` - LD-pruned genotype matrix used for GWAS and construction of kinship matrix
* `total_independent_tests.txt` - number of independent tests determined through spectral decomposition of the genotype matrix

### Mapping folder

#### Raw
* `traitname_lmm-exact_inbred.fastGWA` - Raw mapping results from GCTA's fastGWA program using an inbred kinship matrix
* `traitname_lmm-exact.loco.mlma` - Raw mapping results from GCTA's mlma program using a kinship matrix constructed from all chromosomes except for the chromosome containing each tested variant

#### Processed
* `traitname_AGGREGATE_mapping.tsv` - Combined processed mapping results from lmm-exact_inbred and lmm-exact.loco.mlma raw mappings. Contains additional information nested such as 1) rough intervals (see parameters for calculation) and estimates of the variance explained by the detected QTL 2) phenotype information and genotype status for each strain at the detected QTL.
* `traitname_AGGREGATE_qtl_region.tsv` - Contains only QTL information for each mapping. If no QTL are detected, an empty data frame is written.
* `QTL_peaks.tsv` - contains QTL information for each mapping for all traits combined.


### Plots
* `traitname_manhattan.plot.png` - Standard output for GWA; association of marker differences with phenotypic variation in the population.
* `traitname_LD.plot.png` - If more than 1 QTL are detected for a trait, a plot showing the linkage disequilibrium between each QTL is generated.
* `traitname_[QTL.INFO]_INBRED_effect.plot.png` - Phenotypes for each strain are plotted against their marker genotype at the peak marker for each QTL detected for a trait. The dot representing each strain is shaded according to the percentage of the chromosome containing the QTL that is characterized as a selective sweep region.

#### Fine_Mappings folder

##### Data
* `traitname_bcsq_genes.tsv` - Fine-mapping data frame for all significant QTL

##### Plots
* `traitname_qtlinterval_finemap_plot.pdf` - Fine map plot of QTL interval, colored by marker LD with the peak QTL identified from the genome-wide scan
* `traitname_qtlinterval_gene_plot.pdf` - variant annotation plot overlaid with gene CDS for QTL interval


# Simulation Output Folder Structure

```
Genotype_Matrix
	├── [strain_set]_[MAF]_Genotype_Matrix.tsv
	└── [strain_set]_[MAF]_total_independent_tests.txt
Simulations
	├── NemaScan_Performance.example_simulation_output.RData
	└── [specified effect range (simulate_effect_sizes.csv)]
			└── [specified number of simulated QTL (simulate_nqtl.csv)]
					└── Mappings
							├── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_processed_LMM_EXACT_INBRED_mapping.tsv
							├── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_processed_LMM_EXACT_LOCO_mapping.tsv
							├── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_lmm-exact_inbred.fastGWA
							└── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_lmm-exact.loco.mlma
					└── Phenotypes
							├── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_sims.phen
							└── [nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_sims.par
	└── (if applicable) [NEXT specified effect range]
			└── ...
	└── (if applicable) [NEXT specified effect range]
			└── ...
```

### Genotype_Matrix folder
* `*Genotype_Matrix.tsv` - pruned LD-pruned genotype matrix used for GWAS and construction of kinship matrix. This will be appended with the chosen minor allele frequency cutoff and strain set, as they are generated separately for each strain set.
* `*total_independent_tests.txt` - number of independent tests determined through spectral decomposition of the genotype matrix. This will be also be appended with the chosen minor allele frequency cutoff and strain set, as they are generated separately for each strain set.

### Simulations
* `NemaScan_Performance.*.RData` - RData file containing all simulated and detected QTL from each successful simulated mapping. Contains:
1. Simulated and Detected status for each QTL.
2. Minor allele frequency and simulated or estimated effect for each QTL.
3. Detection interval according to specified grouping size and CI extension.
4. Estimated variance explained for each detected QTL.
5. Simulation parameters and the algorithm used for that particular regime.

### Mappings
* As with the mapping profile, raw and processed mappings for each simulation regime are nested within folders corresponding each specified effect range and number of simulated QTL. QTL region files are not provided in the simulation profile; this information along with other information related to mapping performance are iteratively gathered in the generation of the performance .RData file.

### Phenotypes
* `[nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_sims.phen` - Simulated strain phenotypes for each simulation regime.
* `[nQTL]_[rep]_[h2]_[MAF]_[effect range]_[strain_set]_sims.par` - Simulated QTL effects for each simulation regime. NOTE: Simulation regimes with identical numbers of simulated QTL, replicate indices, and simulated heritabilities should have _identical_ simulated QTL and effects.
