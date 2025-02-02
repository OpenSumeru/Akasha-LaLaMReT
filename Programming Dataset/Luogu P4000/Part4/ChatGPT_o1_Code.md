```cpp
#include <bits/stdc++.h>
using namespace std;

static const int MX = 1 << 18;   // We'll build tables up to 2^18
int MOD;                        // We'll read p into here

// A simple 2x2 matrix struct with operator* under (mod MOD):
struct matrix {
    long long m[2][2];
    matrix() { memset(m, 0, sizeof(m)); }
    long long* operator[](int r) { return m[r]; }
};

matrix operator*(const matrix &A, const matrix &B) {
    matrix R;
    for(int i=0; i<2; i++){
        for(int j=0; j<2; j++){
            __int128 sum = 0;  // to avoid 64-bit overflow before mod
            for(int k=0; k<2; k++){
                sum += (__int128)A.m[i][k] * B.m[k][j];
            }
            R.m[i][j] = (long long)(sum % MOD);
        }
    }
    return R;
}

static matrix T[2][MX + 1];  
// T[0][i] = fibMatrix^i         for i in [0..MX]
// T[1][j] = fibMatrix^(j*MX)    for j in [0..MX]

// The Fibonacci transition matrix
// F_{n+1} = F_n + F_{n-1}, etc.
matrix fibMat;

// We'll store "signatures" in a map: val -> exponent
// The val we store is a 64-bit that encodes the top row of the matrix.
unordered_map<unsigned long long, long long> circ;

// Convert top row of matrix M into a 64-bit key: (M[0][0] << 32) | M[0][1]
static inline unsigned long long makeKey(const matrix &M) {
    // Each entry fits in 32 bits because MOD < 2^31
    return ( (unsigned long long)(M.m[0][0]) << 32 )
         | ( (unsigned long long)(M.m[0][1])       );
}

int main(){
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Read input
    static char n_str[300000002]; // enough to hold up to ~300 million digits
    cin >> n_str >> MOD;
    if (MOD == 1) {
        cout << 0 << "\n";
        return 0;
    }

    // 1) Build the base fibMat = {{0,1},{1,1}}
    fibMat.m[0][0] = 0; fibMat.m[0][1] = 1;
    fibMat.m[1][0] = 1; fibMat.m[1][1] = 1;

    // 2) Precompute T[0][i] = fibMat^i for i in [0..MX]
    //    T[0][0] = identity
    matrix I; 
    I.m[0][0] = I.m[1][1] = 1; // Identity
    T[0][0] = I;
    for(int i = 1; i <= MX; i++){
        T[0][i] = T[0][i - 1] * fibMat; 
    }
    // 3) T[1][0] = identity, T[1][1] = fibMat^MX, T[1][2] = (fibMat^MX)^2 = fibMat^(2*MX), etc.
    T[1][0] = I;
    T[1][1] = T[0][MX];
    for(int i = 2; i <= MX; i++){
        T[1][i] = T[1][i - 1] * T[1][1];
    }

    // 4) Use a random approach to detect the period
    //    We'll generate random exponents x and compute M^x mod p;
    //    If we see the same (top row) again, the difference in exponents is
    //    a multiple of the real period. We'll call that "len" below.
    mt19937_64 rng((unsigned long long)time(nullptr));
    long long period = 0;

    while(true){
        // Pick a random 64-bit "x" but keep it smaller than, say, 2^36
        // (so we don't overflow in x + y differences, etc.)
        // Or you can just mask it to some range.
        unsigned long long x = rng() & ((1ULL << 36) - 1ULL);

        // Compute M^x quickly using the T tables
        // x = (x >> 18)*MX + (x & (MX-1))
        unsigned long long low = (x & (MX - 1));
        unsigned long long high = (x >> 18);

        matrix Mx = T[0][low] * T[1][high];
        unsigned long long key = makeKey(Mx);

        auto it = circ.find(key);
        if (it != circ.end()) {
            // Found a collision
            long long oldx = it->second;
            period = (x > oldx ? x - oldx : oldx - x);
            break;
        }
        // Otherwise record it
        circ[key] = x;
    }

    // 5) Now we reduce big N mod 'period'
    //    N can have up to ~300 million digits, so do normal "mod" with base 10
    long long r = 0;
    for (int i = 0; n_str[i]; i++){
        int digit = n_str[i] - '0';
        r = (r * 10 + digit) % period;
    }

    // 6) Finally, compute fib(r) mod p
    //    i.e. compute fibMat^r, then the top-right element is F(r).
    {
        unsigned long long low = (r & (MX - 1));
        unsigned long long high = (r >> 18);
        matrix Mr = T[0][low] * T[1][high];
        // F(r) = Mr[0][1]
        cout << Mr.m[0][1] << "\n";
    }

    return 0;
}

```