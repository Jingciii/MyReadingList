## Summary on the Google File System

### Motivations

 ---  by the observations of application workloads and tecnological environment

 - Component failures are the norm rather than the exception. Constant monitoring, error detection, fault tolerance, and automatic recovery must be integral to the system
 - Multi-GB files are common
 - Most files are mutated by appending new data rather than overwriting existing data. 
 - Co-designing the applications and the file system API benefits the overall system by increasing the flexibility 

### Design Overview
 
 
#### Assumptions

 - The system must constantly monitor itself and detect, tolerate, and recover promptly from component failures on a routine basis
 - Multi-GB files are common and should be managed efficiently. Small files must be supported and we do not need to optimize for them
 - Performance-conscious applications batch and sort theirsmall reads to advance steadily through the file rather than go back and forth
 - Small writes at arbitrary positions in a file are supported but do not have to be efficient
 - Well-defined semantics for multiple clients that concurrently append to the same file must be implemented
 - High sustained bandwidth is more important tahn low latency

#### Interface

Files are organized hierarchically in directories and identified by pathnames. Usual operations to create, delete, open, close, read, and write files are supported. There is also snapshot and record append operations

#### Architecture

A GFS cluster contains of a single master and multiple chunkservers and is accessed by multiple clients. Each of these is a commodity Linux machine. 

Chunkservers store chunks on local disks as Linux files and read or write chunk data specified by a chunk handle and byte range. 

The master maintains all file system metadata, including namespaces for chunks and files, access control information, the mapping from files to chunks, and the current locations of chunks. Client interact with the master for metadata operations, but all data-bearing communication goes directly to the chunkservers.

#### Single Master


The involvement of reads and writes for master should be minimized so that it does not become a bottlenect (save space for in-memory data structure?)

Typically, the client asks for multiple chunks in the same request and the master can also include the information for chunks immediately following those requested. The extra information sidesteps several future client-master interactions at practically no extra cost

#### Chunk Size

- pros
	- A large chunk size reduces clients' need to interact with the master
	- On a large chunk, a client is more likely to perform many operations on a given chunk and thus it reduces network overhead by keeping a persistenet TCP connection to the chunkserver over an extended period time
	- Large chunk size reduces the size of the the metadata stored on the master
 - cons
	- Easily to form hot spots


#### Metadata

All metadata is kept in the master's memory. namespaces and file-to-chunk mapping are also kept persistent by logging mutations to an operation log stored on the master's local dist and replicated on remote machines

##### In-Memory Data Structures

It provides efficiency for chunk gabage collection, re-replication in the presence of chunkserver failures, and chunk migration to balance load and disk space useage across chunkservers. But the capacity of the whole system is limited by how much memory the master has. Generally, it does not require that much space to store metadata information, and even if it does, the cost of adding extra memory to the master is a small price to pay for the simplicity, reliability, performance and flexibility.


##### Chunk Locations

The master simply polls chunkservers at the startup. Then the master can keep itself up-to-date because it controls all the chunk placement and monitors chunkserver status with regular HeartHeat messages

##### Operation Log

The log files are critical we must store it reliable and not make changes visble to client until metadata changes are made persistent. The logs are replicated on multiple remote machines and repond to a client operation only after flushing the corresponding log record to disk both locally and remotely. The master batches several log records together before flushing thereby reducing the impact of flushing and replication on overall system throughout.

The master recovers its file system state by replaying the operation log. Since all the historical  operations are huge, to minimize the startup time, the log must be kept small. So the master checkpoints its state whenever the log grows beyond a certain size so that it can loading the latest checkpoint from local disk replaying only the limited number of log records after that. Older checkpoints and log files can be freely deleted. A failure during checkpointing does not affect correctness because the recovery code detects and skips incomplete checkpoints

#### Consistency Model

##### Guarantees by GFS

A file region is consistent if all clients will always see the same data, regardless of which replicas they read from. A region is defined after a file data mutation if it is consistent and clients will see what the mutation writes in its entirety.

The mutated file region is guaranteed to be defined and contain the data writen by the last mutation after a sequence of successful mutaions. GFS achieves this by
 - Applying mutations to a chunk in the same order on all its replicas,
 - Using chunk version numbers to detect any replica that has become stale because it has missed mutations while its chunkserver was down.

##### Implications for Applications
 
 - When generating a file, it periodically checkpoints how much has been successfully written. Checkpoints may also include application-level checksums. Readers versify and process only the file region up to the last checkpoint, which is known to be in the defined state.
 - For concurrent appendding in files, a reader can deal with extra padding and record fragments using the checksums. They can also filter the occational duplicates using unique identifiers in the record, which are often needed anyway to name corresponding application entities such as web documents

