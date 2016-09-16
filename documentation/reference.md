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

## Mark policy creation tools

The follwoing tools help build custom mark policies.

### Class `markable_pod_storage_type_tag`

This is an empty class. Mark policies may use it as a base class in order to control the behavior of the instances of class template `markable`. When a mark policy `MP` derives (directly or indirectly) from `markable_pod_storage_type_tag`, `markable<MP>` does no longer store the value directly as member, but instead uses a raw aligned storage layout-compatible with the value type, and uses in-place construction and destruction to manage the life-time of the stored value. This is necessary to implement mark policies that use storage techniques such as one in `markable_pod_storage_type`.

### Class template `markable_pod_storage_type`

This class template can be used for creating a mark policy for types where no logical value of `T` can be used for marked value, but there exists a bit-pattern in memory occupied by `T` that never represents a valid value of `T`.

```c++
template <typename T, typename POD_T = std::aligned_storage_t<sizeof(T), alignof(T)>>
  requires std::is_pod<POD_T>::value
        && sizeof(T) == sizeof(POD_T)
        && alignof(T) == alignof(POD_T)
struct markable_pod_storage_type : markable_pod_storage_type_tag
{
  typedef T value_type;
  typedef POD_T storage_type;
  typedef const T& reference_type;
  
  static const value_type& access_value(const storage_type& s);
  static const storage_type& store_value(const value_type& v); 
};
```

`T` is the type of values we want `markable` to represent. `POD_T` is the type designed to represent the memory occupied by `T`: it has to be a POD, with the same size and alignment as `T`.

#### `const value_type& access_value(const storage_type& s)`
*Returns:* `reinterpret_cast<const value_type&>(s)`.

#### `const storage_type& store_value(const value_type& v)`
*Returns:* `return reinterpret_cast<const storage_type&>(v)`.


### Class template `markable_type`

This class template provides default definitions form most of the requirements in concept `Mark_policy`. When declaring your own mark policy, you probably want to derive from this class template (although it is not strictly necessary). 

```c++
template <typename T, typename STORE = T, typename CREF = const T&>
struct markable_type
{
  typedef T     value_type;
  typedef STORE storage_type;
  typedef CREF  reference_type;
  
  static constexpr const value_type& access_value(const storage_type& v) { return v; }
  static constexpr const value_type& store_value(const value_type& v) { return v; }
  static constexpr value_type&& store_value(value_type&& v) { return std::move(v); }
};
```

## Predefined mark policies


### Class template `mark_int`

```c++
template <typename Integral, Integral MVal>
struct mark_int : markable_type<T>
{
  static constexpr Integral marked_value() noexcept { return MVal; }
  static constexpr bool is_marked_value(Integral v) { return v == MVal; }
};
```

`Integral` represents the stored type. It can be any type suitable for non-type template parameter.

`EV` is the value the empty value representation.

### Class template `mark_fp_nan`

```c++
template <typename FPT>
  requires std::is_floating_point<FPT>::value
        && std::numeric_limits<FPT>::has_quiet_NaN
struct mark_fp_nan : markable_type<FPT>
{
  static constexpr FPT marked_value() noexcept { return std::numeric_limits<FPT>::quiet_NaN(); }
  static constexpr bool is_marked_value(FPT v) { return v != v; }
};
```

### Class template `mark_value_init`

```c++
template <typename T>
struct mark_value_init : markable_type<T>
{
  static constexpr T marked_value() noexcept(see below) { return T{}; }
  static constexpr bool is_marked_value(const T& v) { return v == T{}; }
};
```

`T` is required to be a regular type.

The expression inside `noexcept` should be equivalent to `std::is_nothrow_default_constructible<T>::value && std::is_nothrow_move_constructible<T>::value`.

### Class template `mark_stl_empty`

```c++
template <typename Cont>
struct mark_stl_empty : markable_type<Cont>
{
  static constexpr Cont marked_value() noexcept(see below) { return Cont{}; }
  static constexpr bool is_marked_value(const Cont& v) { return v.empty(); }
};
```

`Cont` is required to be a container in the STL sense. 

The expression inside `noexcept` should be equivalent to `std::is_nothrow_default_constructible<Cont>::value && std::is_nothrow_move_constructible<Cont>::value`.

### Class template `mark_optional`

```c++
template <typename OT>
struct mark_optional : markable_type<typename OT::value_type, OT>
{
  typedef typename OT::value_type value_type;
  typedef OT storage_type;

  static OT marked_value() noexcept { return OT{}; }
  static bool is_marked_value(const OT& v) { return !v; }
  
  static const value_type& access_value(const storage_type& v) { return *v; }
  static storage_type store_value(const value_type& v) { return v; }
  static storage_type store_value(value_type&& v) { return std::move(v); }
};
```

`OT` is required to be either `boost::optional` or `std::experimental::optional` or `std::optional` or a type sufficiently similar to these.

### Class template `mark_bool`

```c++
struct mark_bool : markable_type<bool, char, bool>
{
  static constexpr char marked_value() noexcept { return char(2); }
  static constexpr bool is_marked_value(char v) { return v == 2; }
  
  static constexpr bool access_value(const char& v) { return bool(v); }
  static constexpr char store_value(const bool& v) { return v; }
};
```

### Class template `mark_enum`

template <typename Enum, std::underlying_type_t<Enum> Val> 
  requires std::is_enum<Enum>::value
struct mark_enum : markable_pod_storage_type<Enum, std::underlying_type_t<Enum>>
{
  typedef markable_pod_storage_type<Enum, std::underlying_type_t<Enum>> base;
  typedef typename base::storage_type storage_type;
  
  static storage_type marked_value() { return Val; }
  static bool is_marked_value(const storage_type& v) { return v == Val; }
};

`Enum` is required to be an enumeration type. `Val` a value of integral type, `std::underlying_type_t<Enum>` not necessarily from the range designated by `Enum`.
