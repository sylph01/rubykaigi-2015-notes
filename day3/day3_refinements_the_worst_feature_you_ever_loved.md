# Refinements - the Worst Feature You Ever Loved

Why refinements?

you can open a class and define a method
open class, dynamic class scope, monkeypatching

three use cases

1. DSL
2. convenience methods
    - `1.hour + 20.minutes # => 4800 seconds`
3. method wrappers

```
class String
  alias_method :old_length, :length
  
  def length
    old_length > 5 ? "long" : "short"
  end
end
```

everybody is relying on your length method, so it breaks other code

monkeypatching is evil because it affects things globally

---

refinements

define refinements, then use it

```
module StringExtensions
  refine String do
    def shout
      ...
    end
  end
end

module ModuleThatUseTheRefinement
  using StringExtensions
  "hello".shout
end
```

active from right after the `using` and to the end of the module

top-level: right after the `using` to the end of the file

```
code = 'using StringExtensions; "hello".shout'
eval code
```

what happens:

```
class C
  using Ext
  "hello".shout
end

class C
  "hello".shout
end

class D < C
  "hello".shout
end
```

looks like it's going to work...?

```
C.class_eval do
  "hello".shout
end
```

re-entering the scope of the class. is the refinements active now?

"dynamic scope"

they are supposed to work

----

refinemet gotchas

1 - confusing code

```
SomeClass.class_eval do
  add(1, 1) # => 2
end

SomeOtherClass.class_eval do
  add(1, 1) # => "11"
end

using FloatPointMath
add(1, 1) # => 2.0
```
(SomeClass is not refined, SomeOtherClass is refined)
!?

2 - security risks

3 - Performance issues

looking through the chain is expensive
so the interpreter caches it
sometimes it needs to be invalidated

4 - corner cases

pry is doing an eval

----

dynamically scoped refinemets

- they fix monkeypatches

----

the bad

- make ruby code potentially confusing
- they might impact security
- impact performance
- corner cases

----

refinements today

```
C.class_eval do
  "hello".shout #=> NoMethodError
end
```

Lexically Scoped refinements

```
def add(x, y)
  x + y
end

SomeClass.class_eval do
  add(1, 1) # => 2
end

SomeOtherClass.class_eval do
  add(1, 1) # => 2
end

using FloatPointMath # refines Fixnum#+
add(1, 1) # => 2
```

FloatPointMath refines +, so it doesn't affect the + in the add(x, y)

----

three use cases:

DSLs -> no. unless you write `using` on top of every file
convenience method -> no. if you want them, write refinement on each and every file
Rails, Rspec authors didn't want to do it
big libraries are not doing them
method wrappers -> write using explicitly

----

lexically scoped ref: the bad

- they don't replace monkeypatches
- they still have weird corner cases

---

- they do fix monkeypatching cases
- they don't generally make the code confusing
- they don't impact security

---

the deep problem

language design is a deep problem

