# Design Rationale

... TO BE PROVIDED ...

## Why not just 'fix' Boost.Optional?

...

## Constructor taking marked value

Should the constructor taking `T` have a precondition that the value is not that indicated as marked?

Pros: Such precondition might detect use cases that are potential bugs.

Cons: Without the precondition we can handle a typical use case where we get a raw value that may be potentially marked from an external system and want to apply a `marked` interface on it:

```c++
using opt_index = marked<mark_int<std::string::size_type, std::string::npos>>;

opt_index index {str.find_first_of("/")};
```

## No mutable access to value

(we do not want you to set no-value state through this function) ...

## Contextual conversion to bool

...

## Relational operators

...
