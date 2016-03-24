## Error: Too many open files ##

**Background**:  Most operating systems have an user-defined  limit for the number of open files at any point. On Mac OS X, this is only 256 by default. If you are running a GraphChi program with plenty of _shards_, you may end up having more than the allowed number of files open.

**Solution**:  On Linux and Mac OS X, you can use "ulimit -n X" command. For example:
> ulimit -n 4096

Run "ulimit -n" to display the current limit.

In practice, you can set the limit very high - there is no performance penalty

## Computer becomes unresponsive when running GraphChi, or GraphChi is very slow ##

**Reason**: Your computer is probably "swapping", which means that the system has run out of memory (RAM), and uses the disk to swap memory pages. As a result, the system usually becomes really slow and unresponsive.

**Solution**: You need to decrease the amount of memory GraphChi uses. In the configuration file (conf/graphchi.cnf), there is a parameter "membudget\_mb", which determines the maximum amount of memory used while loading the graph. For a computer with 8-gigabytes of memory, membudget\_mb=3000 is reasonable, but for a 4-gigabyte machine usually just membudget\_mb=1000 is a good choice. However, this all depends on how many other applications you have running etc. You need to just find a suitable value for your system - GraphChi runs well even with a small amount of memory. **Note:** it might be a good idea to delete your shards so the system will recreate them based on the new memory settings.

Mac OS seems to sometimes fragment memory. Reboot can help.


## GraphChi fails to compile on 32-bit system ##

Earlier versions of GraphChi Makefile had flag "-m64" for the compiler, forcing the compilation into 64-bit binaries. You can safely remove this flag (line 4 of Makefile).