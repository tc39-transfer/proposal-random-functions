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

The proposal is split across four somewhat-independent parts:

1. Creating a new `Random` namespace object to host the methods.
2. Defining several new families of random functions.
3. Specifying the PRNG algorithm. 
4. Applying all the functions to `SeededPRNG` as well.

# Part 1: The `Random` Namespace

The current `Math.random()` method is on the `Math` namespace object, 
presumably just because it had to live *somewhere* 
and since it returns a number `Math` seemed vaguely appropriate.

We propose that a new `Random` namespace object be created, 
to host all of the proposed methods of this proposal. 
A new `Random` namespace obviates the need to put "random" into the names of all the new methods.
Several of the proposed new methods also do not directly produce numbers, 
so putting them on `Math` seems inappropriate.

The existing `Math.random()` method will be left as-is (back compat requires that), 
but will also be duplicated into `Random`, as `Random.random()`, for consistency.

We do not propose doing anything with the existing `Crypto` namespace methods that deal with randomness.
They are specifically generating *cryptographically strong* randomness,
which is not a concern of these methods in general,
and their API shapes are geared towards cryptographic use-cases,
which are not necessarily reasonable for JS API design in general.

# Part 2: The New Random Functions

There is a very large family of functions we could *potentially* include.
The list here is, at this point, intentionally a bit large to show off the possibilities;
it is expected that the committee's feedback will reduce them to a smaller set.

## Random Numbers

* `random()`: Identical to the existing `Math.random()`. Takes no arguments, returns a random Number in the range `[0, 1)` (that is, containing 0 but not 1), with a uniform distribution.
* `number(lo, hi, step?)`: Returns a random `Number` in the range `[lo, hi)` (that is, containing `lo` but not `hi`), with a uniform distribution. Similar to the `Iterator.range()` proposal, third argument can be a step value, or an options bag specifying the step value and whether or not to include the endpoint. 
    * If `step` is defined, instead generates a random integer `R` and returns `lo + step*R`, with `R` ranging from 0 to a maximum that ensures the return value is less than or equal to `hi`. (Details below.) 
    * All arguments must be `Number`s, or else `TypeError`.
* `int(lo, hi, step?)`: Returns a random integral `Number` in the range `[lo, hi]` (that is, containing both `lo` and `hi`), with a uniform distribution. `step` works the same as in `number()`. 
    * All arguments must be integers, or else `TypeError`.
* `bigint(lo, hi, step?)`: Identical to `int()`, but returns a `BigInt` instead. All arguments must be `BitInt`s, or else `TypeError`.

> [!NOTE]
> Do we want to enforce an ordering for `lo` and `hi`, or allow them to be out of order? I lean towards allowing them in either order, especially since for `number()` the range is asymmetric; whether you want `[-2, -5)` or `(-2, -5]` can be application-specific. Also, which value is "low" when negatives are used is ambiguous anyway; `Random.number(-2, -5)` and `Random.number(-5, -2)` both potentially look correct.

