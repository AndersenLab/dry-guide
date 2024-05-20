# Using conda on Quest

[TOC]

## Why Conda

__Computational Reproducibility__ is the ability to reproduce an analysis exactly. In order for computational research to be reproducible you need to keep track of the code, software, and data. We keep track of code using git and GitHub (for help, see the [Github page](github.md)). Our starting data (usually FASTQs) is almost always static, so we don't need to track changes to the data. We perform a new analysis when data is added.

To track software, package and environment managers such as Conda and Docker are very useful. Conda works similarly to [brew](https://brew.sh/) or pyenv[pyenv](https://github.com/pyenv/pyenv) that were used in the legacy Andersen-Lab-Env. 

!!! Note
	The software environments on Mac and Linux are not exactly identical...but they are very close.


## Setting up Conda on Rockfish

Anaconda is a Python distribution that contains many packages including conda. Miniconda is a more compact version of Anaconda that also includes conda. So in order to install conda, we usually either install Miniconda or Anaconda. On Quest, Anaconda is already installed. However there are many versions of Anaconda, each can have a different version of Python and Conda. The current lab environments mainly used `module load anaconda3/2022.05` (See **Notes** below for more info).

In your home directory `~/`, create a file called `.condarc` and put the following lines into it. It sets the channel priority when conda searches for packages. If possible, in one environment it is good to use packages from the same channel. 

```
channels:
  - conda-forge
  - bioconda
  - defaults
  
auto_activate_base: false

envs_dirs:
  - ~/.conda/envs/
  - /home/<jheid>/data_eande106/software/conda_envs/
```

## Using Conda

__[Conda Documentation](https://docs.conda.io/projects/conda/en/latest/user-guide/index.html)__

__[Conda Cheatsheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)__

After loading the Anaconda module, one can create an environment and install packages into that environment:

```
# Create conda environment named "name_of_env"
conda create --name name_of_env

# Create conda environment in a specific folder
conda create -p /home/<jheid>/data_eande106/software/conda_envs/test

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
  conda '/home/<jheid>/data_eande106/software/conda_envs/cegwas2-nf_env'

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
    conda = "/home/<jheid>/data_eande106/software/conda_envs/cegwas2-nf_env"
}
```

## Notes on conda versions on Quest vs Rockfish

As of the end of 2020, existing conda environments for the lab were mostly created by `module load python/anaconda` from our previous file system, QUEST (which got automatically loaded with `module git` by accident). It loads Python version 2.7.18 and conda 4.5.2. The other environments were created with `module load python/anaconda3.6` (also from QUEST) which loads Python 3.6.0 and conda 4.3.30. To see versions, use `conda info` or `conda -V`.

As of 2023, conda environments will be generated with `module load anaconda3/2022.05` from Rockfish, which loads Python version 3.9.12 and conda 4.12.0. From our limited testing so far, all environments generated in QUEST (now under `~/data_eande106/software/conda_envs/`) seem to be compatible with the version active in Rockfish

Once you activate an environment with `conda activate env_name` or `source activate env_name`, the default conda usually get re-directed to the conda that were originally used to create the environment. This is good because it helps ensure that all packages in the same environment uses the same version of conda. One can go to `cd ~/.conda/env_name/bin` and `readlink -f conda` or `readlink -f activate` to see which version of conda is used by this environment. This is exactly how Nextflow determines which conda to use when using an existing conda environment. 

## A faster alternative to Conda

In recent years, a 'drop-in' replacement for Conda, called Mamba, was released. Mamba is faster at resolving dependencies, which improves wait times for creating and activating environments. Conda and Mamba seem to be forward and backwards compatible, and they share the same command structure. For example, `conda create` and `mamba create` do the same task and share the same parameters.

If you prefer to use Mamba, you can install it in your home directory by running these commands:

```
cd ~
wget https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh
chmod +x ./Mambaforge-Linux-x86_64.sh
./Mambaforge-Linux-x86_64.sh
```

After completing the installation, the `mamba` binary should be added to `$PATH`, and you should be able to create, configure, and activate environments using the same commands as `conda` (FYI the `conda` binary will still be available to be called from `$PATH` by default). At the time this is being written, this mamba installation will install Python version 3.10.13, with conda version 23.11.0 and mamba version 1.5.5.

You will immediately notice a difference in speed when using mamba. Test it by loading the `nf20` environment:
```
mamba activate /home/<jheid>/data_eande106/software/conda_envs/nf20_env/
```
