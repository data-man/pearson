# Pearson Hashing
This repository contains a custom implementation of a "Fast Hashing of Variable-Length Text Strings" as suggested by Peter K. Pearson in The Communications of the ACM  Vol.33, No.  6 (June 1990), pp. 677-680 – the so called _Pearson Hashing_.

The [Wikipedia](https://en.wikipedia.org/wiki/Pearson_hashing) article contains a good description. Also, it links to a `.pdf` copy the original paper.

The underlying principle of this algorithm is easy to understand. Simplicity shines!

Note that this is not a cryptographicly secure hashing function, so it cannot unencryptedly be used to prove knowledge of a secret without needing to present that secret. But it is well suited for keying hash tables or to compress or expand given entropy of some byte-string to a different number of bytes or just as a checksum. Its design also allows to verify smaller parts of the hash value if required.

For use as checksum of to-be-encrypted data (including the checksum), it is noteworthy that the random permutation as repeating but especially final step makes Pearson Hashing even a better choice than CRC – especially with a view to exclusive-ored stream ciphers.

# Thoughts

To get to output bit-widths beyond eight, Pearson describes an extension scheme which uses different starting values. Unfortunately, output will never contain the same two bytes in the various positions then – diminishing the codomain. This can be avoided by using a different permutation per position (output byte). In that case, there is no need for different starting values anymore.

Not to use another look-up-table for each and every output byte, this implementation changes every byte by a constant value  – different for each position – just before the look-up. If the table is random (and not arithmetically generated), it will seem to be a different permutation (from the new-index-look-up point of view). Remember, this is all modulo 256. After the look-up, the applied change could be reversed but it does not have to (gaining some speed).

Talking about speed, it seems to be feasible to use exclusive-or to change the to-be-looked-up index.  It was Peter K. Pearson – with whom I had a short exchange of eMails on getting permission for use – who pointed out that this is exactly like the key-whitening as found in DES-X for example. By using exclusive-or, a big advantage can be assumed for parallel implementation in regular C then (if not using SSE's vetorized bytewise operations).

In a hunt for more speed, a quick test using a linear congruential generator with full periodicity of 256 (trying to avoid the look-up in memory) resulted in uneven-looking distributed output. I am not able to see that my parameters were badly chosen and I stopped these test maybe too soon, but for now, this road does not want to be gone down.

This hasing function could easily be customized or _tweaked_ (here, I would avoid the term _keyed_) just by changing the permutation along some provided tweak. A possible solution would be to use a tweak as seed for a pseudo-random number generator (maybe not a linear congruential one…) and repeatedly switch any two values in the table indicated by the pseduo-random number generator. Good thing here: The setup happens once _before_ use, it does not impact the actual hashing speedwise.

The hash value equals the innner hashing function state so it can easily be updated whenever further to-be-hashed input data comes along, e.g. in streaming applications.

# This Implementation

The provided code follows the above-elaborated thoughts and offers two different flavors:

One outputs a 128-bit or 256-bit hash value to memory for any hashed string. It is endian-aware. Also, a SSE version is included, just compile using `-march=native`. The 256-bit might see an AVX-imeplementation in the future, probably not the 128-bit hashing as the _shuffle_ is not getting wider (bitwise) and thus does not speed up the table look-up. SSE implementation could turn out slightly slower on some more modern machines than plain C version compiled using `-Ofast`. The reason probably is some CPU feature speed-up or so that came up somewhen between Sandy Bridge and Haswell – I was not able to nail it down more precisely. NEON might follow at a later point in time, too.

The other one returns a 32-bit hash of type `uint32_t`. Those bits are identical to the last 32 bit of the 128-bit and 256-bit hash. As it is a regular return value of a function call, its further use must happen endian-aware. Also, the return value could be used as 'hash' parameter for another call to update the hash  while hashing more of an input.

The fully compiled tool using `gcc -Ofast -mtune=native test.c pearson.c` or `gcc -Ofast -mtune=native -march=native test.c pearson.c`, respectively, shows the following speeds when run (`./a.out`):

 CPU | 32-bit hash | 128-bit hash |
:---:| ---:        | ---:         |
Cortex A53|  60.6 MB/s |  35.1 MB/s |
i7 2860QM | 192.9 MB/s |  82.7 MB/s¹|
i7 5775C  | 232.7 MB/s | 131.0 MB/s²|
i7 7500U  | 221.1 MB/s | 128.6 MB/s²|

¹ this was obtained using SSE code; the non-SSE version delivers 77.5 MB/s

² on these platforms, speeds were obtained with plain 64-bit C code (omitting `-march=native`) as it turned out way faster than the SSE code (85.4 MB/s and 86.7 MB/s)

# Contribute

You are welcome to help with NEON or AVX implementation or any other contribution that would speed-up things. Maybe multi-threading? Don't be shy and leave an _Issue_ or a _Pull Request_!
