# It's Dangerous to GC alone. Take this!

GC OMR Talk

1988: VisualAge SmallTalk
1993: J9 JVM
2016: OMR

Language runtime that can be shared against many languages
shared backbones between many different language runtime
battle-tested with J9

---

GC Book: 「ガベージコレクションのアルゴリズムと実装」
Read the source: gc.c
File is created at 1993

----

Object pointers

Direct Object pointers
Objects in Ruby cannot move

Fixed sized objects
what about variable size objects: objects mallocs additional space

- heavyweight allocation
- malloc fragmentation
- poor cache locality

incremental GC
alleviates long sweep time by runnning GC 

----

goal

- make allocations faster
- improve GC time
- improve mutator time
- 100% compatibility

----

start:
single-threaded mark-sweep

use OMR to allocate/free

-> needed to support conservative collection

----

let's make it parallel

multithreaded GC -> already in OMR

marking: scan roots in parallel, break VM roots into subsets, complete marking in parallel
some changes in internal Ruby API

sweeping: free malloc space in parallel, clean up objects in parallel, move work out of finalization

----

OMR Tools

GC memory visualizer for Ruby MRI with zero changes to the tool
Health Center for Ruby MRI: Live GC Visualization - GC from a running application
Method profiling w/ health center

----

How can we make CRuby faster?

-> Allocations!

making allocations fast

thread-local heaps
better cache locality, aloocate w/out locking

----

-> Let's look at off-heap memory!

malloc/free are expensive
rely on system for memory management concerns

new built-in type OMRBuffers
variable size object
allocate all buffers on the heap as buffers
They don't need to be freed
shorter sweeping with OMRBuffers

----

OMRBuffer
eliminates obj_free()
Fast allocations
Better cache locality

Challenges:
Heap fragmentation
Conservative collection
Concurrency

----

Multithreaded allocations

MRI does some work in background threads
use OMRBuffers in background tasks
Introduce finer grained locking than the GVL
allows for:
  multithreaded allocations
  GCing from a background thread

----

"Is Ruby Fast Yet?" benchmark: 8% improvement

----

Experimenting with non-copying generational GC
Marking is already fast
to tackle heap fragmentation issues

segregated heap in MRI
divided heap into different regions
bounds maximum heap fragmentation

concurrent GC in MRI
Ruby threads incrementally mark
background threads marks

----

what's next

Balanced GC algorithm - takes advantage of bery large heaps
Semispace copying generational collection - + cache locality, + less sweep time; - objects can't move in Ruby now

Open source OMR
don't want to fork Ruby and make IBM ruby
