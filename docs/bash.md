# Bash

[Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) is the default unix shell on Mac OS and most Linux operating systems. Many bioinformatic programs are run using the command line, so becoming familiar with Bash is important.

Start with the fo [Introduction to bash](http://cs.lmu.edu/~ray/notes/bash/). 

## Basic Commands

You should familiarize yourself with the following commands.

* __alias__ - create a shortcut for a command
* __awk__ - file manipulation; Filtering; Rearranging columns
* __cat__ - concatenate files
* __zcat__ - concatenate zipped files
* __cd__ - change directories
* __curl__ - download files
* __echo__ - print strings
* __export__ - Add a variable to the global environment so that they get passed on to child processes.
* __grep__ - filter by pattern
* __egrep__ - filter by regex
* __rm__ - delete files
* __sed__ - quick find/replace
* __sudo__ - run as an administrator
* __sort__ - sorts files
* __source__ - runs a file
* __ssh__ - connect to servers
* __which__ - locate files on your PATH
* __uniq__ - get unique lines. File must be sorted.

## More Advanced

You should learn these once you have the basics down.

* __git__

# Good Guides

Below I link to some good guides for various bash utilities.

## awk

* [awk guide](https://linuxconfig.org/learning-linux-commands-awk)
* [awk by example](https://github.com/learnbyexample/Command-line-text-processing/blob/master/gnu_awk.md) - hundreds of examples

### Rearranging columns

```
cat example.tsv | awk -f OFS="\t" '{ print $2, $3, $1 }'
```

The line above will print the second column, the third column and finally the first column.

### Filtering based on criteria

__Print only lines that start with a comment (#) character__

```
cat example.tsv awk '$0 ~ "^#" { print }'
```

## bcftools

* [bcftools manual](https://samtools.github.io/bcftools/howtos/index.html)

## Screen

Screen can be used to run things in the background. It is extremely useful if you need to run things on quest without worry that they will be terminated if you log out or get kicked off. This is essential when running __nextflow__ because pipelines can sometimes run for many hours and its likely you will be kicked off in that time or lose your connection.

* [Screen basics](https://www.linode.com/docs/networking/ssh/using-gnu-screen-to-manage-persistent-terminal-sessions)

## uniq

Get a count of lines in a file. 
```
cat myfile.tsv | sort | uniq -c
```