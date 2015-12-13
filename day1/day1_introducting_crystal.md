# Introducting Crystal

Will Leinweber @ Heroku
bitfission.com

----

## What is Crystal

compile language
uses LLVM
completely static types, static dispatch
type system is very nice to use, still feels like using Ruby

resulting programs are very fast
very easy to link against C libraries

----

2012-09-04 : first public commit
2013-11-14 : Self hosted
2013-12-05 : Impld Boehm GC

2014-06-19 : First release

2015-10-30 : current release

----

## similarities

```
(1..5).to_a
.sort {|a,b| b<=>a }
.reject {|a| a.even?}
```
works the same in Ruby and Crystal

classes, initialize method, no need for explicit return

open classes (even the standard library!)

modules, extends/includes

do_something unless you_shouldnt? (same syntax = same idioms)

specs

overall structure is similar because looks so much the same
you can take the experience with you

----

## differences

In Crystal removed all aliases
only size, only map

single quotes are characters, double quotes are strings
strings are immutable

instead of `attr_accessor`, `property`

`crystal tool hierarchy` : you can look up how much size the objects are going to take

Symbol.to_proc is a little bit different

```
%w[apple bat cat].map(&.reverse.upcase)
```

### type system

```
def double(i)
  i * 2
end

answer = double(3)
typeof(answer) # => Int32

answer2 = double("hi")
typeof(answer2) # => String
```

Both int and string has 

```
def validate(i)
  if i > 10
    i
  else
    "too low"
  end
end

a = validate(15) # => 15
typeof(a)        # => (Int32 | String)

a * 2 # works
a + 15 # fails
```

one way:

```
a = validate(15)
(a as Int32) + 2
```

### overloading

```
def foo(a : Int, b: Char)
end

def foo(a : Bool, b : Char)
end
```

### abstract

hierarchy -> @pet : Animal+ (anything that inherits animal and the animal itself)

need "abstract" to say that the class can't be instantiated

### macro

defined at compile-time

### linking

9 lines is all you need to link libpq into your Crystal program
minimal postgres driver

### compiling

--release flag: does LLVM optimizations