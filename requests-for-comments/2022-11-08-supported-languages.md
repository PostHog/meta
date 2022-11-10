# Request for comments: Supported languages at PostHog

Since day 1 here at Posthog we've supported two and half core languages: Python and Javascript/Typescript.

This set of tools has carried us a long way and we should both the cpython runtime and node a huge hats off for bringing us this far.

## Problem statement

We have the opportunity coming up to rebuild or greenfield build out services that are critical parts of our data pipeline. It will be important for these services to be correct, fast, and efficient. Considering this now is a good time to ask: Are we using the correct tools for the job?

So why even bring this up?

Frankly:

- Python is slow.
- Node is a memory hog.
- Node tooling is slow.
- Dependencies can be huge.
- No guarantees that code is correct.

Can we be successful continuing to use these languages? Of course.

- They are well understood.
- Our tooling is setup to support these languages
- We have internal expertise.
- Generally they are _good enough_.

Why should we be open to other languages?

- We have different people with experience working with different languages.
- 100% guaranteed static typing is quite beneficial
- Compiled languages provide more confidence shipped code is correct
- There are compiled languages that provide significant efficiency wins on CPU, Memory, and performance
- Using the right tool for the job is typically the right thing to do (if you can ship it)

Candidate Languages:

- [Golang](https://go.dev/) (big surprise)
- [Rust](https://www.rust-lang.org/) 🦀
- [OCaml](https://ocaml.org/)
- [Elixir](https://elixir-lang.org/) (dynamically typed)
- [Scala](https://www.scala-lang.org/)
- [Java](<https://en.wikipedia.org/wiki/Java_(programming_language)>)

## Meet the eligible languages

### Golang

The good:

- Extremely easy to learn. Most engineers can learn and be productive in about a day.
- Built to remove most contentious parts of development
- Designed to make engineering easier
- Built for network services and concurrency
- Great standard lib
- Super fast compile times
- Light on memory
- Used by plenty of organizations big and small
- We have plenty of Gophers here.

The bad:

- Most people would not say the language sparks joy (I disagree)
- Not terribly expressive
- Verbose, but easy to read

### Rust

The good:

- Very hot right now
- Very good for when you need to hyper optimize something but would like to avoid C or C++
- Extraordinarily expressive and fun to program
- Extremely performant and light on memory
- Used by plenty of organizations big and small
- We have a few Crustaceans here!

The bad:

- Slow compile times
- Harder to read because of how expressive
- Slower ramp up time to become proficient in as an Engineer

### OCaml

> OCaml is a general-purpose, industrial-strength programming language with an emphasis on expressiveness and safety.

You may not be familiar with this one but it is _very_ safe and performant. Companies that have huge amounts of money on the line who are risk adverse to defects and slowdowns, like the HFT firm Jane Street use OCaml because it is unforgiving in how type safe it is.

> OCaml’s powerful type system means more bugs are caught at compile time, and large, complex codebases are easier to maintain. This makes it a good language for running critical code. At the same time, sophisticated inference makes the type system unobtrusive, creating a smooth developer experience.

The good:

- Garbage collected
- Algebraic data types
- Pattern matching
- Type inference
- Immutable
- Static Type-checking
- First-class functions
- Parametric polymorphism
- Used by serious programming shops

The bad:

- Not terribly popular relative to the others
- Can be considered relatively academic
- No one experienced here

### Elixir

Oh, Erlang <3

The good:

- Built for streaming data
- The cockroach of runtimes.
- Automatic function level clustering
- Hot reloading of functions
- Compiled
- Relatively functional

The bad:

- Dynamically typed

### Scala / Java

Grouping these together because they really are converging. Really this is any language on the JVM.

The JVM is truly a wonderful runtime. It's very fast. The hotspot detection + Just In Time Compiling is _super_ impressive.

The good:

- Surprisingly performant
- JVM Library interrop means huge availability of libraries out there
- Tons of expertise and companies big and small are building software in this
- Kafka and Zookeeper are built on this

The bad:

- Considered somewhat crusty
- Slow boot times
- JVM tuning is no fun

## Discussion point

So with this overview laid out let's talk about

## Success criteria

_How do we know if this is successful (i.e. metrics, customer feedback), what's out of scope, whats makes this ambitious?_

The goal here is to spark debate about languages that we should use here at Posthog. What are we open to? Why should we not adopt new technologies.

Success here would be to make a decision and effectively enact a policy on this and have engineers aligned and not worry about this again (at least for some time)
