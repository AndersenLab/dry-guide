# cegwas

### Requirements

The cegwas-nf pipeline requires `awesome-slugify`; Install using:

```
pip install awesome-slugify
```

It also requires that cegwas be installed. See the [cegwas](https://github.com/AndersenLab/cegwas) repo for more information.

### Usage

```
# cd to directory containing a trait file.
nextflow run Andersenlab/cegwas-nf --in=<input file>
```

### Input Format

Save trait data as a `.tsv`; Strains in column 1. Traits are in columns 2 and above.

| strain    |   trait1 |  trait2 |   trait3 |   trait4 |
|:----------|--------------------------:|--------------------------:|-------------------------:|--------------------------:|
| AB1       |               0           |                    0      |             19.6825      |              16.5026      |
| AB4       |              14.5294      |                   13.9775 |             18.9721      |              20.6803      |
| BRC20067  |              18.0132      |                   17.1509 |             18.4466      |              21.0243      |
| CB4855_UK |               0           |                   13.2552 |             19.5265      |              21.7389      |
| CB4856    |              14.4711      |                   12.2563 |             19.3584      |              21.2358      |
| CB4932    |               0           |                    0      |             19.8662      |              20.326       |
| CX11254   |              17.5516      |                   16.9135 |             19.5696      |              21.7276      |
| CX11264   |              15.3574      |                   13.8575 |             18.9888      |              21.3832      |
