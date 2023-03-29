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

&#x20;Users can extend and override options in `-with-rtsopts`  running cardano-node with command-line RTS options.  Find the options available with:

```
cardano-node +RTS -?

cardano-node:
cardano-node: Usage: <prog> <args> [+RTS <rtsopts> | -RTS <args>] ... --RTS <args>
cardano-node:
cardano-node:    +RTS    Indicates run time system options follow
cardano-node:    -RTS    Indicates program arguments follow
cardano-node:   --RTS    Indicates that ALL subsequent arguments will be given to the
cardano-node:            program (including any of these RTS flags)
cardano-node:
cardano-node: The following run time system options are available:
cardano-node:
cardano-node:   -?       Prints this message and exits; the program is not executed
cardano-node:   --info   Print information about the RTS used by this program
cardano-node:
cardano-node:   --nonmoving-gc
cardano-node:             Selects the non-moving mark-and-sweep garbage collector to
cardano-node:             manage the oldest generation.
cardano-node:   --copying-gc
cardano-node:             Selects the copying garbage collector to manage all generations.
cardano-node:
cardano-node:   -K<size>  Sets the maximum stack size (default: 80% of the heap)
cardano-node:             Egs: -K32k -K512k -K8M
cardano-node:   -ki<size> Sets the initial thread stack size (default 1k)  Egs: -ki4k -ki2m
cardano-node:   -kc<size> Sets the stack chunk size (default 32k)
cardano-node:   -kb<size> Sets the stack chunk buffer size (default 1k)
cardano-node:
cardano-node:   -A<size>  Sets the minimum allocation area size (default 1m) Egs: -A20m -A10k
cardano-node:   -AL<size> Sets the amount of large-object memory that can be allocated
cardano-node:             before a GC is triggered (default: the value of -A)
cardano-node:   -F<n>     Sets the collecting threshold for old generations as a factor of
cardano-node:             the live data in that generation the last time it was collected
cardano-node:             (default: 2.0)
cardano-node:   -n<size>  Allocation area chunk size (0 = disabled, default: 0)
cardano-node:   -O<size>  Sets the minimum size of the old generation (default 1M)
cardano-node:   -M<size>  Sets the maximum heap size (default unlimited)  Egs: -M256k -M1G
cardano-node:   -H<size>  Sets the minimum heap size (default 0M)   Egs: -H24m  -H1G
cardano-node:   -xb<addr> Sets the address from which a suitable start for the heap memory
cardano-node:             will be searched from. This is useful if the default address
cardano-node:             clashes with some third-party library.
cardano-node:   -xn       Use the non-moving collector for the old generation.
cardano-node:   -m<n>     Minimum % of heap which must be available (default 3%)
cardano-node:   -G<n>     Number of generations (default: 2)
cardano-node:   -c<n>     Use in-place compaction instead of copying in the oldest generation
cardano-node:            when live data is at least <n>% of the maximum heap size set with
cardano-node:            -M (default: 30%)
cardano-node:   -c       Use in-place compaction for all oldest generation collections
cardano-node:            (the default is to use copying)
cardano-node:   -w       Use mark-region for the oldest generation (experimental)
cardano-node:   -I<sec>  Perform full GC after <sec> idle time (default: 0.3, 0 == off)
cardano-node:
cardano-node:   -T         Collect GC statistics (useful for in-program statistics access)
cardano-node:   -t[<file>] One-line GC statistics (if <file> omitted, uses stderr)
cardano-node:   -s[<file>] Summary  GC statistics (if <file> omitted, uses stderr)
cardano-node:   -S[<file>] Detailed GC statistics (if <file> omitted, uses stderr)
cardano-node:
cardano-node:
cardano-node:   -Z         Don't squeeze out update frames on stack overflow
cardano-node:   -B         Sound the bell at the start of each garbage collection
cardano-node:   -h       Heap residency profile (output file <program>.hp)
cardano-node:   -hT      Produce a heap profile grouped by closure type
cardano-node:   -i<sec>  Time between heap profile samples (seconds, default: 0.1)
cardano-node:
cardano-node:   -C<secs>  Context-switch interval in seconds.
cardano-node:             0 or no argument means switch as often as possible.
cardano-node:             Default: 0.02 sec.
cardano-node:   -V<secs>  Master tick interval in seconds (0 == disable timer).
cardano-node:             This sets the resolution for -C and the heap profile timer -i,
cardano-node:             and is the frequency of time profile samples.
cardano-node:             Default: 0.01 sec.
cardano-node:
cardano-node:   -N[<n>]    Use <n> processors (default: 1, -N alone determines
cardano-node:              the number of processors to use automatically)
cardano-node:   -maxN[<n>] Use up to <n> processors automatically
cardano-node:   -qg[<n>]  Use parallel GC only for generations >= <n>
cardano-node:             (default: 0, -qg alone turns off parallel GC)
cardano-node:   -qb[<n>]  Use load-balancing in the parallel GC only for generations >= <n>
cardano-node:             (default: 1 for -A < 32M, 0 otherwise;
cardano-node:              -qb alone turns off load-balancing)
cardano-node:   -qn<n>    Use <n> threads for parallel GC (defaults to value of -N)
cardano-node:   -qa       Use the OS to set thread affinity (experimental)
cardano-node:   -qm       Don't automatically migrate threads between CPUs
cardano-node:   -qi<n>    If a processor has been idle for the last <n> GCs, do not
cardano-node:             wake it up for a non-load-balancing parallel GC.
cardano-node:             (0 disables,  default: 0)
cardano-node:   --numa[=<node_mask>]
cardano-node:             Use NUMA, nodes given by <node_mask> (default: off)
cardano-node:   --install-signal-handlers=<yes|no>
cardano-node:             Install signal handlers (default: yes)
cardano-node:   -e<n>     Maximum number of outstanding local sparks (default: 4096)
cardano-node:   -xp       Assume that all object files were compiled with -fPIC
cardano-node:             -fexternal-dynamic-refs and load them anywhere in the address
cardano-node:             space
cardano-node:   -xm       Base address to mmap memory in the GHCi linker
cardano-node:             (hex; must be <80000000)
cardano-node:   -xq       The allocation limit given to a thread after it receives
cardano-node:             an AllocationLimitExceeded exception. (default: 100k)
cardano-node:
cardano-node:   -Mgrace=<n>
cardano-node:             The amount of allocation after the program receives a
cardano-node:             HeapOverflow exception before the exception is thrown again, if
cardano-node:             the program is still exceeding the heap limit.
cardano-node:
cardano-node: RTS options may also be specified using the GHCRTS environment variable.
cardano-node:
cardano-node: Other RTS options may be available for programs compiled a different way.
cardano-node: The GHC User's Guide has full details.
```

