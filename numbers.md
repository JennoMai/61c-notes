# Number Representation

A set of *n* bits can represent 2<sup>n</sup> values. This section examines how we use bits to
represent numbers.

## Unsigned Integers
An *n*-bit unsigned number represents values from __0 to 2<sup>n-1</sup>__, and cannot represent
negative or fractional numbers. Listed below are the ways of converting between *unsigned values* in various bases.

### Decimal to Base-*n*
Given a base-10 number *x*, we can convert to base-*n* as follows:
1. Find *n'*, the largest power of *n* that is less than *x*.
2. Record *d*, the number of multiples of *n'* that fit in *x*, as a digit of the base-*n* number.
3. Subtract *d* × *n'* from *x*.
4. Set *n'*, as the next lowest power of *n*.
5. Repeat steps 2-4 until *n<sup>0</sup>* is reached.

### Base-*n* to Decimal
If we consider *d<sub>i</sub>* to be the *i*th digit of the base-n number beginning from the right (such that the
rightmost digit is *d<sub>0</sub>*), we simply take the sum of *d<sub>i</sub> × n<sup>i</sup>* for all *i*.

### Binary to Hexadecimal
Given a base-2 number *x*, we can convert to base-16 as follows:
1. Group *x* into 4-bit numbers, padding the left side with 0s as needed.
2. Evaluate each group to a (base-10) number between 0 and 15.
3. Substitute each group with the base-16 equivalent.

### Hexadecimal to Binary
Similarly to the reverse, we can simply replace each base-16 digit with its 4-bit equivalent and remove leading 0s.
___
## Signed Integers
There exist several different strategies for representing signed integers.
### Sign and Magnitude
The simplest method for representing negative numbers uses the leftmost bit to indicate sign. In this
representation, `0` represents positive and `1` represents negative.

This method is rather flawed, as it results in a "positive" and "negative" zero: `0000` and `1000`. Furthermore, it
complicates arithmetic as different steps must be taken depending on the signs of the integers. Finally, incrementing
bits sometimes increases values (if the first bit is `0`) and sometimes decreases values (if the first bit is `1`),
making the system less intuitive.

### One's Compliment
Given a non-negative integer *x*, the one's compliment represents *-x* by flipping all the bits of *x*.
- For example, `1101` (representing +13) would become `0010` (representing -13).
- All negative integers are still preceded by a leading `1` in this system.

While incrementing bits now uniquely increases values, this method still results in two representations of 0: `0000` and `1111`.
Furthermore, arithmetic remains somewhat complicated.

### Two's Compliment
In order to solve the problem of repeated 0s, the two's compliment shifts all negative integers by 1 such that `1111`
corresponds to a value of -1.
- To convert between positive and negative, flip the bits and *add 1*.
- For example, `1101` (representing +13) would be flipped into `0010`, and finally changed to `0011` (representing -13).
- Positive numbers contain a leading `0` while negative numbers contain a leading `1`.
- An *n*-bit number can represent 2<sup>*n*-1</sup> negative integers, 1 zero, and 2<sup>*n*-1</sup> - 1 positive integers.

This representation is the most commonly used. 

### Bias Encoding
With bias encoding, a *bias* of *b* is chosen and all unsigned integers are simply shifted by *b*.
- For an even distribution of positive and negative integers, a bias of -(2<sup>*n*-1</sup> - 1) is typically chosen.
- `0000` would represent 0 - (2<sup>3</sup> - 1) = -7
- `1111` would represent 15 - 7 = 8.
- With a bias of *b*, this representation has *b* negative integers, 1 zero, and 2<sup>*n*-1</sup> - 1 - *b* positive integers.
___

## Floating Point Numbers
Floating point numbers allow us to represent fractional values, as well as very large and very small numbers. They are declared using the `float`
keyword in C.

### Format
Single precision 32-bit IEEE floating point numbers are calculated as follows:

**<p style="text-align: center;">(-1)<sup style="color:#944725">sign</sup> × (1 + <span style="color:#255794;">significand<sub>two</sub></span>) × 2<sup>(<span style="color:#259463">exponent<sub>two</sub></span> - 127)</sup></p>**

* The first bit of the `float` represents the <span style="color:#944725">**sign**</span>
    - `0` for positive, `1` for negative
* The next 8 bits represent the value of the <span style="color:#259463">**exponent**</span>.
    - From left to right, each bit represents non-negative powers of two from 2<sup>7</sup> to 2<sup>0</sup> 
* The last 23 bits represent the <span style="color:#255794">**significand**</span>.
    - From left to right, each bit represents negative powers of two from 2<sup>-1</sup> to 2<sup>-23</sup>
    - The significand will always have a value between 0 and 1

Double precision numbers (and onwards) function identically, albeit with 11 bits for the exponent, 52 bits for the significand, and an exponent bias of 1023.

### Special Cases
Floating point representations incorporate a number of special cases.
* **Zero** is represented by an <span style="color:#259463">**exponent**</span> value of 0 and a <span style="color:#255794">**significand**</span> of 0. Positive and negative 0 both exist and are both valid.
* **Denormalized values** are represented by an <span style="color:#259463">**exponent**</span> of 0 and a nonzero <span style="color:#255794">**significand**</span>.
    * In other words, an <span style="color:#259463">**exponent**</span> of 0 represents a leading 0.
    * We consider there to be an implicit <span style="color:#259463">**exponent**</span> value of -126.
* **Infinity** is represented by the maximum <span style="color:#259463">**exponent**</span> value with a <span style="color:#255794">**significand**</span> of 0.
As per usual, the <span style="color:#944725">**sign**</span> determines whether it is positive or negative.
* **NaN** is represented by the maximum <span style="color:#259463">**exponent**</span> value with a nonzero <span style="color:#255794">**significand**</span>.

### Spacing
The smallest representable positive value of a *normalized* floating point number is **2<sup>-126</sup>**, since exponent values of 0 are reserved for special numbers.

The next smallest number is 2<sup>-126</sup> × (1 + 2<sup>-23</sup>) = **2<sup>-126</sup> + 2<sup>-149</sup>**. We notice then that numbers beyond 2<sup>-126</sup> are spaced by
2<sup>-149</sup>ths, leaving a large gap between 0 and the next representable number.

We use denormalization to resolve this issue.
- An exponent of 0 represents a leading value of 0 and *implicitly* results in a 2<sup>-126</sup> exponential term.
- The smallest representable *denormalized* value is 2<sup>-126</sup> × 2<sup>-23</sup> = **2<sup>-149</sup>**.
- Numbers between 0 and 2 are all evenly spaced by **2<sup>-149</sup>**.

### Discussion
Addition in floating point is not associative.


Precision is the number of bits used to represent a value.
Accuracy is the difference between the representation of a number and its true value.

Floating point addition:
* Denormalize numbers in order to match exponents
* Add significands together
* Keep the same exponents
* Renormalize