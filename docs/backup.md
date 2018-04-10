# Configuring AWS Client

The aws commandline utility is installed as a part of the [andersen-lab-env](quest-andersen-lab-env.md)

You must provide the appropriate s3 credentials in order to backup BAMs. Erik knows where these are.

```
aws configure
```

# Backing up BAMS to Amazon S3

BAMS are uploaded to S3. This allows us to back them up there and access them through CeNDR.

Currently, the only BAMs that we upload to s3 are wild isolate BAMs. Once you've added new sequence data you can sync the new BAMs by running the following:

```
# First, cd to the location of isotype bams
cd /projects/b1059/data/alignments/WI/isotype
# Then run the following command
aws s3 sync s3://elegansvariation.org/bam .
```

# Local Backup

In addition to backing up BAMs on S3, we also backup FASTQs locally. There are four hard drives:

* PortusTotus
* Hawkeye
* Elephant
* Rhino

Each one is labeled with what is backed up on it. After you have added new sequenced data you should sync __FROM__ the data/fastq folder on b1059 __TO__ the drive (Do __not__ go the other way or do a two way sync).