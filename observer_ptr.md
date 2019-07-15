## Prebuttal for Standardizing observer_ptr

Document number: D1495R0  
Date: 2019-02-21  
Audience: LEWG  
Reply-to: Tony Van Eerd. cadged at forecode.com

---


Summary
-------

In P1408, Bjarne wrote a rebuttal against standardizing `observer_ptr`, but no one had yet rewritten a proposal to standardize it. Thus the need for this "prebuttal":

`observer_ptr` (from Library Fundamentals 2 TS) **should be standardized** (but with a better name and better conversions)


Reasons to standardize
----------------------

Reasons to standardize `observer_ptr` (with more in-depth explanations to follow).

1. `observer_ptr` is the safe **_common type_** for `unique_ptr`, `shared_ptr`, other smart pointers, and `T*`.
2. `observer_ptr` is a **_safer alternative to `T*`_**
3. `observer_ptr` **_extends the type safety_** of `shared_ptr` and `unique_ptr` (and other smart pointers)
4. `observer_ptr` makes **_intent_** more **_clear_**
5. `observer_ptr` is a post-modern tool for **_transitioning a codebase_** to more modern C++.

---

### 1. `observer_ptr` is the safe **_common type_** for `unique_ptr`, `shared_ptr`, other smart pointers, and `T*`.

`observer_ptr` for smart and raw pointers, is like `string_view` for `char const *` and `string`.  
ie a common base type that any pointer can be temporarily safely converted to.


<table>
<tr>
<th>C++</th>  <th>C++LibFun2</th>  <th>C++20</th>
</tr>
<tr>
<td  valign="top">

<pre lang="cpp">
void f(Foo * pf);

shared_ptr<Foo> sp = ...;
unique_ptr<Foo> up = ...;
Foo * rp = ...;
    
f(sp.get());
f(up.get());
f(rp);
</pre>
</td>
<td  valign="top">

<pre lang="cpp">
void f(observer_ptr&lt;Foo&gt; pf);
    
shared_ptr<Foo> sp = ...;
unique_ptr<Foo> up = ...;
Foo * rp = ...;
    
f(observer_ptr(sp.get()));
f(observer_ptr(up.get()));
f(observer_ptr(rp));
</pre>
</td>
<td  valign="top">

<pre lang="cpp">
void f(observer_ptr&lt;Foo&gt; pf);
    
shared_ptr<Foo> sp = ...;
unique_ptr<Foo> up = ...;
Foo * rp = ...;
    
f(sp);
f(up);
f(rp);
</pre>
</td>
</tr>
</table>

- The leftmost version has calls to `get()` which require extra scrutiny in a code review (as you are removing a safety latch)
- We will likely never allow smart pointers to implicitly convert to raw pointers
- The leftmost function `f` (taking a raw pointer) also requires extra scrutiny.
- The middle version is just noisy - for no good reason - the noise does not protect anything dangerous.
- The rightmost version is safe - `observer_ptr` doesn't change ownership semantics. Nothing extra should be required.
- In terms of P0705 conversion guidelines, an implicit conversion to `observer_ptr` is safe and sensible.

_Why isn't `T*` the common type?_

You don't want smart pointers to implicitly convert to raw pointers
(as this can too easily lead to accidental misownership - ie `delete someSharedPtr;`), but converting to `observer_ptr` does not increase the risk of misownership.
It does increase risk of dangling - same as string_view does.  Thus, similar to `string_view`, `observer_ptr` is best used as a function param.

### 2. `observer_ptr` is a **_safer alternative to `T*`_**

As mentioned in recent emails on the reflector, `T*` has some downsides:

- `T*` allows `++`, `--`, and `[i]` even when it shouldn't
- `T*` allows derived/base conversions (and then more `++`/`--` with the wrong size)
- `T*` `<` is not a total order
- `T*` opens up questions about ownership (in many codebases)
- `T*` allows conversion to `void *`
- `reinterpret_cast`
- etc

### 3. `observer_ptr` **_extends the type safety of other smart pointers_**

