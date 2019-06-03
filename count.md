## ssize Should be Named count

Document number: DXXXX  
Date: 2019-06-01  
Audience: LEWG  
Reply-to: Tony Van Eerd. count at forecode.com

---


Summary
-------

To resolve (or at least temporarily quell) the signed/unsigned debate, P1227 (approved in Kona) proposes adding a member function `ssize()` to `span`,
which is just like `size()`, but which returns a signed value.  It also proposed adding `std::ssize(container)` for all standard containers,
and `ssize()` member functions for all containers, not just `span`.

Bjarne's P1491 (and P1428) however notes:

> [p1227R1] proposes to change `size()` in the ranges TS and for `span` to `unsigned` (making them bug compatible with the STL)
> and adding `ssize()` to all containers and range accessors
>  * embeds a type in a function name (and it makes me think of Parseltongue :-) )
>  * leaves the wrong solution (IMO) with the better, more established, and simpler name

This proposal: Use the name `count` instead of `ssize`
 * does not embed type info
 * is a better name than `ssize`
 * is actually a better name than `size` (which is confused with `sizeof`)
 * will eventually (20years?) become the established, simple, more intent-ful name
 

Current uses of `count()`
------------------------

Functions named `count` can already be found in the standard.

`bitset<N>::count` returns the number of on bits - consistent???

`AssociativeContainer::count(key)` returns the number of items with key.  This is consistent with count() returning _all_ items

counted_iterator - count is consistent

count(first, last, [proj])
count_if(first,last, pred)
count(first, last, value)

duration.count() - number of ticks










Corollary
---------

Introduce `std::count_t` as the type returned by count()
