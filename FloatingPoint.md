### Floating Point Explained
literature explanation on the components of floating point data structure: (for 32 bit floats)
* 1 bit S for the sign
* 8 bit E for the exponent
* 23 bits M for the mantissa

floating point value is represented as  (-1)^S * 1.M * 2^(E - 127)

a better way to explain about floating point is think of the exponent as "WINDOW", and mantissa as "OFFSET".
the window tells within which 2 consecutive power of two the number of will be in. e.g. [0.5, 1.], [1, 2], [2, 4], [4, 8] and so on up to [2^127, 2^128]. the offset
divides the window in 2^23 = 8388608 buckets. with the window and the offset, you can approximate a real number.

the mechanism protect against overflowing. once a value goes beyond a window, it is represented as a value in the next window with some loss of precision. the loss
of precision depends on which window the value is in. e.g. window [1,2] where 8388608 offsets cover range of 1 gives precision (2-1) / 8388608 = 0.00000011920929.
in window [2048,4096] the 8388608 offsets cover a range of (4096-2048) / 8388608 = 0.0002

to compute how a real number is represented, e.g. 3.14:
1. 3.14 is positive, so S = 0
2. 3.14 is between power of 2 [2,4], so floating window is 2^1, E = 128
3. there are 2^23 offsets available to express with, 3.14 falls within the interval [2,4], it is at (3.14 - 2) / (4 - 2) = 0.57 within the interval which makes the
   offset M = 2^23 * 0.57 = 4781507
the value for 3.14 is therefore approximated to be 3.1400001049041748046875. the corresponding formula: (-1)^0 * 1.57 * 2^(128 - 127)


