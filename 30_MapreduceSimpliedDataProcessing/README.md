1. What did the author(s) try to accomplish?

This paper serves as a cornerstone of MapReduce from *Google*. It introduces the overview and basic components of MapReduce, as well as how it works and main applications. 

2. What were the key elements of the approach(s)?

To summerize the steps of how it works: 

```
  1. The MapReduce library (simplified as library below) splits the input files into M pieces where M is generally the number of mappers assigned for this job ( there should make a distinction between the number of mappers here and in section 3.5 for Task Granularity) so that each piece is tpeically 16 megabytes to 64 megabytes.
  
  2. The master node would picks some idle workers and assigns each one a map task or a reduce task.
  
  3. The workers assigned map tasks read the corresponding input pieces and apply the *Map* function on key/value pairs from these pieces then output some other intermediate key/value pairs which would be buffered in memory.

  4. Periodically, the buffered pairs are written to local disk. *Partition* funtion would be applied on them so they are further partitioned (or hashed, which may not always be the case but easier to understand) into R regions. Then the master would be notified the location of these buffered pairs

  5. Take a step forward, the master would tell reduce workers about the location of those buffered pairs. Once a reduce worker is notified, it uses remote procedure calls to read the buffered data. When all intermediate data are read by a reducer, the reducer would sort it by the intermediate keys.

  
  6. The reducer would iterate over the sorted intermediate data and apply *Reduce* function on each of them. The output of the *Reduce* function is appended to a final output file for this reduce partition.

  7. When all map tasks and reduce tasks have been completed, the master would wake up the user program. At this point, the *MapReduce* call in the user program returns back to the user code.
 ```

3.  List the concepts/techniques that are new to me and highlight whatever I feel necessary to put into my skill set if there is any.

Most concepts are super straightforward, just that some *Computer Science* terminologies are not so intuitive to me. For example, I have no idea what **Network Bandwidth* is. *Deterministic Functions* and *Non-faulting Sequential Execution* also do not make sense to me.

4.  Thoughts and questions

As mentioned above, this paper is more about a instruction and tutorial to me, and I'm unfamiliar with a lot of concepts related perhaps to computer system or processing. So this paper is definitely worthy of revisiting multiple times as this semester goes.