# Ruby3 Challenges

Anger

Larry Wall "3 Virtues of a Programmer"

- Laziness
- Impatience
- Hubris

(怠惰、短気、傲慢)

- laziness: work to save labor
- impatience: angry when computer is lazy
- hubris: the quality that makes you write programs that other people won't want to say bad things about

-> all related to anger. "controlled" anger is good -> source of motivation. but keep inside of you because anger is infectious. So is niceness.

Niceness is Ruby community's greatest virtue

"MINASWAN"
but I am not an inventor of MINASWAN.
Martin Fowler invented it.

----

Recent news

Ruby 2.3.0 will be released on coming Christmas. Preview2 is out.

"Event Driven Programming" -> driven by RubyKaigi, added alot of patches

"Did-you-mean" gem is added

Enumerable#grep_v : grep -v (choose unmatched. -v is for inVerse)

Hash#fetch_values : Strict version of Hash#values_at. If key does not exist, throw exception

Numeric#positive?, negative?

Hash comparisons (包含関係)
必ずしも全順序じゃない。 <=> は存在しない。Not Comparable.

Hash#to_proc

```
h = {1:2, 2:4, 3:6}
(1..3).map(&h)
```

keyに対するvalueを返す関数とみなして

Array, Hash, Struct#dig

```
data[:users] && data[:users][0][:matz]
data.dig(:users, 0, :matz)
```

Indented here document
take indentations out to the shortest indentation

Frozen String Literal
all strings in a current file become frozen. Performs slightly better, able to find errors little bit quickly

```
a = "abc"
a << "cde" # error!
```

Now we can safely ignore "freeze pull-req"

Safe navigation operator
Similar to `?.` in Swift/Groovy
when receiver is nil/none, ignore the rest
in Ruby, `&.`
it used to be `.?` -> question mark is a legal character for a method name
`.?` causes alot of confusion (ex. when making frontend in Swift and backend in Ruby)

`&.` -> came from connecting `&&`
Lonely(bocchi) operator

```
u && u.name && u.name.first
u&.name&.first
```

5-10% faster each year since 2.0
（普通預金・定期預金の金利より高い！）
It's a community effort

----

mruby 1.2

Fixed bunch of bugs
Better build process
More stable

----

Streem

github.com/matz/streem

uploaded README to GitHub as a backup -> buzzed!
someone wrote an interpreter -> worked without writing code!!

----

OCaaS (made it up just right now)

OSS Community as a Shark
we have to keep moving forward to attract the community

Change is cost, change is pain
we like changes, we don't like pain

design is difficult -> conflicting demands
we don't know what we should make

Make something people ~~want~~ need!!
(Y Combinator's motto: Alan Kay fixed the last word)

We shouldn't make what people want.
Henry Ford "If I had asked people what they wanted, they would have said: Faster horses..."

Make what changes their lives
Create something that have never existed
Design is hard because the future is unknown

in 1995, some said we don't need OO for scripting
but they were wrong!

I didn't ask them what they want. I created what I wanted to see in the future. Some didn't understand, I didn't care.

Ruby became new normal


It is a good excuse to throw away old code (to make changes)

When change stops, the system eventually dies

Unfinished system, slow adaptation, ... => We live in a painful world
Programming languages are no exception
Languages live longer

YARV, multilingualization -> 1.9.0
compatibility problems
took 5 or more years to be adapted (made 3 years to create)
We've done slightly better than Python3
No one suggested to give up at least

Replaced object representation
Replaced garbage collector
keep compatibility as much as possible
Prepared migration path
Never tried too drastic changes (one that breaks 80% of ruby programs)
versioning illusion : Python - 2.0 vs 3.0, Perl - 5.0 vs 6.0 => Ruby 1.8 vs 1.9, 1.9 vs 2.0
Give "migration bait" - performance

Lessons learned:

1. Don't throw away everything
2. Don't push (users/changes) too hard
3. Don't break compatibility for no reason
4. Provide user benefit

2.0 had (almost) perfect compatibility

----

Situation changes:

- Multi-core
- Code scalability
- Data scalability

-> we want to address those challenges

Ruby3

but we don't forget the rules

no concrete ideas yet.

----

concurrency
man-machine collaboration

Ruby way: concurrency should be abstract, simple and easy-to-use
several candidates

- actor model: Erlang, Elixir, Scala, goroutine+pipe
    - Thread+Mailbox
    - shared-nothing is difficult
- ownership model: rust
    - use ownership to reclaim memory
- STM(software transactional memory): Clojure
    - Insanely difficult to impl
- Stream model
    - (snip) 
    - Pipeline operator |>
    - Rube Goldberg Programming / OK GO Programming

Event loop runs in the 2nd VM, no GIL
not covers 100% concurrent programming

----

man-machine collaboration

did-you-mean gem

soft-typing
flow-based type inference (Flow / Crystal)
Success Typing (Erlang - Dialyzer)

----

performance

Ruby3x3
Ruby 3 will become 3x faster than Ruby 2!!! (hopefully)

- concurrency
- better DS&Alg
- JIT compiler
- ahead-of-time compiler
- and whatever

Tricks and conditions

Ruby 2 = Ruby 2.0
small but not too artificial benchmark ( we will decide )
It may consume more memory to gain performance
It's minimum memory consumption should be comparable to Ruby2.3

「某heroku社のdynoとかちっちゃくてですね」

We need more power

Heroku, Appfolio, IBM J9? ... "Going to contribute to CRuby"