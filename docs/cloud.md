## AndersenLab cloud resources

[TOC]

For full documentation visit [mkdocs.org](http://mkdocs.org).

# Google Domains

Any domain names the lab uses should be registered with Google Domains. The two ones currently are:

* [andersenlab.org](http://andersenlab.org)
* [elegansvariation.org](http://elegansvariation.org)

Google domains can be used to forward domain-specific email addresses if necessary. 
For example, example@andersenlab.org could be created and forwarded to an email address.

# Google Cloud

Google cloud is used for a variety of services that the lab uses.

### Google Cloud Storage

Google cloud storage is used to store and distribute files on CeNDR, for cegwas, and for some files on the lab website.

#### Buckets

Files are grouped into 'buckets' on google storage. We use the following buckets:

##### elegansvariation.org

This bucket contains all the data associated with elegansvariation.org. It is broken down into six primary directories.

* __browser_tracks__ - for genome-browser tracks that rarely if ever change.
* __db__ - Storage/access to the SQLite database.
* __photos__ - sample collection photos.
* __releases__ - dataset releases. For more detail, see [post-gatk-nf](pipeline-postGATK.md).
* __reports__ - images and data files within reports.
* __static__ - static assets used by the site.
* __bam__ - stores all BAM files at the strain level

##### andersenlab.org

In some cases the data associated with a publication is too large to put on github. We store those data here, along with a 
couple other odds and ends.

##### Other buckets

Google Cloud creates a bunch of other buckets too. Most of these should be ignored as they are part of the 

##### Secret Bucket

There is one other secret bucket. Ask Erik about it.

##### cegwas (deprecated)

Currently this bucket has a SQLite database used by cegwas. This will be replaced to use the same database
used by CeNDR in the __db/__ folder.

Once the new version of cegwas is developed - this section should be removed, and that bucket should be deleted.

# Google datastore

Google datastore used as the database for CeNDR. It stores information on mappings, traits, and more.

# App engine

App engine is the platform CeNDR runs on.

# Error Reporting

Google cloud contains a really nice error reporting interface. Error reports are generated whenever something goes wrong on
CeNDR. A github issue can be created for these errors and they can be addressed.

# BigQuery

We have used bigquery in the past for large query jobs. We are not actively using it as of late.

# AWS

## S3

In the past we stored BAMs on AWS at `elegansvariation.org`, however these data have now been migrated to GCP as of the 20210121 CeNDR release.

## Fargate

Amazon Fargate was used to run the mapping pipeline on CeNDR in the past, but this is now moved to GCP as of 2021
