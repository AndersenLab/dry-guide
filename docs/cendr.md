# CeNDR

[TOC]

## Setting up CeNDR

### Clone the repo

Clone the repo first.

```
git clone http://www.github.com/andersenlab/cendr
```

__Switch to the development branch__

```
git checkout --track origin/development
```

Next you need to install the requirements. You should do this in a virtualenv or pyenv-based environment.

```
pip install -r requirements.txt
```

Once the requirements are installed you need to load credentials. Create a folder called `env_config` and download the appropriate credential files from the secret credentials location. Add these to the `env_config` folder. __Do not commit these credentials to github__.

### Download and install the gcloud-sdk

Install the [gcloud-sdk](https://cloud.google.com/sdk/downloads)

### Create a cendr gcloud configuration

```
gcloud config configurations create cendr
```

### Install direnv

[direnv](https://direnv.net/) allows you to load a configuration file when you enter the development directory. Please read about how it works. CeNDR uses a `.envrc` file within the repo
to set up the appropriate environmental variables.

### Obtain credentials

Credentials are located in a secret credentials location and should be downloaded to a new folder called `env_config` at the base of the repo.

### Load the database

The site uses an SQLite database that can be setup by running:

```
flask init_db
```

This will update the SQLite database used by CeNDR (`base/cendr.db`). The tables are:

* `homologs` - A table of homologs+orthologs.
* `strain` - Strain info pulled from the google spreadsheet __C. elegans WI Strain Info__.
* `wormbase_gene` - Summarizes gene information; Broken into component parts (e.g. exons, introns etc.).
* `wormbase_gene_summary` - Summarizes gene information. One line per gene.
* `metadata` - tracks how data was obtained. When. Where. etc.


!!! Important
    You must have credentials in place to install the database

### Test the site

You can at this point test the site locally by running:

```
flask run
```

----

## Creating a new release

Before a new release is possible, you must have first completed the following tasks:

__See [Add new sequence data for further details](adding-seq-data.md)__.

1. Add new wild isolate sequence data, and process with the `trimmomatic-nf` pipeline.
1. Identified new isotypes using the `concordance-nf`
1. Updated the `C. elegans WI Strain Info` spreadsheet, adding in new isotypes. 
1. Update the release column to reflect the release data in the `C. elegans WI Strain Info` spreadsheet
1. Run and process sequence data with the `wi-nf` pipeline.

Pushing a new release requires a series of steps described below.

### Uploading BAMs

You will need AWS credentials to upload BAMs to Amazon S3. These are available in the secret credentials location. 

```
pip install aws-shell
aws configure # Use s3 user credentials
```

Once configured, navigate to the BAM location on b1059.

```
cd /projects/b1059/data/alignments/WI/isotype
# CD to bams folder...
aws s3 sync . s3://elegansvariation.org/bam
```

Run this command in screen to ensure that it completes (it's going to take a while)

### Uploading Release Data

When you run the `wi-nf` pipeline it will create a folder with the format `WI-YYYYMMDD`. These data are output in a format that CeNDR can read as a release. You must upload the `WI-YYYYMMDD` folder to google storage with a command that looks like this:

```
# First cd to the path of the results folder (WI-YYYYMMDD) from the `wi-nf` pipeline.
gsutil rsync . gs://elegansvariation.org/releases/YYYYMMDD/
```

!!! Important
    Use rsync to copy the files up to google storage. Note that the `WI-` prefix has been dropped from the `YYYYMMDD` declaration.

### Bump the CeNDR version number and change-log

Because we are creating a new data release, we need to "bump" or move up the CeNDR version. The CeNDR version number is specified in a file at the base of the repo: `travis.yml`. Modify this line:

```
- export VERSION_NUM=1-2-8
```

And increase the version number by 1 (e.g. 1-2-9).

You should also update the change log and/or add a news item. The change-log is a markdown file located at `base/static/content/help/Change-Log.md`; News items are located at `base/static/news/`. Look at existing content to get an idea of how to add new items. It is fairly straightforward. You should be able to see changes on the test site.

### Adding the release to the CeNDR website

After the site is loaded, the BAMs and release data are up, and the database is updated, you need to modify the file `base/constants.py` to add the new release. The date must match the date of the release that was uploaded. Add your release with the appropriate 
date and the annotation database used (e.g. `("YYYYMMDD", "Annotation Database")`).

```
RELEASES = [("20180413", "WS263"),
            ("20170531", "WS258"),
            ("20160408", "WS245")]
```

Commit your changes to the __development branch__ of CeNDR and push them to github. Once pushed, travis-ci will build the app and deploy it to a test branch. Use the google app engine interface to identify and test the app.

If everything looks good open a pull request bringing the changes on the development branch to the master branch. Again, travis-ci will launch the new site.

!!! Important
    You need to shut down development instances and older versions of the site on the google-app engine interface once you are done testing/deploying new instances to prevent us from incurring charges for those running instances.

### Adding isotype images

Isolation photos are initially prepared on dropbox and are located in the folder here:

```
~/Dropbox/Andersenlab/Reagents/WormReagents/isolation_photos/c_elegans
```

Each file should be named using the isotype name and the strain name strain name in the following format:

```
<isotype>_<strain>.jpg
```

Then you will use __[imagemagick](https://www.imagemagick.org/)__ (a commandline-based utility) to scale the images down to 1000 pixels (width) and generate a 150px thumbnail.

```
for img in `ls *.jpg`; do
    convert ${img} -density 300 -resize 1000 ${img}
    convert ${img} -density 300 -resize 150  ${img/.jpg/}.thumb.jpg
done;
```

Once you have generated the images you can upload them to google storage. They should be uploaded to the following location:

```
gs://elegansvariation.org/photos/isolation
```

You can drag/drop the photos using the web-based browser or use __gsutil__:

```
# First cd to the appropriate directory
# cd ~/Dropbox/Andersenlab/Reagents/WormReagents/isolation_photos/c_elegans

gsutil rsync -x ".DS_Store" . gs://elegansvariation.org/photos/isolation
```

The script for processing files is located in the dropbox folder and is called 'process_images.sh'. It's also here:

```
for img in `ls *.jpg`; do
    convert ${img} -density 300 -resize 1000 ${img}
    convert ${img} -density 300 -resize 150  ${img/.jpg/}.thumb.jpg
done;

# Copy using rsync; Skip .DS_Store files.
gsutil rsync -x ".DS_Store" . gs://elegansvariation.org/photos/isolation
```

### Update the `current` variant datasets.

The `current` folder located in `gs://elegansvariation.org/releases` contains the latest variant datasets and is used by WormBase to display natural variation data. Once you've completed a new release, update the files in this folder `gs://elegansvariation.org/releases/current` folder. 

## Updating Publications

The publications page (`/about/publications`) is generated using a google spreadsheet. The spreadsheet can be accessed [here](https://docs.google.com/spreadsheets/d/1ghJG6E_9YPsHu0H3C9s_yg_-EAjTUYBbO15c3RuePIs/edit). You can request access to edit the spreadsheet by visiting that link.

The last row of the spreadsheet contains a function that can fetch publication data from Pubmed using its API. Simply fill in column A with the PMID (Pubmed Identifier), and the publication data will be fetched.

Once you have retrieved the latest pubmed data, create a new row and copy/paste the values for any new publications so they are not fetched from the Pubmed API.

Alternatively, you can fill in the details for a publication manually. In either case, any details added should be double checked. Changes should be instant, but there may be some dely on the CeNDR website.
