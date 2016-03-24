# Introduction #

**NOTE: THIS IS CURRENTLY OBSOLETE (SINCE VERSION 0.2)**

Although GraphChi has been primarily developed for running with computers with relatively small amount of memory, it can also utilize large amount of memory, if available.


# Preloading #

_Preloading_  allows GraphChi to permanently keep files in memory, instead of reloading them from disk every iteration.

In the ConfigurationFile, you can specify setting such as
```
  preload.max_megabytes = 10000
```

... which instructs GraphChi to use 10 gigabytes of memory for preloading.

In the end of the run, the changes to preloaded files are commited back to disk.

**Note:** Preloading currently does not work with dynamic graphs.

# Other configurations #

  * **io.blocksize** is the size of the chunk used for I/O. Default setting is one megabyte. Especially on rotational hard-drives, it makes sense to configure it larger. Note, that you need enough memory to store at least one block for each shard. That is, if you have a huge number of shards, this might become an issue.
  * **membudget\_mb** defines the maximum about of megabytes to be used for loading a subgraph in memory at any time. Default is 1024 megabytes. For a computer with 8 gigabytes, 3000 megabytes is a reasonable choice. Larger amount helps avoiding redundant scans of the _memory-shard_ (see IntroductionToGraphChi).