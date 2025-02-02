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