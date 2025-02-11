---
title: "Rust FFI to Julia"
author: "Cristian Bourceanu"
date: 2025-02-11T21:57:00Z
categories: [rust, ffi, julia]
---

== WIP ==

# Rust <-> Julia interface

While I have grown to appreciate the differences in applications for these 2
languages, these are not mutually exclusive and many times the stack or pipeline
of components that compose one application might cross language boundaries.

These might be because one language has much better support, usually in terms of
libraries available, but also in robustness a complex library.

While these 2 language are not truly compatible, some tricks can be added to
ease translation, especially when it comes to the generics `Option` and `Result`
in Rust. There are alternatives for these in Julia, even though by default Julia
does not come equipped with the same paradigm:

- [ResultTypes.jl](https://github.com/0x0f0f0f/ResultTypes.jl)
- `Option` is generally used as `Union{T, Nothing}` in Julia

## Existing solution

You can call Julia from Rust, using the exposed C FFI for Julia as leveraged by
the [jlrs](https://github.com/Taaitaaiger/jlrs) crate, but we are also
interested about the reverse.


## Rust ABI 

There is currently no agreed way even for calling dylib Rust <-> Rust, known as
an ABI, this is because the arguments are not assured to have any layout in
memory, they are decided arbitrarily when compiled:

[rust abi_stable crate](https://lib.rs/crates/abi_stable) 🤣:
> the compiler is free to do whatever it pleases with these aspects of your
> software: the process by which it does that is explicitly unstable, and
> depends on your compiler version, the optimization level you selected, some
> llama's mood in a wool farm near Berkshire... who knows?

### Standard strategy: `extern "C"`

The standard way to call Rust through a FFI is translating it into to expose a C
interface and reuse the system C-ABI. This however causes to drop many of the
guarantees Rust tries to make and induces a huge effort boundary of developing
software that empowers these two languages to talk, since the translation effort
is required two folds as boilerplate glueing code that doesn't add any
value.

## Interim solution to ease integration

=> Develop `RustCall.jl` library similar to `PyCall.jl` which can use cargo
packager to build Rust pacakge and generate the necessary C-FFI shims for the
two sides as follows:

1. Construct a SAT of the public interface of the Rust library
2. Limit the scope of supported types to be translated as follows:
    - Rust::`Result<T, dyn Box<Error>>` <-> Julia::`ResultType{T}`: indirect C-FFI WIP workout common
      representation
    - Rust::`Option<T>` <-> Julia::`Union{T, Nothing}`: simply use C++ union
    - Primitives
        - integer and floats can be directly converted
        - `char*`, `CString`, `CStr`, Rust::`String`, Rust::`&str`,
        Julia::`String`, Julia::`SubString` => These need special attentions
    - enums: ==WARN Julia does not have a first class enum type==, but it
    depends on the global constant declaration similar to C. On the other side,
    Rust::`enum` is more similar to a Julia::`Union`. Also special care needs to
    be taken as a common design pattern in Julia is to do multiple dispatch on
    type hierarchies, whereas Rust dispatches over traits.
    - structs: Every Rust struct must be converted to a C repr struct in order
      to be exposed through the FFI. The same problem seems to occur with Julia
3. While in Julia or structs are considered to be passed by reference, Rust
   defines, clone, immutable ref or mutable ref.
   - WIP: Write some simple tests to understand how structs behave in Julia, but
     from my observations so far, it's either immutable or mutable ref based on
     `struct` definition rather than function definition.


