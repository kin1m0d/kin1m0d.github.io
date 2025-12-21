---
layout: post
title:  "Challenge: Crack the RSA Algorithm - My Solution"
date:   2025-12-20 13:25:00 +0000
categories: jekyll update
---

# My Solution (as far as I can remember, at least :grimacing:)

Check out this repository [crack-rsa-challenge](https://github.com/kin1m0d/crack-rsa-challenge), all those files are from 2022, I kept them unchanged, you might find a little bit of German in there as well lol.

From what I remember my approach was:
* Research how the algorithm works
* Encrypt my own message, and try to reverse engineer it (manually)
* Write some code that does the encryption and also crack it
* Confirm that everything works with some examples I found online
* Solve the challenge with my new scrypt

If you check the notes from the `README.md`, I was doing some tests with Bob and Alice, I guess that was the stuff I found online, based on that I have written some functions (see `functions.py`) that helped me calculating the greatest common divisor `gcd(m,n)`, the extended gcd `egcd(a,b)`, the modular inverse `modinv(a,m)`, and prime factors `prime_factors(n)`. I'm glad I had added some links explaining all the calculations, because I cannot remember at all! And `rsa.py` puts everything together, My class RSA does all the computation and spits out the results. The remaining py files were just little helper files for testing.

```
$ python ./rsa.py
### SOLUTION ###
(1, 289396593721086953, -2)
RSA(p=370248451, q=6643838879, e=17, n=2459871053643326429, phi_n=2459871046629239100, d=289396593721086953)
The lucky number is the following decrypted_plain_text:
DECRYPT: encrypted=35632720329943226 -> decrypted_plain_text=1512720


### TEST THE OTHER WAY AROUND ###
Let's test the reverse, we take the lucky number encrypt it and check if the encrypted value is the same
ENCRYPT: plain_text=1512720 -> encrypted=35632720329943226
Encrypted value: 35632720329943226
RSA(p=370248451, q=6643838879, e=17, n=2459871053643326429, phi_n=2459871046629239100, d=289396593721086953)
SUCCESS
```

I noticed `compute_p_q()` from my RSA class lacks an implementation, I cannot remember why, but I have used my function `prime_factors(n)` to get `p` and `q`, or just the following script which wraps that function.

```
python primeFactorDecomposition.py 2459871053643326429
[370248451, 6643838879]
```

