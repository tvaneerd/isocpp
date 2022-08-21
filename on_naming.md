
### Essence is the essence of naming

Often things are hard to name because you don't actually know what it is you are naming.  Which things are (implementation/unimportant) details, which things are essential to the thing you are naming.  And if you can't narrow down the essence of the thing, there is a good chance the thing isn't designed correctly - fix your design before naming it, or, alternatively by naming it you may figure out what it is you are really building.

Also, knowing the essence can help further design

Some examples:

std::map should probably have been called std::ordered_map - the _ordering_ of the Key is a requirement on the map, and it is _essential_.  It may appear as almost an implementation detail, but it bleeds all over the interface - making it essential.

Yet because it is given the generic "map" name, people expect all things to be mappable, and the ordering part of `map` is actually seldomly used - `map[]` is _always_ used, and the order is _never_ important (for sufficient values of _never_ and _always_).
"Everything should be mappable" leads to "everything should be orderable" leads to people wanting default operator< for types where "less" is nonsensical. Yes, any bag of bytes (ie a struct/class) is orderable, and thus mappable, but the ordering part doesn't make sense.
/rant

#### Corollary - don't separate the essence

`std::fp::binary16_t` is/was a suggested name for a 16 bit floating point number.  This would lead to some code using `binary16_t` (without the namespace qualification) but "binary16_t" doesn't scream number, it screams binary data of some kind, like unsigned shorts or something.  Don't put the essence in the namespace, where it can get separated along the journey.

### Level of generality

This goes along with essence.

Look out for how general your name is.  Example: "map" is a very general name; "unordered_map" is a "map", but more specific; "hash_map" is yet more specific.  Yet, even still, there are many possible hash_maps. "siphash_map", for example, would specify the hash. Alternatively, "redblack_map" would specify the underlying data structure (and thus iterator stability guarantees, etc).

A name is basically never _more_ specific than the class/function it names.  If your function is named "introsort" but does a bubble sort, then you have a bug.  If your function is named "sort", it could be any sort (but typically, after decades of work by computer scientists on sorting, we expect it to fast, and not bubble sort :-).

A name that is exactly specific is probably too long - how do you describe every detail in one word? "flat_map_with_array_of_keyvalues" vs "flat_map_with_array_of_keys_and_array_of_values".  But sometimes you can be close - typically because someone took a general, unused, or made-up name, and now it means something specific - ie "introsort", "redblack tree".

So typically a name will be more general than the thing it names.  That's OK.  But be wary of being tooooo general.
Too general means:

