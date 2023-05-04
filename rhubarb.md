RHuBarb: Speeding up Edge-Centric Processing using Recursive Hilbert Blocking

In Chapter [ch:CaseStudy], we illustrated the performance improvement
that is gained by traversing the edges of the graph using the Hilbert
Curve for a variety of graph datasets and orderings. This motivated our
design and implementation of the Parallel SlashBurn algorithm. However,
our case-study was performed using a single-threaded PageRank
implementation, while all modern multithreaded GPS try to efficiently
use all available cores when computing graph analytics for the purposes
of scalability and speed. This naturally leads to the following
question:

1.  Can we efficiently parallelize the execution of edge-centric
    algorithms that order the edges using the Hilbert curve while
    maintaining the cache locality benefit seen using a single thread?

Our answer to this question is RHuBarb, an edge reordering and blocking
technique that uses Recursive Hilbert Blocking, and is geared towards
parallel computation of edge-centric algorithms.
We begin this chapter by describing the main challenges that we
encountered in our parallelization of the Hilbert Curve:

1.  Ensuring safe writes from multiple threads to vertex data.

    -   Overcoming inefficiencies in OpenMP array reductions.

2.  A thread schedule and assignment of work to threads that maintain:

    -   an even work distribution, and

    -   cache locality of read/write access to shared arrays of vertex
        data.

Next, we describe the techniques we designed and used to address these
concerns, namely:

1.  Recursive Hilbert blocking

2.  Spray: Sparse Reductions of Arrays in OpenMP

Finally, we describe our implementation of three edge-centric
algorithms: PageRank, Connected Components, and Collaborative Filtering
in Rhubarb.

Parallelizing Edge-Centric Computation

Since the Hilbert Space Filling Curve is defined recursively, it follows
that parallelizing of edge-centric computation of the edges that lie on
the Hilbert curve could be done by splitting the graph’s adjacency
matrix into equally sized quadrants. We can then assign each thread the
edges that lie in each quadrant. Two issues arise from this naı̈ve
parallelization:

1.  Potential write contention: Threads may attempt to concurrently
    update the same destination vertex data. For example, in the figure,
    if Thread 1 processes the edge (0, 1) at the same time as when
    Thread 2 processes the edge (2, 1), the updates to destination
    vertex 1 may be clobbered.

2.  Uneven distribution of work between threads: Splitting up the
    adjacency matrix into quadrants of equal side-length may result in
    an uneven distribution of edges between threads. Since quadrants act
    as the base unit of work which assign threads, and there is no
    guarantee that the number of edges that lie within a quadrant is
    distributed evenly amongst the quadrants, this may cause an uneven
    work distribution between threads, causing stagglers.
