# Introduction #

Now you can also use dynamic **chivectors** as your vertex-data type.
The basic usage is similar to DynamicEdgeData, so please read that tutorial
first.


# Usage #

There is currently no example application which uses dynamic vertex data, but there
is a test code that does: http://code.google.com/p/graphchi/source/browse/src/tests/test_dynamicedata_loader.cpp

To enable dynamic vertex data, you need to define **in the beginning of your source code**:

```
  #define DYNAMICVERTEXDATA 1   
```

Then define the vertex datatype:
```
 typedef chivector<size_t> VertexDataType;
```

You can replace **size\_t** with any primitive or struct type.

Here is example how to add and read values from the vertex's vector:

```
        chivector<size_t> * vvector = vertex.get_vector();
         int numitems = vertex.id() % 10;
         for(int i=0; i<numitems; i++) {
             vvector->add(i); // Arbitrary
         }
        
         /* Check vertex data immediatelly */
         for(int i=0; i<numitems; i++) {
            assert(vvector->get(i) == i);
         }
```

No, that instead of get\_data(), you call get\_vector() for the vertex object.

# Accessing vertex data after computation #

After your computation has finished, you can access the vertex values by using
vertex-data iterator callback.

Here is example:

```
    VertexValidator validator;
    foreach_vertices(filename, 0, engine.num_vertices(), validator);
```

Here **validator** is a callback object:

```
class VertexValidator : public VCallback<chivector<size_t> > {
public:
    virtual void callback(vid_t vertex_id, chivector<size_t> &vec) {
        int numitems = vertex_id % 10;
        assert(vec.size() == numitems);
        
        for(int j=0; j < numitems; j++) {
            size_t x = vec.get(j);
            assert(x == j);
        }
    }
};
```

For example, if you want to output your results into a text-file, you simply implement file writing inside the callback.