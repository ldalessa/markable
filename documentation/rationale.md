# Design Rationale

... TO BE PTOVIDED ...

## Why not just 'fix' Boost.Optional?

...

## Constructor taking marked value

Should the constructor taking `T` have a precondition that the value is not that indicated as marked?

Pros: Such precondition might detect use cases that are potential bugs.

Cons: Without the precondition we can handle a typical use case where we get a raw value that may be potentially marked from an external system and want to apply a `marked` interface on it:

```c++
std::string::size_type index = str.find_first_of("/");
marked<mark_int<std::string::size_type, std::string::npos>> opt_index {index};
```

## Contextual conversion to bool

...

## Relational operators

...
