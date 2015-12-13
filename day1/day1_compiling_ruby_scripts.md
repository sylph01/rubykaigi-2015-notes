# Compiling Ruby Scripts


---

ISeq(instruction sequence) tree
basically, each scope has its own iseq

binary format
iseq, id, objects are pointed by index of each lists in each data
objects are serialized manually
dump machine dependent data
no verifier(not for migrations)

----

technique

lazy loading
do not load bytecode at once
load bytecode if needed

(C1, m1: class, method)
(1) load and make an empty toplevel iseq
(2) load toplevel iseq and make empty C1, C2 empty iseq
(3) load C1 and evaluate C1 define m1 and m2 with empty ISeqs
(4) load C2 and evaluate C2

not all classes/methods are loaded

----

interface api and tools

how to store compiled binary?
compile timing
  use compiler explicitly
    C/Java
  loading time
    Rubinius, Python
  -> which is better? tradeoffs exist, undecidable
location of compiled binary
  a file in a same directory of *.rb files
  a file in a special directory
  DB

Current implementation
RubyVM::InstructionSequence.load_iseq
  call this method at every loading time if defined
  this method should return or loaded iseq object or nil
  you can write your own loader
ISeq#to_binary
ISeq#load_from_binary

New require/load process
Load_internal(fname)
Call ISeq.load_iseq(fname) -> ISeq.load_iseq(fname) -> If exists, get ISeq, if not, do normal process

When should we compile?
  Use compile explicitly
    Gem install timing is a good idea
  Loading time
    Matz doesn't like it

Where to store?
  sample/iseq_load.rb provides 3 type of repository
    same directory
    specified directory
    dbm

ruby iseq_load.rb [file / dir]
  compile, serialize, store specified file(s)
ruby -r iseq_load [script]
  enable loader
  load stored files if possible
setting by environment variables


---

note: experimental

---

future work
reduce memory consumption
reduce binary size with some techniques
and more...
