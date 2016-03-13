### tl;dr:

    optional<X> opt = ....;

    assert(( opt >=  opt) == true);‎
    assert((*opt >=  opt) == true);
    assert(( opt >= *opt) == true);
    assert((*opt >= *opt) == false);


‎One of those is not like the others.


Similarly, for some (possibly many*) classes Y:


    optional<Y> opt = ....;

    ... ( opt >=  opt) ... // OK
    ... (*opt >=  opt) ... // OK
    ... ( opt >= *opt) ... // OK
    ... (*opt >= *opt) ... // compile error!!!

(*many people implement < for sake of map, but don't implement the other operators)

This inconsistency in optional is a very simple and small problem with a very simple and small fix:
`optional>=` needs to call T's `operator>=`. (optional currently instead calls `!operator<()`, which is typically,
*but not always*, the same result as `X::operator>=`,
ie consider `X` == `float`)

If you agree with the small fix (or already assumed it worked that way) you don't really need to read any further. It is that simple.





### 3 categories of classes


| Category | Examples |
---:|---
| **Aggregates:** | pair, tuple;  struct { int m, n; };  struct { float m; };  class MyFoo {...}; |
| **Wrappers:** | optional, variant, any, expected, ... |
| **Containers:** | int x[17], array, vector, vector, vector |



Notes:
- you can argue that `int x[17]` is an Aggregate. But it is not a Wrapper.
- I hear there are other containers past vector, but you should just use vector :-)
- `struct { float m; }` is an aggregate, not a wrapper

### Wrappers
('Proxy'? Some better name?)

So what "defines" or at least hints at a Wrapper
- _implicit_ construction from the 'wrappee'
- conversion to the wrappee
- relational operators between wrapper and wrappee ie `optional<X>{} < X{}`
- in general trying to 'act like' the wrappee (proxy etc)

The above is why `struct { float m; }` is not a Wrapper/Proxy, but an Aggregate. 

### Consistency

We want everything to be consistent. Sometimes this is not possible. What should we do?

The following is probably obvious when stated, but still needs to be stated sometimes:

Not all consistency is valued equally. There is a scale:

- Self consistency
- Similar consistency
- ...
- ...
- Global Consistency

(And the value of the scale needs to be weighed with the cost.
Self consistency is more valuable and thus you can/should be willing to spend more effort on that cost.
Consistency at global scale, however, may not be worth the cost.)


So how does consistency apply to the current situation with optional?

Wait. First, however, optional needs to be correct. Being consistently wrong is not near as good as consistently right.
Of these lines:

```
    optional<float> opt = NaN;

    assert(( opt >=  opt) == true);‎
    assert((*opt >=  opt) == true);
    assert(( opt >= *opt) == true);
    assert((*opt >= *opt) == false);
```

only the bottom one is correct. (or you can argue that only the bottom one can't change, as we are not changing how float works)

So we need to change the other three. And it is not that "we have to", it is what makes sense.
If `optional>=` calls T's `>=` we get (of course)

    assert(( opt >=  opt) == false);‎
    assert((*opt >=  opt) == false);
    assert(( opt >= *opt) == false);
    assert((*opt >= *opt) == false);

This makes optional *consistent with itself* and T. 

#### Consistency with neighbours

- The small fix makes optional _closer_ to being consistent with aggregates - `optional<float>` now gives the same results as `struct { float m; }`.
But optional is still slightly inconsistent vs Aggregates when dealing with exotic types.
For Aggregates, `operator>=` calls `operator>` and `operator=` instead of calling memberwise `>=`.
(We can't change Aggregates. For aggregates with more than one member, you cannot build a sensible lexicographical `>=` from only memberwise `>=`.
For *single* member, you could use `>=` directly, but then single-member aggregates would not be consistent with multi-member aggregates.)


- The small fix makes optional _slightly_ inconsistent with Containers. Fine. Optional is not a Container, it is a Wrapper.
And for "normal" types, they are all still consistent. More importantly, they are each consistent within their own category. 

- (Also, regardless of this fix, Containers are not consistent with Aggregates (of floats, for example). We don't suggest changing Containers or breaking code.
Also, Containers (often) are used to find() things via an equivalence (not equality) relation, so maybe Containers aren't broken. Maybe.)
