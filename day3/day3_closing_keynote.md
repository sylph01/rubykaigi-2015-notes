# Closing Keynote / Ruby:2020 - How do we get to Ruby3x3

Evan Phoenix

---

Very technical. "I should lose everyone at some point."

Rubinius, Puma, benchmark-ips

----

- Hacked a truly parallel Ruby 1.8.2 (Sydney)
   - added native threads
- fully parallel ruby impl (Rubinius)
- Built a dynamic optimizing JIT with LLVM

10yrs since release of Sydney
500KB patch (against Ruby head)

----

"Ruby3 will be 3 times faster than Ruby2"

-> How do we do this?

Goal: CRuby Needs a JIT
We will not get to Ruby3 without one. Now is the time to build it. We must be pragmatic with our approach.

---

Part 1: Performance

- Run the same amount of code, but in parallel
- Run less code

----

Parallelism

- With CRuby, harder than running less code
- Core library not designed to run concurrently
- To avoid this problem, need to add "strange" mechanisms

----

"Strange" parallelism

- Actors with unique heaps
- Interesting program construction (Streams)
- Forces users into unfamiliar places

----

Not small changes

touch everything in library? confuse users by bringing in concepts?

----

How can we make it faster?
-> Micro-optimizations

Lots of talks about them here
Micro-optimizations are great, but it is very hard to get to 3x with them

Easy fixes

Missing caches
grep rbfunccall *.c => 364
all of those are slow. every one of them can use cache.

----

Macro-optimizations

we need big jumps all over
big ideas can lead to big jumps

----

JIT : Runs less code

caches - understand how the code works
knowledge to remove dynamic aspects
ruby is no more dynamic than smalltalk or java

Ruby is dynamic, right?
-> 90%+ of all method calls are monomorphic

---

Zen and the Art of Dynamic Language Performance

Part 2: survey of the past

SELF
Dave Ungar
Born in 1987
Required a JIT compiler from the very beginning
Uniform method dispatch and prototype based
Remove all the dynamic features with the JIT

polymorphic inline caches
drove compiler decisions from the caches
extensive method inlining

SELF JIT inlining
without JIT, it would have been far too slow
inlining blocks
inlining primitives

foundation for all dynamic language optimization
dynamic languages can be fast

Strongtalk
Dave Griswold, Urs Hölzle, Lars Bak

Types NOT introduced to improve performance. Only to make developers have a better understanding of the code. Types made it slower. Type checks / guards ran at runtime.

Inline Caching, Method Caching

1997, Sun bought the company -> Team retrofit their engine to run Java -> HotSpot VM was born

Gradual typing is possible, it doesn't improve performance
SELF research can be used to improve other languages

V8
JavaScript engine for the Chrome Browser
Started by Lars Bak
(rumor: Lars' 8th dynamic language VM?)

fast property access w/ hidden classes
not storing anything as hash tables
tiered compilers

JS has to run fast, users expect instant feedback

Survey Results
JIT technology is the driver of performance
even Java wouldn't have been mainstream w/out JIT

----

JIT compiler types

- tracing
- method JIT
- partial evaluation

Tracing
triggered when a place in the code is run enough
(mostly top or bottom of the loop)
records operations until large enough
straightline code without ifs

problems of tracing
exponential number of code paths (-> C3/C4 of taking coverage)
huge number of traces, trace management is important

Method JIT
triggered when a method is run many times
  also triggered by a loop in the method running

bigger the method, longer the compile time
inlining makes methods even bigger!
compiled code only used on next method invocation
  unless advanced techniques (On-Stack Replacement) used
  
Partial Evaluation
old technique, new application
well known is the Truffle+Graal

slow startup
(not used much, not much literature to learn from)

Industry work
Original Mozilla engine was tracing
LuaJIT, PyPy

Compiler performance
the wider the horizon(how much the compiler can see), the faster the code becomes

The Primitive Problem
Ruby code is slower than C code
Developers using core library extensively for performance
C is blackbox to Ruby

Proposal: The Canary (as the canary in the coalmine)

The Canary -> [1,2].min
any sufficient compiler should be able to turn it to 1
to do it in CRuby it is very difficult

"The definition of .min is in C! oh no! 💩"

----

Part 4: Comparison

Rubinius
Design a Ruby implementation from scratch
Still not fast enough

JRuby
allows user to take advantage of the JVM
JVM is not for everyone

Comparison: Simple JIT
convery YARV bytecode to machine code

str.split("\n").map {|i| i.strip }
-> bunch of indirect calls
  - remove bytecode dispatch
  - can't see through rb_funcall!

---

Part 5: A Proposal

Ruby core library is important
Efforts to rewrite it have not been accepted by ruby core
Work with the core library

-> Lift the core

Convert the Ruby core library to LLVM bitcode using Clang
(intermediate format of C code)
Generate LLVM IR from YARV bytecode
Allow LLVM to optimize them together

C methods stop the ability to optimize

LLVM can see inside the core library

----

Proof

(demo)

----

Removing dynamism

Converting rb_block_call and rb_funcall to target method/function

Easy on developers
tooling automatically adds helper code
reuse all of current core library

----

MIPASWAP

Matz is pragmatic, and so we are pragmatic

this one does not require much research
the idea is understood, most of the work is done by LLVM

we've got all the components, we need the will to do it now

-----

Conclusion

3x is hard
CRuby must remove dynamic parts at runtime to improve
a JIT is the best hope

