## std::latest<> Should be Named std::snapshot_source

Document number: DXXXX  
Date: 2019-06-17  
Audience: LEWG  
Reply-to: Tony Van Eerd. snapshot at forecode.com

---

> I've been through the papers, found some stuff with bad names  
> It felt bad to be causing dev pain  
> In the standard you need stuff with good names  
> Cause there ain't no one for to help you explain  
> La la, la la-la la  
>   
> After two days in the standard tome  
> My brain began to turn dead  
> After three days with the standard combed  
> I was looking at a lot of dread  
> And the story it told of those names in code  
> Made me sad to think of our devs  
>   
> You see I've been through the papers, found some stuff with bad names  
> It felt bad to be cause of dev pain  
> In the standard you need stuff with good names  
> Cause there ain't no one for to help you explain  
> La la, la la-la la  
> ....  
> (ie _A Horse with No Name_ by America)

---


P0561R4 proposes a class for deferred reclamation so that multiple threads can easily access the “latest snapshot” of a state.
Currently the class where all snapshots are managed is called `latest` (along with a `raw_latest<>` class template in header `<snapshot>`). 


`std::latest<>` was originally named `std::cell<>`, which was a very vague and thus bad name.  So `lastest` is a better name.

But still not a _good_ name.  There are issues:

- "latest" is a very general term, can be applied to many things, many meanings
- (for example) `std::chrono::latest` is an enum value, completely unrelated to `std::latest<>`
- `std::latest<>` doesn't hint at its relationship to `std::snapshot_ptr` nor the header `<snapshot>`

So rename it `std::snapshot_source<>` (and `raw_snapshot_source<>`)

If you feel these are too long, they are not:
- there will not be many sources, you don't type it often
- the whole feature is not meant to be frequently used by the average programmer
- a long name signals "this thing might be a bit complicated", which it is

