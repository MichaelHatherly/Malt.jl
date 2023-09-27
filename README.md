# Malt.jl

Malt is a multiprocessing package for Julia. It is used by [Pluto.jl](https://plutojl.org/) to manage the Julia process that notebook code is executed in, as a [replacement to Distributed](https://github.com/fonsp/Pluto.jl/pull/2240).

You can find more information on the [**documentation**](https://juliapluto.github.io/Malt.jl).

```julia
julia> import Malt

julia> worker = Malt.Worker();

julia> Malt.remote_eval_fetch(worker, :(1 + 1))
2

julia> Malt.remote_eval_fetch(worker, :(rand(5))) |> sum
3.0618168580350966
```

Example of running code asynchonously, and interrupting the process:

```julia
julia> task = Malt.remote_eval(worker, :(sleep(100)))
Task (runnable) @0x0000023539e7f460

julia> Malt.interrupt(worker)

julia> wait(task)
ERROR: TaskFailedException
Stacktrace:
 ...
    nested task error: Remote exception from Malt.Worker on port 9584 with PID 17584:

    InterruptException:
    Stacktrace:
      ...

julia> Malt.stop(worker);

julia> Malt.isrunning(worker)
false
```

## **Malt.jl** vs **Distributed**

Malt.jl is inspired by the [`Distributed standard library`](https://docs.julialang.org/en/v1/stdlib/Distributed/), but with a focus on process sandboxing, not distributed computing. Important differences are:


### API changes
Malt.jl has different function names, see our [**documentation**](https://juliapluto.github.io/Malt.jl).

One important addition is public API for **evaluating an `Expr`**: 

```julia
worker = Malt.Worker()
Malt.remote_eval_fetch(worker, :(sqrt(123)))
```

### Nested use
With Malt.jl, any **worker** process can also be a **host** process to its own workers. 

In Distributed, only "process 1 can add or remove workers". Malt.jl does not have this limiation. *This means that Malt.jl workers can use Distributed (and Malt.jl) like a regular Julia process.*

### Process isolation
Malt.jl worker processes **do not inherit** `ENV` variables, command-line arguments or the Pkg environment from their host.

### Heterogenous computing
Malt.jl does not have API like `@everywhere` or `Distributed.procs`: Malt is **not the right tool for homogenous computing**.

### Exception handling
Exceptions in Malt.jl workers are converted to plaintext before being rethrown in the host. 

The original exception object is only available to the worker. In Distributed, the original exception object is serialized and rethrown to the host.

