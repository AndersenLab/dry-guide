# [Andersen Lab Dry Guide](http://andersenlab.org/dry-guide/)

The Andersen lab [dry-lab computing guide](http://andersenlab.org/dry-guide/). The guide is built with [mkdocs](http://www.mkdocs.org/).

## Editing the site

1. Clone the repo

```
git clone https://github.com/AndersenLab/dry-guide.git
```

2. Install `mkdocs>=1.5` and `mike>=2.0`

```
# first, you need python3 - make sure you have it, otherwise download

# if you don't have pip...
curl https://bootstrap.pypa.io/get-pip.py > get-pip.py
python3 get-pip.py

# download mkdocs
pip install "mkdocs>=1.5"

# download mike
pip install -Iv "mike>=2.0"

```

3. When edits are complete, use `mike deploy`.

If you are making minor changes, use the same version:

```bash
mike deploy [current date version] latest --update-aliases --ignore-remote-status --push
```

The word `latest` here is an alias which will ensure that the new version is served.


If you have made substantial changes to the site, like adding new sections or removing or rewriting sections, create a new version with the current date.

```bash
mike deploy [today's date] latest --update-aliases --ignore-remote-status --push
```