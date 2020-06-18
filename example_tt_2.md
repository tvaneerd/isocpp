## A Fizzy Library Feature with all the Right Buzzwords

Document number: DXXXXR0  
Date: 20XX-MM-DD  
Audience: LEWG  
Reply-to: Tony Van Eerd. tt2 at forecode.com


### Summary

Not everything should be in the standard library.  Things in the standard library should be one (or hopefully more) of:
- often needed
- often implemented
- hard to get right

FizzBuzz (https://en.wikipedia.org/wiki/Fizz_buzz) checks all the boxes.


<table>
<tr>
<th>
Naive FizzBuzz
</th>
<th>
std::fizzbuzz
</th>
</tr>
<tr>
<td  valign="top">

<pre lang="cpp">

struct fb_gen {
  int max = 100;
  generator<string> gen() {
    for (int i:iota(max)) {
      bool f=!(i%3), b=!(i%5);
      co_yield (f?b?"fizzbuzz":"fizz":b?"buzz":to_string(i));
    }
  }
};

int main()
{
  for (auto s : fb_gen{}.gen())
    std::cout << s;
}

</pre>
</td>
<td  valign="top">

<pre lang="cpp">












int main()
{
  for (auto n : irange(0,100))
    std::cout << std::fizzbuzz(n);
}

</pre>
</td>
</tr>
</table>


## Design Considerations

- efficieny
- ranges?
- ease of use
- include "\n" ?

### What About Efficiency

The simple version of `std::string fizzbuzz(int);` may not be the most efficient.
The resulting string will typically fit into the small buffer optimization (SBO), so there is no allocation, but there is still copying.

A more efficient version would likely be:

    template<typename F>
    std::string fizzbuzz(int n, F & func);

where `func` gets called like `func("fizz")`

    int main()
    {
        for (int i = 1; i <= 100; i++)
        {
            std::fizzbuzz(n, [] (auto s) {
                    std::cout << s << '\n';
                });
        }

Q. Does `func` take `char const *`, `std::string_view`, or anything "string-like"?

A: Anything string-like??

Q: Should the non-fizzbuzz numbers be called like `func(std::to_string(n)` or `func(n)`?

A: We can check if `func(n)` is callable with an `if constexpr`, and call it if true, otherwise convert to string.
(And we assume `func` can take a string-like.)

## Open Questions

### Unicode? Wide Strings?

### Capitalization?

"fizz" or "Fizz" or "FIZZ" or customizable?

### Other Languages?

## Suggested Votes

The authors thank you for your time, and would appreciate specific _directional_ feedback
(in addition to the helpful feedback received during discussion).


#### Do we want (some form of) std::fizzbuzz?

| SF | F | N | A | SA |
|----|---|---|---|----|
| __ | __ | __ | __ | __ |


#### Do we want `std::string std::fizzbuzz(int)`?

| SF | F | N | A | SA |
|----|---|---|---|----|
| __ | __ | __ | __ | __ |

#### Do we want max efficient std::fizzbuzz?

| SF | F | N | A | SA |
|----|---|---|---|----|
| __ | __ | __ | __ | __ |

#### etc...

| SF | F | N | A | SA |
|----|---|---|---|----|
| __ | __ | __ | __ | __ |







### Acknowledgements

@vector_of_bool supplied the (intentionally buggy) "Naive" FizzBuzz.
