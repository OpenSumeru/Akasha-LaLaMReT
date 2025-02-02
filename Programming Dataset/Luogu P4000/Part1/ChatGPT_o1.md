**Analysis**

The core challenge is that \(n\) can be *huge*—up to \(10^{30{,}000{,}000}\)—so any direct attempt to iterate or to do straightforward fast‐exponentiation of the usual Fibonacci‐matrix \( \bigl(\begin{smallmatrix}1 & 1\\1 & 0\end{smallmatrix}\bigr)\) up to exponent \(n\) is utterly infeasible. Even \(\mathrm{O}(\log n)\) steps is too large when \(\log(n)\) is on the order of tens of millions.  

Instead, one exploits the fact that **Fibonacci numbers repeat modulo \(p\)** with some *period*, known as the **Pisano period** (often denoted \(\pi(p)\) or \(\rho(p)\)). Concretely:

\[
F_{k + \pi(p)} \equiv F_k \pmod{p}
\quad\Longrightarrow\quad
F_n \bmod p \;=\; F_{\,n \bmod \pi(p)} \bmod p.
\]

Hence, instead of computing \(\text{Fib}(n)\bmod p\) directly, we can:

1. **Determine the Pisano period** \(\pi(p)\).
2. **Compute** \(n \bmod \pi(p)\).  
3. **Compute** \(\text{Fib}\bigl(n \bmod \pi(p)\bigr)\bmod p\) using fast methods (like fast exponentiation of the Fibonacci Q‐matrix, but now only up to the exponent \(\le \pi(p)\), which is much smaller than \(n\)).

The key questions become:
- How do we *find* \(\pi(p)\) efficiently?
- How do we handle *very large* \(n\) when we do the reduction \(n \bmod \pi(p)\)?

Below is an outline of how to do each step in a typical solution.

---

## 1. Finding the Pisano Period \(\pi(p)\)

The length of the Fibonacci sequence’s repetition modulo \(p\) (the Pisano period) depends on \(p\).  In general:

- \(\pi(p)\) can be *at most* on the order of \(6p\).  
- A naive search for the period by simulating Fibonacci mod \(p\) until we see the repeat `0, 1` could take up to \(\mathrm{O}(p)\) steps—which is too large if \(p\) can be up to about \(2^{31}\approx 2\times 10^9\).

A more refined method uses the fact that:
1. If \(p\) is prime, \(\pi(p)\) divides either \(p - 1\) (if 5 is a quadratic residue mod \(p\)) or \(p + 1\) (if 5 is a nonresidue).
2. For a general \(p\) (not necessarily prime), you can factor \(p\) into prime powers and use the known result:
   \[
   \pi\bigl(\prod_i q_i^{\,a_i}\bigr) \;=\;\mathrm{lcm}\bigl(\,\pi(q_1^{a_1}),\,\pi(q_2^{a_2}),\,\dots\bigr),
   \]
   where each \(\pi(q^a)\) (the Pisano period modulo a prime power) can be computed using known formulas or by smaller searches.

Because \(p < 2^{31}\), **factoring** \(p\) is possible in reasonable time (worst case is roughly \(\mathrm{O}(\sqrt{p})\approx 46{,}000\) steps), and then each prime power factor is comparatively small.  After finding the prime factorization, one can assemble \(\pi(p)\) via:

- Computing \(\pi(q^a)\) for each prime power factor.
- Taking the LCM of these.

Finally, the result \(\pi(p)\) is at most on the order of a few times \(p\), so it is \(\lesssim 10^{10}\) in the worst cases.  That is still big, but it is *way* smaller than \(10^{30{,}000{,}000}\).

> **Why can we afford to do \(\bmod\ \pi(p)\)?**  
> - Once we know \(\pi(p)\), the exponent for Fibonacci is reduced from \(10^{30{,}000{,}000}\) down to \(\pi(p)\le 6p \approx 10^{10}\).  
> - A single fast‐exponentiation by squaring of the \(2\times 2\) Fibonacci‐matrix in \(\mathrm{O}(\log \pi(p))\) steps is around \(\mathrm{O}(\log(10^{10})) \approx 33\) matrix multiplications, which is easy to handle.

