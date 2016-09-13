# Design Rationale

... TO BE PROVIDED ...

## Why not just 'fix' Boost.Optional?

It has been suggested that rather than adding a new library, this 'optimization' should rather been implemented inside `boost::optional`. This is not possible because the idea behind `markable` would visibly affect the smantics of `optional`. The following guarantee offered by `optional` would no longer hold:

```c++
template <typename T>
void expectation(std::optional<T> o, T some_T)
{
  if (o)          // for any 'initialized' optional
  {
    *o = some_T;  // modifying the contained value
    assert (o);   // must not un-initialize the optional
  }
}
```

Some have suggested that `optional` be extended with a policy type, where `optional<T, default_polict>` behaves lik old `optional<T>` used to, but another policy can enable the 'optimization' at the cost of compromizing the generic interface. But having an explicit specialization that behaves differently than the master template is no better than having two separate class templates.

## Constructor taking marked value

Should the constructor taking `T` have a precondition that the value is not that indicated as marked?

Pros: Such precondition might detect use cases that are potential bugs.

Cons: Without the precondition we can handle a typical use case where we get a raw value that may be potentially marked from an external system and want to apply a `marked` interface on it:

```c++
using opt_index = marked<mark_int<std::string::size_type, std::string::npos>>;

opt_index index {str.find_first_of("/")};
```

## No mutable access to value

It is impossible to alter the "contained value" throug the access to function `value`:

```c++
marked<mp_int<int, -1>> oi {1};
oi.value() = 2; // < compile-time error
```

This is so for two reasons. First, we do not want to allow silently changing the state of markable object to marked value through `value`. With the assignment being prevented, we could get the following surprising behaviour:

```c++
marked<mp_int<int, -1>> oi {1};
oi.value() = (int)some_value; // < suppose it was allowed
assert (oi.has_value())       // < might fail!
```

Second, sometimes as in the case of `mp_bool` we do not store any value that we can return a reference to. Instead, we store a three-state `char` value, and we return a temporary `bool` value on the fly, which we return by value rather than by reference.

## Contextual conversion to bool

...

## Relational operators

...