For example, this will extend the default options with **`-qg`** and **`-qb:`**

```bash
cardano-node run --topology configuration/topology.json \
--database-path db \
--socket-path socket/node.socket \
--port 3000 \
--config configuration/config.json \
+RTS -qg -qb
```

Where `+RTS ...`  Signal to the runtime system that we are passing runtime system options. In the above example we are using:

* ****[**-T**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--T)**:** Produce runtime-system statistics, such as the amount of time spent executing the program and in the garbage collector, the amount of memory allocated, the maximum size of the heap, and so on. The three variants give different levels of detail: `-T` collects the data but produces no output. Access the statistics using [GHC.Stats](https://downloads.haskell.org/ghc/latest/docs/libraries/base-4.18.0.0/GHC-Stats.html).
* ****[**-N2**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/using-concurrent.html#rts-options-for-smp-parallelism): Used to specify the number of threads to use for parallel execution. The `-N2` flag specifies that the Haskell runtime system should use two parallel threads.
* **-**[**A16m**:](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--A%20%E2%9F%A8size%E2%9F%A9) Sets the maximum heap size for the generational garbage collector to 16 megabytes.&#x20;
* ****[**-I0:**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--I%20%E2%9F%A8seconds%E2%9F%A9) **** Set the amount of idle time which must pass before a idle GC is performed. Setting `-I0` disables the idle GC.
*   ****[**--disable-delayed-os-memory-return**:](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag---disable-delayed-os-memory-return)  For accurate resident memory usage of the program as shown in memory usage reporting tools (e.g. the RSS column in top and htop). This makes it easier to check the real memory usage.


* \-[**qg**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--qg%20%E2%9F%A8gen%E2%9F%A9): disables parallel GC, use sequential GC.
* \-[**qb**](https://downloads.haskell.org/ghc/latest/docs/users\_guide/runtime\_control.html#rts-flag--qb%20%E2%9F%A8gen%E2%9F%A9): disables GC load balancing