### Steps to find \(\pi(p)\) in practice

1. **Factor** \(p\) into \(p = \prod_i q_i^{\,a_i}\).  
2. For each prime power \(q^a\), **compute** \(\pi(q^a)\).  
   - For example, if \(q\neq 2,5\), we often use the fact that \(\pi(q)\) divides \(q-1\) or \(q+1\), depending on whether 5 is a quadratic residue mod \(q\). One finds the *exact* divisor \(d\) of either \(q-1\) or \(q+1\) that satisfies \(F_{d}\equiv 0\pmod{q}\) and \(F_{d+1}\equiv 1\pmod{q}\).  
   - For powers \(q^a\), a known approach extends the period from \(\pi(q)\) by a factor of \(q^{a-1}\) in specific ways, or you can do a more direct smaller search.  
3. Compute the **LCM** of these \(\pi(q^a)\) values to get \(\pi(p)\).  

In many Fibonacci‐mod‐prime problems, \(p\) itself is prime.  In that simpler setting, we only check one factor \(p\).  For a general \(p\), the prime‐factor approach is the usual route.

---

## 2. Reducing \(n\) Modulo \(\pi(p)\)

We must next compute \(n \bmod \pi(p)\).  The number \(n\) can be up to \(10^{30{,}000{,}000}\) digits, so we obviously cannot store \(n\) in a normal integer variable.  Instead, we read \(n\) as a **string**, then do a standard “big integer mod” technique:

```python
def big_mod(n_str, m):
    # n_str is the decimal string of a huge number n
    # m is an integer up to ~ 10^10 or so
    r = 0
    for ch in n_str:             # each ch is a digit in [0..9]
        r = (r*10 + int(ch)) % m
    return r
```

This yields \(n \bmod m\) in \(\mathrm{O}(\text{length}(n))\) time (i.e. linear time in the number of digits).  

Hence:  
\[
\text{exponent} = n \bmod \pi(p).
\]

---

## 3. Fast Fibonacci Computation for the Reduced Exponent

Once we have \(\ell = n \bmod \pi(p)\), we only need

\[
F_{\ell} \bmod p.
\]

There are multiple well‐known ways:

1. **Fast Doubling** (a specialized form of exponent‐by‐squaring for Fibonacci).  
2. **Matrix Exponentiation**:  
   \[
   \begin{pmatrix}
     F_{k+1} & F_k \\
     F_k     & F_{k-1}
   \end{pmatrix}
   =
   \begin{pmatrix}1 & 1\\[6pt]1 & 0\end{pmatrix}^k.
   \]
   We raise the \(2\times 2\) matrix to the \(\ell\)-th power modulo \(p\).  This takes \(\mathrm{O}(\log \ell)\) matrix multiplications; each multiplication is a few integer multiplications mod \(p\).

Because \(\ell \le \pi(p) \approx 6p \le 1.3\times 10^{10}\) in the worst case, \(\log \ell\) is at most \(\sim 33\), making the computation entirely feasible.

Finally, we output the \((1,0)\) (or \((0,1)\) depending on indexing) entry of that matrix, which equals \(F_{\ell}\).

---

## Putting It All Together

1. **Read** \(n\) (as a string) and \(p\) (as a 32‐bit integer).
2. **Handle edge cases** (e.g., if \(p=0\) or \(1\), though typically the problem constraints might exclude that).
3. **Factor** \(p\) into prime powers: \(p = \prod_i q_i^{\,a_i}\).
4. **Compute** each \(\pi(q_i^{\,a_i})\).  
5. **Take** \(\mathrm{LCM}\) of all those to get \(\pi(p)\).
6. **Reduce** \(n \bmod \pi(p)\) via the standard “big‐number mod” loop over the digits of \(n\).
7. **Compute** \(F_{n \bmod \pi(p)} \bmod p\) by fast‐doubling or matrix‐power mod \(p\).
8. **Output** the result.

---

## Example Walk‐Through

Although the sample inputs in the problem are small, here is how it works in principle:

