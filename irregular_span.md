Should Span be Regular?
-----------------------

Document number: P1085R2  
Date: 2018-09-21  
Audience: LWG/LEWG  
Reply-to: Tony Van Eerd. regular at forecode.com


Revision 2
-----

- Corrected Audience above (ie not EWG).
- Improved wording of editor instructions (thanks @tcanens)


Revision 1
-----

- Added results of LEWG review (see below). Basically, LEWG decision was to remove `==` (and other comparisons) from span.
- Added more about deep const as it relates to copies and threads (which was brought up in LEWG review)
- Added wording (editor instructions actually, on advice of LWG regulars). In this case, added to the top of the doc, for simplicity sake.


-----


Wording (Editor Instructions)
-----

1. Remove subclause 21.7.3.7 [span.comparison].
2. Remove the declarations of operators ==, !=, <, >, <=, and >= in 21.7.2 [span.syn]

see https://github.com/cplusplus/draft/compare/master...tvaneerd:patch-1


LEWG Review
-----

Approval voting:

|     |                                                                   |
|-----|-------------------------------------------------------------------|
| 12  |  Rename to spanning_ptr(ish) + make operator== (etc) shallow      | 
| 18  |  No rename, make operator== (etc) shallow                         |
| 24  |  Drop operator== (etc)                                            |
|  3  |  Do nothing                                                       |
|  9  |  Remove span from IS                                              |
|  3  |  Make span operate only on const T, (rename cspan, obviously :D)  |
| 15  |  Add member .equal() and remove operator== (etc)                  |

Runoff: No rename, make operator== (etc) shallow vs Drop operator== (etc)  
(S=shallow, D=drop)

| SS   |  S   |  N   |   D  |  SD |
|------|------|------|------|-----|
|  2   |  4   |  0   |  13  |  8  |

Add member .equal()?

|  SF  |   F  |   N  |   A  |  SA |
|------|------|------|------|-----|
|   0  |   8  |   9  |   7  |  4  |

(for example, just use ranges::equal)

Motivation
------


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
      |                  \      /
      |                     \/
      |
      |
      /\
   /      \
/ deep assign\ 
\     ?      /  No ---------+
   \      /      
      \/         
      |
     Yes
```


Regular
-----

_Copy or copy not; there is no shallow._  - Master Yoda


`int` is Regular. Pointers are Regular.  `span` currently is not.

Alexander Stepanov, who wrote the STL that we steward, considered the concept Regular to be of utmost importance.  The STL was written with Regularity as its basis.
Concepts, which are appearing in C++20, also strongly lean on Regular; see the Palo Alto TR (http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3351.pdf).

See also Lakos (N2479) on defining Value and the properties of copy and equality, and the assumed relationship between copy and `==`.

**Basically, we all know that overloading operators can be dangerous _when you change the common meaning of the operator_.**

The meaning of copy construction and copy assignment is to copy the _value_ of the object.
The meaning of `==` (and `<`, etc) is to compare the _value_ of the object.

Copy, assign, equality are the holy trinity of C++.  They are expected to go together.  It's fundamental to allowing user-defined types to act as built-in types, and for code to work as expected, at a glance.

Java strings came up in the reflector discussion.  It is well known that people have issues with String's `.equal()` vs `==`.
_This is precisely because java strings do NOT act like built in types_

Java has classes, that work one way (as pointers) and built-ins (ie int) that work like values.  String is in between, thus confusing.

C++ exposes the one model - Regular - and makes pointers work the same as int.  Yes, this means to onus is on the programmer to recognize that a pointer is NOT the thing it points to.  But once this fundamental bridge is crossed, everything is consistent.

Having a type that is a mixed bag of meaning, is dangerous.

I shouldn't need to explain Regular to any LEWG member, so I'll stop here.  Read, again, Elements of Programming by Stepanov.

See also, the LEWG guidelines (https://github.com/cplusplus/LEWG/blob/master/library-design-guidelines.md), in particular,

- When designing a class type, where possible it should be a "regular type"

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

ie using a `span` in a `set` or as a key in a `map`.  (Since the _value_ of `span` can easily change at a distance.)

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

Note also, that `std::cbegin(sp)` is not the same as `sp.cbegin()`.  The first is a (non-const) `iterator` from a `const span`, the other is a `const_iterator`.


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

Note, however, that deep const cannot be maintained. ie:

```
void g()
{
    const span const_sp = some_span();
    span cp = const_sp; // makes a "copy", and is non-const
    cp[1] = 17;  // this also changes _value_ of const_sp
}
```

You can take that example to mean that const should not be shallow.  Others take it to mean that span is fundamentally broken. (ie maybe reconsider making `==` shallow, const shallow and copy shallow!)

Also recall that for the STL (and as a good practice in general) `const` means safe to read from multiple threads. That would also not be true for deep const and span.


Shallow Assignment?
-------------------

If the _value_ of a `span` is the elements to which it refers, why doesn't `sp1 = sp2;` modify the elements of sp1?

It appears that `span` is pointer-like on assignment.

Naming
------

If `span` is to have _some_ reference semantics (deep equality, deep const) (yet shallow assignment?), it should maybe be called `span_ref`; this might lessen confusion.

If `span` is to have pointer semantics (shallow equality, shallow const, shallow copy - ie Regular), it should be called `span_ptr`.  With `ptr` in its name, I doubt anyone will be confused by its shallow semantics.

(Consider also `spanning_ptr` and/or `spanning_ref`.)


Acknowledgements and See Also
-----------------------------

The was a giant reflector discussion that I found enlightening. Thank you all for participating.
