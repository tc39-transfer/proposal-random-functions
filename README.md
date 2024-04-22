# Random Namespace and Functions

Historically, JS has offered a very simple API for randomness: a `Math.random()` function that returns a random value from a uniform distribution in the range [0,1), with no further details. This is a perfectly adequate primitive, but most actual applications end up having to wrap this into other functions to do something useful, such as obtaining random *integers* in a particular range, or sampling a *normal* distribution. While it's not hard to write many of these utility functions, doing so *correctly* can be non-trivial: off-by-one errors are easy to hit even in simple "simulate a die roll" cases.

This proposal introduces a number of convenience functions for dealing with random values, to make common random-related use-cases easier to use. It groups all of these under a new namespace object - `Random` - for convenience and ease of understanding.

## Meta
* Stage 0
* Authors: WhosyVox and Tab Atkins-Bittner
* Champions: Tab Atkins-Bittner
* Spec Text: currently this README

## Details

TODO

## Prior Art

* [Python's `random` module](https://docs.python.org/3/library/random.html)
* [.Net's `Random` class](https://learn.microsoft.com/en-us/dotnet/api/system.random?view=net-8.0)
* [Haskell's `RandomGen` interface](https://hackage.haskell.org/package/random-1.2.1.2/docs/System-Random.html)
* [Ruby's `Random` class](https://ruby-doc.org/core-2.4.0/Random.html)
