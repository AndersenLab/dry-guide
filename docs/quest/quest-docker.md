# Create docker image

[TOC]


[Docker](https://www.docker.com) can help us to maintain our computational environments. Each of our Nextflow pipeline has a dedicated docker image in our lab. And all the docker files should be avalible at [dockerfile](https://github.com/AndersenLab/dockerfile).

## Dockerfile

To simplify the image building, we can use conda to install most of the tools. We can collect the tools available on [conda cloud](https://anaconda.org) into a `conda.yml` file, which might looks like this.

```
name: concordance-nf
channels:
  - defaults
  - bioconda
  - r
  - biobuilds
  - conda-forge
dependencies:
  - bwa=0.7.17
  - sambamba=0.7.0
  - samtools=1.9
  - picard=2.20.6
  - bcftools=1.9
  - csvtk=0.18.2
  - r=3.6.0
  - r-ggplot2=3.1.1
  - r-readr=1.3.1
  - r-tidyverse=1.2.1
```

Then, build the `Dockerfile` as below.

```
FROM continuumio/miniconda
MAINTAINER Katie Evans <kathrynevans2015@u.northwestern.edu>

COPY conda.yml .
RUN \
   conda env update -n root -f conda.yml \
&& conda clean -a

# install other tools not avalible on conda cloud -- tidyverse sometimes need to be installed here separately...
RUN apt-get update && apt-get install -y procps
RUN R -e "install.packages('roperators',dependencies=TRUE, repos='http://cran.us.r-project.org')"
```

!!! Note
    Put the `conda.ymal` and `Dockerfile` under the same folder.

## Build docker image

To build the docker image, you need [docker desktop](https://www.docker.com/products/docker-desktop) installed on your local machine. Also you should sign up the dockerhub to enable pushing docker image to cloud.

Go to the folder which have `conda.ymal` and `Dockerfile`, run

```
docker build -t <dockerhub account>/<name of the image> . # don't ingore the dot here
```

You can use `docker image ls` to check the image list you have in your local machine.

Importantly, you have to check if the tools were installed sucessfully in your docker image. To test the docker image, run 

```
docker run -ti <dockerhub account>/<name of the image> sh
```

The above command will let you into the docker image, where you can check the tools by their normal commands. Make sure all the tools you need have been installed appropriately. 

## Tag image with a version

There are sometimes issues with Nextflow thinking it has the latest docker image when it really doesn't. To avoid this issue, it is useful to tag each updated docker image with a version tag. Remember to update the docker call in nextflow to use the new version!

```
docker image tag <dockerhub account>/<name of the image>:latest <dockerhub account>/<name of the image>:<version tag>
```

## Push docker image to dockerhub

After the image check, you are ready to push the image to dockerhub which allows you to download the image when ever you need to use.

```
docker push <dockerhub account>/<name of the image>:<version tag>
```

## Running Nextflow with docker

If running Nextflow locally, a docker container can be used with the following line (check out the [documentation](https://www.nextflow.io/docs/latest/docker.html)):

```
nextflow run <your script> -with-docker [docker image]
```

Alternatively, a docker container can be specified within the `nextflow.config` script to avoid adding an extra parameter:

```
process.container = 'nextflow/examples:latest'
docker.enabled = true

// if on quest:
// singularity.enabled = true
```

!!! Important
    When running Nextflow with a docker container on QUEST, it is necessary to replace the `docker` command with `singularity` (although you still must build a docker container). You must also load singularity using `module load singularity` before starting a run.

### Caching singularity images on QUEST

To make the most out of using a shared cache directory for singularity on b1059, make sure to add this line to your `~/.bash_profile` before you run a pipeline for the first time (Note: this is not needed to USE a previously cached image, but only when you ADD a new one).

```
export SINGULARITY_CACHEDIR='/projects/b1059/singularity/'
```
