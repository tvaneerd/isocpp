
### Essence is the essence of naming

Often things are hard to name because you don't actually know what it is you are naming.  Which things are (implementation/unimportant) details, which things are essential to the thing you are naming.  And if you can't narrow down the essence of the thing, there is a good chance the thing isn't designed correctly - fix your design before naming it, or, alternatively by naming it you may figure out what it is you are really building.

Also, knowing the essence can help further design

Some examples:

std::map should probably have been called std::ordered_map - the _ordering_ of the Key is a requirement on the map, and it is _essential_.  It may appear as almost an implementation detail, but it bleeds all over the interface - making it essential.

Yet because it is given the generic "map" name, people expect all things to be mappable, and the ordering part of `map` is actually seldomly used - `map[]` is _always_ used, and the order is _never_ important (for sufficient values of _never_ and _always_).
"Everything should be mappable" leads to "everything should be orderable" leads to people wanting default operator< for types where "less" is nonsensical. Yes, any bag of bytes (ie a struct/class) is orderable, and thus mappable, but the ordering part doesn't make sense.
/rant



### Describe the thing in detail – what words did you use?

The name is probably in there. Or, if you can't describe something clearly and succinctly, you may have a bad design.



### Be Consistent

This is the number 1 API design guideline (applies not only to naming, but param order, contracts, etc)
Consider any/optional/variant/etc before Vicente's papers - `any::clear()`, `optional::reset()`; `any::empty()`, `optional::operator bool()`, `has_value()` ... ?

Not all consistency is equal. Local consistency is more important than Global.consistency
• Self consistency
• Similar consistency
• ...
• ...
• Global Consistency

(eg. `optional<T>::operator>=()` is consistent with `T`, mildly inconsistent with `vector`)


### Be Consistent in "warning" signs

std::optional uses operator* _precisely_ because developers have learned to see operator* as "dangerous" - ie "what if that pointer is null?" By using the same operator, you get the same warning signs - "what if that optional is empty?"

smart-ptr get() is no more "dangerous" than the raw pointer it wraps. Whereas optional adds the additional empty state, smart-ptrs don't. So `get()` doesn't throw like optional's `value()` does.  So `get()` isn't "dangerous" (more than the pointer already is) but `get()` is a _different_ warning sign - it is a sign that you are removing some type-safety.  A `unique_ptr` guarantees you are the only owner, but if the code is littered with `up.get()` everywhere, you start to question that "guarantee".


### Be _Glaringly_ Inconsistent

    optional<float> op;
    expected<float> ex;
    any an;
    variant<float,int> vr;

    op.has_value()
    ex.has_value()
    an.has_value()
    vr.valueless_by_exception()

IF/WHEN you have consistency, you can use the power of INconsistency for good.  An inconsistency tells the developer "look here, this is NOT the same as the rest, and since our API is always so consistent, there must be a REASON why it's inconsistent - make sure you know that reason."


### NOT understanding is better than MISunderstanding ("name collision")

`observer_ptr` - does that relate to the Observer Pattern? `view_ptr`? Does that relate to other uses of `view` in the STL (`string_view`, etc)?

The big problem of MISunderstanding is that you *think* you understand. So you proceed.
NOT understanding at least gives you a chance to read the docs instead of guessing/assuming.
(check out [On Being Wrong](https://www.ted.com/talks/kathryn_schulz_on_being_wrong) TED talk by Kathryn Schulz - Being wrong feels exactly like being right. *Realizing* you are wrong feels different.)

This is somewhat the flip side of the consistency coin.



### Co-opt a term

The problem with name collision is that most of the good names are already taken.  For example, we recently took a fairly generic term "view' and imbued it with more specific semantics (read-only, etc).  That's OK.  Co-opting a term gives us a short name that encapsulates a larger idea (that is the purpose of names).

But as good names get taken, we may need to get a bit more creative when finding new terms that don't collide.
eg. Boost had a somewhat-like-partitioning algorithm, and the suggested name was `stratify` (as it separated data into strata - not completely ordered, but...etc).
As far as I'm concerned, that term is now taken.

The co-opted term, needs to be *close* to the right meaning, so that programmers can quickly latch onto how it applies to the named thing.
`cadged_ptr` is an *accurate* name for `observer_ptr` (look up meaning of 'cadged'), but most people are unfamiliar with the term, and thus can't latch onto it.



### Avoid negatives – thus avoiding double negatives

`noexcept(false)` ??? I need to think twice every time I see it.



### Avoid spoken ambiguity (or learn to pronounce _, Capitals, etc)

"You need to use a raw pointer here"  Did I mean `T*` or `raw_ptr<T>` ?
I've experienced this confusion with `std::function`.  (The solution is to say "you need a _standard_ function here" (yes, `std` is pronounced "standard"!)



### Avoid verb/noun ambiguity

`empty()`



### Be *Conceptually* Concise. Avoid sub-concepts.

That is the point of words, basically.

`delayed_computation_range` vs `lazy_range`.  One is easy to 'grok' quickly.  And was that delayed-computation range, or delayed computation-range?
`not_my_ptr` vs `notmy_ptr`. (also double negative)



### By use or by functionality?

`void_t` is named by functionality, but doesn't hint at typical use.  Sometimes use is better; sometimes functionality is better.  This relates to top-down vs bottom-up.

(In "normal" (non-STL) code this is often decided by future plans - I work with projectors. `projector.getRelativeBightness(x,y)` doesn't really return correctly calculated brightness.
Should I admit that by the name (ie call it what it really does - `getRelativePixelSize(x,y)`) or should I promise that it will become more correct in the future?)
And then you need to worry about whether people use it for what it says, vs what is does.  This relates to specification vs implementation, and users relying on implementation...)

The more general, the more likely it is to be named by functionality (because if it is general, you can't know how it will be used).



