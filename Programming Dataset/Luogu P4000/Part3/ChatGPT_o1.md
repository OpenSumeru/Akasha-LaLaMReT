Below is an outline of **why** your current solution times out for large \(p\) and **how** to fix it so it can handle the biggest possible inputs (including the case \(n\) having tens of millions of digits).

---

## Why Your Current Code Gets TLE

1. **Naive Pisano-Period Computation**  
   In your code, `pisano_period(p)` does a simple, step-by-step search for the period up to \((p \times p)\) iterations. For \(p\) near \(2^{31}\), this can be on the order of \(10^{18}\) operations—which is far too large for typical time limits.

2. **No Problem with Fast Doubling Itself**  
   Once you have \(n \bmod T\), you use fast doubling to compute \(F(n \bmod T) \bmod p\). Fast doubling is fine for relatively small exponents (like \(n \bmod T\)), but the crux is you cannot afford to iterate up to \(p^2\) to find the period in the first place.

Hence, your code may pass smaller test cases (where \(p\) is small enough that \(p^2\) is not too big) but will time out when \(p\) is large.

---

## The Key Idea to Fix

**Stop trying to find the Pisano period for large \(p\).** Instead, directly compute \(F(n) \bmod p\) using a method that can handle huge \(n\) (with up to \(3\times10^7\) digits) **without** linear iteration in \(n\).

### But How Can We Possibly Do Fibonacci of Such a Huge \(n\)?  

There is a well-known technique to compute \(A^N \mod M\) even when \(N\) is extremely large, using the exponent’s decimal representation:

\[
N = \underbrace{( \dots ((0 \times 10 + d_1) \times 10 + d_2 ) \times 10 + \cdots }_{\text{reading digits }d_i}.
\]

Usually, for an integer base \(a\), we do:

```
result = 1
for each digit d in N_str:
    // old exponent -> old_exponent * 10 + d
    // so  a^(old_exponent * 10 + d) = (a^(old_exponent))^10 * a^d
    result = pow(result, 10)  mod M
    result = result * pow(a, d)  mod M
```

We can adapt this trick to **Fibonacci** by viewing it as exponentiation of the 2×2 Fibonacci matrix
\[
M = \begin{pmatrix}0 & 1 \\ 1 & 1\end{pmatrix}.
\]
Remember that
\[
M^n = 
\begin{pmatrix}
F(n-1) & F(n) \\
F(n)   & F(n+1)
\end{pmatrix}
\]
(if you index \(F(0)=0, F(1)=1\)).  
So \(F(n)\) is exactly the top-right (or bottom-left) entry of \(M^n\).

Thus:

1. Let `R` = the 2×2 identity matrix \(I\).
2. For each digit \(d\) in the decimal string of \(n\):
   - Update `R = R^10 \mod p` (which is done by fast exponentiation of a 2×2 matrix—but “exponent by 10” only takes a handful of multiplications, about 3 or 4).
   - Then update `R = R × (M^d) \mod p` (where we can precompute \(M^0, M^1, \dots, M^9\) and multiply by the correct one in **1** multiplication).
3. After processing all digits, `R = M^n (mod p)`.
4. The answer is `R[0][1]` or `R[1][0]`, whichever holds \(F(n)\).

### Complexity Analysis

- For each digit of \(n\), you do:
  - **3 or 4** matrix multiplications to get `R^10`.  
  - **1** matrix multiplication to multiply by \(M^d\).  
- Each 2×2 multiplication mod \(p\) is just 8 multiplications + 4 additions mod \(p\), which is fast in C++ if done carefully (inline, etc.).  
- If \(n\) has \(D\) decimal digits (worst case \(D = 3\times10^7\)), you end up with about \(4 \times D = 1.2\times10^8\) matrix multiplications in the worst scenario.

In a fast C++ environment (and depending on the judge’s time limit), **this is borderline but can be optimized** enough to pass when carefully implemented (fast I/O, efficient inline ops, no big overhead). It is definitely more practical than trying to find the Pisano period with \(p\) up to \(2^{31}\).

---

## A Reference Implementation

Below is a **simplified** version showing the core matrix-approach steps. It omits a lot of minor optimizations (like inlining, etc.) but illustrates the structure you need. You can refine/optimize it as necessary to pass all tests.

```cpp
#include <bits/stdc++.h>
using namespace std;

static const int MAT_SIZE = 2;

// Multiply 2x2 matrices A, B under mod p
void matMul(const long long A[MAT_SIZE][MAT_SIZE],
            const long long B[MAT_SIZE][MAT_SIZE],
            long long C[MAT_SIZE][MAT_SIZE],
            long long p) {
    // Using 64-bit to avoid overflow before taking mod
    // (p < 2^31, so intermediate fits in 64-bit)
    long long temp[MAT_SIZE][MAT_SIZE];
    memset(temp, 0, sizeof(temp));
    for(int i = 0; i < MAT_SIZE; i++){
        for(int j = 0; j < MAT_SIZE; j++){
            __int128 sum = 0; // or long long but carefully
            for(int k = 0; k < MAT_SIZE; k++){
                sum += (__int128)A[i][k] * B[k][j];
            }
            temp[i][j] = (long long)(sum % p);
        }
    }
    memcpy(C, temp, sizeof(temp));
}

// Compute A^exp (mod p) for 2x2 matrix A using fast exponentiation
// (Here exp <= 10, so we can just do repeated squaring or unrolled.)
void matPow(long long A[MAT_SIZE][MAT_SIZE], int exp, long long p) {
    // We'll raise A in-place to the power exp.
    // Because exp <= 10 in this step, we can do
    // repeated squaring or just do naive multiply in a loop.

    long long R[MAT_SIZE][MAT_SIZE] = {{1,0},{0,1}}; // identity
    long long base[MAT_SIZE][MAT_SIZE];
    memcpy(base, A, sizeof(base));

    int e = exp;
    while(e > 0){
        if(e & 1){
            long long tmp[MAT_SIZE][MAT_SIZE];
            matMul(R, base, tmp, p);
            memcpy(R, tmp, sizeof(tmp));
        }
        // square base
        long long tmp2[MAT_SIZE][MAT_SIZE];
        matMul(base, base, tmp2, p);
        memcpy(base, tmp2, sizeof(tmp2));

        e >>= 1;
    }

    // Copy R back into A
    memcpy(A, R, sizeof(R));
}

// Matrix exponent for M^digit, digit in [0..9], can be precomputed
void precomputeMD(long long M[10][MAT_SIZE][MAT_SIZE], long long p) {
    // M[1] = original fib matrix
    // M[2] = M[1]^2
    // ...
    // M[9] = M[1]^9
    // M[0] = identity
    long long fibMat[MAT_SIZE][MAT_SIZE] = {{0,1},{1,1}};
    // identity
    for(int i=0; i<MAT_SIZE; i++){
        for(int j=0; j<MAT_SIZE; j++){
            M[0][i][j] = (i == j ? 1 : 0);
        }
    }
    // M[1] = fibMat
    memcpy(M[1], fibMat, sizeof(fibMat));

    for(int d = 2; d <= 9; d++){
        matMul(M[d-1], fibMat, M[d], p);
    }
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Read n as string
    string n_str;
    cin >> n_str;

    long long p;
    cin >> p;
    if(p == 1){
        // mod 1 => always 0
        cout << 0 << "\n";
        return 0;
    }

    // The Fibonacci matrix
    long long fibMat[MAT_SIZE][MAT_SIZE] = {{0,1},{1,1}};

    // We'll store R = I (2x2 identity), accumulate R = R^(n)
    long long R[MAT_SIZE][MAT_SIZE] = {{1,0},{0,1}};

    // Precompute M^d for d=0..9
    static long long Mpow[10][MAT_SIZE][MAT_SIZE];
    precomputeMD(Mpow, p);

    for(char c : n_str){
        int digit = c - '0';

        // Step 1: R = R^10
        {
            // Make a copy so we don't overwrite R while computing
            long long tmp[MAT_SIZE][MAT_SIZE];
            memcpy(tmp, R, sizeof(tmp));
            matPow(tmp, 10, p);   // raise tmp to power 10
            memcpy(R, tmp, sizeof(tmp));
        }

        // Step 2: R = R * (fibMat^digit)
        {
            // fibMat^digit is precomputed in Mpow[digit]
            long long tmp2[MAT_SIZE][MAT_SIZE];
            matMul(R, Mpow[digit], tmp2, p);
            memcpy(R, tmp2, sizeof(tmp2));
        }
    }

    // After processing all digits, R = fibMatrix^(n).
    // So F(n) mod p is R[0][1] if we consider F(0)=0, F(1)=1.
    cout << (R[0][1] % p) << "\n";
    return 0;
}
```

### Notes & Optimizations

- We only do **4 matrix multiplications per digit**:
  - 3 (or fewer) to do exponent by 10 (actually it can be done in \(\log_2(10)\approx3\) multiplications).
  - 1 to multiply by \(M^d\).
- `matPow(tmp, 10, p)` can be streamlined even further since 10 is a small constant—one might unroll it or do repeated squaring explicitly:
  ```cpp
  // to compute tmp^10 quickly:
  // tmp^2, tmp^4, tmp^8, multiply as needed
  ```
- Precomputing `M^d` for d=0..9 allows you to multiply by them in just one 2×2 multiplication.
- Make sure all multiplications and additions are under modulo `p`.
- Watch out for 64-bit overflow if `p` can be up to 2^31. Intermediate multiplications can reach up to ~2^62, so either use 128-bit built-ins (`__int128` in C++) or do careful 64-bit multiplication with `uint64_t` + checks.

If you implement these details carefully (e.g. inline the matrix multiplication, use `-O2` or `-O3`, etc.), you have a good shot at getting all AC, including the largest test cases where \(p\) is near \(2^{31}\) and \(n\) has tens of millions of digits.

---

## Summary

- **Remove** your `pisano_period` and all logic that depends on it.  
- **Directly compute** \(F(n) \bmod p\) via matrix exponentiation “digit-by-digit in base 10.”  
- Implement the 2×2 matrix multiplication carefully and efficiently.  
- This avoids the \(\mathbf{O}(p^2)\) trap altogether and should pass all test files if optimized.  

That is the standard, proven approach to handle \(\mathbf{n \le 10^{30,000,000}}\) with \(\mathbf{p < 2^{31}}\) in competitive programming. Once you switch to this method, you can get **all AC** instead of TLE.