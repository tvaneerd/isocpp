## Potential Guidelines for LEWG/LWG Reviewers and Authors

### What?

This doc is what it is.  Maybe when it is something then we'll know what it is.

------------



### Regular

Should my type be Regular (see definition in EoP)?

| Regular | Irregular |
| --- | --- |
| Yes | you better have a good reason |

Everything works better with Regular types, and most of the STL assumes or works best with Regularity
(although we have added some support for move-only types, etc).

See EoP.



### Const implies thread-safe reads

Yes, yes it does.  Don't break that.  Don't forget to check that.

If there is a large cost incurred to ensure that guarantee, instead of breaking the guarantee, find a different design.




### Const-correctness and deep-const

Const is tied together with ownership.  A container like vector _owns_ its elements, thus vector::begin() const returns a const_iterator which propagates constness.

A Range, however, doesn't own the elements, a range is more like a pointer.  So a const Range can still range over mutable elements (like a const pointer can point to non-const data).

\<insert Geoffrey here\>




### Explicit vs Implicit

When should a constructor or conversion be implicit vs explicit?

| Explicit | Implicit | Notes/Examples |
| --- | --- | --- |
| usually |  |  |
| if might throw | doesn't throw / noexcept | what about string from char * ? If we had string_view earlier... |
| if info/accuracy is lost | if both types represent the same "platonic" thing | string/string_view/char* all represent "strings"; int, long represent "numbers" |
| if performance penalty | no performance penalty (time nor space) |   |
| if dangerous (eg dangling pointer/ref) | not dangerous | char * from string would be performant, (mostly) accurate, noexcept; but forms a dangerous relationship |


#### Should the conversion exist at all

... and should it be a separate function, or a conversion operator?
ie `someByte.to_integer<int>()` or `int(someByte)` ?

The crux is, as above in Explicit vs Implicit, whether the two types represent the same "thing".

For example, An `EmployeeRecord` and an `EmployeeId` do both represent the same thing - an employee.  Conversion between these makes sense.

`EmployeeRecord` ->  `EmployeeId` could probably be implicit as there is no cost nor risk, and no info is lost (that can't be recovered).(?)

`EmployeeId` -> `EmployeeRecord` would be explicit as it could probably throw (probably allocates strings),
and it probably has a cost (look up Record from Id).

Now you could also probably convert a `EmployeeRecord` -> `PostalCode`, but that is a "hasa" relationship not an "isa".
They don't represent the same external thing.

`EmployeeRecord` -> `EmployeeId` may also seem "hasa" but `EmployeeRecord` and `EmployeeId` are actually isomorphic (modulo performance)
and do represent the same thing.




### When to use the same name

When should we use the same name (ie, typically function name), and when should we use a different name.

This applies both to same name within a class (ie overloading) and same name _across_ classes - ie size() on containers, ie concepts/categories of classes.

| Same name | different name |
| --- | --- |
| when a template would still make sense | |
| same performance | | 
| same "intent" | |

Basically, if you have a template that calls `foo.size()` you have expectations on what `size()` means, how it performs, etc.

Whenever we reuse a name within the standard library, we should imagine a template that uses that name.  Does the template work with all uses of that name?




### Naming

_**view**_ see `string_view`.  A `view` ranges over _immutable_ elements.  
_**span**_ see `span` (?). A `span` ranges over _mutable_ elements.  
_**_ref**_ ? (array_ref?)  
_**_ptr**_ ?  

_**empty**_ vs is_empty etc?
_**get_x**_ vs x()  (and set_x(X val) vs x(X val))



### More on Naming

\<insert thoughts from "On Naming" here\>




### member functions vs free functions




### Consistency

We want everything to be consistent. Sometimes this is not possible. What should we do?

The following is probably obvious when stated, but still needs to be stated sometimes:  
**_Not all consistency is valued equally._** There is a scale (from greatest to least value):

- Self consistency
- Similar consistency
- ...
- ...
- Global Consistency

For example, `optional<T>` is, first and foremost, consistent with `T`. (For example, see `operator>=`).  
Then, where possible, it is consistent with similar types (`variant`, `any`, `expected`, smart pointers, etc)  
Then, where possible, it is consistent with other std::library types.  
Etc.  


### Categories of Types

The standard contains many related types.  It may be useful to categorize them and understand their commonalities and differences.

**Aggregates:** pair, tuple;  struct { int m, n; };  struct { float m; };  class MyFoo {...};  
**Wrappers:** optional, variant, any, expected, ...  
**Containers:** int x[17], std::array, vector, vector, vector, ...

**Smart Pointers:** are these just wrappers?  Is it a orthogonal property?