A smart pointer, when used correctly, ensures safe lifetime management. The only (well almost only) way to break the safety of a smart pointer, is improper use of `get()`.  Since `get()` exposes the underlying pointer, it exposes the responsibility of the smart pointer's invariants, breaking the "air-tight seal" of the smart pointer.

`observer_ptr` avoids `get()` and allows the invariant to stay protected throughout more code. A function that previously used a raw pointer (so as to be used by both `shared_ptr` and `unique_ptr` clients), can now avoid `get()` completely.  This means `get()` can be pushed to the edge of your codebase - only needed at the border with 3rd party or unchangeable APIs that require other pointer types.

`observer_ptr` extends the safety net of a smart pointer over more of your codebase.

### 4. `observer_ptr` makes **_intent_** more **_clear_**

From the original proposal (N4282) "it is intended as a near drop-in replacement for raw pointer types, with the advantage that,  as a vocabulary type, it indicates its intended use without need for detailed analysis by code readers". `observer_ptr` makes code more clear, easier to read, alleviating nagging lifetime questions. Some codebases can use `T*` to mean non-owning, but many popular 3rd party libraries still use raw pointers with various meanings, thus adding ambiguity to even modern codebases.

### 5. `observer_ptr` is a post-modern tool for **_transitioning a codebase_** to more modern C++.

This is the reasoning most often discussed. Note that it is listed _fifth_. The idea is to review all raw pointers in a codebase, replacing each case with the correct smart pointer (ie `unique_ptr` hopefully, else `shared_ptr`, etc).
Since this replacement takes time, and not all cases will be fixed at once, "unowned" pointers need a thing to be replaced with (ie `observer_ptr`) else it is hard to know which raw pointers have been reviewed, and which haven't.
(If *all* owned pointers in a codebase are wrapped with smart pointers,
then a raw pointer can mean "unowned". But most codebases are still mixed with new and old uses.)

Postbuttal
---------

Rebuttal rebuttal:  There was concern about the potential proliferation of smart pointers in the standard. Although there are a few other potential smart pointers not (yet) in the standard (ie an intrustive_ptr, see P0468), smart pointers have been around for about 20 years, and there really isn't that much variation.  Boost has shared_ptr and intrusive_ptr, many codebases have something like observer_ptr. There is also inout_ptr (P1132), which can be found in many codebases (and is not really a smart pointer, but more of a helper, working alongside smart pointers), and also a clone_ptr or value_ptr, now called polymorphic_value (P0201).  But the list of common, widely used (and thus candidates for standardization) smart pointers is finite and short.  Ironically, the more we have, the more useful `observer_ptr` becomes as a common type that works seemlessly with all of them.


Changes
-------

Before standardizing `observer_ptr`, we should make a few small changes (more in-depth below)

1. Allow **_implicit conversions from smart and raw pointers_**
2. **_Rename/remove `release()`_** (as it does not transfer ownership)
3. **_Rename `observer_ptr`_**


### 1. Allow **_implicit conversions from smart and raw pointers_**

The original proposal did not include implicit conversions.  Most coding guidelines now favour `explicit` constructors - _when in doubt_; however, in the case of `observer_ptr`, there is no danger in an implicit conversion, only benefit.
See the table earlier in this paper - implicit conversion is required to allow `f(observer_ptr<T>)` to take smart pointers and raw pointers without calls to `get()`.  See P0705 for a more complete explanation as to when implicit conversion is acceptable. `observer_ptr` checks all the right boxes.  In particular:
- a smart/raw pointer and an `observer_ptr` both represent the same "platonic" thing (or `observer_ptr` is a strict subset of a pointer, since it offers a subset of functionality). Thus conversion (of some form) is worth considering.
- the conversion is safe. The `observer_ptr` won't delete the pointer, etc.  For a smart-pointer, conversion to `observer_ptr` does *not* break the smart-pointer's invariants. (whereas `get()` on a smart-pointer _does_ break (or expose for breakage) a smart pointer's invariants).  There is a slight concern with dangling (the `observer_ptr` doesn't extend the lifetime of the pointer), but this is the exact same level of concern (and same risk/reward) as with `string_view`.

This change has been implemented a few implementations without issue.


