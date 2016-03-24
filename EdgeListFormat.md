# Edge List - dataformat #

Edge list is a text file with each edge on its own line:

```
  src1 dst1 value1
  src2 dst2 value2
```

Value can be omitted if each edge has default value. Vertex ids are numerical.

Lines start with # or % are treated as comments.