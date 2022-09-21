# My-Public-CTF-Challenges
This is the repository of some CTF challenges I made, maybe including the source code and solution.

## **ToC**
- [0CTF/TCTF 2022](#0CTF/TCTF-2022)
  - [real magic dlog](#real-magic-dlog)
  - [Leopard](#Leopard)
  - [ezRSA++](#ezRSA++)
  - [Login](#Login)
  - [Chimera](#Chimera)
- [0CTF/TCTF 2021 Finals](#0CTF/TCTF-2021-Finals)
  - [ezMat](#ezMat)
  - [ezHash](#ezHash)
  - [ezRSA](#ezRSA)
  - [ezRSA+](#ezRSA+)
  - [halfhalf](#halfhalf)
- [0CTF/TCTF 2021 Quals](#0CTF/TCTF-2021-Quals)
  - [zer0lfsr-](#zer0lfsr-)
  - [zer0lfsr+](#zer0lfsr+)
  - [zer0lfsr++](#zer0lfsr++)


<a id="0CTF/TCTF-2022"></a>

## 0CTF/TCTF 2022

<a id="real-magic-dlog"></a>
### real-magic-dlog
Category: **Crypto**  
Difficulty: ★  
Solved: **66 / 158**  
Tag: **Smoothness, Coppersmith**  

#### Details

[Source](0CTF-TCTF-2022/real-magic-dlog/)

This challenge is based on HITCON CTF 2021 [**magic dlog**](https://ctf2021.hitcon.org/dashboard/#19), but we add a 60-seconds limit this time. According to its author @lyc, the intended solution is [[1]](#magic-ref1). But many players solved it with simple brute-force attack, i.e., randomly generating numbers and hope it is smooth enough.

#### Solution

I underestimate the success rate of the original brute-force method, so it also works for this challenge. The intended solution is to implement the algorithm in [[1]](#magic-ref1). But in our case, it can not guarantee the full smoothness of the output number, the number always contains one or two large factors. So, we hope these factors are not too large, then the DLP can be solved within 1 minute. Another trick is the parameter setting, the algorithm works better if we set `S` (for details, see [[1]](#magic-ref1)) as the product of first 45 primes, I don't know why.

**Reference**  
<a id="magic-ref1">[1]</a> Boneh, Dan. "Finding smooth integers in short intervals using CRT decoding." *Proceedings of the thirty-second annual ACM symposium on Theory of computing*. 2000.

<a id="Leopard"></a>
### Leopard
Category: **Crypto**  
Difficulty: ★★☆  
Solved: **6 / 158**  
Tag: **AEAD, Algebraic Attack**  

#### Details

[Source](0CTF-TCTF-2022/Leopard/)

This challenge implements a homemade AEAD scheme named *Leopard*, which is inspired from an AEAD scheme called *Panther* [[1]](#leopard-ref1). Note that *Panther* suffers from a straightforward attack [[2]](#leopard-ref2), while *Leopard* has been patched. And in this challenge, we are asked to apply the differential fault analysis on *Leopard*. Specifically, given a fixed plaintext, its ciphertext as well as a fault ciphertext, one should recover the secret key and IV within 300s.

#### Solution

There are at least three methods to solve this task.

- Approach 1 (team *More Smoked Leet Chicken*): This is an algebraic attack, and it is the intended solution. The SBOX of *Leopard* is not generated in the same way as the SBOX of *AES*. One can check this [wiki page](https://en.wikipedia.org/wiki/Rijndael_S-box), note that the linear transform is done over $\mathbb{F}_{2}^8$. But in this challenge, one can figure out the linear transform is actually done over $\mathbb{F}_{2^8}$, it can be represented as `SBOX[x]=55/x+186`. Now consider the internal state of the cipher just before the fault will be injected, partial state are known to us since they are equal to the ciphertext bytes, so we can treat the remaining unknown state bytes as variables over $\mathbb{F}_{2^8}$, then simulate the state update function locally. And we can get several equations in the form as `55/f(x_0,x_1,...)+186=ct[i]+pt[i]`, **SageMath** keeps these equations in fraction form, but we can extract the numerator part of the fraction, which yields a multivariate equation. After collecting enough equations, we can find the roots with Gröbner basis. The fault is used to providing more equations.
- Approach 2 (team *perfect r00t*): This is a SAT-based solution, one can check [here](https://github.com/perfectblue/ctf-writeups/tree/master/2022/0ctf-2022/leopard).
- Approach 3 (other teams): This is a guess-and-determine attack, which exploits the internal state update function. Several players shared
their codes in Discord channel, but there are no detailed writeups now. If you have a detailed analysis, please contact me :)

**Reference**  
<a id="leopard-ref1">[1]</a> 
Bhargavi, K. V. L., Chungath Srinivasan, and K. V. Lakshmy. "Panther: A Sponge Based Lightweight Authenticated Encryption Scheme." International Conference on Cryptology in India. Springer, Cham, 2021.  
<a id="leopard-ref2">[2]</a> 
Boura, Christina, Rachelle Heim Boissier, and Yann Rotella. "Breaking Panther." Cryptology ePrint Archive (2022).

<a id="ezRSA++"></a>
### ezRSA++
Category: Crypto  
Difficulty: ★★★☆  
Solved:  **4 / 158**  
Tag: **RSA, Coppersmith**  

#### Details

[Source](0CTF-TCTF-2022/ezRSA++/)

This challenge is based on 0CTF/TCTF 2021 Finals [**ezRSA+**](#ezRSA+).
We are given an CRT-RSA public key where `N` and `e` are around 2000 bits.
We also know both `dp` and `dq` are 105 bits, and we are given 55 LSBs of them. 

#### Solution

- Approach 1 (team *idek*): This is the intended solution, just implement the algorithm in the paper [[1]](#ezRSApp-ref1). @gripingberry has released his neat [code] (https://gist.github.com/willfisher/79292934bf2a08cc41174619ae69a70c).
- Approach 2 (team *Super Guesser* and team *Straw Hat*): Please read rkm's [writeup](https://rkm0959.tistory.com/264).

**Reference**  
<a id="ezRSApp-ref1">[1]</a> May, Alexander, Julian Nowakowski, and Santanu Sarkar. "Partial Key Exposure Attack on Short Secret Exponent CRT-RSA.  


<a id="Login"></a>
### Login
Category: **Crypto**  
Difficulty: ★★☆  
Solved: **2 / 158**  
Tag: **Lattice, RLWE, IPFE**  

#### Details

[Source](0CTF-TCTF-2022/Login/)

This challenge implements an RLWE-based inner-product functional encryption scheme. You have to login as player at first then login as admin to get flag. The scheme is modified from [[1]](#login-ref1). 

#### Solution

The original scheme encodes a l-dimension vector to l polynomials, and this is provably secure. But in this task, both the vectors `x` and `y` are treated as one polynomial `X` and `Y`, it is possible to recover `y` from the coefficients of the result `X*Y` with some linear algebra when `x` is known. One may refer to *perfect r00t* 's [writeup](https://github.com/perfectblue/ctf-writeups/tree/master/2022/0ctf-2022/login) for detials

**Reference**  
<a id="login-ref1">[1]</a> Mera, Jose Maria Bermudo, et al. "Efficient lattice-based inner-product functional encryption." *IACR International Conference on Public-Key Cryptography*. Springer, Cham, 2022.


<a id="Chimera"></a>
### Chimera
Category: **Crypto**  
Difficulty: ★★★★☆  
Solved: **0 / 158**  
Tag: **Feistel, SPN, Boomerang Attack**  

#### Details

[Source](0CTF-TCTF-2022/Chimera/)

This challenge implements a homemade block cipher named *Chimera*. The 16-bytes plain text will be encrypted by two parallel 4-rounds DES-like cipher, then permutated, and finally encrypted by a 4-rounds (without shift rows in first round) AES-like cipher. One can set one byte among the master keys of three components to a specified byte of flag. And the server provide an encryption oracle as well as a decryption oracle. The time limit is 300 seconds, and one can call encryption/decryption oracles for 16-bytes input plaintext at most 8192 times.

#### Solution

The intended solution is a combination of differential cryptanalysis of the Feistel cipher [[1]](#chimera-ref1) and Yoyo tricks with the SPN [[2]](#chimera-ref2), which can be regarded as a retracing boomerang attack [[3]](#chimera-ref3). The basic idea is to find a differential path of one of the Feistel cipher such that the non-zero bytes in the output differential after the permutation are placed in one column of the input to SPN, then the condition of Yoyo tricks will be satisfied. And one may hope 
when decryption, the input difference of another Feistel cipher are exactly zero, thus the right half of two plaintexts are the same. Once this happens, the differential path of the first Feistel cipher is likely activated, and we can extract some bytes of the first round key, since the input as well as the difference of the output of first round function are known. Detailed writeup is coming soon...

**Reference**  
<a id="chimera-ref1">[1]</a> Biham, Eli, and Adi Shamir. Differential cryptanalysis of the data encryption standard. Springer Science & Business Media, 2012.  
<a id="chimera-ref2">[2]</a> Rønjom, Sondre, Navid Ghaedi Bardeh, and Tor Helleseth. "Yoyo tricks with AES." *International Conference on the Theory and Application of Cryptology and Information Security*. Springer, Cham, 2017.  
<a id="chimera-ref3">[3]</a> Dunkelman, Orr, et al. "The retracing boomerang attack." *Annual International Conference on the Theory and Applications of Cryptographic Techniques*. Springer, Cham, 2020.



<a id="0CTF/TCTF-2021-Finals"></a>
## 0CTF/TCTF 2021 Finals

### ezMat
Category: **Crypto**  
Difficulty: ★  
Solved: **12 / 12**  
Tag: **Matrix, Linear Algebra**  

#### Details

[Source](0CTF-TCTF-2021-Finals/ezMat/)

This task is inspired from the challenge [**Onclude**](https://ctftime.org/task/16872) in *Crypto CTF 2021*. 
The ciphertext computes as `E = L_inv * S * X = U * X = U * (A + R)`, we know `E` and `R`, need to recover `A`. And `U` is the LU decomposition of a secret matrix, `A` is a matrix which contains most of zero and flag bytes. We are also given a sha256 digest of flag.

#### Solution

Notice that `U` is a triangular matrix, and there are lots of zero elements in `A`, thus we may solve linear equations to recover most elements in `A`,  then bruteforce the remaining bytes according to the given sha256 digest.

### ezHash
Category: Crypto  
Difficulty: ★★★☆  
Solved: **3 / 12**  
Tag: **Matrix, LPS Graph**  

#### Details

[Source](0CTF-TCTF-2021-Finals/ezHash/)

The script implements an algorithm involves 2x2 matrix multiplication over Fp. Our input will be converted to base 5 sequence, then each time choose one of the six generator matrix multiple to the result according to the sequence.
The server will generate a random matrix `X` per connection, we have 10 seconds to find an input which yields a matrix `Y`, where `Y ≡ aX (mod p)` for some `a ∈ Fp`.

#### Solution
In fact, the algorithm is a hash function based on LPS graph. This paper [[1]](#ezHash-ref1) describes a preimage attack, just implement it.

**Reference**  
<a id="ezHash-ref1">[1]</a> Petit, Christophe, Kristin Lauter, and Jean-Jacques Quisquater. "Full cryptanalysis of LPS and Morgenstern hash functions." *International Conference on Security and Cryptography for Networks*. Springer, Berlin, Heidelberg, 2008.

### ezRSA
Category: Crypto  
Difficulty: ★★  
Solved: **11 / 12**  
Tag: **RSA, Hint**  

#### Details

[Source](0CTF-TCTF-2021-Finals/ezRSA/)

The script generates a CRT-RSA key pair, where `n` is 2000 bits, `e` is 1000 bits,  `dp`, `dq`, `k`, `l` are only 60 bits, then encrypt the flag. We are given the public key, encrypted flag and the lowest 10 bits of both `dp`  `dq` as well as a magic number `m = 1337 * k ** 4 + 7331 * l ** 3 + 73331 * k ** 2 + 13337 * l ** 2 + 7 * k * l + 2 * k + l`. 

#### Solution
Take 4th root of `m // 1337` may get an approximation of `k`, then search in a small range, for each candidate `k_bar`, take 3th root of `(m - 1337 * k_bar ** 4) // 7331` to get an approximation of `l`, then search again and use `m` to determine `l` and `k`. As `k * p - k = e * dp - 1`, we can recover `p`, then decrypt flag.  


<a id="ezRSA+"></a>
### ezRSA+
Category: Crypto  
Difficulty: ★★☆  
Solved:  **6 / 12**  
Tag: **RSA, Lattice, Continued Fraction**  

#### Details

[Source](0CTF-TCTF-2021-Finals/ezRSA+/)

Same as **ezRSA**, but no magic numebr `m`, and we get the lowest 12 bits of both `dp`  `dq`.

#### Solution

- Approach 1: After some arithmetic, we have `e * e * X + e * Y + Z = W * n`, where `X = dp * dq`,`Y = dp * (l - 1) + dq * (k - 1)`, `Z = (k - 1) * (l - 1)`, `W = k * l`.  Notice that `X, Y, Z, W` are small enough, so we can construct a lattice where the first column is `[n, -e^2, -e]`, then apply LLL to get `k` and `l`, which leads to factor `n`. 
- Approach 2: Notice `dp` and `dq` are small enough, we may apply [Wiener's attack](https://en.wikipedia.org/wiki/Wiener's_attack), to get `X = dp * dq` (for data in this task, `X = dp * dq // 3`) from the continued fraction of `e * e / N`. Then we may guess `dp` from the factors of `X`, and recover `p` via `GCD(pow(pow(2, e, n), dp, n) - 2, n)` .
- Approach 3: Both the above methods are unintended, intended solution involves a Partial Key Exposure attack on CRT-RSA [[1]](#ezRSA-ref1) or a small CRT-Exponent attack (slower) [[2]](#ezRSA-ref2).

**Reference**  
<a id="ezRSA-ref1">[1]</a> May, Alexander, Julian Nowakowski, and Santanu Sarkar. "Partial Key Exposure Attack on Short Secret Exponent CRT-RSA.  
<a id="ezRSA-ref2">[2]</a> Takayasu, Atsushi, Yao Lu, and Liqiang Peng. "Small CRT-exponent RSA revisited." Journal of Cryptology 32.4 (2019): 1337-1382.

### halfhalf
Category: Reverse  
Difficulty: ★★☆  
Solved: **7 / 12**  
Tag: **Rust, Jacobi Symbol**

#### Details

[Source](0CTF-TCTF-2021-Finals/halfhalf/)

We are given a Rust ELF file. It requires you solve a PoW, then input a magic word, next the server will generate a RSA key and a secret number. You can interact with server, there are five options: Get the RSA modulus; Guess the secret; Reset secret; Get flag; Exit. Each time you input a guess, the server may encrypt`9x^2` or `18x^2` for some random `x`, which depends whether your guess is greater than the secret. And if your guess is same as the secret you may get the flag. Note that all the number should be encoded with emoji.

#### Solution

The PoW requires us to find a 4-byte phrase whose sha256 digest match the given value for the last 3 bytes. And it is easy to find the magic word, which are a series of emoji. To distinguish `9x^2` and `18x^2`, we can compute its Jacobi symbol, so a binary search can determine the secret and get the flag. 

#### Issues

- I deployed this challenge with xinetd, due to a mistake in the wrapper, the program is executed under root directory, so it can't read the flag from file. Thanks @FORMALIN for pointing out this.
- @RBTree tells me the 60s time limitation is too tight, and he asks to increase it. But the source code of this challenge is lost, due to a VM broken, so the only way is to patch the binary. And he quickly finds the location of the corresponding parameters, then I patch it from 60s to 200s. Thanks RBTree's help. 

<a id="0CTF/TCTF-2021-Quals"></a>

## 0CTF/TCTF 2021 Quals
<a id="zer0lfsr-"></a>
### zer0lfsr-
Category: **Crypto**  
Difficulty: ★★  
Solved: **35 / 198**  
Tag: **LFSR, NFSR, Z3**  

#### Details

[Source](0CTF-TCTF-2021-Quals/zer0lfsr-/)

There are three generators composed of LFSR or NFSR.

- Generator1 contains a 48-bit NFSR and a 16-bit LFSR

- Generator2 contains a 16-bit NFSR and a 48-bit LFSR

- Generator3 contains a 64-bit LFSR

The output functions of each generator are filted, which are not linear from its internal state. We are asked to recover the initial state of any two generators within 50 seconds after obtained its consecutive 5000-byte output and the hash digest of its initial state. 

#### Solution

Most teams choose to attack Generator1 and Generator3.

- Approach 1: Use z3
- Approach 2: See [rkm's writeup](https://rkm0959.tistory.com/229?category=765103) for details 

<a id="zer0lfsr+"></a> 
###  zer0lfsr+
Category: **Crypto**  
Difficulty: ★★★★  
Solved: **2 / 198**  
Tag: **LFSR, NFSR, FCA**  

#### Details

[Source](0CTF-TCTF-2021-Quals/zer0lfsr+/)

The three generators remain unchanged. But we are given 20000 bytes from the majority function of the output of three generators, and are required to recover all the initial states within 100s. The initial state of Generator2 is derived from Generator1 with KDF function, the initial state of Generator3 is derived from Generator2 with KDF function.

#### Solution

Please check [rkm's writeup](https://rkm0959.tistory.com/229?category=765103) for details.

<a id="zer0lfsr++"></a>

### zer0lfsr++
Category: **Crypto**  
Difficulty: ★★★★★  
Solved: **1 / 198**  
Tag: **LFSR, NFSR, FCA**  

#### Details

[Source](0CTF-TCTF-2021-Quals/zer0lfsr++/)

Same as **zer0lfsr+**, but there are no KDF, three initial states are independent. And we have a hint, can get the highest 16 bits of one of the three state.

#### Solution

- FCA attack recover full state of Generator3.
- FCA attack recover the LFSR state of Generator2 and use the hint to get the NFSR state of Generator2. 
- Similar attack in [[1]](#zer0lfsr-ref1) to recover full state of Generator1.

**Reference**  
<a id="#zer0lfsr-ref1">[1]</a> Berbain, Côme, Henri Gilbert, and Alexander Maximov. "Cryptanalysis of grain." International Workshop on Fast Software Encryption. Springer, Berlin, Heidelberg, 2006.  