# Andersenlab.org

The Andersen Lab website was built using [jekyll](https://jekyllrb.com/) and runs using the [Github Pages](https://pages.github.com/) service.

## Announcements

## Lab members

## Papers

## Protocols

## Research

## Photo Albums

Photo albums can be added easily to the Andersen Labsite. Adding albums requires two utilities to be installed on your computer:

* (a) Image Magick
* (b) exiftool

These two can be installed using the following

```
brew install imagemagick
brew install exiftool
```

#### (1) Place images in a folder and name it according to the following schema:

`YYYY-MM-DD-title`

For example, `2017-08-05-Hawaii Trip`.

#### (2) Run the `build.sh` script in the root of the `andersenlab.github.io` repo.

The `build.sh` script will do the following:

* (a) Construct pages for the album being published.
* (b) Decrease the size of the images in the album (max width=1200). 

You can run the script using:

```
bash build.sh
```

#### (3) Add the images using git and push to GitHub

You can easily add all images using:

```
git add *.jpg
```

