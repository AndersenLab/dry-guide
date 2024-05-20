__1. Download and Install Fuse for Mac OS__

[https://osxfuse.github.io/](https://osxfuse.github.io/)

__2. Install sshfs__

You can use the link on https://osxfuse.github.io/ or use:

```
brew install sshfs
```

__3. Create a folder in your documents called `b1059`__

```
mkdir ~/b1059
```

__4. Mount our labs quest project folder (`b1059`) to the `b1059` folder you created locally__

```
sshfs <NETID>@quest.it.northwestern.edu:/projects/b1059/ ~/Documents/b1059 -ovolname=b1059
```

__To mount alignments of isotypes at this location:__

```
sshfs <NETID>@quest.it.northwestern.edu:/projects/b1059/data/alignments/WI/isotype ~/Documents/b1059 -ovolname=b1059
```