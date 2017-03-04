

### 0. Describe the thing in detail – what words did you use?

The name is probably in there. Or, if you can't describe something clearly and succinctly, you may have a bad design.



### 1. Be Consistent

This is the number 1 API design guideline (applies not only to naming, but param order, contracts, etc)
Consider any/optional/variant/etc before Vicente's papers - `any::clear()`, `optional::reset()`; `any::empty()`, `optional::operator bool()`, `has_value()` ... ?

Not all consistency is equal. Local consistency is more important than Global.consistency
• Self consistency
• Similar consistency
• ...
• ...
• Global Consistency

(eg. `optional<T>::operator>=()` is consistent with `T`, mildly inconsistent with `vector`)



### 2. NOT understanding is better than MISunderstanding ("name collision")

`observer_ptr` - does that relate to the Observer Pattern? `view_ptr`? Does that relate to other uses of `view` in the STL (`string_view`, etc)?

The big problem of misunderstanding is that you *think* you understand. So you proceed.
NOT understanding at least gives you a chance to read the docs instead of guessing/assuming.
(check out [On Being Wrong](https://www.ted.com/talks/kathryn_schulz_on_being_wrong) TED talk by Kathryn Schulz - Being wrong feels exactly like being right. *Realizing* you are wrong feels different.)

This is somewhat the flip side of the consistency coin.



### 3. Co-opt a term

The problem with name collision is that most of the good names are already taken.  For example, we recently took a fairly generic term "view' and imbued it with more specific semantics (read-only, etc).  That's OK.  Co-opting a term gives us a short name that encapsulates a larger idea (that is the purpose of names).

But as good names get taken, we may need to get a bit more creative when finding new terms that don't collide.
eg. Boost had a somewhat-like-partitioning algorithm, and the suggested name was `stratify` (as it separated data into strata - not completely ordered, but...etc).
As far as I'm concerned, that term is now taken.

The co-opted term, needs to be *close* to the right meaning, so that programmers can quickly latch onto how it applies to the named thing.
`cadged_ptr` is an *accurate* name for `observer_ptr` (look up meaning of 'cadged'), but most people are unfamiliar with the term, and thus can't latch onto it.



### 4. Avoid negatives – thus avoiding double negatives

`noexcept(false)` ??? I need to think twice every time I see it.



### 5. Avoid spoken ambiguity (or learn to pronounce _, Capitals, etc)

"You need to use a raw pointer here"  Did I mean `T*` or `raw_ptr<T>` ?
I've experienced this confusion with `std::function`.  (The solution is to say "you need a _standard_ function here" (yes, `std` is pronounced "standard"!)



### 6. Avoid verb/noun ambiguity

`empty()`



### 7. Be *Conceptually* Concise. Avoid sub-concepts.

That is the point of words, basically.

`delayed_computation_range` vs `lazy_range`.  One is easy to 'grok' quickly.  And was that delayed-computation range, or delayed computation-range?
`not_my_ptr` vs `notmy_ptr`. (also double negative)



### 8. By use or by functionality?

`void_t` is named by functionality, but doesn't hint at typical use.  Sometimes use is better; sometimes functionality is better.  This relates to top-down vs bottom-up.

(In "normal" (non-STL) code this is often decided by future plans - I work with projectors. `projector.getRelativeBightness(x,y)` doesn't really return correctly calculated brightness.
Should I admit that by the name (ie call it what it really does - `getRelativePixelSize(x,y)`) or should I promise that it will become more correct in the future?)
And then you need to worry about whether people use it for what it says, vs what is does.  This relates to specification vs implementation, and users relying on implementation...)

The more general, the more likely it is to be named by functionality.



### 9. Be _Glaringly_ Inconsistent

    optional<float> op;
    expected<float> ex;
    any an;
    variant<float,int> vr;

    op.has_value()
    ex.has_value()
    an.has_value()
    vr.valueless_by_exception()

IF/WHEN you have consistency, you can use the power of INconsistency for good.  An inconsistency tells the developer "look here, this is NOT the same as the rest, and since our API is always so consistent, there must be a REASON why it's inconsistent - make sure you know that reason."


