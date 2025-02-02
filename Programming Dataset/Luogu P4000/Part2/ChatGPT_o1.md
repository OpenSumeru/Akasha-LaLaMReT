Below is a **complete C++ solution** that computes \(F_n \bmod p\) for extremely large \(n\) (up to \(10^{30{,}000{,}000}\) digits!) and for \(p < 2^{31}\). 

The key ideas are:

1. **Pisano Period**  
   The Fibonacci sequence taken modulo \(p\) is **periodic**. The length of this period is often denoted \(\pi(p)\). Once you know \(\pi(p)\), you only need to compute \(n \bmod \pi(p)\), because  
   \[
   F_n \bmod p \;=\; F_{\,n \bmod \pi(p)} \bmod p.
   \]

2. **Finding \(\pi(p)\)** (Naive approach)  
   A well-known method is to generate Fibonacci numbers modulo \(p\) until the pair \((0, 1)\) reappears. That signals the start of a new cycle, so the index at which \((0,1)\) reappears is the Pisano period \(\pi(p)\). In the worst case, this can be up to \(6p\) (for certain \(p\)), which can be quite large. However, this is the most straightforward method.

3. **Modding Large \(n\) by \(\pi(p)\)**  
   Since \(n\) can have up to \(10^{30{,}000{,}000}\) digits, we **cannot** store it in standard integer types. Instead, we read \(n\) as a string and compute  
   \[
   n \bmod \pi(p)
   \]  
   by processing each digit in a standard “big number to modulus” fashion:
   \[
   n_{\text{mod}} = \Bigl(\bigl(n_{\text{mod}} \times 10\bigr) + \text{(next digit)}\Bigr) \bmod \pi(p).
   \]

4. **Fast Fibonacci Computation (Fast Doubling)**  
   After reducing \(n\) to \(n_{\text{mod}} = n \bmod \pi(p)\), we can compute \(F_{n_{\text{mod}}} \bmod p\) in \(O(\log n_{\text{mod}})\) time using a “fast doubling” method (or matrix exponentiation).  

> **Important Note on Performance**  
> - If \(p\) is very large (close to \(2^{31}\)), the naive search for the Pisano period \(\pi(p)\) can take billions of steps in the worst case, which might be **too large for typical time limits**. In many practical/competitive-programming environments, one would implement more advanced techniques (e.g., factorizing \(p\), using properties of prime moduli, or using more elaborate period-finding algorithms).  
> - For smaller \(p\), this naive approach is usually acceptable.  
> - The problem as stated does not specify time limits or environment constraints, so we provide the most straightforward “textbook” solution below.

---

## C++ Code

```cpp
#include <bits/stdc++.h>
using namespace std;

/**
 * Computes the Pisano period for modulus p by naive simulation.
 * Returns the length of the period pi(p).
 *
 * NOTE: For very large p, this can be very slow in practice!
 */
long long pisano_period(long long p) {
    long long prev = 0;
    long long curr = 1;

    // The period always starts with 0, 1
    // We look for the next occurrence of (0, 1)
    for (long long i = 0; i < p * p; i++) {
        long long temp = (prev + curr) % p;
        prev = curr;
        curr = temp;

        // Check if we restarted the sequence
        if (prev == 0 && curr == 1) {
            return i + 1;  // +1 because i is zero-based
        }
    }
    // Theoretically should never reach here for a valid p > 0
    return -1;
}

/**
 * Fast Doubling method to compute Fibonacci(n) mod p.
 * Returns pair(F(n), F(n+1)) mod p.
 */
pair<long long, long long> fib_fast_doubling(long long n, long long p) {
    if (n == 0) {
        return {0, 1};
    }
    // Recursively get F(k), F(k+1) where k = n // 2
    auto half = fib_fast_doubling(n >> 1, p);
    long long a = half.first;   // F(k)
    long long b = half.second;  // F(k+1)

    // F(2k) = F(k) * [2*F(k+1) − F(k)]
    // F(2k+1) = F(k+1)^2 + F(k)^2
    long long c = (a * ((2LL * b - a + p) % p)) % p; 
    long long d = ((a * a) % p + (b * b) % p) % p;

    if (n & 1) {
        // n is odd
        return {d, (c + d) % p};
    } else {
        // n is even
        return {c, d};
    }
}

/** 
 * Computes F(n) mod p using Fast Doubling.
 */
long long fibonacci_mod(long long n, long long p) {
    return fib_fast_doubling(n, p).first;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Read n as a string, since it can be extremely large
    string n_str;
    cin >> n_str;

    long long p;
    cin >> p;

    // Special case if p == 1 (mod 1 -> always 0)
    if (p == 1) {
        cout << 0 << "\n";
        return 0;
    }

    // 1) Find the Pisano period T = pi(p)
    long long T = pisano_period(p);

    // 2) Compute n mod T, where n is given as a string
    long long n_mod_T = 0;
    for (char c : n_str) {
        // For each digit, build up n_mod_T in base 10
        n_mod_T = (n_mod_T * 10 + (c - '0')) % T;
    }

    // 3) Compute Fibonacci(n_mod_T) mod p
    long long answer = fibonacci_mod(n_mod_T, p);

    // Print the result
    cout << answer << "\n";

    return 0;
}
```

### How It Works

1. **Pisano Period Search (`pisano_period`)**  
   - Initialize `prev = 0`, `curr = 1`.  
   - Repeatedly compute the next Fibonacci number modulo \(p\).  
   - Once we see the pattern `(0, 1)` again, we know the period length.

2. **Reduce \(n\) (given as a string) modulo \(\pi(p)\)**  
   - We keep a running remainder `n_mod_T`.  
   - For each digit `d` of \(n\), we do:
     \[
       n_{\text{mod}} = (n_{\text{mod}} \times 10 + d) \bmod T.
     \]

3. **Compute the Fibonacci of that reduced exponent**  
   - Use **Fast Doubling**:  
     \[
       \begin{aligned}
         F(2k)   &= F(k) \,\bigl[\,2F(k+1)-F(k)\bigr],\\
         F(2k+1) &= F(k+1)^2 + F(k)^2.
       \end{aligned}
     \]  
   - This method computes \(F(n)\bmod p\) in \(O(\log n)\) steps, which is very efficient once \(n\) is reduced.

---

### Caveats / Further Optimizations

- For very large \(p\) (close to \(2^{31}\)), the naive period-finding loop can be up to \(6p\) iterations, which might be billions of steps and *could* be too slow in a strict time-limit environment. In that scenario, one typically implements:
  - **Prime factorization** of \(p\) (feasible for \(p < 2^{31}\) using fast methods).
  - Combine periods via **Chinese Remainder Theorem** if \(p\) factors into prime powers.
  - Or use known properties for primes (like \(\pi(p)\) divides \(p - 1\) or \(p + 1\) depending on whether 5 is a quadratic residue mod \(p\), etc.).

- If your environment has generous time/memory limits or \(p\) is typically smaller, the above solution suffices.

---

**Summary**:  

1. Read \(n\) as string, read \(p\).  
2. Compute the Pisano period \(\pi(p)\).  
3. Compute \(r = n \bmod \pi(p)\) via string processing.  
4. Compute \(F_r \bmod p\) using fast doubling.  
5. Output the result.  

This solves the problem of finding the \(n\)-th Fibonacci number modulo \(p\) even when \(n\) has up to \(10^{30,000,000}\) digits!