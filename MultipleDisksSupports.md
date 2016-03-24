**NOTE: AS OF VERSION 0.2, THIS FEATURE IS CURRENTLY NOT WORKING**

# Introduction #

GraphChi supports loading the data from multiple disks in parallel, using striping.

**Note:** the multiple disks support is not currently well tested.

# Configuration #


**1.** Preprocess your graph data into shards by running bin/sharder\_basic or running your program once (you can kill it after the preprocessing has ended).

**2.** Create a directory with D symbolic links that point to the directories on different hard-disks. The symbolic links should be named as follows:
```
  path-to-directory/
   graphdisk1  ---> directory on disk 1
   graphdisk2  ---> directory on disk 2
   ..
   ..
   graphdiskD
```

You can rename "graphdisk" as you wish.


**3.** Copy the preprocessed files (graph-name.**) to each of the directories above.**

**4.** Run GraphChi with following configuration

```
   multiplex D multiplex_root path-to-directory
```

Above, D is the number of disks and _path-to-directory_ path to the directory containing the symbolic links.