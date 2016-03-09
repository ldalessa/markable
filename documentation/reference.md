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

## Class template `markable`

```c++
namespace ak_toolbox
{
  namespace markable_ns
  {
    template <typename MP, typename Tag = default_tag>
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
