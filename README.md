# Random Namespace and Functions

* Stage 0
* Authors: WhosyVox and Tab Atkins-Bittner
* Champions: Tab Atkins-Bittner
* Spec Text: currently this README

-----

Historically, JS has offered a very simple API for randomness: a `Math.random()` function that returns a random value from a uniform distribution in the range [0,1), with no further details. This is a perfectly adequate primitive, but most actual applications end up having to wrap this into other functions to do something useful, such as obtaining random *integers* in a particular range, or sampling a *normal* distribution. While it's not hard to write many of these utility functions, doing so *correctly* can be non-trivial: off-by-one errors are easy to hit even in simple "simulate a die roll" cases.

This proposal introduces a number of convenience functions for dealing with random values, to make common random-related use-cases easier to use. It groups all of these under a new namespace object - `Random` - for convenience and ease of understanding.

Goals:
* To introduce the lowest common set of functionality for random number generation
* To build a consistent and easily understood interface, free of ambiguity
* In time, to extend the API to the seeded-random-number proposal, making every function in `Random` available to the seeded PRNG


## Details

All functions will be available from a global `Random` namespace.
The functions are split up into three categories: single randoms, lists/arrays of randoms, and array-specified methods.

Every random number function is in the form [x,y) to be fully consistent.


### Single randoms
|Function           | Description|
|-------------------|------------|
Random.number()            | returns a random decimal value in the range [0,1) |
Random.numberBetween(x,y)  | returns a random value in the range [x,y)         |
Random.integerBetween(x,y) | returns a random integer in the range [x,y)       |
Random.boolean()           | randomly returns either true or false             |

### Lists of randoms
|Function                    | Description|
|----------------------------|------------|
Random.numberList( size )           | returns a `size` sized list of decimal values in the range [0,1)  |
Random.numberBetweenList( size, x, y )  | returns a `size` sized list of random values in the range [x,y)   |
Random.integerBetweenList( size, x ,y ) | returns a `size` sized list of random integers in the range [x,y) |
Random.booleanList( size )              | returns a `size` sized list of random boolean values              |

### array methods
|Function             | Description|
|---------------------|------------|
Random.pickFromList( array ) | returns a random element from [0,array.length) (can be undefined for empty/sparse arrays) |
Random.shuffle( array )      | performs an in-place random shuffle of the array                                          |
Random.asShuffled( array )   | returns a copy of the the provided array with the elements randomly shuffled              |

## Interaction With Other Proposals

It is intended that this proposal and the [Seeded Random proposal](https://github.com/tc39/proposal-seeded-random/) expose the same APIs. Either proposal can advance ahead of the other, however. This proposal is intentionally not touching seeded randomness, instead focusing on functions that are agnostic as to their random source.

## Prior Art

* [Python's `random` module](https://docs.python.org/3/library/random.html)
    * random float, between 0 and 1
    * random ints, with any min/max bounds (and any step)
    * random bytes
    * random selection (or N selections with replacement) from a list, with optional weights
    * random sampling (or N samples, *without* replacement) from a list, with optional counts
    * randomly shuffle an array
    * sample various random distributions: binomal, triangular, beta, exponential, gamma, normal, log-normal, von Mises, Pareto, Weibull
* [.Net's `Random` class](https://learn.microsoft.com/en-us/dotnet/api/system.random?view=net-8.0)
    * random ints, with any min/max bounds
    * random floats, with any min/max bounds
    * random bytes
    * randomly shuffle an array
* [Haskell's `RandomGen` interface](https://hackage.haskell.org/package/random-1.2.1.2/docs/System-Random.html)
    * random u8/u16/u32/u64, either across the full range or between 0 and max
    * <s>generate two RNGs from the starting RNG</s> (seeded-random use-case)
    * random from any type with a range (like all the numeric types, enums, etc), or tuples of such
    * random bytes
    * random floats between 0 and 1
* [Ruby's `Random` class](https://ruby-doc.org/core-2.4.0/Random.html)
    * random floats between 0 and max
    * random ints between 0 and max
    * random bytes
* [Common Lisp's `(random n)` function]([https://www.cs.cmu.edu/Groups/AI/html/cltl/clm/node133.html](http://clhs.lisp.se/Body/f_random.htm))
    * random ints between 0 and max
    * random floats between 0 and max
* [Java's `Random` class](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html)
    * random doubles, defaulting to [0,1) but can be given any min/max bounds
    * random ints (or longs), with any min/max bounds
    * random bools
    * random bytes
* [JS `genTest` library](https://www.npmjs.com/package/gentest)
    * random ints (within certain classes)
    * random characters
    * random "strings" (relatively short but random lengths, random characters)
    * random bools
    * random selection (N, with replacement) from a list
    * custom random generators
