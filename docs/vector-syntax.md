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

The `vector.create` function and the `<x, y, z>` syntax are interchangable.

```lua
local Foo = {
    [vector.create(1, 2, 3)] = "Foo",
    [<4, 5, 6>] = "Bar",
}

print(Foo[<1, 2, 3>]) -- Foo
print(Foo[vector.create(4, 5, 6)]) -- Bar

print(vector.create(1, 2, 3) == <1, 2, 3>) -- true
```

If a number is put next to a vector without a comma separating it, it will just compare the two numbers instead of constructing a vector.

```lua
function Foo(...)
  for i, v in {...} do
    print(i, v)
  end
end

Foo(1 <2, 3, 4> 5)
-- 1, true
-- 2, 3
-- 3, false

Foo(1 <2, 3, 4>)
-- Expected identifier when parsing expression, got ')'
```

This RFC does not propose any changes when printing or stringifying vectors. Printing vectors varies based on what runtime you use, and stringifying vectors will still return `"x, y, z"`.

```lua
print(<1, 2, 3>) -- varies on how the runtime prints vectors

print(tostring(<1, 2, 3>)) -- 1, 2, 3
print(`<{tostring(<1, 2, 3>)}>`) -- <1, 2, 3>
```

---

Since the angle brackets are already used by generics, there are rules that define when angle brackets refer to generics or vectors.

When angle brackets are used after a type name, it will be interpreted as a generic instead of a vector.

```lua
type Foo<T> = {T}
type Bar<T = typeof(<1, 2, 3>)> = F<T>
type Bax
<T>
=
Bar<T>
```

Angle brackets will also refer to generics within function definitions where they are expected. 

Double brackets used between the function name and the parameters are used for putting types into type parameters when calling functions.

```lua
type Foo<T> = {T}

local Bar = function<T>(foo: T)
  -- code
end

Bar<<vector>>(<1, 2, 3>)

function Baz<T>(foo: T)
  -- code
end

Baz<<{typeof(<1, 2, 3>)}>>({<4, 5, 6>} :: Foo<typeof(<7, 8, 9>)>)
```
Any other usage interprets it as a vector.

## Drawbacks

This will add more syntax to Luau, making the language more complex and more bloated.

Complex constraints must be introduced to differentiate vectors from generics. This could be resolved altogether by using another form of syntax.

## Alternatives

Do nothing; vectors can already be constructed using `vector.create` or any other runtime provided vector constructor.

Use different syntax for constructing vectors instead of using angle brackets. A few of these were suggested in the vector library RFC as possiblities.
- `(x, y, z)` is often used for tuples and functions, so it would not work.
- `[x, y, z]` is used for table indexing, so writing out `newtable[[x, y, z]]` may confuse readers. People may also write `newtable[x, y, z]` expecting for it to index the vector.
- `vector(x, y, z)` would give the `vector` library a `__call` metamethod, which requires more discussion over whether this is a good idea.
- `|x, y, z|` shouldn't conflict with anything, however, `<x, y, z>` is the better choice for vector constructor syntax.