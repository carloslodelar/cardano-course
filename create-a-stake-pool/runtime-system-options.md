# Runtime System Options

The Haskell runtime system is a software layer that provides a set of services that enable Haskell programs to execute on a computer. It is responsible for managing the execution of Haskell programs, including memory management, garbage collection, and scheduling computations on available processor cores.

At a high level, the Haskell runtime system provides the following services:

1. Memory Management: The Haskell runtime system manages the allocation and deallocation of memory used by the program. This includes allocating memory for data structures, managing the stack and heap, and freeing memory that is no longer in use.
2. Garbage Collection: The Haskell runtime system automatically collects and frees memory that is no longer being used by the program. This ensures that memory is used efficiently and helps to prevent memory leaks.
3. Concurrency and Parallelism: The Haskell runtime system provides support for concurrency and parallelism through features like lightweight threads, software transactional memory, and parallel arrays. This enables Haskell programs to take advantage of modern multicore processors and to execute computations in parallel.
4. Exception Handling: The Haskell runtime system provides a mechanism for handling exceptions that occur during program execution. This includes both synchronous exceptions, which occur as a result of a program error, and asynchronous exceptions, which occur as a result of external events such as signals.

In summary, the Haskell runtime system provides a set of essential services that enable Haskell programs to execute on a computer, including memory management, garbage collection, concurrency and parallelism, and exception handling.

{% embed url="https://downloads.haskell.org/ghc/latest/docs/users_guide/runtime_control.html#runtime-system-rts-options" %}

We can use RTS options to achieve a better perfromance from cardano-node, for example:

```
cardano-node run --topology configuration/topology.json \
--database-path db \
--socket-path socket/node.socket \
--port 3000 \
--config configuration/config.json \
+RTS -N2 -A16m -qg -qb --disable-delayed-os-memory-return -RTS
```

Where `+RTS ... -RTS`  Signal to the runtime system that we are passing runtime system options.

* **-N2**: When we specify "-N", the runtime system creates that many operating system threads and assigns each thread a Haskell computation to execute. The runtime system can then schedule these threads to execute concurrently on different cores of the CPU, which can lead to significant performance gains.
* **-A16m**: Set the allocation area size used by the garbage collector.  In general settings >= 4MB can reduce performance in some cases, in particular for single threaded operation. However in a **parallel setting** increasing the allocation area to `16MB`, or even `64MB` can increase gc throughput significantly.
* \-**qg**: Enables parallel garbage collection support.
* **-qb**: Use load-balancing in the parallel GC in generation ⟨gen⟩ and higher. Omitting ⟨gen⟩ disables load-balancing entirely. Load-balancing shares out the work of GC between the available cores.
* **--disable-delayed-os-memory-return**:  To control the behavior of the garbage collector when it releases memory back to the operating system. By default, the Haskell runtime system's garbage collector delays releasing memory back to the operating system. This is because releasing memory back to the operating system can be an expensive operation, and delaying it can improve performance.