### Sample Input #1
```
5
1000000007
```
- \(n = 5\), \(p = 10^9 + 7\).  
- For a small exponent, we do not even need the period. But the method still works:
  1. Factor \(p=10^9+7\). This is a known prime.  
  2. Because \(p\) is prime and \(10^9+7\equiv 2 \pmod{5}\), 5 is not a quadratic residue, so \(\pi(p)\) divides \(p+1 = 1000000008\).  
  3. We would find the exact divisor \(d\) of \(10^9+8\) satisfying the Fibonacci boundary conditions. That yields \(\pi(10^9+7)\).  
  4. Meanwhile, \(n=5\) mod \(\pi(p)\) is just 5 because \(\pi(p)\) is certainly \(\ge 5\).  
  5. Then compute Fib(5) mod \(10^9+7\) = 5.

### Sample Input #2
```
10
1000000007
```
Similarly, output is 55.

Of course, these examples are tiny \(n\). Where the method *really* shines is something like:

```
999999999999999999999999999999999  (a huge n)
1000000007
```
You would:
- Still factor \(p=10^9+7\) (prime).
- Find the Pisano period \(\pi(10^9+7)\).
- Reduce \(n\) mod \(\pi(p)\).
- Compute \(\mathrm{Fib}(n \bmod \pi(p)) \bmod p\).

All steps remain efficient because \(\pi(10^9+7)\) is at most around \(2\times 10^9\), so we only do \(\mathrm{O}(1)\) or \(\mathrm{O}(\log \pi(p))\) final arithmetic steps after factoring.

---

## Final Sketch of a Solve Function

A high‐level pseudocode combining everything:

```python
def solve():
    import sys
    
    # Read input as strings
    n_str = sys.stdin.readline().strip()
    p = int(sys.stdin.readline().strip())
    
    # Special case if p <= 1? (Depends on problem constraints; often omitted.)
    if p == 0:
        print(0)
        return
    if p == 1:
        print(0)
        return
    
    # 1) Factor p into prime powers
    prime_powers = factor_into_prime_powers(p)
    
    # 2) Compute the Pisano period for each prime power
    sub_periods = []
    for (q, a) in prime_powers:
        sub_periods.append( pisano_prime_power(q, a) )
    
    # 3) LCM of sub_periods
    from math import gcd
    def lcm(a, b):
        return a // gcd(a,b) * b
    
    pi_p = 1
    for sp in sub_periods:
        pi_p = lcm(pi_p, sp)
    
    # 4) Compute n mod pi_p via big integer mod
    n_mod_pi_p = 0
    for ch in n_str:
        n_mod_pi_p = (n_mod_pi_p*10 + (ord(ch)-ord('0'))) % pi_p
    
    # 5) Compute Fib(n_mod_pi_p) mod p
    #    via fast matrix exponentiation or fast doubling
    ans = fib_mod(n_mod_pi_p, p)
    
    # 6) Print result
    print(ans)
```

Where:

- `factor_into_prime_powers(p)` returns a list of \((q, a)\) such that \(p = \prod_i q_i^{\,a_i}\).
- `pisano_prime_power(q, a)` returns \(\pi(q^a)\).  
- `fib_mod(k, p)` does fast doubling or matrix exponentiation to get \(F_k \bmod p\).

> **Note.** Implementing `pisano_prime_power` or even `pisano_prime(q)` for large prime \(q\) can itself be involved (one typically factors \(q-1\) or \(q+1\), then checks divisors). That is an entirely standard approach in advanced Fibonacci‐mod problems, but it is not “one line of code.”  

---

## Summary

1. **Pisano period** \(\pi(p)\) is the key to handling huge \(n\).  
2. **Factor** \(p\) and use known number‐theory results to find \(\pi(p)\).  
3. **Reduce** \(n\) modulo \(\pi(p)\) by reading \(n\) as a string and doing big‐integer mod.  
4. **Compute** Fibonacci of that reduced exponent mod \(p\) with fast exponentiation of the \(2\times 2\) matrix (or fast‐doubling).

This approach lets you handle \(n\) of size up to \(10^{30{,}000{,}000}\) (or even bigger) so long as \(p < 2^{31}\).