## Conversions

### Three Types

To be clear:

#### 1. Implicit Constructor and/or Implicit Cast

    struct Int
    {
       Int(int);
       operator int() { return 17; }
    };
    
    Int J = 17;
    int k = J;
    
#### 2. Explicit Constructor and/or Explicit Cast

    struct Ent
    {
       explicit Ent(int) {};
       explicit operator int() { return 23; }
    };
    
    // three "spellings"
    Ent E = Ent(17);
    Ent F = (Ent)17;
    Ent G = static_cast<Ent>(17);
    // three "spellings"
    int x = int(E);
    int y = (int)E;
    int z = static_cast<int>(E);

#### 3. "Named" Conversion

    struct Nnt
    {
        Nnt();
        // inside class...
        static Nnt from_int(int) { return Nnt(); }
        int to_int() { return 31; }
    };
    
    // ...or free function:
    Nnt to_Nnt(int) { return Nnt(); }
    int to_int(Nnt) { return 13; }
    
    
    Nnt N = Nnt::from_int(5);
    Nnt M = to_Nnt(7);
    
    int w = N.to_int();
    int v = to_int(N);  // more generic
    

### Which should be used When, and Why?

Starting Suggestions:

| **Consideration** | Implicit | Explicit  | Named |
| --- | --- | --- | --- |
| **same platonic thing?** | completely | yes, but info/accuracy lost | not "same thing". |
| **performance?** | (almost) none (time nor space) | small performance penalty |  performance penalty  |
| **throws?** | noexcept | might throw?  | can throw |
| **danger?** | not dangerous | if dangerous (eg dangling pointer/ref) | dangerous |
| **code review?** | fine | self-policed | greppable / policeable |
| **generic code?** | if all implicit | if all implicit or explicit  | most generic - "extension point"  |
| **modify class?** | yes | yes | no |
|  |  |  |  |
|  |  |  |  |
| **if in doubt** | | if "same thing"  | if not "same thing" |

Note: a class could have more than one.  In particular, could have named in addition to implicit/explicit (for genericity). Particularly free function form.

#### Examples/Explanations

- "platonic": `int`, `long` represent "numbers". `string`/`string_view`/`char*` represent "strings".
All date classes attempt to model "real world" dates. 17 is not the same thing as "17"
- "dangerous": ie dangling ref/pointer. `char *` from `string` is performant, accurate (except embedded nulls), but dangerous
- generic code: consider `std::to_string`. Can be written outside class. Can be an extension point.
- modify class: We can modify any STL class - _however_ it might break existing code. We can definitely modify any new class.
If you can't modify the class, use named free functions.

#### Extension points

Named free-funciton conversions can be good extension points.
_However_, they do require agreement on the name - which is fine for STL, but harder for independent libraries.
ie my library called it `to_int` but your library called it `to_integer`. De facto standands - _plural_. :-(

Extensions points in STL have their own problems. ADL, `using std::swap`. etc.

#### How are we (std::) doing?
- `string(char *)` should be explicit? (can throw).  But sooo convenient.  But now we have string_view - doesn't throw.
- chrono :-)
- std::to_string
- std::byte to_integer

#### Elsewhere
- boost lexical_cast
