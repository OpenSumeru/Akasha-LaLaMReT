# Fibonacci Sequence (Modulo Calculation)

## Problem Description

Everyone knows that the Fibonacci sequence follows these rules:

- \( f_1 = 1 \)
- \( f_2 = 1 \)
- \( f_n = f_{n-1} + f_{n-2} \) (for \( n \geq 2 \), where \( n \) is an integer)

Your task is to compute \( f_n \mod p \).

## Input Format

- **Line 1:** An integer \( n \).
- **Line 2:** An integer \( p \).

## Output Format

- **Line 1:** The value of \( f_n \mod p \).

## Example #1

### Sample Input #1

```
5
1000000007
```

### Sample Output #1

```
5
```

## Example #2

### Sample Input #2

```
10
1000000007
```

### Sample Output #2

```
55
```

## Hints

For **100%** of the test cases:
- \( n \leq 10^{30,000,000} \)
- \( p < 2^{31} \)