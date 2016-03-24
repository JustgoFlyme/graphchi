# Announcement: GraphChi v0.2 (C++) #

(January 21, 2013)

GraphChi version 0.2 is the first major update to the GraphChi software for disk-based computation on massive graphs.  This upgrade brings two major changes: compressed data storage (shards) and support for dynamically sized edges.

We also thank for your interest on GraphChi so far: since the release in July 9th 2012, there has been over 8,000 unique visitors to the Google Code project page, at least 2,000 downloads of the source package, several blog posts and hundreds of tweets. GraphChi is a research project, and your feedback has helped us tremendously in our work.

## Compressed Shards ##

GraphChi processes graphs efficiently by dividing the graph into a number of **shards**, using a specific partitioning model (see IntroductionToGraphChi).  Previously, each shard was stored as one big file. This works well if the size of the data remains constant, but if we want to introduce compression, we must handle the data in smaller chunks. For example, if the data in the beginning of the shard shrinks or expands, we would have needed to recreate the whole file on the disk, with huge performance penalty.

This update brings a major overhaul to the shards. Instead of storing them as big files, each shard is split into 4-megabyte _blocks_. Each block is now loaded and written separately, so we can compress and uncompress them on the fly. For compression, we use inflate/deflate algorithm from the [zlib](http://www.zlib.net/) library.  The level of compression depends on your algorithm: if you store floating point numbers, the level of compression is likely quite small, but for algorithms like label propagation (with integer labels), you might experience very good compression.

Compression should improve performance on hard drives, but on SSD the computation time needed for compression/decompression roughly matches the time saved in I/O.  (Please let us know if you experience bad performance in some cases - we might want to add a switch to disable compression in such cases).

## Dynamic edge values ##

So far, GraphChi has restricted the values stored in _edges_ to be of fixed size. This release includes support for _dynamically sized arrays_ (called "chivectors") in edges. This will allow implementation of such algorithms where the number of data points in edges varies over the course of the computation. For example, particle-based random walks.

The current implementation has some limitations, so let us know (on the email list: http://groups.google.com/group/graphchi-discuss) if these constraints prevent you from implementing your applications.

To learn about this feature, please consult this tutorial: DynamicEdgeData.

## Dynamic vertex values ##

(Updated Jan 22, 2013): DynamicVertexData explains how to use dynamic vectors for the vertex values.


## How to obtain the new version ##

We recommend checking out the latest version from the Mercurial source control:

```
hg clone https://code.google.com/p/graphchi/ 
```

# Other announcements #

## Toolkits ##

Danny Bickson has been working hard to extend improve the _collaborative filtering toolkit_ for GraphChi. It contains now a wide variety of state-of-the-art algorithms. Read more from Danny's blog: http://bickson.blogspot.com/2012/08/collaborative-filtering-with-graphchi.html

## GraphChi-Java ##

The Java-version of GraphChi has also been upgraded to version 0.2, and now is self-contained (including its own preprocessing/sharding) and the code has been throughly documented: http://code.google.com/p/graphchi-java/


Yours,
Aapo Kyrola (akyrola -- cs.cmu.edu)
Follow me at Twitter: https://twitter.com/kyrpov