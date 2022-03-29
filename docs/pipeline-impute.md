# VCF Imputation

[TOC]

Although there is not a full "pipeline" for imputation, there is a script to run and this step is crucial, as fine-mapping for NemaScan requires an imputed VCF.

# Script overview

```
#!/bin/bash
#SBATCH -J beagle      
#SBATCH -A b1042               
#SBATCH -p genomicsguestA              
#SBATCH -t 6:00:00            
#SBATCH -n 24 # can maybe be lower now
#SBATCH --mem=50G # can change if needed

#################
# edit these variables:
vcf='/projects/b1059/data/c_briggsae/WI/variation/20210901/vcf/WI.20210901.hard-filter.isotype.vcf.gz'
species='c_tropicalis'
date='20210901'
#################

# load modules:
module load bcftools

# do the imputation with beagle5.2:
for i in I II III IV V X
do
    java -jar /projects/b1059/software/beagle/beagle5.2/beagle.28Jun21.220.jar chrom=${i} \
    window=5 \
    overlap=2 \
    impute=true \
    ne=100000 \
    nthreads=3 \
    imp-segment=0.5 \
    imp-step=0.01 \
    cluster=0.0005 \
    gt=$vcf \
    map=/projects/b1059/data/$species/genomes/genetic_map/chr${i}.map \
    out=${i}.b5
    
    bcftools index ${i}.b5.vcf.gz
done

# now do mitochondria - genetic map might not exist for briggsae/tropicalis so use default, should be okay just mito...
if [ -f "/projects/b1059/data/$species/genomes/genetic_map/chrMtDNA.map" ]
then 
    java -jar /projects/b1059/software/beagle/beagle5.2/beagle.28Jun21.220.jar chrom=MtDNA \
    window=5 \
    overlap=2 \
    impute=true \
    ne=100000 \
    nthreads=3 \
    imp-segment=0.5 \
    imp-step=0.01 \
    cluster=0.0005 \
    gt=$vcf \
    map=/projects/b1059/data/$species/genomes/genetic_map/chrMtDNA.map \
    out=MtDNA.b5
else 
    java -jar /projects/b1059/software/beagle/beagle5.2/beagle.28Jun21.220.jar chrom=MtDNA \
    window=5 \
    overlap=2 \
    impute=true \
    ne=100000 \
    nthreads=3 \
    imp-segment=0.5 \
    imp-step=0.01 \
    cluster=0.0005 \
    gt=$vcf \
    out=MtDNA.b5
fi

bcftools index MtDNA.b5.vcf.gz

# concat all the chrom vcfs
bcftools concat *.b5.vcf.gz -O z -o WI.${date}.impute.isotype.vcf.gz
bcftools index -t WI.${date}.impute.isotype.vcf.gz
bcftools stats --verbose WI.${date}.impute.isotype.vcf.gz > WI.${date}.impute.isotype.stats.txt
```

## Software requirements

You can run this script on QUEST using the latest version of Beagle (5.2) in the `/projects/b1059/software/` folder. Also requires `bcftools` (loaded in script).

## Usage

1. Copy the script above and create a new file on QUEST (i.e. `impute.sh`). 
2. Manually edit the `vcf` input, the `species`, and the `date` (to be the release date for all VCFs). 
3. Submit the job with `sbatch impute.sh`

## Parameters

These parameters have been checked and decided on by previous lab members and Erik. Some chromosomes might require a window size of 3 and an overlap of 1. In recent conversation with the person who manages Beagle, they mentioned we should probably use default values unless we have done simulations to show these values are better. Note for the future maybe.
