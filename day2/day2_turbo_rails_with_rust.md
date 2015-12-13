# Turbo Rails with Rust

Godfrey Chan, Yehuda Katz

----

(+) High-level, predictable, dynamic, OOP. metaprogramming
(-) slow

machine learning, number crunching -> some do it on python, most do it in C

C
(-) lowlevel, imperative, dangerous
(+) faster, closer to the metal

native extensions : best of both worlds*
JSON::Pure, JSON::Ext

*-> for the end-user
for the author of extensions, you have to write C anyways, low-level, dangerous....

String#blank? that shows up on profiler alot
unfortunately, it is used unexpectedly and becomes costly

(GH)SammSaffron/fast_blank
up to 20x faster

----

Why not use Rust?

----

developer experience <-> performance

joined TC39 (that makes JS)

joined Rust core team
motto: "Hack without fear!"

many people want to write low-level code...

----

Structs

basic unit of data in Rust

```
struct Circle {
  radius: f64
}

impl Circle {
  fn new(r: f64) -> Circle {
    Circle { radius: r}
  }
}

let c = Circle::new(10.0)
```

type inference system

static dispatch
  static dispatch allows function inlining
  primary benefit of inline expansion is to allow further optimizations
  "gatekeeper optimization" that unlocks other optimizations

Allocation
  Heap allocation
    in Ruby, all your values will be alloc'd on Ruby
    you need to free it, but you cannot free it if others have reference to it
  Stack allocation
    compiler finds out how many space it needs and saves space for them automatically
    when you're done, you can just pop that stack frame, all the values go away
    stack pointer handles freeing memory
    in Ruby, it is an optimization, you cannot rely on it
    but:
      the exact size for each function must be known at compile time
      it's dangerous to give out pointers to the stack
    Rust ownership makes stack allocation safe

Traits and Generics : polymorphism

```
trait Shape {
  fn area(&self) -> f64;
}

impl Shape for Circle {
  fn area(&self) -> f64 {
    PI * self.radius.powf(2.0)
  }
}
```

like Refinements in Ruby

---

Back to fast_blank

In C -> alot of lines
In Rust -> one-liner! looks like it's slow, but it's faster than C code!

---

There is a fineprint

-> Glue code

function is one-liner, but a lot of glue code to expose it to Ruby

the glue code is generic

libcruby in rust
-> take out all the glue code

----

trb files (trb: turbo ruby)

```
class String
  def blank? -> bool
    <--RUST
      self[..].chars().all(|c| c.is_whitespace())
    RUST
  end
end
```

trb + libcruby

----

Turbo Rails with Rust?

----

Shared of Mutable, Pick one

- Channels: Forbid Aliasing
- Functional: Forbit Mutation

-> it's a whole programming-model thing

Rust: Forbid both at the same time