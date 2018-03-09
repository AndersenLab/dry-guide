# The Andersen Lab Software Environment

## andersen-lab-env

In order for comutational research to be reproducible you need to keep track of the code, software, and data. We keep track of code in GitHub. Our starting data is static in the sense that it will not change over the course of our analysis and when it does we will perform a new analysis. 

To track software we've developed the 'andersen-lab-env'. The andersen-lab-env is a GitHub repo that helps install and set up a software environment for use locally on a Mac and on Quest. The installed software is not identical but it is very similar between Mac and Linux. More importantly, we rely on three tools to manage the software environment. Each of them provide advantages. In concert they provide a lot of flexibility when it comes to setting up the software environment.

### pyenv

pyenv is used to install and manage different versions of python. For example, you might have a python 3 script for one project and a python 2 script for another. You want to be able to run both scripts on your system. One option is to modify the python 2 script to work with python 3, but this is not always an option.

The solution is to be able to install __multiple__ versions of python simultaneously. `pyenv` allows you to do this. More than this, pyenv allows you to set the python version that you want to use at the local or global level.

* __local__ - Sets the python version to a specific directory.
* __global__ - Sets the python version to use everywhere unless a local version is set.

Lets look at an example of this. First, lets install two different versions of python.

```
pyenv install 2.7.11
pyenv install 3.6.0
```

Now you can see the installed versions by typing `pyenv versions`:

```
$ pyenv versions
*  system
   2.7.11
   3.6.0
```

The `*` indicates that that is the current version of python you are using. In the case above it is set to use the `system` python which is preinstalled and is often python 2.

#### Setting the global version

__Example setting the global version of python to 3.6.0__

```
pyenv global 3.6.0
```

#### Setting the local version

__Example setting the local version of python to 2.7.11__

```
mkdir my_python2_project
cd my_python2_project
pyenv local 2.7.11
```




Now if we go into a particular directoy and type `pyenv local 2.7.11`, a `.python-version` file is created that says `2.7.11` and makes it so that directory always uses python 2.7.11.

Now lets see what this looks like:

<image src='/img/pyenv.svg' style='width: 400px;'/>

As is illustrated above, versions are defined hierarchically. When you `cd` to a directory, pyenv searches up through each directory looking for a `.python-version` file to identify which version of python to use. If it reaches the top before finding one it uses the global version.

#### How pyenv sets versions



```
pyenv global 3.6.0
```

Now when we type `Python` we should see something that looks like this:

```
Python 3.6.0 (default, Jan 22 2017, 12:12:27)
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.42.1)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
```

And running `pyenv versions` gives us:

```
$ pyenv versions
   system
   2.7.11
*  3.6.0
```



For example lets set the global version of python:

```
pyenv global 3.6.0
```

Now when we run `python`

For example, if you run:

```
mkdir my_new_project
cd my_new_project
pyenv local 2.7.11
```

`pyenv` will create a `.python-version` file in the directory `my_new_project` which makes it so that when you are in that folder you will be using python version 2.7.11. Nested folders will also use 2.7.11 unless an inner directory has a different local version of python set.



__The advantages of using pyenv__

* You can install and maintain multiple versions of python
* You can set those 

* The ability to track all the software that is installed and is being used.
* The ability to maintain __multiple__ software environments concurrently.
* The ability to set the software environment that should be used at the directory level. This means you can install a different set of tools in one directory vs. another.
* __hierarchically define environments. 

The __andersen-lab-env__ is a precisely defined set set of configurations and software. The environment can be created on Quest by running a script. The __andersen-lab-env__ environment is implemented using two software tools: [conda](https://conda.io/docs) and [pyenv](https://github.com/pyenv/pyenv). The details of how this are implemented are provided in further detail on the repo where the environment is maintained.

## How the environment is structured




#### [andersenlab/andersen-lab-env](https://github.com/AndersenLab/andersen-lab-env)

* You can install additional software, but if you expect others in the lab will use it you should update the default software environment.


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
