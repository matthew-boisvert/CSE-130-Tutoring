# Assignment 3 Resources


## Load Balancing
![enter image description here](https://i.imgur.com/w13u9m6.png)
Prior to implementing caching, it might be best to try implementing your load-balancing functionality. From the perspective of the servers, your load balancer will be a client (see client.c for starter code). From the perspective of the clients, your load balancer will be a server. Try leveraging your Assignment 2 strategy for your load-balancing implementation. Specifically, you can use a dispatcher thread to accept incoming client connections and spool up work. Then, your worker threads can recv() the incoming requests, choose a server to forward the request to, and send() the data directly to that server (instead of write()ing to a file like in assignment 2). Finally, your worker threads can recv() data from the server that they forwarded the request to and send() the response back to the client. Note: For choosing the least-loaded server, you will need an internal state to keep track of each server's status (# of errors, # of entries). This state is updated every time a healthcheck is performed or every time you send a request to a server (because you know that you will be contributing to the entries/errors of that server).
![enter image description here](https://i.imgur.com/9QyUORN.png)
Note: To implement healthchecks, you might want to have 1 thread be solely responsible for this (in addition to the 1 dispatcher and the worker threads). Hint: To deal with getting responses back from multiple servers at once, look at the man page for the select() system call.

## Caching
As a high-level overview, caching is a technique used to speed up the memory accesses of a service's frequently accessed pieces of data in order to improve the average memory access latency. For the purposes of this assignment, we will use an in-memory cache so that frequently accessed files can be retrieved from the load balancer's memory rather than from the disk of a separate server.

Caches store data on the granularity of cache lines, where each cache line is a bounded group of elements which share a common identifier. In the CPU architecture of your machine, this "identifier" is likely the higher-order bits of each memory address (known as the index). For our cache, all of the elements will be stored in single cache line. The number of elements in the cache line is known as the set "associativity" of the cache. For example, if the associativity of the cache is specified is 5, then we would say that the cache is 5-way set associative. This means that the maximum number of elements that can be mapped to a given same cache line would be 5. Hence, if a cache line is full and you want to add another element (because it maps to that same cache line), you will need to choose an element in the cache line to evict. There is a great body of research related to the practical applications of caches and what the best replacement policies are, but the general intuition is that you want an eviction policy which removes "cold" data (data that is likely never going to be accessed again in the future) and preserves "hot" data (data that is likely to be accessed again frequently -- exhibiting a property what we refer to as "temporal locality"). 

For this assignment, you can choose between a FIFO eviction policy or an LRU eviction policy (for extra credit). For both policies, it might be easiest to implement your cache as a linked list so that you can add/lookup elements to the cache in O(1) time. Note: If we had multiple cache lines, you could then scale this to a chained hash map to maintain O(1) lookups and additions (something something left as an exercise for the reader). With a FIFO eviction policy, you always evict the element which was added least recently. With a LRU eviction policy, you always evict the element which was accessed least recently.

Here is an example of performing insertions/lookups with a FIFO cache:![enter image description here](https://i.imgur.com/PskjSSG.png)

Here is an example of performing lookups/insertions with an LRU cache:
![enter image description here](https://i.imgur.com/4AEse3U.png)

Note: This is just a high-level overview to get started on Assignment 3. Please refer to the spec or ask clarifying questions on Discord

-- Matt (mboisver)
