# Using conda on Quest

[TOC]

## Why Conda

__Computational Reproducibility__ is the ability to reproduce an analysis exactly. In order for computational research to be reproducible you need to keep track of the code, software, and data. We keep track of code using git and GitHub (for help, see the [Github page](github.md)). Our starting data (usually FASTQs) is almost always static, so we don't need to track changes to the data. We perform a new analysis when data is added.

To track software, package and environment managers such as Conda and Docker are very useful. Conda works similarly to [brew](https://brew.sh/) or pyenv[pyenv](https://github.com/pyenv/pyenv) that were used in the legacy Andersen-Lab-Env. 

!!! Note
	The software environments on Mac and Linux are not exactly identical...but they are very close.


## Setting up Conda on Quest

Anaconda is a Python distribution that contains many packages including conda. Miniconda is a more compact version of Anaconda that also includes conda. So in order to install conda, we usually either install Miniconda or Anaconda. On Quest, Anaconda is already installed. However there are many versions of Anaconda, each can have a different version of Python and Conda. The current lab environments mainly used `module load python/anaconda3.6` (See **Notes** below for more info).

In your home directory `~/`, create a file called `.condarc` and put the following lines into it. It sets the channel priority when conda searches for packages. If possible, in one environment it is good to use packages from the same channel. 

```
channels:
  - conda-forge
  - bioconda
  - defaults
  
auto_activate_base: false

envs_dirs:
  - ~/.conda/envs/
  - /projects/b1059/software/conda_envs/
```

## Using Conda

__[Conda Documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/index.html)__

__[Conda Cheatsheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)__

After loading the Anaconda module, one can create an environment and install packages into that environment:

```
# Create conda environment named "name_of_env"
conda create --name name_of_env

# Create conda environment in a specific folder
conda create -p /projects/b1059/software/conda_envs/test

# activate conda environment
# for some `conda activate` works and for others `source activate`
conda activate test
source activate test

# install package into conda environment
conda install bcftools
```

When looking to install a package, one resource to check out is [anaconda.org](https://anaconda.org), search for your package and run the install command listed on the page.

!!! Note
	Remember to keep in mind the different versions of software/packages. You could have bcftools-v1.10 or bcftools-v.1.12, so make sure you install the correct one!

!!! Important
	You can also install R packages with conda, however conda and R don't always work well together. Check out our quick fix [here](r.md)

## Running Nextflow with conda

When running Nextflow, conda environments can be specified as part of a process or in the `nextflow.config` file to apply to the entire pipeline (check out the [documentation](https://www.nextflow.io/docs/latest/conda.html)):

**Conda within a process:**

```
process foo {
  conda '/projects/b1059/software/conda_envs/cegwas2-nf_env'

  '''
  your_command --here
  '''
}
```

**Conda for the entire pipeline:**

```
// in the nextflow.config file:
conda { 
    conda.enabled = true 
    conda.cacheDir = ".env"  
}

process {
    conda = "/projects/b1059/software/conda_envs/cegwas2-nf_env"
}
```

## Notes on conda versions on Quest

__tl;dr;__ If having trouble with conda, or Nextflow gives conda-related errors, try to load a different version of anaconda on Quest. At some point it may be worth re-creating all conda environments in the lab with a consistent version of conda. 

As of the end of 2020, existing conda environments for the lab were mostly created by `module load python/anaconda` (which got automatically loaded with `module git` by accident). It loads Python version 2.7.18 and conda 4.5.2. The other environments were created with `module load python/anaconda3.6` which loads Python 3.6.0 and conda 4.3.30. To see versions, use `conda info` or `conda -V`.

Once you activate an environment with `conda activate env_name` or `source activate env_name`, the default conda usually get re-directed to the conda that were originally used to create the environment. This is good because it helps ensure that all packages in the same environment uses the same version of conda. One can go to `cd ~/.conda/env_name/bin` and `readlink -f conda` or `readlink -f activate` to see which version of conda is used by this environment. This is exactly how Nextflow determines which conda to use when using an existing conda environment. 

Quest people recommended `module load python-anaconda3`, but that version does not mix well with the versions mentioned above. But if one were to re-install all environments we have, this is probably the version to stick to. 

