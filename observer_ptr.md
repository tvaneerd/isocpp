If/When observer_ptr is standardized...

Reasons to standardize
----------------------

1. `observer_ptr` is a post-modern tool for transitioning a codebase to more modern C++.  
ie review all raw pointers, replace each case with the correct smart pointer (ie `unique_ptr` hopefully, else `shared_ptr`, etc).
"unowned" pointers need a thing to be replaced with (ie `observer_ptr`) else it is hard to know which raw pointers have been review, and which haven't.
(If *all* owned pointers in a codebase are wrapped with smart pointers,
then a raw pointer can mean "unowned". But most codebases are mixed with new and old uses.)

2. `observer_ptr` for `unique_ptr` and `shared_ptr` is like `string_view` for `char *` and `string`.  
(For even the most modern of codebases...) ie a common base type that any pointer can be temporarily safely converted to.  You don't want smart pointers to implicitly convert to raw pointers
(as this can too easily lead to accidental misownership - ie `delete someSharedPtr;`), but converting to `observer_ptr` does not increase the risk of misownership.
It does increase risk of dangling - same as string_view does.  Thus, similar to `string_view`, `observer_ptr` is best used as a function param.

Changes
-------

**Implicit Conversions**

Allow implicit conversion from other smart pointers. (Just std ones or detection?)   
Allow implicit conversion from raw pointers. ie `T *`. (This also covers anything convertible to `U*` that is convertible to `T*`)  

Why? 


<table>
<tr>
<th>C++</th>  <th>C++LibFun2</th>  <th>C++20</th>
</tr>
<tr>
<td  valign="top">

<pre lang="cpp">
void f(Foo * pf);
    
f(shared_p.get());
f(unique_p.get());
f(raw_p);
</pre>
</td>
<td  valign="top">

<pre lang="cpp">
void f(observer_ptr&lt;Foo&gt; pf);
    
f(observer_ptr(shared_p.get()));
f(observer_ptr(unique_p.get()));
f(observer_ptr(raw_p));
</pre>
</td>
<td  valign="top">

<pre lang="cpp">
void f(observer_ptr&lt;Foo&gt; pf);
    
f(shared_p);
f(unique_p);
f(raw_p);
</pre>
</td>
</tr>
</table>

- The leftmost version has calls to `get()` which require extra scrutiny in a code review (as you are removing the safety latch), as does the function `f` that takes a raw pointer.
- The middle version is just noisy - for no good reason - the noise does not protect anything dangerous.
- The rightmost version is safe - `observer_ptr` doesn't change ownership semantics. Nothing extra should be required.

Basically:
- In terms of P0705 conversion rules: They represent the same values, and it is safe.
- As mentioned above,  
`obeserver_ptr` is to `shared_ptr`, `unique_ptr`, and `T*`  
as `string_view` is to `char *` and `string`.



**Review  `release()` and `reset(ptr)`.**

`release()` - The name implies ownership - and ownership transfer. use `get()` or `reset()`.  In generic code, you might call `release()` to take ownership.  With `observer_ptr` is does NOT tranfer ownership.  Different semantics require different name.
Actually, `shared_ptr` doesn't transfer ownership on `release` either, as some other `shared_ptr` might still own it.

`reset(ptr)` - for all other smart pointers, this _suggests_ taking ownership (although not as obvious as `release()`).   Maybe use `=`?


Naming
------
This paper is really about naming.

`observer_ptr` is a bad name.  It is bad because "observer" already has common meaning in programming (ie "the observer pattern" https://en.wikipedia.org/wiki/Observer_pattern).
`observer_ptr` is so bad that it is a great name to use throughout this paper, as it is a clearly only a placeholder, not a reasonable suggestion,
and thus doesn't bias to any of the other good names to follow.

Criteria:

- not understanding is better than MISunderstanding.  (Mark Twain: “It ain't what you don't know that gets you into trouble. It's what you know for sure that just ain't so.”)
- coin a term is OK, it will forever have that meaning (ie "observer" means observer pattern), if you can find a good term not already used - an arbitrary term (like _iota_) is not good "coinage". A coined term should at least contain a hint that our brains can cling to.
- avoid negatives, as these quickly lead to double negatives in code (ie `if (!noSoup)...`)
- avoid spoken ambiguity.  ie `raw_ptr` vs "raw pointer"

A list of names
---------------

* ONWERSHIP: smart pointers tend to be about ownership.  This one is lack of ownership.  But the pointer is still owned (hopefully!), just not by you.  So `unowned_ptr` is not correct. `notmy_ptr` is more correct.
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
|   | exempt_ptr | ownership, obviously | exempt from what? |
|   | | | |
|   | USAGE | | |
|   | | | |
| 1 | access_ptr | grants access, no more no less | |
|   | temp_ptr  | use | |
| 1 | brief_ptr |  | i before e |
|   | transient_ptr | intent | long |
|   | ephemeral_ptr | intent | long |
|   | guest_ptr  | | |
|   | sojourn_ptr | intent | uncommon |
|   | | | |
|   | FUNCTIONALITY | | |
|   | | | |
|   | basic_ptr | basic_string? | captures functionality, but not intent |
|   | common_ptr | | functionality, not intent |
|   | view_ptr | | a pointer to a view? |
|   | ptr_view | like string_view | doesn't end in ptr? |
|   | | | |
|   | COINAGE | | |
|   | | | |
|   | naive_ptr | gives fair warning | |
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
blond_ptr  
blonde_ptr  
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

