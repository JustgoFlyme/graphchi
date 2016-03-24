## Command line parameters ##

Command-line parameters can be used to _override_ configuration file parameters as follows:

```
   bin/myapps/myprogram file GRAPH-FILE  config1 configvalue1 config2 configvalue2
```


## Most common parameters ##

  * **file** your input graph


  * **filetype**: edgelist or adjlist  (interactive option, if you omit, it will be asked if needed)
  * **execthreads**: the number of threads used for parallel computation
  * **nshards**:  omit or 0 for automatic discovery
  * **membudget\_mb**: amount of megabytes to be used for loading the graph