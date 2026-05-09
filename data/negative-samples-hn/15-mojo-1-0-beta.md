# Mojo 1.0 Beta

Source: https://mojolang.org/
HN score: 359

Mojo

IMPORTANT: To view this page as Markdown, append `.md` to the URL (e.g. /docs/manual/basics.md).
For the complete Mojo documentation index, see 
llms.txt
.

Skip to main content

Install

Docs

Packages

Releases

Community

GitHub

1.0.0b1

Nightly

1.0.0b1

Search

For the complete Mojo documentation index, see 
llms.txt
. Markdown versions of all pages are available by appending .md to any URL (e.g. /docs/manual/basics.md).

Write like Python, run like C++.
Write fast code for diverse hardware, from CPUs to GPUs, without vendor lock-in, in a language that's both user friendly and memory safe.

Install Now

Stable: 
1.0.0b1
 (
May 7
)

 | 

Latest nightly
 
May 9

Quickstart
Learn the basics in 5 minutes

Releases
See what's new

Roadmap
See what's coming next

Contribute
Explore Mojo on GitHub

Built different

Modern
Mojo draws inspiration from the best parts of modern languages - like Python's intuitive syntax, Rust's memory safety, and Zig's powerful and intuitive compile-time metaprogramming.

AI native
Mojo is built from the ground up to deliver the best performance on the diverse hardware that powers modern AI systems. As a compiled, statically-typed language, it's also ideal for agentic programming.

Simply performant
No more choosing between productivity and performance - Mojo gives you both. You can start with simple and familiar programming patterns, and add complexity as you need it.

GPU programming
Mojo makes GPU programming accessible to everybody, without vendor-specific libraries and without separately-compiled code. You can finally write high-performance GPU kernels in the same language you use for CPUs.

def

 

vector_add

(

    a

:

 TileTensor

[

float_dtype

,

 type_of

(

layout

)

,

 element_size

=

1

,

 

.

.

.

]

,

    b

:

 TileTensor

[

float_dtype

,

 type_of

(

layout

)

,

 element_size

=

1

,

 

.

.

.

]

,

    result

:

 TileTensor

[

        

mut

=

True

,

 float_dtype

,

 type_of

(

layout

)

,

 element_size

=

1

,

 

.

.

.

    

]

,

)

:

    

var

 i 

=

 global_idx

.

x

    

if

 i 

<

 layout

.

size

(

)

:

        result

[

i

]

 

=

 a

[

i

]

 

+

 b

[

i

]

Python interop
Mojo natively interoperates with Python so you can eliminate performance bottlenecks in existing code without rewriting everything. You can start with one function, and scale up as needed to move performance-critical code into Mojo. Your Mojo code imports naturally into Python and packages together for distribution. Likewise, you can import libraries from the Python ecosystem into your Mojo code.

# SIMD-vectorized kernel squaring array elements in place.

def

 

mojo_square_array

(

array_obj

:

 PythonObject

)

 raises

:

    

comptime

 simd_width 

=

 simd_width_of

[

DType

.

int64

]

(

)

    ptr 

=

 array_obj

.

ctypes

.

data

.

unsafe_get_as_pointer

[

DType

.

int64

]

(

)

    

def

 

pow

[

width

:

 Int

]

(

i

:

 Int

)

 unified 

{

mut

 ptr

}

:

        elem 

=

 ptr

.

load

[

width

=

width

]

(

i

)

        ptr

.

store

[

width

=

width

]

(

i

,

 elem 

*

 elem

)

    vectorize

[

simd_width

]

(

len

(

array_obj

)

,

 

pow

)

Compile-time metaprogramming
Mojo metaprogramming uses the same language as the run-time code, providing an intuitive system to maximize performance. You can build hardware-specific optimizations with conditional compilation, ensure memory safety with compile-time evaluation, eliminate costly runtime branches, and more—all with clearly defined intentions and zero-cost abstractions.

# Generic struct equality using compile-time reflection.

@always_inline

def

 

__eq__

(

self

,

 other

:

 Self

)

 

-

>

 Bool

:

    

comptime

 r 

=

 reflect

[

Self

]

(

)

    

comptime

 names 

=

 r

.

field_names

(

)

    

comptime

 types 

=

 r

.

field_types

(

)

    

comptime

 

for

 i 

in

 

range

(

names

.

size

)

:

        

comptime

 T 

=

 types

[

i

]

        

comptime

 

assert

 conforms_to

(

T

,

 Equatable

)

,

 

"All fields must be Equatable"

        

if

 trait_downcast

[

Equatable

]

(

            r

.

field_ref

[

i

]

(

self

)

        

)

 

!=

 trait_downcast

[

Equatable

]

(

r

.

field_ref

[

i

]

(

other

)

)

:

            

return

 

False

      

return

 

True

Roadmap

Mojo was born in late 2022 and has come a long way, but there's still a lot to do.

Phase 0

Initial bring-up

Implementing the core parser, defining memory types, functions, structs, initializers, argument conventions, and other language foundations.

Phase 1

in progress

High performance CPU + GPU coding

Making Mojo a powerful and expressive language for writing high-performance kernels on CPUs, GPUs, and ASICs, while empowering developers to extend Python seamlessly.

Phase 2

Systems application programming

Expanding Mojo to support more application-level programming, with a guaranteed memory-safety model and more abstraction features that developers expect for systems programming.

Phase 3

Dynamic object-oriented programming

Supporting more of Python's dynamic features like classes, inheritance, and untyped variables to maximize compatibility with Python code.

For more details, see the 
Mojo roadmap

Open source

The Mojo standard library is fully
 
open-source on GitHub
 
and we welcome contributions! We also plan to open-source the Mojo compiler in 2026.
We're committed to open-sourcing all of Mojo, but the language is still very young and we believe a tight-knit group of engineers with a common vision moves faster than a community-driven effort.
If you'd like to get involved,
 
join our developer community!

Get started

Install

Quickstart guide

Stable: 
1.0.0b1
 (
May 7
)

 | 

Latest nightly
 
May 9

Learn Mojo

Beginner tutorial
Learn Mojo by building the Game of Life.

GPU puzzles
Learn GPU programming with Mojo by solving puzzles.

Intro to Mojo
Read all about all the Mojo language features.

Join the community

Developer forum
Ask questions and stay up to date on Mojo ongoings.

Events
See upcoming events, meetups, talks, hackathons, and more.

Contributions
Browse open issues, help with docs, and share your projects.

Releases

Roadmap

License

Contributing

Install now
