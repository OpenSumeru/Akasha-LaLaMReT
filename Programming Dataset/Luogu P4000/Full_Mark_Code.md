```cpp
#include<bits/stdc++.h>
using namespace std;

#define ll long long
#define ull unsigned long long
unordered_map < ull , ll > circ; ll len;
int MOD , MX = 1 << 18; mt19937_64 rnd(time(0));
struct matrix{
	ll arr[2][2];
	matrix(){memset(arr , 0 , sizeof(arr));}
	ll* operator [](int x){return arr[x];}
	friend matrix operator *(matrix p , matrix q){
		matrix x;
		for(int i = 0 ; i < 2 ; ++i)
			for(int j = 0 ; j < 2 ; ++j)
				for(int k = 0 ; k < 2 ; ++k)
					x[i][k] += p[i][j] * q[j][k];
		for(int i = 0 ; i < 2 ; ++i)
			for(int j = 0 ; j < 2 ; ++j) x[i][j] %= MOD;
		return x;
	}
}G , T[2][1 << 18 | 1];

signed main(){
	static char str[300000003]; scanf("%s %d" , str + 1 , &MOD);
	T[0][0][0][0] = T[0][0][1][1] = T[1][0][0][0] = T[1][0][1][1] = 1;
	T[0][1][0][1] = T[0][1][1][0] = T[0][1][1][1] = 1;
	for(int i = 2 ; i <= MX ; ++i) T[0][i] = T[0][i - 1] * T[0][1];
	T[1][1] = T[0][MX]; for(int i = 2 ; i <= MX ; ++i) T[1][i] = T[1][i - 1] * T[1][1];
	while(1){
		ll x = (rnd() << 28 >> 28); matrix C = T[0][x & (MX - 1)] * T[1][x >> 18];
		ull val = ((1ull * C[0][0]) << 32) | C[0][1];
		if(circ.find(val) != circ.end()){len = abs(circ[val] - x); break;}
		circ[val] = x;
	}
	ll sum = 0; 
	for(int i = 1 ; str[i] ; ++i) sum = (sum * 10 + str[i] - '0') % len;
	cout << (T[0][sum & (MX - 1)] * T[1][sum >> 18])[0][1];
	return 0;
}
```