- users will expect it to work for all cases (I need a map, it is called `map` why isn't it right for this case?)
- usage then guides future changes (ie change `map` to be unordered because that's more efficient and the common use case?) and new features. These new features might not "fit" well with the original design, because the original design was targeting one niche whereas current use targets another, but the class was able to "drift" between design spaces because it was given a generic name that covered both niches.  A more specific name may have made it more obvious that a different class was needed.

Of course, in day-to-day code, you often don't yet know what the class is or will be in the future - you know it is a work in progress.  But even then, at least _thinking_ about where on the "generality spectrum" the name is, might help you better understand the thing you are designing.

A more general name tends to imply more of a commitment to be usable in more general situations.

See also "essence" and "by use vs by functionality"

### Describe the thing in detail – what words did you use?

The name is probably in there. Or, if you can't describe something clearly and succinctly, you may have a bad design.



### Be Consistent

This is the number 1 API design guideline (applies not only to naming, but param order, contracts, etc)
Consider any/optional/variant/etc before Vicente's papers - `any::clear()`, `optional::reset()`; `any::empty()`, `optional::operator bool()`, `has_value()` ... ?

Not all consistency is equal. Local consistency is more important than Global.consistency

- Self consistency
- Similar consistency
- ...
- ...
- Global Consistency

(eg. `optional<T>::operator>=()` is consistent with `T`, mildly inconsistent with `vector`)


### Be Consistent in "warning" signs

std::optional uses operator* _precisely_ because developers have learned to see operator* as "dangerous" - ie "what if that pointer is null?" By using the same operator, you get the same warning signs - "what if that optional is empty?"

smart-ptr get() is no more "dangerous" than the raw pointer it wraps. Whereas optional adds the additional empty state, smart-ptrs don't. So `get()` doesn't throw like optional's `value()` does.  So `get()` isn't "dangerous" (more than the pointer already is) but `get()` is a _different_ warning sign - it is a sign that you are removing some type-safety.  A `unique_ptr` guarantees you are the only owner, but if the code is littered with `up.get()` everywhere, you start to question that "guarantee".

#### Be greppable

'get' is hard to grep. Or way too easy - too many results.  Of course, code analytics and search tools are getting better, and you can search carefully for ".get(" or "get<", etc, but something like "unwrap" is still easier to search for - possibly easier for our _eyes_ as well, as much searching - particularly in code reviews - is just visual, not textual.

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

This should include being consistent with other programming languages and "nearby" fields (math, science, ...).  "Generators" already has meaning in Python, we shouldn't use that term to mean something else.



### Co-opt a term

The problem with name collision is that most of the good names are already taken.  For example, we recently took a fairly generic term "view' and imbued it with more specific semantics (read-only, etc).  That's OK.  Co-opting a term gives us a short name that encapsulates a larger idea (that is the purpose of names).

But as good names get taken, we may need to get a bit more creative when finding new terms that don't collide.
eg. Boost had a somewhat-like-partitioning algorithm, and the suggested name was `stratify` (as it separated data into strata - not completely ordered, but...etc).
As far as I'm concerned, that term is now taken.

The co-opted term, needs to be *close* to the right meaning, so that programmers can quickly latch onto how it applies to the named thing.
`cadged_ptr` is an *accurate* name for `observer_ptr` (look up meaning of 'cadged'), but most people are unfamiliar with the term, and thus can't latch onto it.

Reminder: _"Extent" can mean many things, but once we use it, it can only mean one thing._



### Avoid negatives – thus avoiding double negatives

`noexcept(false)` ??? I need to think twice every time I see it.



### Avoid spoken ambiguity (or learn to pronounce _, Capitals, etc)

"You need to use a raw pointer here"  Did I mean `T*` or `raw_ptr<T>` ?
I've experienced this confusion with `std::function`.  (The solution is to say "you need a _standard_ function here" (yes, `std` is pronounced "standard"!)



### Avoid verb/noun ambiguity

`empty()`


### hints of complexity

A Name should hint at the complexity/performance of the function.

`getFoo()` should be fast, like return an existing member variable, or some very simple calculation like `getArea() const { return w * h; }`  
`find` should typically be O(logN) or linear  
`determine` or `calculate` could be worse than linear


### Be *Conceptually* Concise. Avoid sub-concepts.

That is the point of words, basically.

`delayed_computation_range` vs `lazy_range`.  One is easy to 'grok' quickly.  And was that delayed-computation range, or delayed computation-range?
`not_my_ptr` vs `notmy_ptr`. (also double negative)



### By use or by functionality?

"functionality", here, also means "by structure".  `std::pair` is a pair by structure, and offers the common functionality that can be offered by that structure.

`void_t` is named by functionality, but doesn't hint at typical use.  Sometimes use is better; sometimes functionality is better.  This relates to top-down vs bottom-up.

(In "normal" (non-STL) code this is often decided by future plans - I work with projectors. `projector.getRelativeBightness(x,y)` doesn't really return correctly calculated brightness.
Should I admit that by the name (ie call it what it really does - `getRelativePixelSize(x,y)`) or should I promise that it will become more correct in the future?)
And then you need to worry about whether people use it for what it says, vs what is does.  This relates to specification vs implementation, and users relying on implementation...)

The more general, the more likely it is to be named by functionality (because if it is general, you can't know how it will be used).

Note that "by use" implies more of a commitment to that usage - future features will follow from that usage.

# General Rule for Abbreviations

If it is the first search result in google, you can use the abbreviation, otherwise probably not.

ie search for `NFC` - the first result is about Near-field Communication.

But search for `ISM` - you don't get anything about the ISM frequency band that NFC runs on.

