## It's the Sign of the Lines

Document number: DXXXXR0  
Date: 2020-06-15  
Audience: EWG  
Reply-to: Tony Van Eerd. signed at forecode.com


### Summary

- The return type used by `source_location` for `line()` and `column()` is unsigned, but should be signed.
- Is this a big deal?
- maybe yes, maybe no, but it affects (ie will be repeated in):
    - Contracts
    - stace trace
    - reflection
    - unknown future proposals

### Recommendation

- `int_least32_t` instead of `uint_least32_t`
- as a **Defect Report** against C++20

### Is it too late???

Actually, No.

- not yet broadly supported.
   - libstc++ - No
   - MSVC - No
   - libc++ - No
- #line is int (effectively)
- int vs uint have the same size, and #line is UB past 2147483647, no one has files that big
- Defect Reports are a thing.

### Why?

"Developers expect numbers to work like numbers" - Tony Van Eerd  
"Nicely put" - Bjarne Stroustrup

But I don't really want to debate signed vs unsigned _here_.  I want to debate that as a general policy over there: PXXXX.

We _did_ debate signed vs unsigned vigorously wrt `std::span`.  That was a close vote, but many voted for
unsigned for the sake of compatibility, even though they would prefer signed in isolation.
Thus I believe (and via being a LEWG regular) that LEWG (and WG21?) prefers signed for numbers, all else being equal.

I don't think there is any compatibility argument between `source_location` and `std::size`,
but there _will be_ compatibility arguments
in the future between { Contracts, stack trace, reflection, feature X, ... } and `source_location`.

### See Also
For more on signed vs unsigned see https://www.youtube.com/watch?v=Puio5dly9N8 Interactive panel with  Bjarne Stroustrup, Andrei Alexandrescu, Herb Sutter, Scott Meyers, Chandler Carruth, Sean Parent, Michael Wong, and Stephan T. Lavavej.

at 9:50,  
41:08,  
1:02:50 short answer (for why size() uses unsigned): “sorry”

