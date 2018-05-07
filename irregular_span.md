Should Span be Regular?
-----------------------

Document number: PXXXXR0  
Date: 2018-05-04  
Audience: EWG  
Reply-to: Tony Van Eerd. regular at forecode.com




```
                        Span

                         /\
                      /      \
                   /   Regular  \
      +------ No   \      ?     /  Yes ------+
      |               \      /               |
 spanning_ref            \/             spanning_ptr
      |
      |
      /\
   /      \
/ deep const \ 
\     ?      /  No ---------+
   \      /                 |
      \/                    /\
      |                  /      \
      |               /     WTF    \
     Yes              \      ?     /
                         \      /
                            \/
```


Regular
-----

_Copy or copy not; there is no shallow._  - Master Yoda


`int` is Regular. Pointers are Regular.  `span` currently is not.

Alexander Stepanov, who wrote the STL that we steward, considered the concept Regular to be of utmost importance.  The STL was written with Regularity as its basis.
Concepts, which are appearing in C++20, also strongly lean on Regular; see the Palo Alto TR (http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).

See also Lakos (N2479) on defining Value and the properties of copy and equality, and the assumed relationship between copy and `==`.

I shouldn't need to explain Regular to any LEWG member, so I won't.  Read, again, Elements of Programming by Stepanov.

Span
----

Span is currently not Regular.

In particular, copy is "shallow" (only ptr and length are copied), whereas `==` is "deep" (ptr and length are _not_ compared, instead the elements pointed at are compared, ie via `std::equal` algorithm.

This matches `string_view`, however `string_view` can't modify the elements it points at, and thus the shallow copy of `string_view` can be thought of as similar to a copy-on-write optimization.

What are the _salient_ parts that make up the _value_ of a span?

A type with mistaken `==` literally doesn't know what it is.  What is span?

It appears that the intention of `span` is to be a lightweight representation of the elements it references.  Its current `==` is useful and possibly "the point" of `span`.  A `span` is trying to _act like_ a collection of the elements over which it spans.

Yet it is not Regular, and thus may not work as expected compared to most types, when a novice copy/pastes code patterns and then applies them to `span`, or when using `span` in generic templates.

Basically `span` has, in some sense, reference semantics, which tends to trip people up eventually.

```
T oldx = x;
change(x);
assert(oldx != x);
return oldx;
```


Deep Const
----------

We often talk about _logical const_; the idea that a type, when const, shouldn't just protect its members, but should also extend that protection to any _parts_  (see EoP) of the type that it references/owns (via pointers, references, etc).  We do this whenever the extended parts constitute part of the _value_ of the type, exemplified by how those parts are used in `==` and copy.  You compare the values, you copy the values.

See also `propagate_const`.

If `span` has "deep" equality, it should have deep (logical) const.
Deep equality implies that the _value_ of a span is not the ptr+length, but the elements it refers to.
Thus if a span is const, ie `const span<T> csp;`, then the _value_ of the span (ie the elements) should not be modifiable.

If you feel that `const span<T>` should be "shallow" similar to a smart pointer, ie `const unique_ptr<T>`, and thus allow T to be modified,
then you should also want `==` to work like a pointer and be shallow.  _Be consistent._


```
void read_only(const T & x);

void f()
{
	T tmp = x;
	read_only(x);
	assert(tmp == x);
}
```

(Currently, the above can easily fail when T is `span`).



Naming
------

If `span` is to have reference semantics (deep equality, deep const, shallow copy), it should maybe be called `span_ref`; this might lessen confusion.

If `span` is to have pointer semantics (shallow equality, shallow const, shallow copy - ie Regular), it should be called `span_ptr`.  With `ptr` in its name, I doubt anyone will be confused by its shallow semantics.

(Consider also `spanning_ptr` and/or `spanning_ref`.)


Acknowledgements and See Also
-----------------------------

The was a giant reflector discussion that I found enlightening. Thank you all for participating.
