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

As mentioned above, `obeserver_ptr` is like `string_view`, and it would be good to have a type that smart pointers can safely convert to. `f(observer_ptr<Foo> p)` should be safe to call with a `shared_ptr` or a `unique_ptr`.

And it is safe. And they represent the same values. (see P0705 for in depth discussion on conversion rules.)



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



| vote | name | pros | cons |
|---|------|------|------|
|   | | | |
|   | OWNERSHIP | | |
|   | | | |
|   | notmy_ptr | intent | cheeky, double negative |
|   | nonowning_ptr | intent | double negative | 
|   | cadged_ptr | very correct, coins a term | not well known |
|   | borrowed_ptr | | but how do you give it back? |
|   | loaned_ptr | | |
|   | someones_ptr | intent | cheeky |
|   | dang/danged_ptr | dangling/danger, coins a term | dang_ptr! :-) |
|   | dependent_ptr | | `[[carries_dependency]]`? |
|   | exempt_ptr | ownership, obviously | exempt from what? |
|   | | | |
|   | USAGE | | |
|   | | | |
|   | access_ptr | grants access, no more no less | |
|   | temp_ptr  | use | |
|   | brief_ptr |  | i before e |
|   | transient_ptr | intent | long |
|   | ephemeral_ptr | intent | long |
|   | guest_ptr  | | |
|   | sojourn_ptr | intent | uncommon |
|   | | | |
|   | basic_ptr | basic_string? | captures functionality, but not intent |
|   | common_ptr | | functionality, not intent |
|   | view_ptr | | a pointer to a view? |
|   | ptr_view | | doesn't end in ptr? |
|   | | | |
|   | naive_ptr | gives fair warning | |
|   | neutral_ptr | | |
|   | thin_ptr | | |
|   | tepid_ptr | coin | |
|   | lax_ptr | (relaxed/lackadaisical), coins a term | |
|   | loose_ptr | | |
|   | assumed_ptr | | |
|   | presumed_ptr | | |
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

