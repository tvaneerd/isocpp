## `std::order` A (mostly) Library Solution to a (mostly) Library Problem

Document number:    
Date: 2016-07  
Audience: EWG / LEWG  
Reply-to: Tony Van Eerd. order at forecode.com

## Summary

A new `std::order<T>` that forms an ordering of any type, based on member-wise ordering.

## Motivation for ordering

There has been some support for default `operator<()` (as in P0221R1) which would define a `<` for many types by default.
Most of the motivation for this, that I have heard, is

- I want types to automatically work in `std::set` and `std::map`
- I want types to be easily sortable in arrays and `std::vector`, to enable fast binary searching

In particular, what I haven't heard much/any of is

- I want to be able to write the expression `x1 < x2` for almost all my types.

It seems to me that building a default `operator<` onto every type, so that they work well with library constructs
is somewhat the tail wagging the dog.[*]  Could we not simply build some library constructs to solve these library usability issues?

Well, in fact, we con't _simply_ build a library construct, at least not without reflection.  But we could with a bit of compiler help.
Enter `std::order`.

## `std::order`

`std::order<T>` would be defined roughly as "orders T "by value" in some implementation defined way consistent with memberwise ordering".
Most likely, we would just define it the same way memberwise `<` is defined in P0221R1,
although we could maybe get away with less strict rules, such that order-of-members need not be exposed,
or compilers could have a bit more optimization leeway, etc.
We can also order more types - we need not impose restrictions on pointer members, mutable members, etc -
Since we are limiting the scope of the feature to a library solution,
we need not be as conservative, thus "Buyer beware" works here, I think.

### Interaction with `<`

Should `std::order<T>` call T's `operator<` when available?  
Yes.  

Step 0.  Does/should `std::order<T>` call T::member's `operator<` for each member that has an `operator<`?  And same for T::bases?  
If the answer to that is "of course", then the logical answer is to call T's `operator<` when available.

(Wait, should it actually call each member's `operator<` or instead use `std::less` on each member? ie what about pointers?)

### Interaction with Library

`std::order` still, on its own, doesn't do anything automatically.
Our goal of having types "just work" with `set` and `map` and `sort()` has not been reached.
To enable this we have three choices, only one of which is actually currently palatable:

1. make `set` and `map` and `sort` et al default to `std::order`
2. make `set` and `map` and `sort` et al default to `std::less` but have `std::less` call `std::order` when `<` is not well-formed
3. make `set` and `map` and `sort` et al default to some `conditional<is_lessable<T>, std::less<T>, std::order<T>>::type`

Which one is palatable?

1. is not :-(  It would be ideal I think, and can be revisited for a "stl2", but would break ABI compatibility for "stl classic".
2. is not :-(  For things like pointers to work correctly, and to support existing practice of specializing `std::less`,
we need `std::order` to use `std::less` on its members.
Which means, actually, that `std::order<T>` should use `std::less<T>` when available.
So that would mean `std::order<T>` calls `std::less<T>` which calls `std::order<T>`...

3. is the only palatable solution (that I could up with anyhow)

### Should `std::order` be specializable?
¯\\\_(ツ)_/¯  

`std::less` should NOT specializable.  It would be great to "take-back" `std::less`.
It might be interesting to specialize `std::order<MyImmutableString>` that compares underlying `char *` pointers (if strings are immutable equal strings can share memory, and equality becomes just a pointer comparison. Ordering (_an_ ordering, not lexical ordering) is also just a pointer comparison).  However, `map` will always choose `std::less<MyImmutableString>` which probably still exists (for when you want "real" string comparisons).  So specializing `std::order` doesn't by much.

Alternative:

`std::representation_order<T>` - compiler generated.  Not specializable. Clear purpose.  Not used _directly_ by STL.  But does it recursively call itself, or `std::order` or `std::less` ???  
`std::order<T>` - used by STL.  Can be specialized.  Used by STL when it exists, else STL uses `std::less`.  _However_, `std::order` does not 'just' call `std::less` - see above.  
`std::less<T>` - calls `operator<`.  Cannot be specialized.

On one hand, we want "the thing map uses" (ie `std::map_order` maybe).  On the other hand, we want "the thing that will always work, regardless of what the user has defined" (ie `std::some_order`) which calls `std::less` or `representation_order` or whatever it needs to do.


## Footnotes
[*] library tails wagging language dogs isn't always a bad thing;
sometimes libraries, and coding in general, highlight areas where language features make sense.
I just don't think this is one of those cases.

