weak_equality considered harmful
-----------------------

Document number: PXXXX  
Date: 2018-10-08  
Audience: EWG  
Reply-to: Tony Van Eerd. regular at forecode.com


_If I had more time, I would have written a shorter letter._

-- Blaise Pascal (and others, see https://quoteinvestigator.com/2012/04/28/shorter-letter/)

----


_I hope that most programmers will learn the fundamental semantic properties of fundamental operations. What does assignment mean? What does equality mean? How to construct data structures._

_At present C++ is the best vehicle for this style of programming._

-- Alex Stepanov, http://stepanovpapers.com/drdobbs-interview.html


Synopsis _(and about that title)_
----

This paper will explain why `weak_equality` (as the return type of `<=>`, introduced in P0515)
should not lead to the generation of `==` (nor `!=`).
The title of this paper is obviously "click-bait" like the (retitled! https://en.wikipedia.org/wiki/Considered_harmful)
goto paper it alludes to was, but it is also accurate - a `==` operator that is "weakly equal" is an oxymoron
and is _harmful_ to quality software; it will break std algorithms, and goes against Concepts,
the Palo Alto paper (N3351), the Lakos "value paper" (N2479), Stepanov and the _Elements of Programming_ ("EoP") book,
and the very fundamentals of C++.

P.S. same goes for generating `==` from `partial_ordering` or `weak_ordering`. Also not good.


Background
----

To ensure everyone clearly understands the terms `strong_equality` and `weak_equality` from P0515.

`strong_equality` implies _substitutibility_, ie that `x == y` implies `f(x) == f(y)`, for all (Regular) `f`
that only accesses the _salient_ attributes of `x` and `y`.
What is _salient_? See Lakos.  Either N2479, or see actual John Lakos.
But basically "salient" means "the important parts" and
the parts that are _guaranteed_ to be copied/moved via construction and assignment.
For example, the elements of a vector are salient, and form the value of the vector, but capacity is not salient.
Basically, as this paper will explain, `strong_equality` means equality, as it is commonly known.

(It sounds like "salient" is arbitrary.  To an extent, it is.  It must be defined by each particular class.
But whatever the class defines as salient, it is important for it to be consistent about it -
copy, move, assign, and ==, should all agree.  Again, see Lakos. And later examples in this paper.)

So `weak_equality`?  It is an oxymoron, but in P0515 means that `==` is still an _equivalence relation_
in the mathematical sense (ie reflexive, symmetric, transitive - see Wikipedia),
but not "equal" in the substititable sense.
In fact, not "equal" in the _equal_ (common English) sense.
Like 2 and 7 are _equivalent_ mod 5, but 2 and 7 are not equal, they do not represent the same _value_.

So fundamentally, `weak_equality` generates an `==` such that `==` does not mean "equal". Thus it is an oxymoron. :-(


More on "equal"
----

**The C++ Working Draft:**

Searching for "equal" and "equiv" in the Working Draft...

equal (~700 occurences)
- some syntactic, like "brace-or-equal-initializers"
- Almost all usage is as in English, ie:
  * equal in the sense of two numbers being mathematically equal ie "`sizeof(std::nullptr_t)` shall be equal to `sizeof(void*)`"
  * or two expressions being equal/equivalent - ie substitutible.

- "equal" is used in the sense of "weak_equality" exactly 0 times

equivalent (550+ occurences)
- "Effects: equivalent to..." - ie substitutible
- "equivalence of keys" is carefully explained as an exception,
to be an equivalence relation (not equality) on keys in map and set.  Note that `==` is not used for key-equivalence. 


**The Palo Alto Paper (N3351)**

There is an entire section: **3.1.1 Equality**  
First sentence:

> "Reasoning about computer programs is facilitated by _equational reasoning_, which allows us to substitute equals for equals."

Next:  

> "If two values represent the same entity, then they are equal."

But what is meant by "same entity"?  It is not in Palo Alto.  We need to turn to Elements of Programming.
For equality, EoP is (unsurprisingly) similar:

> "Two values of a value type are equal if and only if they represent the same abstract entity"

But EoP is more thorough.  Page **1**:

> "An _abstract entity_ is an individual thing that is eternal and unchangeable,... Blue and 13 are examples of abstract entities"

And Page 2 explains that values represent abstract entities (via datum - sequence of 0s and 1s).  The important part being the abstract thing, like 13 - this is the key to "value".

This is also explained similarly in Lakos (N2479) which Palo Alto references in the **Equality** section.

And more directly, Palo Alto, Page 50:

> So, EqualityComparable<T> is true if T
> 1. has == and != with result types that can be converted to bool
> 2. == compares for true equality
> 3. == is reflexive, symmetric, and transitive
> 4. != is the complement of ==

Note #2, implying that #3 isn't sufficient. "true equality" isn't defined anywhere, but you can look at their definition of eq(), or the definition of equality and the quotes above.


**Concepts (in the C++ Working Draft)**

The Ranges/Concepts papers (now in the working paper) lean heavily on Palo Alto. `EqaulityComparable<T>` requires the expected syntactic constraints of `==`, but also "is satisfied only if `bool(a == b)` is `true` when `a` is equal to `b`, and `false` otherwise". The "equal" in that sentence is "English equal" not "syntactic equal" (else it would be redundant) and is further explained in the `[concepts.equality]` section as strong equality (substitution).

To be clear: **Concepts and Ranges assume `==` means strong equality**

Thus some Concept-based std algorithms may break on types with weak equality.  (As strong equality is an assumption that an implementation may assume, which algorithms will break may differ per implementation.)


But standard algorithms tend to only need partial ordering?
----

It was pointed out to me that

> Alex Stepanov (or someone quoting him) would likely stomp on the notion of equivalence being problematic. He’s retired now but
> famously stomps of people about mathematics, and he designed STL to use equivalence for associative containers
> and I’ve never heard him regret it.

(equivalence there meaning weak equality)

It is important to understand that _algorithms_ tend to (and should) set minimal requirements _for that algorithm_, whereas _types_ are expected to be useful in a larger set of algorithms, and thus tend to support a super-set of requirements.  In particular, _Regular_ is the concept that captures all these common and expected requirements.

EoP page 7: "A type is _regular_ if and only if its basis includes equality, assignment, destructor, default constructor, copy constructor, total ordering, and underlying type."

Note _total ordering_.  Not partial.  Even though most of Stepanov's STL only requires partial ordering, a type is expected to have total ordering, (to be clear - the difference between the two is strong equality).

So to answer the Stepanov question, agreed, equivalence is not problematic _for an algorithm_, but Stepanov would find it problematic for a _type_.

Why?

It goes back to Palo Alto's "Reasoning about computer programs is facilitated by _equational reasoning_, which allows us to substitute equals for equals."

> Every important optimization technique is affiliated with some abstract property of programming objects. Optimization, after all,
> is based on our ability to reason about programs and to replace one program with its faster equivalent. 

-- Alexander Stepanov, _Notes on Programming_, http://stepanovpapers.com/notes.pdf

Another way to consider it is what Stepanov called "optimizing programmers":

> The operations we have discussed here, equality and copy, are central because they
> are used by virtually all programs.  They are also critically important because they are
> the  basis  of  many  optimizations...
> Such optimizations include, for example, common subexpression elimination,
> constant and copy propagation, and loop-invariant code hoisting and sinking. These
> are routinely applied today by optimizing compilers to operations on values of built-in
> types. Compilers do not generally apply them to operations on user types because
> language  specifications  do  not  place  the restrictions  we  have  described  on  the
> operations of those types.

> However,  users  do  apply  such  optimizations  by  hand.    They  often  do  so  without
> thinking  because  they  intuitively  expect  the  conditions  to  apply.

> If  they  are  toproduce   efficient  generic  components  without  seeing  the  underlying  type  definitions,
> they  must  be  able  to  make  the  assumptions  which  allow  such  optimizations.    Our
> axioms,  then,  are  necessary  to  allow  users  to  reliably  make  the  optimizations
> commonly made both by optimizing compilers and by optimizing programmers.
> Ultimately, we would like compilers to be able to perform such optimizations at a
> high  semantic  level  as  well  as  they  do  at  the  built-in  type  level. 

-- James C. Dehnert and Alexander Stepanov, _Fundamentals of Generic Programming_, http://stepanovpapers.com/DeSt98.pdf 

ie we, as programmers, assume equality means substitution.
We do this regularly.

_The algorithms that require "strong" equality are the everyday lines of code we write._


The motivation _for_ `weak_equality`
----

Why was `weak_equality` proposed in the first place, and is its motivation compelling _enough_ to include it in the standard?

**No.**

I see 2 motivations for `weak_equality` in P0515 (the original paper proposing `<=>`)

1. "completeness"
2. `CaseInsensitiveString` (and/or filenames)
3. building on Lawrence Crowl's comparison work (P0474, P0100)


**Completeness** - ie it makes the table of the relationship between equality and ordering more complete (and teachable):

```
+-----------------+------------------+
|                 | partial_ordering | 
|  weak_equality  +------------------+
|                 |  weak_ordering   |
+-----------------+------------------+
| strong_equality | strong_ordering  |
+-----------------+------------------+
```

Note however that it is not actually complete - `partial_equality` is missing. ie An equality that is strong where defined, but not defined over the full set of values - ie this could be used for <=> over floating point types (ie with strong equality everywhere except NaN). Rust, for example, has the `PartialEq` trait for this.

In fact, strong vs weak (ie "is it substitutible") and partial vs total ("does it cover all values") are orthogonal axes, conflated by P0515.

**`CaseInsensitiveString`** (and/or case insensitive filenames)

The original <=> paper (P0515) used `CaseInsensitiveString` as a motivating example of a class that might want to use `weak_equality` or `weak_ordering`.

Basically I don't find the example sufficiently motivating.  It is an anti-pattern:

I feel that a class like this needs to decide whether case is _salient_ or not.
A typical litmus test is "if we put a bunch of CISs into a set, would you be surprised when some disappeared?".
Alternatively, would the rest of the code (and user-base) be OK is the string was converted to lowercase in the constructor?

If case is not salient, not important, then equality can ignore it, and it is actually _strong equality_
(like vector ignores capacity - it is not part of the value).

Alternatively, if case is salient, make it part of `==`.

Neither case results in a weak `==`.

A weak `==` is _harmful_ in that it can lead to simple mistakes, reminiscent of Stepanov's "optimizing programmers":


```
void set(X const & newX)
{
    if (oldX == newX)
       return;

    ... do stuff ...
}
```

A programmer would need to know not to use this pattern with CaseInsensitiveString, since "important" case information would be lost.


Now you are free to disagree with me when I say that the example `CaseInsensitiveString` is a bad class, and that you will use it with a weak equality anyhow.  Or that there a examples here and there that might use weak equality.

_Fine_.

The real question here is not whether we allow you to write bad code. We always do.  The question is whether we encourage it, enable it, make it easy.

True, the standard isn't a guideline, but be very clear - the whole `<=>` feature is about making it easier to be more correct. We don't add things to the language without _sufficient motivation_.

> C++ as a language does not impose any constraints. You can define your equality operator to do multiplication. But equality should be equality.

-- Alexander Stepanov, http://stepanovpapers.com/drdobbs-interview.html

We don't offer a shortcut on how to make `==` do multiplication, even though it's possible.



**Building on Crowl's P0474 and P0100**


Much of the categorization in P0515 is based on Lawrence Crowl's very detailed comparison papers, P0474 and P0100.  Lawrence suggested a `weak_equivalence()` function (which makes sense on types like CaseInsensitiveString that have weak or partial ordering), but never suggested promoting that to `==`.  His papers are very clear that `==` should always be "total equality".  (In fact, his earlier papers called it `weak_equal`, but he (correctly :-) later changed it to `weak_equivalence`).

(Similarly, an explicit function like `weak_equivalence(CaseInsensitiveString const &, CaseInsensitiveString const &)` might be more clear and avoid mistakes.)


It seems `weak_equality`, and the generation of a weak `==`, was new in P0515 and did not get enough scrutiny.


Suggested Actions
----

Some of these are alternatives to others (ie contradictory to each other, or one moots the other, etc).

- do not generate `==` from anything except `strong_ordering` and `strong_equality` (ie not weak_ordering nor partial_ordering)
- rename `weak_equality` to `weak_equivalence`,
- rename `strong_equality` to `equality`
- remove `weak_equality` completely
- remove `weak_ordering` as well
- consider introducing `partial_equality`


