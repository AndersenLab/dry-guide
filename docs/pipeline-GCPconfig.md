# Running Nextflow pipeline on GCP

[TOC]

Google genomic API allows auto scale for computational resources by creating and closing VMs automatically. We have a dedicated google project `caendr` which using Google genomic APT for all the nextflow pipelines in our lab. To access it, you should provide your gmail accout to Erik and ask Erik give you a `project owner` role for `caendr`. I already preset the project to enable running Nextflow pipelines using [Google genomic API](https://cloud.google.com/life-sciences/docs/how-tos/migration). See below for more details.

## Enable API

Go the the main page of [google cloud platform](https://console.cloud.google.com/home/dashboard?project=caendr). In the Produck & Services menu, click `APIs & Services`, and then click `Enable APIs and Services`. The following APIs should be enabled to run nextflow pipeline on GCP.

* Genomics API
* Cloud Life Sciences API
* Compute Engine API
* Google Container Registry API

## Create service account

Go to the [IAM & admin](https://console.cloud.google.com/iam-admin/serviceaccounts?project=caendr), find the `Service accounts`. Click `CREATE SERVICE ACCOUNT` to create a new service accounts. **Note** this service accounts must have a `project owner` role to run Nextflow pipelines. The service account I created here is called `nextflowRUN`.

---
You don't need to do the above processes when you use GCP. But you have to do all the following processes to make sure you have the right permissions to `caendr`.


## Generate credential for the service account

After you get the access to `caendr`, go to the [API & Services](https://console.cloud.google.com/apis/credentials?project=caendr). Click the `Create credentials` button, select `Service account key`. And choose `nextflowRUN` to generate a JSON file, which is a privite key file for using `nextflowRUN`. Download the file and save it in a safe place. Finally, define the `GOOGLE_APPLICATION_CREDENTIALS` variable in `.bash_profile` with the directory of the JSON file. Which should looks like the example.

```
export GOOGLE_APPLICATION_CREDENTIALS=$HOME/google_creds/caendr-2cae6210c8d1.json
```

## Nextflow version and mode

Only the version of 19.07.0 or higher of Nextflow are compatible with GCP. And also, the Nextflow should have a google mode. You can define the version and mode in `.bash_profile`.

```
export NXF_VER=19.07.0
export NXF_MODE=google
```

Then, run the following code to update or install Nextflow.

```
curl https://get.nextflow.io | bash
```

## Configure Nextflow for GCP

To run Nextflow pipelines on GCP, you need to build docker images for them. Check the [docker file repo](https://github.com/AndersenLab/dockerfile) of our lab for more information.

The google genomic API has its own executor called `google-pipelines`, you need to define the `executor` variable with `google-pipelines` in the `nextflow.config` file. Here is the example for [concordance-nf](http://andersenlab.org/dry-guide/pipeline-concordance/).

```
docker {

    enabled = true

}

process {
    executor = 'google-pipelines'

    withLabel: bam_coverage {
        container = 'faithman/bam_toolbox:latest'
    }

    container = 'faithman/concordance:latest'
    machineType = 'n1-standard-4'
}

google {
    project = 'caendr'
    zone = 'us-central1-a'
}

cloud {
	preemptible = true
}

executor {
    queueSize = 500
}
```

!!! Important
    The file system of google buckets is not like S3 that can read/write directly by most softwares. You have to use gsutil tool to interact with google buckets to read/write files in most situations. Nextflow has built-in functions to interact with google buckets, but you still can not read/write files directly in your script. All the files have to be read and write via channels in Nextflow!

    
