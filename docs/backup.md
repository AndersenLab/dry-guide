# Backup data

## Backup files to google cloud

**1. Download and setup google cloud SDK - only need to do once**

```
# download from google
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-325.0.0-linux-x86_64.tar.gz

# un-tar
tar -xf google-cloud-sdk-325.0.0-linux-x86_64.tar.gz

# add to path
./google-cloud-sdk/install.sh

# initialize and authenticate
gcloud init

# follow prompts
```

**2. Add files from QUEST folder to google cloud bucket**

You can see the structure of the google buckets by going [here](https://console.cloud.google.com/storage/browser?authuser=1&project=andersen-lab&prefix=). Most things are currently in the `elegansvariation.org` bucket.

```
# copy all files in current directory to specified bucket
gsutil cp * gs://<YOUR_BUCKET_NAME>

# copy all files in parallel (good for multiple files)
gsutil -m cp * gs://<YOUR_BUCKET_NAME>

# view list of files in bucket
gsutil ls gs://<YOUR_BUCKET_NAME>

# example - move all vcf files to the variation folder under 20210121 release
gsutil -m cp * gs://elegansvarition.org/releases/20210121/variation
```

!!! Note
    We store `.bam` and `.vcf` files on google because they are used by CeNDR. The `.bam` files are not separated by release but the `.vcf` files (and accompanying files for a CeNDR release) are.


## Local Backup

We also store all `.fastq` files locally in duplicate. If anything ever happens to any downstream data we can always recreate it with the original FASTQ files. **This step is extremely important**.

A list of all data and which hardrive it is backed up on can be found [here](https://docs.google.com/spreadsheets/d/16jlaDIRPNu0jnVCg395goF9B70UP1skXdrWO6bWrwrA/edit?usp=sharing). The main copies of all data can be found on **Pangolin, Armadillo, Raven, Karasu, and Turkey**

1. Choose a hardrive that has enough space for the data you need to back up and plug it in to your computer
2. Change directories on your local computer into the hardrive `cd /Volumes/{name_of_harddrive}/{path}`
3. Sync data from QUEST with `Rsync -avh <netid>@quest.northwestern.edu:<path_to_folder_quest> .`. *Don't forget the '.'!!!*
4. When finished, check that the total file size is the same on the hard drive and on QUEST, you can always re-run the Rsync command to verify it is complete.
5. Repeat with the second hard drive
6. Complete the [data backup google sheet](https://docs.google.com/spreadsheets/d/16jlaDIRPNu0jnVCg395goF9B70UP1skXdrWO6bWrwrA/edit?usp=sharing) with your new backup.

## Adding FASTQ to SRA project

Checkout this [guide](sra.md) for how to upload data to SRA. This should be done with wild isolate FASTQ files after each new CeNDR release. The SRA submission ID might need to be cited in publications.