### 2. **_Rename/remove `release()`_** (as it does not transfer ownership)

`unique_ptr` has a `release()` function, which transfers ownership. Note that `shared_ptr` does NOT have a `release` function (as you don't gain exclusive ownership from the shared pointer). 

`observer_ptr::release()` **does not transfer ownership**.  It should be consistent with STL `shared_ptr`, and not have this function.

In any programming language, it is important that **_Different semantics require different names_** but it is particluarly important in a language with templates that use compile-time duck-typing.  If `release()` is called in a template, the expectation would be ownership transfer.

`release` can be renamed `detach` or just removed - the user can call `get()` then `reset()`.

(P.S. same with `retain_ptr::release()` (P0468), although its semantics are even subtler - the refernce count lives with the raw pointer, not the smart pointer, and only responsibility is being transferred.)


### 3. **_Rename `observer_ptr`_**

Naming
------

`observer_ptr` is a bad name.  It is bad because "observer" already has common meaning in programming (ie "the observer pattern" https://en.wikipedia.org/wiki/Observer_pattern).
`observer_ptr` is so bad that it is a great name to use throughout this paper, as it is a clearly only a placeholder, not a reasonable suggestion, and thus doesn't bias to any of the other good names to follow.

Criteria:

There are some criteria to use when considering naming:

- Not understanding is better than MISunderstanding.  (“It is better to know nothing than to know what ain’t so.” - Josh Billings, 1874)
- Coining a term is OK - it will forever have that meaning (ie "observer" means observer pattern). The hard part is finding a good term not already used - an arbitrary term (like _iota_) is not good "coinage" (but it does show how coining a term - attaching meaning - works). A coined term should at least contain a hint that our brains can cling to.
- Avoid negatives, as these quickly lead to double negatives in code (ie `if (!noSoup)...`)
- Avoid spoken ambiguity.  ie `raw_ptr` vs "raw pointer".

A list of names
---------------

Mostly the names can be grouped into a few piles:

* ONWERSHIP: smart pointers tend to be about ownership.  This one is lack of ownership.  But the pointer is still owned (hopefully!), just not by you.  So `unowned_ptr`, for example, is not correct. `notmy_ptr` is more correct.
* USAGE: Instead of defined-by-contrast, we could focus on how it is meant to be used - it is best used as a param (like string_view) and is only temporary. Thus names like temp/brief/transient/sojourn/... It is also meant to grant _access_ to an object, no-more-no-less, thus `access_ptr`.
* FUNCTIONALITY: We can define a class by what it is and what it offers, and let users decide how to use it.
* COINAGE: Picking a word that is currently unused, and give it meaning in the programming context.  But it should at least hint at meaning.

| vote | name | pros | cons |
|---|------|------|------|
|   | | | |
|   | OWNERSHIP | | |
|   | | | |
|   | notmy_ptr | intent | cheeky, double negative |
|   | nonowning_ptr | intent | double negative | 
| 1 | cadged_ptr | very correct, coins a term | not well known |
|   | borrowed_ptr | | but how do you give it back? |
|   | loaned_ptr | | |
|   | someones_ptr | intent | cheeky |
|   | dang_ptr | dangling/danger, coins a term | cheeky |
|   | dependent_ptr | | `[[carries_dependency]]`? |
|   | trust_ptr | | I trust it will live long enough |
|   | exempt_ptr | ownership, obviously | exempt from what? |
|   | | | |
|   | HOW (USAGE) | | |
|   | | | |
| 1 | access_ptr | grants access, no more no less | |
| 1 | alias_ptr | aliases a ptr out there somewhere | |
|   | temp_ptr  | use | |
|   | brief_ptr |  | i before e |
|   | transient_ptr | intent | long |
|   | ephemeral_ptr | intent | long |
|   | guest_ptr  | | |
|   | param_ptr | | |
|   | sojourn_ptr | intent | uncommon |
|   | | | |
|   | WHAT | | |
|   | | | |
| 1 | object_ptr | Anthony Williams library | |
|   | basic_ptr | basic_string? | captures functionality, but not intent |
|   | common_ptr | | functionality, not intent |
|   | view_ptr | | a pointer to a view? |
|   | ptr_view | like string_view | doesn't end in ptr? |
|   | | | |
|   | COINAGE | | |
|   | | | |
|   | naive_ptr | gives fair warning | |
| 1 | klein_ptr | https://en.wikipedia.org/wiki/Minimalism#/media/File:IKB_191.jpg | not Klein bottle |
|   | neutral_ptr | | |
|   | thin_ptr | | |
|   | tepid_ptr | | |
|   | lax_ptr | (relaxed/lackadaisical) | |
|   | loose_ptr | | |
|   | assumed_ptr | | |
|   | presumed_ptr | | |
|   | | | |
|   | etc | | |
|   | | | |
|   | dumb_ptr | | politically incorrect? |
|   | bum/freeload/mooch | | slang |
|   | viewing_ptr | | is that read only? |



From the original paper (N3740)

aloof_ptr  
agnostic_ptr  
apolitical_ptr  
ascetic_ptr  
attending_ptr  
austere_ptr  
bare_ptr  
blameless_ptr  
classic_ptr  
core_ptr  
disinterested_ptr  
disowned_ptr  
disowning_ptr  
dumb_ptr  
emancipated_ptr  
estranged_ptr  
excused_ptr  
exempt_ptr  
faultless_ptr  
free_ptr  
freeagent_ptr  
guiltless_ptr  
handsoff_ptr  
ignorant_ptr  
impartial_ptr  
independent_ptr  
innocent_ptr  
irresponsible_ptr  
just_a_ptr  
legacy_ptr  
naked_ptr  
neutral_ptr  
nonown_ptr  
nonowning_ptr  
notme_ptr  
oblivious_ptr  
observer_ptr  
observing_ptr  
open_ptr  
ownerless_ptr  
pointer  
ptr  
pure_ptr  
quintessential_ptr  
severe_ptr  
simple_ptr  
stark_ptr  
straight_ptr  
true_ptr  
unfettered_ptr  
uninvolved_ptr  
unmanaged_ptr  
unowned_ptr  
untainted_ptr  
unyoked_ptr  
virgin_ptr  
visiting_ptr  
watch_ptr  
watcher_ptr  
watching_ptr  
witless_ptr  
witness_ptr  


Very Scientific poll
---------------
by Hadriel Kaplan  
https://strawpoll.com/c4fd88ap  
https://www.reddit.com/r/cpp/comments/808c5z/bikeshedding_time_poll_for_a_new_name_for/


| count | name               | description/reason |
--------|--------------------|------------------------------------------- |
|   121 | pointy_mcpointface | because it's reddit |
|   100 | access_ptr         | it grants access, not ownership |
|    57 | observer_ptr       | don't care if it's misleading |
|    40 | unowned_ptr        | a non-owning "smart" pointer |
|    37 | pointer or ptr     | the word already describes it |
|    37 | use_ptr            | you can use the object, but not destroy it |
|    24 | borrowed_ptr       | its borrows something someone else owns |
|    21 | view_ptr           | something to view an object |


Implementation Experience
----------

- Anthony Williams - https://www.justsoftwaresolutions.co.uk/cplusplus/object_ptr.html - includes implicit conversions, removed `release()`
- Martin Moene - https://github.com/martinmoene/observer-ptr-lite
- (IIUC) Ville has tried these changes against libstdc++ and its testsuite

Acknowledgement
----------

Thank you Walter for the original proposal. Thanks to Ville, Anthony, Martin, and others for their encouragement and implementation experience.

References
---------

N4282 - A Proposal for the World’s Dumbest Smart Pointer, v4 - Walter E. Brown  
P1408 - Abandon observer_ptr - Bjarne Stroustrup  
P0705 - Implicit and Explicit Conversions - Tony Van Eerd  
P0468 - An Intrusive Smart Pointer - Isabella Muerte  
P1132 - out_ptr - a scalable output pointer abstraction - JeanHeyd Meneide, Todor Buyukliev, Isabella Muerte  
P0201 - A polymorphic value-type for C++ - Jonathan Coe, Sean Parent  

