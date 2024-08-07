# Random Namespace and Functions

* Stage 0
* Authors: WhosyVox and Tab Atkins-Bittner
* Champions: Tab Atkins-Bittner
* Spec Text: currently this README

-----

Historically, JS has offered a very simple API for randomness: a `Math.random()` function that returns a random value from a uniform distribution in the range [0,1), with no further details. This is a perfectly adequate primitive, but most actual applications end up having to wrap this into other functions to do something useful, such as obtaining random *integers* in a particular range, or sampling a *normal* distribution. While it's not hard to write many of these utility functions, doing so *correctly* can be non-trivial: off-by-one errors are easy to hit even in simple "simulate a die roll" cases.

This proposal aims to introduces a number of convenient functions for dealing with random values, to make common random-related code both easier to use and less suseptable to errors.

Goals:
* To introduce at least the lowest common set of methods for properly generating random numbers
* To build a consistent API between the pre-seeded and seeded PRNG proposal
* To use readily understood method names, in line with what has been done in other areas of the language (e.g. `Array.toSorted`, `Aray.at`, `Object.fromEntries`)


## Details

This proposal suggests that all new Random-related functions be accessible from a global `Random` namespace.

This does not impact either the Crypto methods or Math.random, which would remain as-are.


## The current state of Random

As of today the main way to generate a random number (outside of library inclusion) is by direclty manipulating the output of `Math.random()`.

The common pattern for an integer is `Math.floor(Math.random()*(maxValue-MinValue)+maxValue)`. This is, at best, cumbersome.
This proposal would tidy it up to something closer to `Random.integerBetween(minValue,maxValue)`.

(This is akin to how we can use `Array.at(-1)` rather than `Array[Array.length-1]` to convey the intent of the code.)


## Example methods that may warrant inclusion
This is neither exhaustive, concrete concrete or yet well-researched, but merely serves as an example.

### Single randoms
|Function           | Description|
|-------------------|------------|
random()            | return a random decimal value in the range [0,1) |
integerBetween( x, y ) | return a random integer between x and y       |
boolean()           | randomly returns either true or false             |

### Lists of randoms
|Function                    | Description|
|----------------------------|------------|
randomList( size )               | return a `size` sized list of decimal values in the range [0,1)  |
integerBetweenList( size, x, y ) | return a `size` sized list of random integers in the range [x,y) |
booleanList( size )              | return a `size` sized list of random boolean values              |

### array methods
|Function             | Description|
|---------------------|------------|
pickFromList( array ) | return a random element from the array |
shuffle( array )      | perform an in-place random shuffle of the array                                          |
asShuffled( array )   | return a copy of the the provided array with the elements randomly shuffled              |

There might also be a good case to include a method for generating [Normal Distributions](https://en.wikipedia.org/wiki/Normal_distribution).


## Q&A

### Why use a new `Random` namespace?

Firstly the reason is to avoid polluting the existing `Math` namespace with a set of collective methods related to randomness.

Secondly...

### Why should the methods be symmetric with the [Seeded Random proposal](https://github.com/tc39/proposal-seeded-random/)?

There are some strong benefits in keeping these methods synchronised with the Seeded PRNG.

For methods or classes which require some element of randomness, test suites could take a fixed-seed PRNG with certain outcomes. And then, for the application itself, passing in `Random` should yield the same set of methods but with a random seed.

### Couldn't you just intantiate a Seeded PRNG with `Math.random()`?

I expect you could absolutely do that. It would be more verbose and remove some of the ease-of-use of this proposal however.

It would also require a new proposal to add these methods to the Seeded PRNG, as this wouldn't solve some of the original issues.


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
