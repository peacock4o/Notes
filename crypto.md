crypto
======
- Category: Learning
- Tags: 
- Created: 2024-12-15T18:03:52-08:00






## Euclidean algorithm
- Find the GCD of two numbers.
- euclid(a, b) = euclid(b, r), where a = bq + r
- Proof -> `https://www.youtube.com/watch?v=H_2_nqKAZ5w`
	- Theoreom: if a = bq + r, then gcd(a,b) = gcd(b,r)
	- A divisor d of a and b must be a divisor of aq + b. d|a, d|b, d|(a +/- bq)
	- Remember our original theoreom. a = bq + r, so r = a - bq (manipulate the divisor statement above a little bit to make this work)
	- This means that d|r
	- Let's now take another divisor e, this time of b and r
	- e|b, e|r. Like above, we know that this means e|(bq +/- r)
	- This means that e|a
	- d is a common divisor of a and b if and only if d is a common divisor of b and r
	- The set of common divisors of a and b must be the same as that of the set of b and r, so the GCD in each set is the same, so the GCD of a, b is the same as b, r
- Coded iteratively, but it's a recursive algorithm by nature

## Extended Euclidean algorithm
- Find the GCD of two numbers, but document our steps along the way.
- Linear Diophantine equation -> ax + by = c.
	- We're interested in solving for the integer solutions - we already have a,b,c. Need to find x, y with x, y in set Z
	- Set Z - all integers. This is also what Diophantine refers to - we're only interested in integer solutions
- In this situation, we're trying to solve the linear diophantine equation ax + by = gcd(a,b)
- "Add and subtract copies of a and b together until we get the gcd" - like the Euclidean algorithm
	- Remember - as you do Euclid. algo., the GCD of each consecutive pair is the same
	- Ex. gcd(196, 180) -> 196 = 180 * 1 + 16 -> 180 = 16 * 11 + 4 -> 16 = 4 * 4 + 0
- For EEA, our challenge would be to solve 196x + 180y = 4 = gcd(196, 180)
- To achieve this, we start off with normal EA. 196 = 1 * 180 + 16
	- Then, represent the first two numbers as how many of x and y are needed to make that value.
	- (x=1,y=0) = 1 * (x=0,y=1) + 16
		- We can solve the last value by evaluating the "cards" like an equation. So 16 = (x=1,y=-1)
- Then, carry on to the next step. Do normal EA, but for EEA, we can reuse the cards for the previous values and solve for the new unknown.
	- 180 = 11 * 16 + 4
	- (x=0, y=1) = 11 * (x=1,y=-1) + 4
	- Therefore, 4 = (x=0,y=1) - 11(x=1,y=-1) = (x=0,y=1) + (x=-11,y=11) = (x=-11,y=12)
- NOTE!
	- This only works for ax + by = gcd(a,b)
	- Only one solution for this lin. diaph. equation
- Proof -> `https://www.youtube.com/watch?v=IwRtISxAHY4`

## Modular arithmetic
- a = b mod m
	- Read this as:
		- "a % m = b"
		- "b is the remainder of a / m"
		- "a becomes be when modded by m"
- When you hear "modulus", this refers to m, the "degree" of modulus (so to speak)
	- Imagine we've picked a prime modulus p
	- The set of Z mod p define a field, denoted F sub p
	- Set of integers 0, 1, ..., p-1. Under addition and subtraction, you have inverse elements b+ and b* for every a in the set where a + b* = 0 and a * b* = 1. 
		- Here, 0 and 1 are the "identity element" and are different.

## Fermat's little theorem
- If p is a prime number and p !| a, then a^(p-1) is congruent to 1 (mod p)
	- Ex. 273246787654^65536 mod 65537 = 1
	- Ex. if p=5, a^(p-1) = a^4. 2^4 cong. to 1 mod 5, like 3^4, like 4^4. 
		- Not 5^4 or 10^4, though! Divides by 5, so 
- When modding, if p is a prime number, a^p = a mod p.
	- Ex. 3^17 mod 17 = 3
- We can also use this in another way - finding other exponents
	- Ex. what's the remainder when you divide 3^100000 by 53?
	- Start with our mod value 53, so p = 53.
	- 53 !| 3, so a^(p-1) = 3^(53-1) = 3^52.
	- Then, raise 3^52 to the highest power less than 3^100000. Find this with division - 100000/52 = 1923 r4.
	- 3^(52 * 1923) -> 3^99996 cong. 1 mod 53. 
	- Our remainder was 4, so we multiply each side by 3^4
		- Can multiply both sides of a congruence statement by a value. It's just like an equals sign.
	- 3^100000 cong. to 3^4 mod 53 -> 3^100000 cong. to 81 mod 53 -> 3^100000 cong. to 28 mod 53. Remainder is 28
- Proof -> `https://www.youtube.com/watch?v=w0ZQvZLx2KA`
	- Proving theorem that if p is prime and p !| a, then a^(p-1) cong. 1 mod p
	- Assume p is a prime number and p !| a (as stated in theorem definition)
		- Ex. p=11
	- Every integer in Z is congruent to one of 0, 1, 2, ..., p-1 ()
		- Ex. every integer mod 11 has congruence between 0 and 10.
	- Remember that we assumed p !| a, so our field is actually 1 ... p-1, since a as a multiple of p would mod to 0
		- Ex. a=22, p=11 is ineligible based on our assumptions.
	- Multiply our working field of congruence classes by any a. 
		- Now working with a, 2a, 3a, ..., (p-1)a
		- ^^ IMPORTANT! We need to show that this is just a rearrangement of the original congruence classes (1, 2, ..., )
		- Case 1: None of the integers are congruent to 0 in this new set.
			- Suppose r * a cong. to 0 mod p. (!) r is from our new group, up to p-1
			- If r * a (some multiple of a) was cong. to 0 mod p, this would mean that p | r * a, which is impossible, since we know p !| a and r is smaller than p (why? idk)
			- p !| a and r !| a. Therefore, 0 cannot be a congruence class.
		- Case 2: These are distinct values; no two unique a are congruent to each other
			- Pick two values, r * a and s * a from our new group
			- a


## Inverse of a number

- We can work within the finite field F sub p, adding and multiplying elelemnts to obtain another element of the field.
- For all elements g in the field, there is a unique integer d such that g * d cong. to 1 mod p
	- We call d the multiplicative inverse of g. 
	- Ex. 7 * 8 = 56 cong. to 1 mod 11
- How can we use little theorem to find the inverse element of a value?
	- Little theorem: given value a and modulus p, a^(p-1) cong. to 1 mod p 
	- a^(p-1) cong. 1 mod p
	- Trying to find a^(-1)
	- a^(p-1) cong. 1 mod p
	- a^(p-1) * a^(-1) cong. a^(-1) mod p
	- a^(p-2) * a * a^(-1) cong. a^(-1) mod p
	- a^(p-2) cong. a^(-1) mod p
	- Meaning a^(-1) = a^(p-2) % p

## Root modulo

- 

