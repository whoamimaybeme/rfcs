# Backwards compatibility configuration in `.config.luau`

## Summary 

Implement a way to make changes to Luau that break backwards compatibility via a new property in `.config.luau` files.

## Motivation

Backwards compatibility is very important, however it prohibits breaking changes from being made, even if the feature would be very helpful for various developers.

Adding some sort of way to allow developers to enable these breaking changes without breaking older codebases that aren't being updated would be helpful.

## Design

Add a new property named `compatibility` to the `luau` configuration within `.config.luau` files:

```luau
return {
    luau = {
        compatibility = { -- everything is true by default
            ["*"] = false, -- everything is disabled
            ["0.746"] = false, -- everything added to this before 0.746 is disabled
            ContextualConst = true, -- `local const = 1` works when this is true 
        },
    }
}
```

The table is similar to `lint`, but every property points to a certain feature in Luau. If it's `true` or `nil`, then the feature is enabled, but if it's false, the feature is disabled. The currently supported properties are:
- `FunctionsArentConstByDefault` - When this property is set to false, functions are `const` by default. Implements [#277](https://github.com/luau-lang/rfcs/pull/277).
- `ContextualConst` - When this property is set to false, code that defines a variable named `const` will error. Setting this to false may remove performance penalties from [#260](https://github.com/luau-lang/rfcs/pull/260).

There are certain properties that determine the setting for all rules not within the `compatibility` table. You can only have one of these within a table.
- Similar to `lint`, `*` determines all rules that aren't defined within the `compatibility` table. However, this may cause your code to break in the future if we introduce new rules, so it is recommended to just use the version number instead.
- You can add a specific version number to only control compatibility rules added before the release version or on the release version.

## Drawbacks

This feature may not warrant being in Luau due to its currently presented features

## Alternatives

There could be a script directive for this. However, the functionality of that may not be clear and may clog up the top of the scripts with a lot of script directives.

Swap the roles of `true` and `false` within the `compatibility` table, alongside changing all the names to make sense with that change.

## Prior Art

I don't know.
