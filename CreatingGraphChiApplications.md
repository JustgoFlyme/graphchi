# Introduction #

GraphChi computation is based on the vertex-centric model, popularized by [!GraphLab](http://www.graphlab.org) and [Pregel](http://dl.acm.org/citation.cfm?id=1807184). Under this model, programmer provides a **vertex update-function**, which is invoked for each vertex separately. Update function can read and modify data in the edges, and optionally ask the system to run the update on its neighbors. Typical programs are iterative, and terminate when the values of vertices (or edges) do not change (significantly) anymore after an update.


# Application template #

To get started with coding, copy file example\_apps/application\_template (http://code.google.com/p/graphchi/source/browse/example_apps/application_template.cpp) and rename it to match your program myappname.cpp. Place it in directory "myapps", and then you can compile your program by entering
```
  make myapps/myappname
```

To run your program,
```
  bin/myapps/myappname
```

Before writing your own program, it is good idea to study the ExampleApps.

Read on to understand the execution semantics provided by GraphChi.

## Vertex and Edge Data ##

GraphChi programs (like GraphLab) programs store the program data into the graph. Each edge and vertex can store a value.

To define your types, modify the following typedefs in the application template:

```
  typedef my_vertex_type VertexDataType;
  typedef my_edge_type EdgeDataType;
```

GraphChi requires that both edge and vertex data types are basic C structs, and must be of constant size. That is, you may not use STL vectors or other dynamic data types.

## Preprocessing ##

For GraphChi to run, your input graph needs to be converted to shards (see IntroductionToGraphChi). Moreover, the shards must conform to the datatypes your defined (more precisely, the size of the edge data must be compatible with the shard). Following line in the application template does it automatically.

```
   convert_if_notexists<EdgeDataType>(filename ...)
    
```

# Update schedule and semantics #

GraphChi executes vertex updates in round-robin manner. That is, on each iteration it executes the update functions on vertices in ascending order. Optionally, programs can use **selective scheduling**, in which GraphChi only updates vertices that a _scheduled_. It is typical for an update function to schedule its neighbors if its own value changes significantly.

GraphChi runs updates in parallel. We next discuss how dependencies are handled.

## Dependencies ##

If two vertices, with id A and B, share an edge, we say that A and B depend on each other. GraphChi dependencies under parallel execution are governed by the following rule:

  * If vertices A and B share and edge, and A < B, A is always executed before B.
  * Otherwise, if A and B do not share an edge, the update order is not defined.

That is, GraphChi provides a deterministic execution order guarantee. See the (upcoming) paper for more discussion.

Note: deterministic execution can also be disabled.


## Asynchronous Updates ##

A key property of GraphChi is _asynchronous_ execution, in contrast with the more common (bulk) synchronous execution. In the latter model, all on iteration K+1 see values of their neighbors as they were on previous iteration K. However, in the asynchronous setting, an update function will observe the _most recent_ values available. That is, if vertices A and B are neighbors, and A < B, then update of B will see the changes done by previous update on A.

Asynchronous execution is a powerful model, and can lead to much more efficient convergence of iterative algorithms. However, for programmers used to the synchronous setting, it might take some care to adopt.

It is also straight-forward to simulate synchronous execution on GraphChi. This requires storing two values for each edge: value of the previous iteration and value for the current iteration. Based on the iteration (even or odd), update function reads the other, and writes the other version of the value.