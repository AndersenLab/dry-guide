## Andersen Lab Coding Best Practices

These best practices are just a few of the important coding tips and tricks for reproducible research. If you have more ideas, contact Mike!

### General

* You should be doing most (if not all) of your analyses in `/Volumes/vast/projects/YourName` (this requires having mapped VAST to your computer. See [here](../rockfish/VAST.md) for instructions)
	* This is (1) to make sure the data is backed up/saved with version history and (2) to allow other lab members to access your code/scripts when necessary
* Do **NOT** use spaces, quotation marks, or brackets in names of files or folders. Try not to use these characters in column names either (although sometimes it is necessary for a final table output)
	* Computers often have a hard time reading spaces and code used to ignore spaces can vary from program to program
	* Instead, you can use `_` or `.` or `-` or capitalization (`fileName.txt`)
* **NEVER** replace raw data!!!!!
	* You should save your raw data in the rawest format, write a script to analyze it then if you wish, save the processed data for further use. This is important because it always allows you to go back to the original raw data in case something happens
	* Some suggested project folder structures might look like something below:
![directory_structure](../img/directory_structure.png)
* Include a `README.md` (or `README.txt`) file in each project folder to explain where the data and scripts can be found for certain analyses. Trust me, after a few years you will definitely forget…
	* And don’t forget to update the `README` regularly, an old `README` doesn’t do anyone good!
* Either use full path names in scripts or be explicit about where the working directory is
	* This is important to allow other people to run your code (or might even be helpful for you if you ever reorganize folders one day)
* Date your files, especially when you update an existing file. Write dates in `YYYYMMDD` format
* As much as possible, ensure that your processed data is “tidy” (see below). This doesn’t work for all complex data types, but it should be a general norm to follow.
	* Each variable must have its own column
	* Each observation must have its own row
	* Each value must have its own cell
	* No color or highlighting 
	* No empty cells (fill with `NA` if necessary)
	* Save data as plain text files (`.csv`, `.tsv`, `.txt` etc. -- NOT `.xls`!!!)

### R

* **ALWAYS** use namespaces before functions from packages (i.e. `dplyr::filter()` instead of `filter()`)
	* This includes `ggplot2` and especially `dplyr`!!!
	* Some packages have functions with the same name, so adding a namespace is crucial for reproducibility.
	* Also, this helps other people read your code by knowing which functions came from which packages
* When piping with Tidyverse (`%>%`), press `<Enter>` to go to the next line after a pipe
	* This makes your code more readable 
	* In fact, general practices state no more than 80-100 characters per line of code EVER to increase readability

### Rockfish

* You should be doing most (if not all) of your analyses in `/vast/eande106/projects/yourName` (not your home directory (i.e. `/home/<jheid>`))

!!! Important
	Main exception: Nextflow temporary working directories should NOT be on `/vast/eande106` but rather in the scratch space `/scratch4/eande106/` (files get automatically deleted here periodically).
	A correctly setup [bash profile](../rockfish/rf-nextflow.md) file will take care of this.

### QUEST

* You may still be doing some of your analyses in `/projects/b1059/projects/yourName` (not your home directory (i.e. `/home/netid`))

!!! Important
	Main exception: Nextflow temporary working directories should NOT be on `b1059` (it will fill us up!) but rather in the scratch space `b1042` (files get automatically deleted here every 30 days).
	A correctly designed [nextflow.config](../quest/quest-nextflow.md) file will take care of this.

### Python

* **ALWAYS** indent using spaces. Mixing spaces and tabs will case your script to fail
* Use clear and descriptive variable names
	* The names should help someone reading the code track the logic of the coder
	* Only use alphanumeric characters or underscores
	* Never start a name with a number
* Comment as much as you can to help others (and future you) understand what each part of the code is for
* Make logical blocks of code into functions
	* This helps with interpretation
	* Anything that is done more than once should be a function
* Try to follow [PEP8](https://peps.python.org/pep-0008/) style rules

