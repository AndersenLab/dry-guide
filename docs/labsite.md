# Andersenlab.org

[TOC]

## Getting Started

The Andersen Lab website was built using [jekyll](https://jekyllrb.com/) and runs using the [Github Pages](https://pages.github.com/) service.

#### Software-Dependencies

Several software packages are required for editing/maintaining the Andersen Lab site. They can be installed using [Homebrew](brew.sh):

```
brew install ruby imagemagick exiftool python
gem install jekyll # you may need to sudo install this.
# If you get an error when trying to run pip, try:
# brew link --overwrite python
pip install metapub pyyaml
```

#### Cloning the repo

To get started editing, clone the repo:

```
git clone https://github.com/andersenlab/andersenlab.github.io
```

This repo contains documents that get compiled into the Andersen Lab website. When you make changes to this repo and push them to GitHub, Github will trigger a 'build' of the labsite and update it. This usually takes less than a minute. Instructions for changing various aspects of the site are listed below.

You can also use [Github Desktop](https://desktop.github.com) to manage changes to the site. 

If you want to edit the site locally and preview changes, run the following in the git repo directory:

```
jekyll serve
```

The site should become available at `localhost:4000` and any changes you make will be reflected at that local url.

## andersenlab.github.io

The structure of the Andersen Lab repo looks like this:

```
CNAME
LICENSE
README.md
build.sh
index.html
_config.yml
_data/
_includes/
_layouts/
_posts/
_site/
assets/
feeds/
files/
pages/
people/
publications/
scripts/
Protocols/
```

## Announcements

## Lab members

### Adding new lab members:

* __(1)__ - Add a photo of the individual to the `people/` folder.
* __(2)__ - Edit the `_data/people.yaml` file, and add the information about that individual.

Each individual should have - at a minimum, the following:

```
- first_name: <first name>
  last_name: <last name>
  title: <One of: Graduate Student; Research Associate; Undergrad; Postdoctoral Researcher>
  photo: <filename of the photo located in the people/ directory>
```

Additional fields can also be added:

```
- first_name: <first name>
  last_name: <last name>
  title: <One of: Graduate Student; Research Associate; Undergrad; Postdoctoral Researcher>
  pub_names: ["<an array>", "<of possible>", "<publication>", "<names>"]
  photo: <base filename of the photo located in the people/ directory; e.g. 'dan.jpg'>
  website: <website>
  description: <a description of research>
  email: <email>
  github: <github username>
```

!!! Note
    `pub_names` is a list of possible ways an individual is referenced in the author list of a publication. This creates links from the publications page back to lab members on the people page.

### Set Status to Former

Lab members can be moved to the bottom of the people page under the 'former member' area. To do this add a `former: true` line for that individual and a `current_status:` line indicating what they are up to.

For example:

```
- first_name: Mostafa
  pub_names: 
    - Zamanian M
  last_name: Zamanian
  description: My research broadly spans "neglected disease" genomics and drug discovery. I am currently working to uncover new genetic determinants of anthelmintic resistance and to develop genome editing technology for human pathogenic helminths.
  title: Postdoctoral Researcher, 2015-2017
  photo: Mostafa2014.jpg
  former: true
  github: mzamanian
  email: zamanian@northwestern.edu
  current_status: Assistant Professor at UW Madison -- <a href='http://www.zamanianlab.org/'>Zamanian Lab Website</a>
```

### Remove lab members

Remove the persons information from `_data/people.yaml`; Optionally delete their photo.


## Protocols

## Research

## Publications

Elements used to construct the publications page of the website are stored in two places:

1. `_data/pubs_data.yaml` - The publications data stores authors, pub date, journal, etc.
2. `publications/` - The publications folder for PDFs, thumbnails, and supplementary files.

#### (1) Download a PDF of the publication

You will want to remove any additional PDF pages (e.g. cover sheets) if there are any present in the PDF. See [this guide](https://support.apple.com/kb/PH20221?locale=en_US&viewlocale=en_US) for information on removing pages from a PDF.

Save the PDF to `/publications/[year][tag]`

Where `tag` is a unique identifier for the publication. In general, these have been the first author or the journal or a combination of both.

#### (Optional) PMID Known

If the PubMed Identifier (PMID) is known for the publication, you can add it to the file `publications/publications_list.txt`.

#### (2) Run `build.sh`

The `build.sh` script does a variety of tasks for the website. For publications - it will generate thumbnails. It will also fetch information for publications and add it to the `_data/pubs_data.yaml` file __if__ a PMID has been provided. If you did not add a PMID, you will have to manually add authors, journal, etc. to the `_data/pubs_data.yaml` file.

#### (3) Edit `_data/pubs_data.yaml`

The publication should now be added either manually or automatically to `_data/pubs_data.yaml` and should look something like this:

```
- Authors: [Laricchia KM, Zdraljevic S, Cook DE, Andersen EC]
  Citations: 0
  DOI: 10.1093/molbev/msx155
  Date_Published: 2017 May 09
  Journal: Molecular Biology and Evolution
  PMID: 28486636
  Title: Natural variation in the distribution and abundance of transposable elements
    across the Caenorhabditis elegans species
```

The first thing you will want to do is associate the publication with the PDF you added. Add a `PDF` line for that:

```
- Authors: [Laricchia KM, Zdraljevic S, Cook DE, Andersen EC]
  Citations: 0
  DOI: 10.1093/molbev/msx155
  Date_Published: 2017 May 09
  Journal: Molecular Biology and Evolution
  PMID: 28486636
  Title: Natural variation in the distribution and abundance of transposable elements
    across the <em>Caenorhabditis elegans</em> species
  PDF: 2017Laricchia
```

You can also italicize text by adding `<em>` tags around words as shown above (`<em>Caenorhabditis elegans</em>`).

!!! IMPORTANT

    Before pushing any changes to GitHub, you will want to preview the changes locally using `jekyll serve`.

#### (4) Add supplementary data

Supplemental data and figures are stored in `publications/[pdf_name]`. For example, 2017Laricchia has an associated folder in `publications/` where supplemental data and figures are stored.

## Photo Albums

Photo albums can be added to the Andersen Labsite. Adding albums requires two utilities to be installed on your computer:

* (a) Image Magick
* (b) exiftool

These can easily be installed with [homebrew](https://brew.sh/). should have been installed during Setup (above), but if not you can install them using the following:

```
brew install imagemagick
brew install exiftool
```

#### (1) Place images in a folder and name it according to the following schema:

`YYYY-MM-DD-title`

For example, `2017-08-05-Hawaii Trip`.

#### (2) Move that folder to `/people/albums/`

Now copy the renamed folder to the `/people/albums/` directory.

#### (3) Run the `build.sh` script in the root of the `andersenlab.github.io` repo.

The `build.sh` script will do the following:

* (a) Construct pages for the album being published.
* (b) Decrease the size of the images in the album (max width=1200). 

!!! Note

    The `build.sh` script also performs other maintenance-related tasks. It is fine to run this script at anytime.

You can run the script using:

```
bash build.sh
```

#### (4) Add the images using git and push to GitHub

You can easily add all images using:

```
git add *.jpg
```

#### (5) Push changes to github

```
git push
```

