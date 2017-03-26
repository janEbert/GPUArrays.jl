# JTensors (formerly GPUArrays)

[![Build Status](https://travis-ci.org/SimonDanisch/JTensors.jl.svg?branch=master)](https://travis-ci.org/SimonDanisch/JTensors.jl)


Prototype for a GPU Array library.
It implements the Base AbstractArray interface for Julia's various GPU backends.

We're using Julia's JIT to generate optimized kernels for map/broadcast operations.
The compilation for the GPU is done with [CUDAnative.jl](https://github.com/JuliaGPU/CUDAnative.jl/)
and for OpenCL and OpenGL [Transpiler.jl](https://github.com/SimonDanisch/Transpiler.jl) is used.
In the further future it's planned to replace the transpiler by the same approach
CUDAnative.jl is using (via LLVM + SPIR-V).

This allows to get more involved functionality, like complex arithmetic, for free, since we can compile what's already in Julia Base.

JTensors relies heavily on dot broadcasting. The great thing about dot broadcasting in Julia is, that it [actually fuses operations syntactically](http://julialang.org/blog/2017/01/moredots), which is vital for performance on the GPU.
E.g.:

```Julia
out .= a .+ b ./ c .+ 1
```

Will result in one GPU kernel call to a function that combines the operations without any extra allocations.
This allows JTensors to offer a lot of functionality with minimal code.

#### Main type:

```Julia
type JTensor{T, N, B, C} <: DenseArray{T, N}
    buffer::B # GPU buffer, allocated by context
    size::NTuple{N, Int} # size of the array
    context::C # GPU context
end
```

#### Scope

Current backends: OpenCL, CUDA

Planned backends: OpenGL, Vulkan

Implemented for all backends:

```Julia
map(f, ::JTensor...)
map!(f, dest::JTensor, ::JTensor...)

# maps
mapidx(f, A::JTensor, args...) do idx, a, args...
    # e.g
    if idx < length(A)
        a[idx+1] = a[idx]
    end
end


broadcast(f, ::JTensor...)
broadcast!(f, dest::JTensor, ::JTensor...)

```


CLFFT, CUFFT, CLBLAS and CUBLAS will soon be supported.
A prototype of generic support of these libraries can be found in [blas.jl](https://github.com/JuliaGPU/JTensors.jl/blob/sd/glsl/src/blas.jl).
The OpenCL backend already supports mat mul via `CLBLAS.gemm!` and `fft!`/`ifft!`.
CUDAnative could support these easily as well, but we currently run into problems with the interactions of `CUDAdrv` and `CUDArt`.


# TODO / up for grabs

* mapreduce (there is a first working version for cudanative)
* stencil operations
* more tests
* tests, that actually only switch the backend but use the same code
* performance improvements!!
* implement push!, append!, resize!, getindex, setindex!
* interop between OpenCL, CUDA and OpenGL is there as a protype, but needs propper hooking up via `Base.copy!` / `convert`
* share implementation of broadcast etc between backends. Currently they don't, since there are still subtle differences which should be elimated over time!
