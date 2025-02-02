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