# Runtime System Options

The Haskell runtime system is a software layer that provides a set of services that enable Haskell programs to execute on a computer. It is responsible for managing the execution of Haskell programs, including memory management, garbage collection, and scheduling computations on available processor cores.

At a high level, the Haskell runtime system provides the following services:

1. Memory Management: The Haskell runtime system manages the allocation and deallocation of memory used by the program. This includes allocating memory for data structures, managing the stack and heap, and freeing memory that is no longer in use.
2. Garbage Collection: The Haskell runtime system automatically collects and frees memory that is no longer being used by the program. This ensures that memory is used efficiently and helps to prevent memory leaks.
3. Concurrency and Parallelism: The Haskell runtime system provides support for concurrency and parallelism through features like lightweight threads, software transactional memory, and parallel arrays. This enables Haskell programs to take advantage of modern multicore processors and to execute computations in parallel.
4. Exception Handling: The Haskell runtime system provides a mechanism for handling exceptions that occur during program execution. This includes both synchronous exceptions, which occur as a result of a program error, and asynchronous exceptions, which occur as a result of external events such as signals.

{% embed url="https://downloads.haskell.org/ghc/latest/docs/users_guide/runtime_control.html#runtime-system-rts-options" %}

IOG-released binaries are built with the following RTS options (-with-rtsopts):  `-T -I0 -A16m -N2 --disable-delayed-os-memory-return` . Meaning that by default the node uses these options at runtime.  You can check it with:

```
cardano-node +RTS --info

 [("GHC RTS", "YES")
 ,("GHC version", "8.10.7")
 ,("RTS way", "rts_thr")
 ,("Build platform", "x86_64-unknown-linux")
 ,("Build architecture", "x86_64")
 ,("Build OS", "linux")
 ,("Build vendor", "unknown")
 ,("Host platform", "x86_64-unknown-linux")
 ,("Host architecture", "x86_64")
 ,("Host OS", "linux")
 ,("Host vendor", "unknown")
 ,("Target platform", "x86_64-unknown-linux")
 ,("Target architecture", "x86_64")
 ,("Target OS", "linux")
 ,("Target vendor", "unknown")
 ,("Word size", "64")
 ,("Compiler unregisterised", "NO")
 ,("Tables next to code", "YES")
 ,("Flag -with-rtsopts", "-T -I0 -A16m -N2 --disable-delayed-os-memory-return")
 ]
```

### Customized RTS options

Users can choose to use different options by tweaking  `-with-rtsopts` on the node's [cabal file](https://github.com/input-output-hk/cardano-node/blob/master/cardano-node/cardano-node.cabal), and building the node with the new options. For example:

```
ghc-options:    "-with-rtsopts=-T -I0 -A16m -N2 --disable-delayed-os-memory-return"
```

Alternatively, users can override options in `-with-rtsopts`  running cardano-node with command-line RTS options, for example:&#x20;

```
cardano-node run --topology configuration/topology.json \
--database-path db \
--socket-path socket/node.socket \
--port 3000 \
--config configuration/config.json \
+RTS -N2 -A16m -I0 --disable-delayed-os-memory-return -RTS
```

Where `+RTS ... -RTS`  Signal to the runtime system that we are passing runtime system options. In the above example we are using:

* ****[**-T**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--T)**:** Produce runtime-system statistics, such as the amount of time spent executing the program and in the garbage collector, the amount of memory allocated, the maximum size of the heap, and so on. The three variants give different levels of detail: `-T` collects the data but produces no output. Access the statistics using [GHC.Stats](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.18.0.0/GHC-Stats.html).
* ****[**-N2**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/using-concurrent.html#rts-options-for-smp-parallelism): Used to specify the number of threads to use for parallel execution. The `-N2` flag specifies that the Haskell runtime system should use two parallel threads.
* **-**[**A16m**:](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--A%20%E2%9F%A8size%E2%9F%A9) Sets the maximum heap size for the generational garbage collector to 16 megabytes.&#x20;
* ****[**-I0:**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--I%20%E2%9F%A8seconds%E2%9F%A9) **** Set the amount of idle time which must pass before a idle GC is performed. Setting `-I0` disables the idle GC.
*   ****[**--disable-delayed-os-memory-return**:](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag---disable-delayed-os-memory-return)  For accurate resident memory usage of the program as shown in memory usage reporting tools (e.g. the RSS column in top and htop). This makes it easier to check the real memory usage.

    \
