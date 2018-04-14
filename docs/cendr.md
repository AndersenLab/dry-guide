# CeNDR

[TOC]

## Adding isotype images

Isolation photos are initially prepared on dropbox and are located in the folder here:

```
~/Dropbox/Andersenlab/Reagents/WormReagents/isolation_photos/c_elegans
```

Each file should be named using the isotype name and the strain name strain name in the following format:

```
<isotype>_<strain>.jpg
```

Then you will use __imagemagick__ to scale the images down to 1000 pixels (width) and generate a 150px thumbnail.

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

## Creating a new release

Before a new release is possible, you must have first completed the following tasks:

__Add a wild isolate sequence data__

1. Add new wild isolate sequence data, and process with the `trimmomatic-nf` pipeline.
1. Identified new isotypes using the `concordance-nf`
1. Updated the `C. elegans WI Strain Info` spreadsheet, adding in new isotypes. 
1. Update the release column to reflect the release data in the `C. elegans WI Strain Info` spreadsheet
1. Run and process sequence data with the `wi-nf` pipeline.

__Update the CeNDR Database__

To update the CeNDR database you need to first clone the CeNDR repo.

```
git clone http://www.github.com/andersenlab/cendr
```

Next you need to install the requirements. Preferably, you will do this in a virtual environment.

```
pip install -r requirements.txt
```

Once installed, you can use the command line utilities built into the CeNDR repo to update the database.

```
flask init_db
```

This will update the SQLite database used by CeNDR. The tables within this database are:



The above command will download the latest set of 