### System Interactions
 
 
#### Leases and Mutation Order

Each mutation is performed at all the chunk's replicas.

Master grants chunk lease to primary --> primary picks a serial order for all mutations --> all replicas follow this order when applying mutations (if the write had failed at the primary, it would not have been assigned a serial number and forwarded). Figure 2 in the paper has shown this process clearly.

If a write by the application is large or exceed a chunk boundary, GFS client code breaks it down into multiple write operations, all of which follow the contral flow in Figure 2 but may be interleaved with and overwritten by concurrent openrations from other clients. 

#### Data Flow

To fully utilize each machine's network bandwidth, the data is pushed linearly along a chain of chunkservers rather than distributed in some other topology. To avoid network bottlenecks and high-latency links as much as possible, each machine forwards the data to the closest machine in the network topology (distance can be found by IP address) that has not received it.

#### Atomic Record Appends


Follow the data flow in Figure 2, after the client pushes the data to all replicas of the last chunk of the file, then it sends its request to the primary. The primary checks to see if appending the record to the current chunk would cause the chunk to exceed 64 MB. If so it pads the chunk to the maximum size and tells secondaries to do the same, and replies to the client indicating that the operation should be retried on the next chunk. 

Record append is restricted to be at most one-forth of the maximum chunk size to keep worst-case fragmentation at an acceptable level

#### Snapshot

- When the master received a snapshot request, it revokes any outstanding leases on the chunks in the files it is about to snapshot
- After the leases have been revoked or have expired, the master logs the operation to disk. then it applies this log record to its in-memory state by duplicating the metadata for the source file or directory tree
- The first time a client wants to write to a chunk C after the snapshot operation, it sends a request to the master to find the current lease holder. The master notices that the reference count for chunk C is greater than one. It defers replying to the client request and instead picks a new chunk handle C'
- Then the master asks each chunkserver that has a current replica of C to ceate a new chunk called C'
- Thus in the same chunkserver, data can be copies locally

### Master Operation

#### Namespace Management and Locking

GFS does not have a per-directory data structure that lists all the files in that directory. Nor does it support alias for the same fileor directory. Instead, GFS logically represents its namespace as a lookup table mappin gfull pathnames to metadata. 

Locks are acquired in a consistent total order to prevent deadlock: they are first ordered by level in the namespace tree and lexicographically within the same level

#### Replica Placement

Two purposes:
 - Maximize data reliability and availability
 - Maximize network badwidth utilization

#### Creation, Re-replication, Rebalancing
 
Creation:

 - Place new replicas on chunkservers with below-average disk space utilization
 - Limit the number of "recent" creations on each chunkserver
 - Spread replicas of a chunk across racks

Re-replication: 

 - How far it is from its replication goal
 - We prefer to first re-replicate chunks for live files as opposed to chunks that belong to recently delited files
 - To minimize the impact of failures on running applications, we boost the priority of any chunk that is blocking client progress

Rebalancing:
 
  - The master examines the current replica distribution and moves replicas for better disk space and load balancing
  - It gradually fills up a mew chunkserver rather than instantly swamps it with new chunks and the heavy write traffic that comes with them
  - The master must also choose which existing replica to move. In general, it prefers to remove those on chunkservers with below-average free space so as to equalize disk space usage

  
#### Garbage Collection

##### Mechanism

When a file is deleted by the application, the master logs it by renaming it with a hidden name with the timestamp of the deletion, and in fact they can still be restored at this time. During the master's regular scan, it removes any hidden file if they have existed over 3 days and erases its in-memory metadata when the hidden file is removed from the namespace. Similar for orphaned chunks.


##### Discussion

Several advantages of garbage collection over eager deletion:

 - It is simple and reliable in a large-scale distributed system where component failures are common
 - It merges storage reclamation into the regular background activities of the master, such as the regular scans of namespaces and handshakes with chunkservers. Thus, it is done in batches and the cost is amortized. Plus, it is done only when the master is relatively free.
 - The dalay in reclaiming storage provides a safety net against accidental, irreversible deletion

Main disadvantage: 
 
 - The delay sometimes hiders user effort to fine tune usage when storage is tight

  
#### Stale Replica Detection


 Whenever the master grants a new lease on a chunk, it increases its version number and informs its up-to-time replicas. The master will detect a chunkserver has stale replica when the chunkserver restarts and reports its set of chunks and their associated version numbers. 
 
 The master removes stale replicas in its regular garbage collection.







