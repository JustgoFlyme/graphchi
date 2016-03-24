# Introduction: Dynamic Edge Data #

Each GraphChi program needs to define specific types for the vertices and edges of the data graph. First version of GraphChi supported only fixed-size types for edges (primitive types or structs). Since version 0.2, GraphChi now allows definition of **chivectors** as edge data type, which are arrays that have variable size. Including dynamic edge data support was not easy, and the implementation has several limitations. This document will walk through an example application that shows how to use this new feature.

## Note: dynamic vertex data ##

Also vertex data can be dynamic. Read [DynamicVertexData](DynamicVertexData.md)


# Tutorial: Random walks #

In this tutorial we implement a _particle_ based random walk application for GraphChi. This program will simulate [PageRank](http://en.wikipedia.org/wiki/PageRank) by starting one hundred (100) random walks from a subset of the vertices (i.e 100 walks from each vertex), and recording the number of times they visit each vertex during five iterations. Thus, we simulate the "random surfer" model that motivates the PageRank algorithm. (Note: it is much faster to compute PageRank using power-iteration - see the example applications for GraphChi).

Random walks require dynamic edge data, because the number of "walks" traversing through each edge differs at any point of time.

## Source code ##

The source program for this application is found from the example\_apps directory: [randomwalks.cpp](http://code.google.com/p/graphchi/source/browse/example_apps/randomwalks.cpp)/

## Algorithm description ##

  1. On the first iteration, we launch 100 random walks from every 50th vertex. Random walk is defined by an integer that contains the id of the vertex that created the walk.
  1. On subsequent iterations, for each vertex:
    1. Scan in-edges for random walks.
    1. For each random walk, choose a random out-edge and insert the walk into that edge.
    1. Add the total number of walks passing by to the vertex value
  1. In the end, order the vertices by the number of walks that passed by. Normalizing this count with the number of vertices gives estimate of the PageRank.

## Step 0: Enabling dynamic edge data ##

The first line of the application needs to contain following definition:

```
#define DYNAMICEDATA 1
```

This definition will enable the machinery for managing dynamic edge values.

## Step 1:  Defining the data types ##

```
typedef unsigned int VertexDataType;
typedef chivector<vid_t>  EdgeDataType;
```

For each **vertex**, we store an unsigned integer that keeps track of the number of walks passed by.

Each **edge** contains a dynamic **chivector**, i.e a dynamic array that contains values of type **vid\_t** (meaning "vertex-id type"). That is, each walk is represented by an integer depicting the id of the source vertex.

In general, chivectors can contain any primitive or struct type, but not other chivectors.

## Step 2: Define the update-function ##

GraphChi applications are written by using update-functions that are invoked for each vertex in turn.

We first check what is the current iteration. On first iteration we initialize the walks:

```

   int walks_per_source() {
        return 100;
    }
    
    bool is_source(vid_t v) {
        return (v % 50 == 0);
    }
    

  void update(graphchi_vertex<VertexDataType, EdgeDataType > &vertex, graphchi_context &gcontext) {
        if (gcontext.iteration == 0) {
            if (is_source(vertex.id())) {
                for(int i=0; i < walks_per_source(); i++) {
                    /* Get random out edge's vector */
                    graphchi_edge<EdgeDataType> * outedge = vertex.random_outedge();
                    if (outedge != NULL) {
                        chivector<vid_t> * evector = outedge->get_vector();
                        /* Add a random walk particle, represented by the vertex-id of the source (this vertex) */
                        evector->add(vertex.id());
                        gcontext.scheduler->add_task(outedge->vertex_id()); // Schedule destination
                    }
                }
            }
            vertex.set_data(0);
```

On third line of the update-function which check if the vertex is chosen to be a "source" vertex. That is, whether random walks are initiated from the vertex. For this example, we just simply choose vertices with have id multiple of 50 as sources.

If the vertex is a source, we initiate 100 walks from the vertex. This is done by choosing a random out-edge:

```
    graphchi_edge<EdgeDataType> * outedge = vertex.random_outedge();
```

(We need to check if the edge is null, because the vertex might not have any out-edge.)

Then, we add the walk to the out-edge by first asking for a pointer to the **chivector** stored in that edge, and then adding the value to the vector.

```
                  chivector<vid_t> * evector = outedge->get_vector();
                   /* Add a random walk particle, represented by the vertex-id of the source (this vertex) */
                   evector->add(vertex.id());
```

**Note:** unlike in programs that do not use dynamic edge data, we do not call edge->get\_data() but edge->get\_vector().

And last, we need to ensure that GraphChi calls the update function for the vertex at the other end of the edge:

```
   gcontext.scheduler->add_task(outedge->vertex_id()); // Schedule destination
```



On subsequent iterations, the operation is similar. But instead of creating new walks, the update function just _forwards_ walks from its in-edges to out-edges.


```
        } else {
            /* Check inbound edges for walks and advance them. */
            int num_walks = 0;
            for(int i=0; i < vertex.num_inedges(); i++) {
                graphchi_edge<EdgeDataType> * edge = vertex.inedge(i);
                chivector<vid_t> * invector = edge->get_vector();
                for(int j=0; j < invector->size(); j++) {
                    /* Get one walk */
                    vid_t walk = invector->get(j);
                    /* Move to a random out-edge */
                    graphchi_edge<EdgeDataType> * outedge = vertex.random_outedge();
                    if (outedge != NULL) {
                        chivector<vid_t> * outvector = outedge->get_vector();
                        /* Add a random walk particle, represented by the vertex-id of the source (this vertex) */
                        outvector->add(walk);
                        gcontext.scheduler->add_task(outedge->vertex_id()); // Schedule destination
                    }
                    num_walks ++;
                }
```

After _forwarding_ the walks, it is important to clear the values from the in-edge's vector (otherwise we would be creating new walks indefinetely):

```
                /* Remove all walks from the inbound vector */
                invector->clear();
            }
            /* Keep track of the walks passed by via this vertex */
            vertex.set_data(vertex.get_data() + num_walks);
        }
    }
```



## Step 3: Initialize computation ##

The main() of the application looks very similar to any other GraphChi example applciation.
However, there are couple of subtle points:

To create the shards (see IntroductionToGraphChi),  the type template parameter is not EdgeDataType but the element type of the chivectors. That is, our EdgeDataType was vid\_t, so we create the shards with type vid\_t.

```
    bool preexisting_shards;
    int nshards          = convert_if_notexists<vid_t>(filename, get_option_string("nshards", "auto"), preexisting_shards);
```

Note, that we also defined a boolean variable _preexisting\_shards_. That flag will be set to true if the shards were already created. If they were, we need to clean them up first (since they contain the random walks of the previous execution):

```
  if (preexisting_shards) {
        engine.reinitialize_edge_data(0);
    }
```


## Step 4: Start the program and output results ##

Rest of the code is similar to the PageRank example provided in the example applications:

```
/* Run */
    RandomWalkProgram program;
    graphchi_engine<VertexDataType, EdgeDataType> engine(filename, nshards, scheduler, m);
    if (preexisting_shards) {
        engine.reinitialize_edge_data(0);
    }
    engine.run(program, niters);
    
    /* List top 20 */
    int ntop = 20;
    std::vector< vertex_value<VertexDataType> > top = get_top_vertices<VertexDataType>(filename, ntop);
    std::cout << "Print top 20 vertices: " << std::endl;
    for(int i=0; i < (int) top.size(); i++) {
        std::cout << (i+1) << ". " << top[i].vertex << "\t" << top[i].value << std::endl;
    }

```

### Running the program ###

To try this out, you can download a medium size graph for the LiveJournal community from here: http://snap.stanford.edu/data/soc-LiveJournal1.html.

Uncompress the file into graphs-directory in your graphchi-directory and execute:

```
   make example_apps/randomwalks
   bin/example_apps/randomwalks graphs/soc-LiveJournal1.txt  filetype edgelist
```

The final output should look something like:

```

Print top 20 vertices: 
1. 8737	7264
2. 2914	6602
3. 18964	5481
4. 10029	4201
5. 39295	3745
6. 214538	3527

```

The results will of course vary, because the walks are random.

### Remarks ###

PageRank algorithm defines a reset-probability, which is the probability that a random walk is terminated and restarted from a random vertex. You can easily simulate this behavior by starting walks with random probability from each vertex on each iteration, and on the other hand terminating walks with the same probability. This is left as an exercise :).




# Other features #

## Specifying input values ##

Your graph input can contain values for edges. In the case of dynamic edge values, the edges will be initialized with chivectors containing one element (the defined edge value).

Currently GraphChi supports only one iniitial value per edge, but this will be improved later on (please tell us if you require this feature).