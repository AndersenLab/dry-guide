# Adding NIL sequence data to lab site

[TOC]

This section is for adding genomic sequencing data (.tsv files) onto the existing dataset provided and displayed on the [NILs browser](http://andersenlab.org/NILs/) page on [Andersenlab.org](http://andersenlab.org/). 

## Basic Commands To Know Prior

You should freshen up on the following terminal commands. 

* __cd__ - change directories
* __rm__ - delete files
* __cp__ - make a copy
* __git__ 

## Your Sequencing Data 

You will use the `gt_hmm_fill.tsv` file output from the `nil-ril-nf` pipeline for this step.

!!! Important   
    In order for your sequencing data to be properly added, it is important to make sure that there are no empty/additional lines located at the bottom of your .tsv file. 

__What you do not want__
![What you do not want](img/WhatNotToDo.png)

__What you do want__
![What you do want](img/WhatYouDoWant.png)

Once your file has no empty lines at the bottom, save the file and move onto the next instructions below. 

## Step By Step Instructions 
1. Clone the `andersenlab.org` repo

```
git clone https://github.com/AndersenLab/andersenlab.github.io.git
```

2. [Optional] Create a new branch. This is a good idea if you are newer to github and want to make sure you don't break the lab website by doing something weird.

3. Add your .tsv file into the `pages` folder of your Andersenlab github directory. 

4. Open terminal and use the `cd`command to change directories into the `pages` folder in your Andersenlab github directory. 
If you did everything correctly, when you type `ls` into your terminal, it should look something like this. 

![What your terminal should look like](img/PagesDirectory.png)

5. Then run the following commands in your terminal (while still in your `pages` directory): 
```
# create a new file called 'copy.tsv' with your data
cp yourFileName.tsv copy.tsv

# run python script which will add your data to full dataset
python addDataTogt_hmm.tsv.py
```

6. After running the above commands, your sequencing data has now been added to the existing NILs dataset on Andersenlab.org. You can now remove your .tsv from the `pages` directory by using the `rm` command in your terminal. 

```
rm yourFileName.tsv
```

7. Finally, commit your changes and push your code to update the Andersenlab github. 

8. [Optional] If you created a new branch, you need to merge this new branch back into the master branch for changes to take effect.

## Viewing your results

Once all changes are pushed, you should be able to view your NIL genotypes in the [NIL browser shiny app](https://andersen-lab.shinyapps.io/nil-browser/).