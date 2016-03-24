(Note, this wiki page is direct copy-paste from the paper)

## Evolving Graphs and Streaming Graph Updates ##

We now modify the PSW model to support changes in the graph _structure_. Particularly, we allow adding edges to the graph efficiently, by implementing a simplified version of I/O efficient buffer trees .

Because a shard stores edges sorted by the source, we can divide the shard into  P  logical parts: part  j  contains edges with source in the interval  j . We associate an in-memory **edge-buffer(p, j)** for each logical part  j , of shard  p. When an edge is added to the graph, it is first added to the corresponding edge-buffer (Figure below). When an interval of vertices is loaded from disk, the edges in the edge-buffers are added to the in-memory graph.

![http://www.cs.cmu.edu/~akyrola/graphchiwiki/edgebuffers.png](http://www.cs.cmu.edu/~akyrola/graphchiwiki/edgebuffers.png)

After each iteration, if the number of edges stored in edge-buffers exceeds a predefined limit, PSW will write the buffered edges to disk. Each shard, that has more buffered edges than a shard-specific limit, is recreated on disk by merging the buffered edges with the edges stored on the disk. The merge requires one sequential read and write. However, if the merged shard becomes too large to fit in memory, it is _split_ into two shards with approximately equal number of edges. Splitting a shard requires two sequential writes. PSW can also support removal of edges: removed edges are flagged, and ignored when graph is loaded. When a shard is rewritten to disk, the removed edges are removed permanently.

Finally, we need to consider consistency. It would be complicated for programmers to write update-functions that support vertices that can change during the computation. Therefore, if an addition or deletion of an edge would affect a vertex in current execution interval, it is added to the graph only after the execution has finished.