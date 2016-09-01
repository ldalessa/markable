# Reference

```c++
namespace ak_toolkit
{
  namespace markable_ns
  {
    template <typename MP, typename Tag>
    class markable;
    
    void swap(markable & l, markable & r);
  }

  using markable_ns::markable;
}
```

## Concept `Mark_policy`

```c++
template <typename MP>
concept bool Mark_policy =
  requires
  {
    typename MP::value_type;
    typename MP::storage_type;
    typename MP::reference_type;
  } &&
  requires(const typename MP::storage_type & s,
           const typename MP::value_type &   cv,
                 typename MP::value_type &&  rv)
  {
    { MP::marked_value() }              -> typename MP::storage_type;
    { MP::is_marked_value(s) } noexcept -> bool;
    
    { MP::access_value(s) }             -> typename MP::reference_type;
    { MP::store_value(cv) }             -> typename MP::storage_type;
    { MP::store_value(std::move(rv)) }  -> typename MP::storage_type;
  };
```

#### `value_type`
This represents the type 'logically' stored by the markable object. Markable object tries to make an impression on the users that it is internally storing an objectof type `value_type`, which is often the case, but not always.

#### `storage_type`
This represents the type of the sub-object physically stored inside markable object, which is used to store the value of the object as well as the empty-state mark. 

#### `reference_type`
This represents the type returned when the user requests read access to the stored value. Typically, this type is defined as `const value_type&`, however, sometimes when the accessed value is computed on the fly, this type may be defined as `value_type`.

#### `{ marked_value() } -> storage_type`
Returns a marked value encoded in `storage_type`. This will be later used to represent a markable object with no value.

#### `{ is_marked_value(s) } -> bool`
Checks if the given value represents a marked value.

#### `{ access_value(s) } -> reference_type`
*Preconditions:* `!is_marked_value(s)`.

Given a value encoded in `storage_type`, provides access to it through type `reference_type`. Typically, when `reference_type` and `const storage_type&` are same type, this is an identity function.

#### `{ store_value(v) } -> storage_type`
Given a value of type `value_type` (possibly marked), returns its representation as `storage_type`. Typically, when `value_type` and `storage_type` are same type, this is an identity function.

## Class template `markable`

```c++
namespace ak_toolkit
{
  namespace markable_ns
  {
    template <Mark_policy MP, typename Tag = default_tag>
    class markable
    {
    public:
      typedef typename MP::value_type     value_type;
      typedef typename MP::storage_type   storage_type;
      typedef typename MP::reference_type reference_type;

      constexpr markable() noexcept(noexcept(storage_type{MP::marked_value{}}));
      constexpr explicit markable(const value_type& v);
      constexpr explicit markable(value_type&& v);
      constexpr markable(const markable&) = default;
      constexpr markable(markable&&) = default;
      
      constexpr markable& operator=(const markable&) = default;
      constexpr markable& operator=(markable&&) = default;
      
      constexpr bool has_value() const;
      constexpr reference_type value() const;
      constexpr storage_type const& storage_value() const;
      
    private:
      storage_type val_; // exposition only
    };
  }
}
```

#### `markable()`

*Effects:* Initializes storage value with expression `MP::marked_value()`. 

*Postconditions:* `!has_value()`.

#### `markable(const value_type& v)`

*Effects:* Initializes storage value with expression `MP::store_value(v)`. 

*Postconditions:* `has_value()`.

*Note:* The semantics are in fact slightly different when `MP` is derived from `markable_pod_storage_type_tag`.

#### `markable(value_type&& v)`

*Effects:* Initializes storage value with expression `MP::store_value(std::move(v))`. 

*Postconditions:* `has_value()`.

*Note:* The semantics are in fact slightly different when `MP` is derived from `markable_pod_storage_type_tag`.

#### `reference_type has_value() const`

*Returns:* `!MP::is_marked_value(val_)`.

*Throws:* Nothing.


#### `reference_type value() const`

*Preconditions:* `has_value()`.

*Returns:* `MP::access_value(val_)`.

*Throws:* Whatever `MP::access_value` throws. Also, if `reference_type` is in fact not a reference type, whatever `reference_type`'s move constructor throws.


#### `storage_type const& storage_value() const`

*Returns:* `val_`.

*Throws:* Nothing.

## Predefined mark policies

## Class template `mark_int`

```c++
template <typename T, T Val>
struct mark_int : markable_type<T>
{
  static constexpr T marked_value() noexcept { return Val; }
  static constexpr bool is_marked_value(T v) { return v == Val; }
};
```
