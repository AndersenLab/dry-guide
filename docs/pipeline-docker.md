# Create docker iamge

[Docker](https://www.docker.com) can help us to maintain our computational environments. Each of our Nextflow pipeline has a dedicated docker iamge in our lab. and all the docker files should be avalible at [dockerfile](https://github.com/AndersenLab/dockerfile).

## Dockerfile

To simplify the image building, we can use conda to install most of the tools. We can collect the tools avalible on [conda cloud](https://anaconda.org) into a `conda.yml` file, which might looks like this.

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
#  - vcfkit=0.1.6
  - csvtk=0.18.2
  - r=3.6.0
  - r-ggplot2=3.1.1
  - r-readr=1.3.1
  - r-tidyverse=1.2.1
```

Then, build the `Dockerfile` like this.

```
FROM continuumio/miniconda
MAINTAINER XXX <email>

COPY conda.yml .
RUN \
   conda env update -n root -f conda.yml \
&& conda clean -a

# install other tools not avalible on conda cloud
RUN apt-get install -y procps  
RUN R -e "install.packages('roperators',dependencies=TRUE, repos='http://cran.us.r-project.org')"
```

!!! Note
    Put the `conda.ymal` and `Dockerfile` under the same folder

## Build docker image

To build the docker iamge, you need [docker desktop](https://www.docker.com/products/docker-desktop) installed in your local machine. Also you should sign up the dockerhub to enable pushing docker image to cloud.

Go to the folder which have `conda.ymal` and `Dockerfile`, run

```
docker build -t <dockerhub account>/<name of the iamge> .
```

You can use `docker image ls` to check the image list you have in your local machine.

Importantly, you have to check if the tools were installed sucessfully in your docker image. To test the docker iamge, run 

```
docker run -ti <dockerhub account>/<name of the iamge> sh
```

The above command will let you into the docker image, where you can check the tools by their normal commands. Make sure all the tools you need have been installed appropriately. 

## Push docker image to dockerhub

After the image check, you are ready to push the image to dockerhub which allows you to download the image when ever you need to use.

```
docker push <dockerhub account>/<name of the iamge>
```

