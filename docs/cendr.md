# CeNDR

[TOC]

## Adding isotype images

Isolation photos are initially prepared on dropbox and are located in the folder here:

```
~/Dropbox/Andersenlab/Reagents/WormReagents/isolation_photos/c_elegans
```

Each file should be named using the strain name (_e.g._ `ECA744.jpg`). Then you will use __imagemagick__ to scale the images down to 1000 pixels (width) and generate a 150px thumbnail.

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