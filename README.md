# Random Namespace and Functions

* Stage 0
* Authors: WhosyVox and Tab Atkins-Bittner
* Champions: Tab Atkins-Bittner
* Spec Text: currently this README

-----

Historically, JS has offered a very simple API for randomness: a `Math.random()` function that returns a random value from a uniform distribution in the range [0,1), with no further details. This is a perfectly adequate primitive, but most actual applications end up having to wrap this into other functions to do something useful, such as obtaining random *integers* in a particular range, or sampling a *normal* distribution. While it's not hard to write many of these utility functions, doing so *correctly* can be non-trivial: off-by-one errors are easy to hit even in simple "simulate a die roll" cases.

This proposal introduces a number of convenience functions for dealing with random values, to make common random-related use-cases easier to use. It groups all of these under a new namespace object - `Random` - for convenience and ease of understanding.

Goals:
* To introduce at least the lowest common set of functionality for generating random numbers
* To build a consistent and easily understood API

Applications currently outside the scope of this proposal:
* Random string functions
* Specialised probability functions
* Iterator/Generators

Anything on this list would ideally require a libray or separate proposal.

## Details

This proposal suggests that all new Random-related functions be accessible from a global `Random` namespace.

This does not impact either the Crypto methods or Math.random, which would remain as-are.


## Example methods

### Single randoms
|Function           | Description|
|-------------------|------------|
random()            | return a random decimal value in the range [0,1) |
numberBetween( x, y )  | return a random value between x and y         |
integerBetween( x, y ) | return a random integer between x and y       |
boolean()           | randomly returns either true or false             |

### Lists of randoms
|Function                    | Description|
|----------------------------|------------|
randomList( size )               | return a `size` sized list of decimal values in the range [0,1)  |
numberBetweenList( size, x, y )  | return a `size` sized list of random values in the range [x,y)   |
integerBetweenList( size, x, y ) | return a `size` sized list of random integers in the range [x,y) |
booleanList( size )              | return a `size` sized list of random boolean values              |

### array methods
|Function             | Description|
|---------------------|------------|
pickFromList( array ) | return a random element from [0,array.length) (can be undefined for empty/sparse arrays) |
shuffle( array )      | perform an in-place random shuffle of the array                                          |
asShuffled( array )   | return a copy of the the provided array with the elements randomly shuffled              |



### Some open questions

How and when to clamp input values of ±Infinity.

Should integers for range (x,y) be generated as [x,y) or \[x,y\]?

How should integers be generated when passed non-integer boundaries?


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
    * sample Gaussian distribution
* [JS `genTest` library](https://www.npmjs.com/package/gentest)
    * random ints (within certain classes)
    * random characters
    * random "strings" (relatively short but random lengths, random characters)
    * random bools
    * random selection (N, with replacement) from a list
    * custom random generators
