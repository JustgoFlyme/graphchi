Unfortunately I have not written a program to initialize vertex values. It is quite easy though:

  * The vertex data is stored in one flat file. This file simply stores a binary array of the vertex values. So vertex j is stored as vertex-arrayj?.
  * The filename can be retrieved programmatically:  ` filename_vertex_data<VertexDataType>(graphname) `

So basically following program should do the trick (assuming your vertex-datatype is int):

```
     std::string filename = filename_vertex_data<int>("mygraph") 
     FILE*  f = fopen(filename.c_str(), "w");
     for(int i=0; i<numvertices; i++) {
        int value = vertex_value_of(i); // TODO 
        fwrite(&value, sizeof(int), 1, f);
     } 
     fclose(f);
```

This is not the fastest way to do it, but probably fast enough.

## Java ##

In the Java-version, you can define vertex values by defining a self-edge in the edge list input format.