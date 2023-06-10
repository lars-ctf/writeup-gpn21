# Ref4ctory

This challenge involved finding the factors of multiple composite
numbers.

``` python
def check_factors(a,b,ab):
    if abs(a)<=1 or abs(b)<=1:
        print("too easy")
        return False
    if type(a*b) == float:
        print("no floats please")
        return False
    return a*b == ab
```

The `check_factors` function disallows floats and factors $-1\le a\le 1$
and $-1\le b\le 1$.

Our quick-and-dirty strategy was therefore to use the factor $a=2$ and
$b=ab/2$ if $a\pmod2=0$. If $a\pmod2\ne0$, we would use wolframalpha's
prime factorization (because primes are awesome). While we had some
problems with typing fast enough (`ncat` timeouted), we got the flag
using this approach in the end.
