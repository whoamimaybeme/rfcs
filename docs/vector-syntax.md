# Vector constructor syntax

## Summary

Implement new syntax to construct vectors, `<x, y, z>`.

## Motivation

Within the [future work section of the vector library RFC](https://rfcs.luau.org/vector-library.html#future-work), it was stated that there would be a "better" vector syntax that is less verbose. This proposal defines that syntax and how it is used.

## Design

A new vector constructor is introduced, the `<x, y, z?>` constructor. When 4-wide mode is enabled, the constructor would be `<x, y, z?, w?>`.

In practice, it would look something like this:
```lua
local newvector = <1, 2, 3>
vector.magnitude(<1, 2, 3>)

-- when using 2 components of a vector
local newvector2 = <1, 2>
vector.magnitude(<1, 2>)

-- in 4 wide mode
local newvector4 = <1, 2, 3, 4>
vector.magnitude(<1, 2, 3, 4>)

-- example of something that would error
<1, 2, 3> -- Expected identifier when parsing expression, got '<1, 2, 3>'
```

This RFC does not propose any changes when printing or stringifying vectors. Stringifying vectors will still return `x, y, z` instead of something like `<x, y, z>`.

## Drawbacks

This may add more syntax to Luau and bloat the language.

This syntax is already used for generics, but with how vectors are currently used, this shouldn't be an issue at all.

## Alternatives

Do nothing; we can already construct vectors with `vector.create` or any other runtime provided vector constructor.

A few other syntax designs were proposed in the vector library RFC.
- `(x, y, z)` is often used for tuples and is used for functions.
- `[x, y, z]` is used for table indexing, and may raise potential questions with tables with the `__call` metamethod.
- `vector(x, y, z)` would give the `vector` library a `__call` metamethod, which might require more discussion over whether this is a good idea.
- `|x, y, z|` is possible, but `<x, y, z>` is probably the better option.