GraphChi supports most of the core functionality of GraphLab's new (version 2.1) Gather-Apply-Scatter model of computation ( see http://www.graphlab.org ).

As a difference to basic GraphChi programs, GraphLab-programs have their vertex data in-memory. That is, you need to have enough memory to store values of each vertex in your main memory. In most cases, this is not a problem, since the number of edges vastly outnumber the number of vertices.

While waiting for the complete documentation, you can study an example GraphLab program under GraphChi: http://code.google.com/p/graphchi/source/browse/example_apps/matrix_factorization/graphlab_gas/als_graphlab.cpp