# Setting Up Quest

### Setting up your environment

In order to do any sort of analysis you will need to set up your environment first. Daniel Cook has written a script that will set up the environment on quest. The script works as of 2017-12-05. However, because software gets updated frequently it may break and may need to be updated. However - it will get you 90% of the way there. You need to __read__ the script before running it and to understand what it is doing. And you may need to make adjustments if it is out of date.

__Please read the script first__

[Quest Setup Scripts](https://gist.github.com/danielecook/aed4a6fd53195fca7a3297f054d613c7)

__Then read about what the script is doing below__

### What do these scripts do?

#### Cleans up the existing environment - Removes `~/.linuxbrew` and `~/.cache`

These older files are removed.

#### Installs linuxbrew

Linuxbrew is the tool used to install additional software.

#### Installs the following software

```
curl
git
nextflow
pigz
igvtools
vcfanno
snpeff
fastq-tools
muscle
sambamba
google-cloud-sdk
# utilities
pyenv
pyenv-virtualenv
autojump
```

#### Replaces your .bash_profile

The __.bash_profile__ will be replaced with the one found on the [Quest Setup Scripts](https://gist.github.com/danielecook/aed4a6fd53195fca7a3297f054d613c7) page.

#### Installs Python 2.7.11 and Python 3.6.0

These are installed using [pyenv](https://github.com/pyenv/pyenv).

__python 2.7.11__ is set as the default global version.

#### Installs python modules for python 2.7.11

```
pip
numpy
setuptools
cython
scipy
gcloud
vcf-kit
bam-toolbox (for coverage calculations)
```

### Configures nextflow

See [Quest-Nextflow](quest-nextflow) for further details on what exactly is being done here.


### ... And More

Please __read__ the script prior to running it.

__Run the script using this command__

__Run the script using the command below__

```
curl https://gist.githubusercontent.com/danielecook/aed4a6fd53195fca7a3297f054d613c7/raw/quest_setup.sh | bash
```
