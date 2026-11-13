## Introduction

We discuss the 2025 IOCCC's [dogon](https://www.ioccc.org/2025/dogon/index.html) winning entry and present a solution to the fun challenges 0 and 1.

Written in 2026 by Antoine Mathys.

## First steps
We format the code and substitute the defines.

We assume that `e` is initialized to `0`, that `calloc()` is successful and that the machine uses 32 bits little endian integers.

We then systematically apply the following transformations:
- Since after `calloc()`, `e` is non-zero, we replace `!!e` with `1` and `!e` with `0`.
- We replace `x == x` with `1`, `x < x` with `0` and `x > x` with 0.
- We replace `~x` with `-x - 1`, `-~x` with `x + 1` and `~-x` with `x - 1`.
- We perform logical and arithmetic operations on constants.
- We replace `(e + i)[j]` with `e[i + j]` and `e[i][e + j]` with `e[e[i] + j]`.

`calloc()` initializes the elements of `e` to `0`. Early on, the fragment `*((char *)(e + 2)) = 1` thus becomes equivalent to `e[2] = 1`, because we assume little endian encoding. The fragment `e[2] <= 1 ? ... : ...` tests for endianness. Because of our assumption, we know the condition is true and keep only the first expression.

The code now looks like this:

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int *e;
    for (e = (int *)calloc(262144, 2) + 5,
         e[3] = 100000,
         e[-2] = 8192,
         e[1] = 11826,
         e[-1] = 2105376,
         e[65536 - 1] = 10,
         printf((char *)(e - 1)),
         printf((char *)(e + 1)),
         e[-4] = 1681207333,
         e[2] = 0,
         e[1] = 1,
         e[65536] = 1,
         e[5] = 1,
         e[-1] = 0;
         e[2] < e[-2] ||
         (e[-1] = 19126859,
          e[1] = 1128484681,
          e[2] = 0,
          printf((char *)e + 4),
          e[-4] = 1681204547,
          printf((char *)(e - 4), 2026),
          0);
         e[1] += !(e[-1] %= e[-2]))
        e[65536 + e[-1] + 1] += e[65536 + e[-1]] % e[1] * e[3],
        e[e[-1] + 4] +=
          (e[(e[2] += !e[65536 + e[2]] &&
           (e[2] < 2 || !printf((char *)(e - 4), e[e[2] + 4]) ||
            (e[2] % 16 && e[2] < (e[-2] - 1)) ||
            printf((char *)(e + 65536 - 1))),
           e[-1] + 1) + 4] += e[65536 + e[-1]] /= e[1]) / e[3],
        e[-1]++,
        e[e[-1] + 4] %= e[3];
}
```

## Fun challenge 1
Let's look at how `printf()` is used. Take the following fragment:

```c
e[-4] = 1681204547,
printf((char *)(e - 4), 2026),
```

`e[-4]` consists of the following bytes: `43, 25, 35, 64` and we know at that point that `e[-3]` is `0` and therefore starts with a terminating null-byte. Thus, we can replace the `printf()` with
`printf("\x43\x25\x35\x64", 2026)`
or with
`printf("C%5d", 2026)`.

As you can verify, there is a `printf("IOCC")` right before that point.

We systematically replace the first argument of `printf()` calls with a character string. 

## Fun challenge 0

Early on, `e[-2]` is set to `8192`. This constant is the number of base 100000 digits produced by the algorithm.

You can change it. However, it is used to calculate the string constants for the last line of output. Thus, you will need to adjust how they are set if you want to keep that line unchanged.

The program first prints `2.` directly. The first two generated digits are skipped so that we only print the digits after the decimal point.

This happens in the following fragment:
`e[2] < 2 || !printf((char *)(e - 4), e[e[2] + 4])`
which we rewrite as
`e[2] < 2 || !printf("%05d", e[e[2] + 4])`.

Each base 100000 digit corresponds to five base 10 digits.

Thus, the number of digits after the decimal point is `(8192 - 2) * 5 = 40950`.

## Final improvements
We can now replace the `for` with a `while`, replace the short-circuit operators with tests, and remove the comma operators.

We then observe that the first entries of `e` are used as scratch space, for constants, and for variables. Then follow two separate arrays.

Specifically, we write `N` for `e[-2]`, `B` for `e[3]`, `i` for `e[2]`, `j` for `e[-1]`, and `x` for `e[1]`. We write `e` for the array that starts at `e[4]`, and `f` for the array that starts at `e[65536]`.

We can now make the code clearer.

```c
#include <stdio.h>

#define B 100000
#define N 8192

int e[N + 1] = { 0, 1 };
int f[N + 1];

int main()
{
    int i = 0;
    int j = 0;
    int x = 1;

    printf("   2.");

    f[0] = 1;

    while (i < N) {
        f[j + 1] += f[j] % x * B;
        f[j] /= x;

        if (f[i] == 0) {
            if (i >= 2) {
                printf("%05d", e[i]);

                if (i % 16 == 0) {
                    printf("\n");
                }
            }

            i++;
        }

        e[j + 1] += f[j];
        e[j] += e[j + 1] / B;
        e[j + 1] %= B;

        j++;
        if (j == N) {
            j = 0;
            x++;
        }
    }

    printf("\nIOCCC 2026");
}
```

## Explanation
The program uses a spigot algorithm to calculate the digits of `e`. I intended to write an explanation but found that a good AI chatbot can explain how the final version of the code works.