> [!NOTE]
> The `step` behavior for these methods is taken directly from [CSS's `random()` function](https://drafts.csswg.org/css-values-5/#random).
> It's also [being proposed for `Iterator.range()`](https://github.com/tc39/proposal-iterator.range/issues/64#issuecomment-2881243363).

> [!NOTE]
> It's probably generally a good thing to match [`Iterator.range()`](https://github.com/tc39/proposal-iterator.range/) in the signatures, which would mean including `inclusive` as an options-bag argument, and defaulting `int()` and `bigint()` (and `number()` using a `step`) to exclude their `hi` value. That would mean, to simulate a d6, you'd need to write either `Random.int(1, 7)` or `Random.int(1, 6, {inclusive:true})`. I'm unsure if the consistency is worth it, tho... 

## Collection Methods

* `shuffle(arr)`: Shuffles an `Array` in-place. Argument must be an `Array` or Array-like.
* `toShuffled(coll)`: Returns a fresh `Array` containing the shuffled values from `coll`, which can be any iterable.
* `take(coll, n, {replace, counts, weights})`: Returns an `Array` containing `n` randomly-selected entries from `coll` (which must be an iterable). Optional arguments are:
    * `replace`, a boolean: if `false` (the default), takes without replacement; every value in the returned Array will be from a unique index in the original collection. (If `n` is larger than the length of `coll`, throw a `RangeError`?) If `true`, takes with replacement; values in the returned Array can be repeats (and you can take any amount of them without error).
    * `counts`, an iterable of non-negative integers: provides the "multiplicity" of the entries in the matching index from `coll`. `Random.take(["a", "b"], 2, {counts:[2, 3]})` is identical to `Random.take(["a", "a", "b", "b", "b"], 2)`. (In particular, this example could return "a" or "b" multiple times, even tho it's not using replacement, just like the desugared example can.) If omitted, all counts are `1`. If the iterable is too short, missing entries are treated as `0`; if too long, excess entries are ignored.
    * `weights`, an iterable of non-negative numbers: provides the "weight" for the entries in the matching index from `coll`, allowing some entries to be more likely to be selected than others. If omitted, all weights are `1`. If the iterable is too short, missing entries are treated as `0`; if too long, excess entries are ignored.
    * `counts` and `weights` can be used together; the specified weight for an entry is treated as applying to each of the multiple implied entries (not divided between them). That is, `Random.take(["a", "b"], 2, {counts: [2, 3], weights:[1, 2]})` is equivalent to `Random.take(["a", "a", "b", "b", "b"], 2, {weights: [1, 1, 2, 2, 2]})`.

## Distribution Methods

[Python includes a decent selection of distributions.](https://docs.python.org/3/library/random.html#discrete-distributions)
We should probably *at least* include the normal/gaussian distribution, given its high degree of usefulness. Should we include more? All of Python's distributions? Other distributions?

* `normal(mean=0, stdev=1)`: the gaussian/normal distribution
* `lognormal(mean=0, stdev=1)`: the [lognormal distribution](https://en.wikipedia.org/wiki/Log-normal_distribution) - a distribution whose *log* has a `normal(mean,stdev)` distribution
* `vonmisse(mean=0, kappa=0)`: the [von Misse distribution](https://en.wikipedia.org/wiki/Von_Mises_distribution), basically a gaussian over a circle
* `triangular(lo=0, hi=1, mode=(lo+hi)/2)`: a [triangular distribution](https://en.wikipedia.org/wiki/Triangular_distribution) with a high point of `mode`.
* `exponential(lambda=1)`: an [exponential distribution](https://en.wikipedia.org/wiki/Exponential_distribution) (from 0 to infinity). (The mean is `1/lambda`.)
* `binomial(n, p=0.5)`: the [binomial distribution](https://en.wikipedia.org/wiki/Binomial_distribution) - how many successes in N trials with P chance of success? (sampling with replacement)
* `geometric(p=.5)`: the [geometric distribution](https://en.wikipedia.org/wiki/Geometric_distribution) - how many failures before the first success, with P chance of success? (sampling with replacement)
* `hypergeometric(n, N, K)`: the [hyper-geometric distribution](https://en.wikipedia.org/wiki/Hypergeometric_distribution) - how many successes in n trials if exactly K of the N possible outcomes are a success? (sampling *without* replacement)
* `beta(alpha, beta)`: the [beta distribution](https://en.wikipedia.org/wiki/Beta_distribution)
* `gamma(alpha, beta)`: the [gamma distribution](https://en.wikipedia.org/wiki/Gamma_distribution)
* `pareto(alpha)`: the [Pareto distribution](https://en.wikipedia.org/wiki/Pareto_distribution)
* `weibull(alpha, beta)`: the [Weibull distribution](https://en.wikipedia.org/wiki/Weibull_distribution)

## Byte Methods

* `bytes(n)`: Returns a `Uint8Array` of the specified length, filled with random bytes.
* `fillBytes(buffer)`: Fills the passed typed array/view/etc with random bytes. (Pass views to fill only a chunk of an array.)

## Other Value Methods

* `boolean(p=0.5)`: Returns a bool, with propability `p` of returning `true`. (Exactly equivalent to `Random.random() < p`, so possibly not worth it?)

# Part 3: Specifying The PRNG

`Math.random()` was historically unspecified, and because some benchmarks unintentionally depended on it, has been driven toward a fast but terrible PRNG algorithm in implementations.

We propose that the `Random` methods use a *specified* PRNG algorithm, specifically the same one used by the [Seeded Random proposal](https://github.com/tc39/proposal-seeded-random/) - ChaCha12. This algorithm produces high-quality pseudo-randomness (but intentionally not cryptographic quality), and has good performance characteristics relative to PRNGs of similar quality. Plus, it would mean a shared implementation between the two very similar proposals, which is likely appealing to implementations, since all of the `Random` methods are intended to appear on `SeededPRNG` as well.

We will probably leave the choice of initial PRNG state to still be up to the UA.

# Part 4: Interaction with `SeededPRNG`

It is intended that this proposal and the [Seeded Random proposal](https://github.com/tc39/proposal-seeded-random/) expose the same APIs; every method on `Random` should exist on `SeededPRNG` objects as well. Either proposal can advance ahead of the other, however. This proposal is intentionally not touching seeded randomness, instead focusing on functions that are agnostic as to their random source.

# Prior Art

* [Python's `random` module](https://docs.python.org/3/library/random.html)
    * random float, with any min/max bounds (and any step)
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
