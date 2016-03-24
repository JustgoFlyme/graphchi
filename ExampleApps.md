# Example Applications: Introduction #

Best way to get familiar with GraphChi is to run and study the example applications. You should start with the simple applications, Pagerank and Connected Componenents prior to looking at matrix factorization or triangle counting.

## Input data ##

To run the examples, you need an input graph (network). Currently GraphChi can read graphs in the standard EdgeListFormat or AdjacencyListFormat, but it is fairly easy to write your own parsers. Remember that GraphChi can run even very big graphs on a normal PC or laptop!

A good set of graphs of different size are available below:

  * http://snap.stanford.edu/data/

Great collection of graphs is available at ( http://law.di.unimi.it/datasets.php ), but notice that they are in the compressed format of WebGraph Framework ( http://webgraph.dsi.unimi.it/ ). A simple, not too efficient, tool for converting these graphs to AdjacencyListFormat is available here: (TODO).  **Contribution request:** proper support for the WebGraph Framework's data.


## Preprocessing of graphs ##

All example programs automatically convert the input graph to GraphChi's internal **shards** (see IntroductionToGraphChi). That is, you just need to compile and run the application. This conversion needs to be done only once for each application, and the system will detect whether preprocessing is needed. Also, if the value type stored in edges is of same size, different applications can use same shards.







# Example Applications #

## Pagerank (Easy) ##

What you will learn:
  * Basic structure of GraphChi programs
  * How to "communicate" via edges
  * Flavor of asynchronous computation

**Source**: [example\_apps/pagerank.cpp](https://code.google.com/p/graphchi/source/browse/example_apps/pagerank.cpp)

To build and run:
```
   make example_apps/pagerank
   bin/example_apps/pagerank file GRAPH-NAME
```

Above, replace GRAPH with the input graph. If the graph has not been preprocessed, the program will ask for the format of the graph (edgelist or adjlist).

You may adjust the number of iterations by specifying parameter "niters" as follows:


```
   bin/example_apps/pagerank file GRAPH-NAME niters 10
```

The application will print the ids of the top 20 vertices with highest pagerank.

### Analyzing the output ###

GraphChi will write the values of the edges in a binary file, which is easy to handle in other programs. Name of the file containing vertex values is GRAPH-NAME.4B.vout.  Here "4B" refers to the vertex-value being a 4-byte type (float).

You can also analyze the vertex-data programatically, in memory efficient manner, by using VertexAggregators.


### Remark ###

While this Pagerank example is simple, it is not the fastest way to compute Pagerank with GraphChi. Particularly, it unnecessary loads the values for out-edges, while the value is never read, it is just written. There is another version of Pagerank available, [| example\_apps/pagerank\_functional.cpp](https://code.google.com/p/graphchi/source/browse/example_apps/pagerank_functional.cpp), which uses an experimental alternative "functional" API, which is roughly 50% more efficient for Pagerank.

## Connected Components (Easy) ##

What you will learn:
  * Dealing with undirected edges
  * Understanding asynchronous computation
  * Selective (dynamic) scheduling

**Source**: [example\_apps/connectedcomponents.cpp](https://code.google.com/p/graphchi/source/browse/example_apps/connectedcomponents.cpp)

To build and run:
```
   make example_apps/connectedcomponents
   bin/example_apps/connectedcomponents file GRAPH-NAME
```


### Output ###

The program will produce output GRAPHNAME\_components.txt, which on each line has
```
  component id,num-of-vertices
```

As an interesting analysis, you can plot a histogram of the sizes of the connected components.



## Community Detection (Easy+) ##

What you will learn:
  * How to exchange information via edges in both directions

**Source**: [example\_apps/communitydetection.cpp](https://code.google.com/p/graphchi/source/browse/example_apps/communitydetection.cpp)

To build and run:
```
   make example_apps/communitydetection
   bin/example_apps/communitydetection file GRAPH-NAME
```


### Output ###

Same output as connected components, but instead of components, it lists community-ids and their sizes.


## Matrix factorization: Alternative Least Squares - ALS (Moderate) ##

What you will learn:
  * How to integrate linear algebra package (Eigen)
  * How to deal with a bipartite graph
  * Special graph input / output
  * Two strategies of reading neighbor vertex value (via edges, and in-memory)

There are two versions of the program. First version, edge-version, writes a vertex's value (a latent factor vector) to its adjacent edges; vertices-in-memory version uses a separate array to store vertices in-memory. See the code for more details.

### Dependency: Eigen ###

You need the Eigen matrix library for compiling the matrix factorization programs. It is available, as a headers-only installation, from http://eigen.tuxfamily.org/index.php?title=Main_Page

### Edge-version ###
**Source**: [example\_apps/als\_edgefactors.cpp](http://code.google.com/p/graphchi/source/browse/example_apps/matrix_factorization/als_edgefactors.cpp)

To build and run:
```
   make example_apps/matrix_factorization/als_edgefactors
   bin/example_apps/matrix_factorization/als_edgefactors file MATRIX-NAME
```

### Vertices in-memory version ###

**Source**: [example\_apps/als\_vertices\_inmem.cpp](http://code.google.com/p/graphchi/source/browse/example_apps/matrix_factorization/als_vertices_inmem.cpp)

```
   make example_apps/matrix_factorization/als_vertices_inmem
   bin/example_apps/matrix_factorization/als_vertices_inmem file MATRIX-NAME
```

### Input ###

For this program, you need to provide your input in the Matrix Market format: http://math.nist.gov/MatrixMarket/

For example, if you are running ALS for the Netflix movie ratings dataset, the matrix is a sparse matrix containing the ratings that users have given each movie.

### Output ###

The matrix factors are output in the Matrix Market format with names
```
  GRAPH-NAME_U.mm
  GRAPH-NAME_V.mm
```



## Triangle Counting (Advanced) ##

What you will learn:
  * A special design pattern, which requires working outside the basic abstraction
  * Writing a special preprocessor that orders the vertices by their degree

**Source**: http://code.google.com/p/graphchi/source/browse/example_apps/trianglecounting.cpp


## Streaming Graph + Pagerank ##

This application starts with a base-graph, and streams additional edges to the graph from a streaming graph file (which must be in the EdgeListFormat), while simultaneously computing pagerank.  The source code contains also example on how to use a HTTP-based admin dashboard for GraphChi applications. More information is added later, and be warned that the code in the example application is not finished and contains a lot of demonstration code.

**Source:** http://code.google.com/p/graphchi/source/browse/example_apps/streaming_pagerank.cpp