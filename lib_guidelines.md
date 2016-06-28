## Potential Guidelines for LEWG/LWG Reviewers and Authors

### What?

This doc is what it is.  Maybe when it is something then we'll know what it is.

------------



### Regular

| Regular | Irregular |
| --- | --- |
| Yes | you better have a good reason |

Everything works better with Regular types, and most of the STL assumes or works best with Regularity
(although we have added some support for move-only types, etc).

See EoP.



### Const implies thread-safe reads

Yes, yes it does.  Don't break that.  Don't forget to check that.

If there is a large cost incurred to ensure that guarantee, instead of breaking the guarantee, find a different design.




### Explicit vs Implicit

When should a constructor or conversion be implicit vs explicit?

| Explicit | Implicit |
| --- | --- |
| if in doubt |  |
| if might throw | doesn't throw / noexcept |
| if info/accuracy is lost | if both types represent the same "platonic" thing (string/string_view/char* all represent "strings") |
| if performance penalty | no performance penalty (time nor space) |

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



### Consistency

Todo: copy/paste from https://github.com/tvaneerd/isocpp/blob/master/making_optional_greater_equal.md

