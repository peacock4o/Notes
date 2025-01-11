crypto_102
==========
- Category: Learning
- Tags: 
- Created: 2025-01-10T17:49:27-08:00

*Presented by Dhruv Ashok*

## Review
- Symmetric
	- Both parties use a shared key
	- AES, ChaCha20, DES
- Asymmetric
	- Both parties have a public/private key pair
	- RSA, Diffie Hellman, Elliptic Curves

## Trapdoor Functions

- Some computation that is reasonably efficient one way, but not the other
- All public key crypto needs smething like this
	- Allows us to link public/private key but cannot derive one from another
- You know what mod is already.

## Asymmetric Cryptosystems

- Starting with RSA
	- Pick two large primes, p and q
		- Best way to get this is with Miller-Ray(something), which is probibalistic
	- Let N = p * q and let phi(N) = (p-1) * (q-1)
		- AKA totient
	- Now, pick an integer e such that e < phi(N) and gcd(e, phi(N)) = 1
		- Often, people choose a prechosen e instead of generating each time
	- The pair (N, e) is our public key
	- To get the corresponding private key:
		- To get the corresponding private key:
			- Let d =- e^-1 (mod N)
				- Find e such that d*e =- 1 (mod N)
			- The pair (phi(N), d) is the private key
- Proof
	- 
- CTF Examples
	- Factoring N
		- This gives us p, q, which gives us the totient (p-1, q-1), which gives us the unknown part of our decryption key.
		- Check out ``factordb.com``
		- Often, they just give you N
	- Bad prime selection
		- Never pick two primes that are too close together, vulnerable to Fermat's attack
		- See "hardcopy" challenge - don't establish a relationship between p and q.
			- This can be a direct relationship or just nearby primes
	- Decryption oracle
	- Small value of e
		- Defeated using coppersmiths attack
		- Use e=65537
	- Small d (Wiener's attack)
		- Check out wiener.py
	- 20 years of attacks on RSA - check out this paper!

## Diffie-Hellman

- Remember the paint example
	- AKA discrete logarithm problem
- Pick g, a generator (typically g=2 or g=3)
- Let p be a large prime (~2048 bits)
- Choose a < p-1 (this is your private key)
- Choose A = g ^ a (mod p) (this is your public key)
- Given g and A = g^a, tje DLP is the task of finding a
	- THis is the trapdoor problem for well-chosen parameters.
