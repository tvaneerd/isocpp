Document number:

Date: 2015-11-12

Project: ISO C++, LEWG

Reply-to: Tony Van Eerd. first initial + last name at gmail dot com

#### I. Table of Contents

everything ................ page 1

#### II. Introduction/Summary, tl;dr.


variant's `corrupted_by_exception()` should be called `valueless_by_exception()`.
It's been discussed by the committee (a lot) and polled (https://docs.google.com/forms/d/1DE3ebnyeDKA2KUp0tRdZBYtkFotpgviq6VJAVeheKfg/viewanalytics?usp=form_confirm)
and although it is not overwhelming, there is consensus (as of writing!) for `valueless_by_exception`.

#### III. Motivation

`std::experimental:variant`'s function `corrupted_by_exception()` is poorly named.  In particular:

1. Regulated Industries - it has been suggested that "corrupted" may scare away usage in tightly regulated industries with stringent code reviews
2. "corrupted" isn't correct - the variant object is not "corrupted" in the way memory, for example, gets corrupted by UB such as out-of-bounds writes

However, there are good aspects to the name:

1. it is long and hard to type.  This is typically not good, but in this case, similar to "reinterpret_cast",
there is a belief that some things _should_ be hard to type, and that variant's checking function should be one of those things
(because it is believed that typical correct usage should not require the check)
2. it is easy to grep for.  If you don't want it to be overused, you want it to be easily found and "policed"
3. The "by_exception" part of the name hints as to why it happened, and more importantly,
why is should rarely happen and why it is most likely often handled when the exception is thrown (and thus not needed in "normal" code)
4. A hard-to-type name is part of the "the Kona Kompromise".  Those who compromised on a rarely-invalid variant (instead of a thicker never-invalid variant) really want a don't-type-this name


#### V. Design Decisions

1. 1-4 as above. It should be an ugly name.
2. "valueless" is correct - the variant has no value.  "corrupted" is not quite correct.
3. A shorter name, such as just `valueless()`, seems _carelessly_ inconsistent with names like `has_value()` (suggested for optional/any in P0032R0),
whereas `valueless_by_exception()` looks _purposely_ inconsistent, not carelessly inconsistent.
4. purposely inconsistent can be a good thing, when you want developers to question and understand why/how the underlying behaviour is inconsistent


#### VII. Technical Specifications

`:%s/currupted_by_exception/valueless_by_exception/g`

