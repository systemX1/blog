# **Download**

http://research.google.com/archive/gfs-sosp2003.pdf

# Notes

也许叫摘要更加合适

## **Architecture**

A GFS cluster consists of a single *master* and multiple *chunkservers* and is accessed by multiple *clients*.

Files are divided into fixed-size chunk(64 MB). Each chunk has three replicas by default, identified by an unique 64 bit chunk handle assigned by the master at the time of chunk creation.

The master maintains all file system metadata(namespace, control information, mapping from files to chunks and chunks' current locations).

Clients interact with the master for metadata operations, but all data-bearing communication goes directly to the chunkservers.

High sustained bandwidth is more important than low latency,  and mutated more by appending new data rather than overwriting existing data.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211209133846-29e6d40479853ee5a655cf60a3daaab1-GFSFigure1.png" style="zoom:40%;" />

### **Chunk Size**

*2.5 Chunk Size* 64 MB is much larger than typical file system

Advantages: reduces clients’ need to interact with the master; reduce network overhead by keeping a persistent TCP connection since client is more likely to perform many operations on a given chunk; reduces the size of the metadata stored on the master.

Disadvantages: A small file consists of a small number of chunks, perhaps just one. The chunkservers storing those chunks may become hot spots if many clients are accessing the same file(Haven't been a major issue in practice because applications mostly read large multi-chunk files sequentially).

Lazy space allocation: the physical allocation of space is delayed until data at the size of the chunk size is accumulated. It will avoids wasting space due to internal fragmentation.

Most chunks are full because most files contain many chunks, only the last of which may be partially filled.

### **Master**

The master stores three major types of metadata: the file
and chunk namespaces, the mapping from files to chunks,
and the locations of each chunk’s replicas.

### **Read Files**

*2.4 Single Master*

1. The client translates the file name and byte offset specified by the application into a chunk index within the file at the first. And then, the client sends the master a request containing the file name and chunk index. 

2. The master replies with the corresponding chunk handle and locations of the replicas.

3. The client will caches the information for a limited time and then sends a request to one of the replicas, most likely the closest one. 

In fact, the client typically asks for multiple chunks in the same request and the master can also include the information for chunks immediately following those requested. This extra information sidesteps several future client-master interactions at practically no extra cost.







## **Consistency**

Weak consistency



### **Master Recovery**

Master all in RAM

*2.6.3 Operation Log* The operation log contains a historical record of critical metadata changes. Is it the only persistent record of metadata and serves as a logical time line that defines the order of concurrent operations. Files and chunks, as well as their versions are all uniquely and eternally identified by the logical times at which they were created.

Replicate it on multiple remote machines and respond to a client operation only after flushing the corresponding log record to disk both locally and remotely.





### **Write Files**

***3.1 Leases and Mutation Order***

Each mutation is performed at all the chunk’s replicas. Lease mechanism is used to maintain consistency. Only the primary replica will have lease. 

1. The client asks for the primary and the secondary replicas.

2. The master replies and the client cached. No need to contact until the primary unreachable or lease expired. 

3. The client pushes the data to all the replicas(in any order). Each chunkserver will store the data in an internal LRU buffer cache until the data is used or aged out.

4. Once all the replicas have acknowledged receiving the
   data, the client sends a write request to the primary.

5. The primary forwards the write request to all secondary replicas.

6. The secondaries all reply to the primary indicating
   that they have completed the operation. Each secondary replica applies mutations in the same serial number order assigned by the primary.

7. The primary replies to the client. In case of errors, the write may have succeeded at the primary and an arbitrary subset of the secondary replicas. It will make a few attempts at steps (3) through (7) before falling back to a retry from the beginning of the write.

If a write straddles a chunk boundary, it will be broken into multiple write operations. They may be interleaved with and overwritten by concurrent operations from other clients(consistent but undefined, *2.7 Consistency Model*).

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211209133846-e8b01af34bba92ef166837a3655fedb9-GFSFigure2.png" style="zoom:50%;" />

### **3.2 Data Flow**

To fully utilize each machine’s network bandwidth, the data is pushed linearly along a chain of chunkservers. Each machine forwards the data to the closest machine in the network topology that has not received it. Once a chunkserver receives some data, it starts forwarding immediately(receive and send are happened in the same time).

Without network congestion, the ideal elapsed time for transferring B bytes to R replicas is B/T + RL where T is the network throughput and L is latency to transfer bytes between two machines(Transmsission delay + Propagation delay).

### **3.3 Atomic Record Appends**

After the client pushes the data to all replicas of the last chunk of the file, a request will be sent to the primary. The primary checks to see if appending the record to the current chunk would cause the chunk to exceed the maximum size (64 MB). If so, it pads the chunk to the maximum size, tells secondaries to do the same, and replies to the client indicating that the operation should be retried on the next chunk. (Record append is restricted to be at most one-fourth of the maximum chunk size to keep worst case fragmentation at an acceptable level.)

一次append最大16MB 如果越界就pad chunk剩下的部分然后让client重发 因为有append大小限制因此碎片最大不会超过16MB



如果append失败不会覆盖failed部分 而是在后面pad 然后at the same offset on all replicas of some chunk 继续append. 但这样会造成duplication. GFS does not guarantee that all replicas are bytewise identical.



### **Snapshot**

The snapshot operation makes a copy of a file or a directory tree almost instantaneously.

**Copy-On-Write**

When the master receives a snapshot request, it first revokes any outstanding leases on the chunks in the files it is about to snapshot. This ensures that any subsequent writes to these chunks will require an interaction with the master to find the lease holder.

leases全部不可用之后, logs the operation to disk, then duplicating the metadata for the source file or directory tree.

如果client对某个chunk C有新的写操作 master会先通知所有replicas本地复制一份C' 然后对C'写



## **Master operation**

### **4.1 Namespace**

Prefix compression. Each node in the namespace tree has an associated read-write lock.

Each master operation acquires a set of locks before it
runs. 对于/d1/d2/.../dn/leaf /d1, /d1/d2, ..., /d1/d2/.../dn都会获取读锁 对/d1/d2/.../dn/leaf获取读锁或写锁

The read lock on the name is sufficient to protect the parent
directory from deletion. One nice property of this locking scheme is that it allows concurrent mutations in the same directory.



<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211209133846-8bceb779ec85cd490bae4c471befeea5-GFSTable1.png" style="zoom:50%;" />

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211209133846-cdb5ab16f88d920ba7bdb17f40f893ea-GFSTable2.png" style="zoom:50%;" />



## **Questions**







*I.e.* is an abbreviation for the phrase *id est*, which means "that is." *I.e.* is used to restate something said previously in order to clarify its meaning. *E.g.* is short for *exempli gratia*, which means "for example." *E.g.* is used before an item or list of items that serve as examples for the previous statement.

# **Reference**

[The Google File System](http://research.google.com/archive/gfs-sosp2003.pdf)

[6.824 Schedule: Spring 2021](https://pdos.csail.mit.edu/6.824/index.html) 

[mit-public-courses-cn-translatio](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/)

[What is lazy space allocation in Google File system](https://stackoverflow.com/questions/18109582/what-is-lazy-space-allocation-in-google-file-system)

[大数据经典论文解读](https://time.geekbang.org/column/intro/100091101)

[Google工程师 の 大数据论文解读2 - GFS](https://www.bilibili.com/video/BV1GR4y1b71o)

幻读phantom problem

脏读dirty read

不可重复读unrepeatable read
