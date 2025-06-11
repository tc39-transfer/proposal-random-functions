Random Generation Methods
=========================

## Number in [0, 1)

Used by Rust/numpy

1. Pull 64 bits. 
2. Shift it down to 53 bits.
3. Cast to f64.
4. Divide by 2^-53.

* 53 bits used; 64 bits consumed.
* Generates all possible floats in [.5, 1). Generates half of all possible floats in [.25, .5). Generates 1/4 of all possible floats in [.125, .25). Etc.
* Smallest non-zero value is 2^-53.
* Equal to "generate a random real, then floor to nearest returnable float". All values equally likely.

## Number in (0, 1)

Invented by me, closely follows <https://www.corsix.org/content/higher-quality-random-floats>

1. Pull 128 bits.
2. Put 52 bits into the mantissa of an f64.
3. Count leading 0s in the remaining bits. 
4. Set exponent of f64 to (-ZeroCount - 1).

* 128 bits used; 64 or 128 bits consumed.
* Generates all possible floats in [2^-77, 1).
* Smallest non-zero values is 2^-77. (Can't generate 0 itself.)
* Can be split into pulling 64 bits + an extra 64 in 2^-12 (~1 in 4000) of cases.
* Almost equal to "generate a random real, then floor to nearest float", except floats in [2^-77, 2^-76) occur twice as often as "perfect", and floats in [0, 2^-77) never occur. (Seeing this is much less likely than generating a perfect 0 in the previous method.)
	* Flooring to nearest float not *quite* equal to "convert real to nearest float"; the unit dyadics (1/2, 1/4, 1/8, etc) show up slightly more often (4/3 higher weight than "perfect").
* Avoiding possibility of 0 (and 1) helps some algorithms that blow up at 0. (Can never depend on 0 showing up anyway, previous method only generates it 1 in 2^53 times.)

## Int in [0, max) (max < 2^64)

[Canon's Method](https://github.com/swiftlang/swift/pull/39143)

1. Cast max into a u64.
2. Pull 64 bits.
3. 128-bit multiply with max.
4. High 64 is result. Low 64 is leftover.
5. Pull 64 more bits, and 128-bit multiply with max again.
6. Add the high 64 to the leftover. If the result overflows, +1 to result.
7. Cast result back to Number. 

* 64 bits used (+64 to remove bias); 64 or 128 bits consumed.
* Bias is less than 2^-64 (would require >= 2^128 samples to notice).
* Can skip the second 64 bits pull if leftover+max-1 doesn't overflow (result can't change), or if result is >= 2^53 and bottom N bits aren't all 1 (+1 won't change the Number)
* Handles any [min, max] where min+max+1 < 2^64. (Second early-exit should be if result is >= 2^54, to be safe.)


## Int in [0, max] (max >= 2^64)

Invented by me

1. Let |bound| be the most significant 63 bits of |max|, +1.
2. Use Canon's method to generate a u64 in [0, bound).
3. Set the corresponding bits of |result| to the number. Fill all remaining bits of |result| randomly.
4. If result is >= max, restart.

* log2(max)+1 bits used; ceil(log2(max) / 64) \* 64 + 64 bits consumed
* Bias is less than 2^-64, as Canon's method, and only affects the high bits.
* Chance of rejection less than 2^-62, effectively nil in practice.
* In theory could check results as you fill remaining bits and reject early, but not worth complexity.
* Can be used both for `Random.int()` (for large bounds) and `Random.bigint()`.

## Number in (min, max)

Trivial, already ubiquitous in the literature.

1. Generate a random in (0, 1).
2. Multiply by (min - min), add min.
3. If result equals min or max, restart.

* Same bits used/consumed as (0,1) range (except when rejection occurs).
* Rejection should be extremely rare.
* Straightforward and easy to understand.
* Not remotely uniform in many cases, bad properties in general.
* Without rejection, can easily generate |min| or |max| for many possible bounds, even small ones.

## Number in (min, max)

Drawn from <https://hal.science/hal-03282794v5/document>

function γsectionOO(a,b)
	g = γ(a,b)
	hi = ceilint(a,b,g)
	k = rand(DiscreteUniform(1, hi-1))
	(k1,k2) = splitint64(k)
	if abs(a) <= abs(b)
		return 4*(b/4-k1*g)-k2*g
	else
		return 4*(a/4+k1*g)+k2*g
	end

1. Let |step| be the largest step between floats between |min| and |max|.
2. Let |values| be ceilint(|min|, |max|, |step|).
3. Let |k| be a random int in (1, |values| - 1), using Canon's method.
4. Let |k1| be |k| >> 2, and |k2| be the low 2 bits of |k|.
5. If abs(|min|) <= abs(|max|), return 4\*(|max|/4 - |k1|\*|step|) - |k2|\*|step|.
	Otherwise, return 4\*(|min|/4 - k1\*|step|) + |k2|\*|step|.


ceilint(|a|, |b|, |g|), implements ceil((b-a)/g) perfectly, and without overflow

1. Let s = b/g - a/g.
2. If s is not an integer, return ceil(s).
3. Otherwise, set |s| to ceil(s).
4. If abs(a) <= abs(b), let |eps| = -a/g - (s - b/g).
	Otherwise, let |eps| = b/g - (s + a/g).
5. If |eps| > 0, set |s| to |s|+1.
6. Return |s|.

* 52-54 bits of precision; 128 bits consumed.
* Perfectly uniform, and equidistributed.
* No rejection sampling at all.
* Generates all possible floats in the highest exponent; etc, same as the [0,1) method above.
* Could modify it to consume 64 more bits to densely generate floats, if desired.
	* If in the next exponent down, take 1 bit to decide between the two possible floats. If two exponents down, take 2 bits. etc.
	* Like, if one endpoint is 2^54, all values generated are even ints right now. Probably good to avoid that.
	* min(52, Max exponent - result exponent) = number of bits to fill in on the little end of mantissa. Special case for 0: consume one extra bit for sign (if needed), and count leading 0s from 11 bits to push the exponent range even lower.