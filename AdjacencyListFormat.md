Adjacency lists are more efficient format than EdgeListFormat for storing graphs, but do not allow storing the edge value.

Each vertex is stored on its own line. Line starts with vertex-id, followed by the number of edges and then all the neighbors as a list.

Example
```
   src1 4 dst1 dst2 dst3 dst4
   src2 2 dst5 dst6
```

File can contain comments starting with # or %.