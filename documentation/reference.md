# Reference

```c++
namespace ak_toolbox
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
concept bool Mark_policy = requires()
{
  typename MP::value_type;
  typename MP::storage_type;
  typename MP::reference_type;

  requires requires(const typename MP::storage_type & s,
                    const typename MP::value_type & cv,
                    typename MP::value_type && rv)
  {
    { MP::access_value(s) }            -> typename MP::reference_type;
    { MP::store_value(cv) }            -> typename MP::storage_type;
    { MP::store_value(std::move(rv)) } -> typename MP::value_type;
  };
};
```

#### `value_type`
represents the type 'logically' stored by the markable object. Markable object tries to make an impression on the users that it is internally storing an objectof type `value_type`, which is often the case, but not always.

#### `storage_type`
represents the type of the sub-object physically stored inside markable object, which is used to store the value of the object as well as the empty-state mark. 

#### `reference_type`
represents the type returned when the user requests read access to the stored value. Typically, this type is defined as `const value_type&`, however, sometimes when the accessed value is computed on the fly, this type may be defined as `value_type`.

## Class template `markable`

```c++
namespace ak_toolbox
{
  namespace markable_ns
  {
    template <Mark_policy MP, typename Tag = default_tag>
    class markable
    {
      typedef typename MP::value_type     value_type;
      typedef typename MP::storage_type   storage_type;
      typedef typename MP::reference_type reference_type;

      constexpr markable() noexcept(noexcept(storage_type{MP::marked_value{}}));
      constexpr markable(const value_type& v);
      constexpr markable(value_type&& v);
      
      constexpr bool has_value() const;
      constexpr reference_type value() const;
      constexpr storage_type const& storage_value() const;
    };
  }
}
```

#### `reference_type value() const`

*Requires:* `has_value()`.
