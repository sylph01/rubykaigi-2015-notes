# Experiments in Sharing Java VM technology with CRuby

Matthew Gaudet @MattStudies @ IBM

Compilation Technology since 2008, JIT since last year

----

IBM J9

----

Why does IBM have a Ruby JIT?

IBM is interested in Cloud
Polyglot world out there -> want to support these languages
Let developers choose the right language for the job, rather than select by capabilities

How can we minimize duplicated effort?

The plan: OMR
Open source toolkit for language runtime
Annouced by Mark Stoodley at JVMLS 2015 https://www.google.co.jp/url?sa=t&rct=j&q=&esrc=s&source=web&cd=13&cad=rja&uact=8&ved=0ahUKEwjlrqyHhtPJAhXBFaYKHSPXDrwQtwIIQzAM&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DkOnyJurioyw&usg=AFQjCNHtksPUliD95jJMR1u4N7yxahskLA&sig2=7f3GGGP2DnyDb--b_cTu_g&bvm=bv.109910813,d.dGY

Goal: compatibility!

JIT, monitoring, GC, ...

----

Why so quiet?

Sort of the joke answer: IBM.

"Future is already here, it's just not very evenly distributed"

IBM is getting better at open source
coming from a traditionally closed-source part of IBM
we are learning

Proving our proof of concept
we wanted to ensure we were happy with the concept

this is real technology
"I dont always re-engineer my runtime tech, but when i do, i do it in production" - the ibm runtimes team

----

Testarossa
dynamic language JIT for Java

Java C++ COBOL SystemZ -> SystemZ, POWER, X86, ARM

written in C++, 100+ optimizations

----

YARV interpreter
->
+ IL generator : Intermediate Language is created
-> 
+ Optimizer
->
+ Code generator
->
+ Code cache
->
YARV interpreter
-> 
+ Runtime
+ Profiler

+ : done in Testarossa

---

- functional correctness
- no big changes to how MRI works
- no restrictions on native code used by extension modules
- no benchmark tuning
- very simple compilation control

----

We can run Rails applications

-----

`vm_jit_init(vm. globals)` (inside void Init_BareVM(void))

----

Testarossa IL uses a tree-based representation
tree represents a single exp
treetop represents a statement

Ruby IL generation creates trees for each bytecode
complicated bytecode behavior leads to size expansion (one operation -> 100+ lines of C code)

our strategy:
mimic interpreter for maximum compatibility
implement simple opcodes directly in IL

build callbacks to VM for complex opcodes
  automattically generate callbacks from the instruction
fast-path particular patterns (e.g. trace)

don't try to handle everything: let interpreter handle hard cases!

based on 2.2.3
almost all opcodes supported
test-all, tested spree

----

Performance

microbenchmarks: doing okay
"production" benchmarks: some slowdowns
22.5% geomean improvement against interpreter only

----

onwards and upwards

challenges
- languages itself is highly dynamic
- unmanaged direct use of internal data structures by extensions
- complicated runtime, with many subtle details

opportunities to bring to Ruby
- speculative optimizations w/ decompilation and code-patching
  - class hiearchy based optim
  - guarded inlining
  - recompilation
- recompilation
- interpreter and JIT profiling
- asynchronous compilation
- more optimization (10/100+ today)


cycle of
- value/type propagation
- type specilization
- inlining

- profiling
- recompiliation

---

Ruby 3x3 -> IBM wants to help

we're not open source yet

goo.gl/P3yXuy

----

Long term:

OMR near-equal LLVM