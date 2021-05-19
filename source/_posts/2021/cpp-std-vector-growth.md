---
title: Growth of `std::vector`
date: 2021-05-16 21:35:54
tags: [CPP, MSVC, GCC, STL]
---

### *`Overview`*

Nowadays, STL is an essential component in almost every CPP project. That is why several questions about STL are asked in a technical interview. Especially, the `std::vector` is a popular subject. In this post, we gonna check the codes related to `std::vector`'s growth, which is a hot topic in STL.
The term "growth" in `std::vector` means an event to increase a size of instance by some actions. The action would be inserting an element (e.g. `push_back()`) or tuning its size (e.g. `resize()`). Some of us say "When the growth happens, its size become twice.", but some of others say "No, it is exactly 3/2 times.". Well...both of saying are not wrong. Let us find out why it is.

The environment is below:
- MSVC
    - Windows 10 `2004 OS Build 19041.985`
    - MSVC `19.28.29914`
- GCC
    - Ubuntu 18 `18.04.4 LTS`
    - GCC `7.5.0`

The reference is below:
- [https://en.cppreference.com/w/cpp/container/vector][reference #1]
- [https://github.com/microsoft/STL/blob/f17f2e72001f570cb6fe5a6e1c0c32fcc90ee53a/stl/inc/vector][reference #2]
- [https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/stl_vector.h][reference #3]

[reference #1]:https://en.cppreference.com/w/cpp/container/vector
[reference #2]:https://github.com/microsoft/STL/blob/f17f2e72001f570cb6fe5a6e1c0c32fcc90ee53a/stl/inc/vector
[reference #3]:https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/stl_vector.h

### *`Growth in MSVC`*

First, let us trace codes about `push_back()`. Suppose we use a code below:
```cpp
int Number = 0;
std::vector<int> Instance;
Instance.push_back(Number);
```
The function below will be called in this code. You can find all of STL codes for MSVC at [reference #2].
```cpp
_CONSTEXPR20_CONTAINER void push_back(const _Ty& _Val) { // insert element at end, provide strong guarantee
    emplace_back(_Val);
}
```
```cpp
template <class... _Valty>
_CONSTEXPR20_CONTAINER decltype(auto) emplace_back(_Valty&&... _Val) {
    // insert by perfectly forwarding into element at end, provide strong guarantee
    auto& _My_data   = _Mypair._Myval2;
    pointer& _Mylast = _My_data._Mylast;
    if (_Mylast != _My_data._Myend) {
        return _Emplace_back_with_unused_capacity(_STD forward<_Valty>(_Val)...);
    }

    _Ty& _Result = *_Emplace_reallocate(_Mylast, _STD forward<_Valty>(_Val)...);
#if _HAS_CXX17
    return _Result;
#else // ^^^ _HAS_CXX17 ^^^ // vvv !_HAS_CXX17 vvv
    (void) _Result;
#endif // _HAS_CXX17
}
```
As you can see, there is a branch on returning the function. When `_Mylast != _My_data._Myend` is true, the growth not happens. Because the logic in the `_Emplace_back_with_unused_capacity()` does not reallocate memory, but reuse unused memory. FYI, values about `_Mypair` have the relationship like below:
```cpp
_Compressed_pair<_Alty, _Scary_val> _Mypair;
```
```cpp
// https://github.com/microsoft/STL/blob/c12089e489c7b6a3896f5043ed545ac8d1870590/stl/inc/xmemory
template <class _Ty1, class _Ty2, bool = is_empty_v<_Ty1> && !is_final_v<_Ty1>>
class _Compressed_pair final : private _Ty1 { // store a pair of values, deriving from empty first
public:
    _Ty2 _Myval2;
```
```cpp
// CLASS TEMPLATE vector
template <class _Ty, class _Alloc = allocator<_Ty>>
class vector { // varying size array of values
private:
    template <class>
    friend class _Vb_val;
    friend _Tidy_guard<vector>;

    using _Alty        = _Rebind_alloc_t<_Alloc, _Ty>;
    ...
    using _Scary_val = _Vector_val<conditional_t<_Is_simple_alloc_v<_Alty>, _Simple_types<_Ty>,
        _Vec_iter_types<_Ty, size_type, difference_type, pointer, const_pointer, _Ty&, const _Ty&>>>;
```
```cpp
// CLASS TEMPLATE _Vector_val
template <class _Val_types>
class _Vector_val : public _Container_base {
public:
    using value_type      = typename _Val_types::value_type;
    using size_type       = typename _Val_types::size_type;
    using difference_type = typename _Val_types::difference_type;
    using pointer         = typename _Val_types::pointer;
    using const_pointer   = typename _Val_types::const_pointer;
    using reference       = value_type&;
    using const_reference = const value_type&;

    _CONSTEXPR20_CONTAINER _Vector_val() noexcept : _Myfirst(), _Mylast(), _Myend() {}

    _CONSTEXPR20_CONTAINER _Vector_val(pointer _First, pointer _Last, pointer _End) noexcept
        : _Myfirst(_First), _Mylast(_Last), _Myend(_End) {}

    _CONSTEXPR20_CONTAINER void _Swap_val(_Vector_val& _Right) noexcept {
        this->_Swap_proxy_and_iterators(_Right);
        _Swap_adl(_Myfirst, _Right._Myfirst);
        _Swap_adl(_Mylast, _Right._Mylast);
        _Swap_adl(_Myend, _Right._Myend);
    }

    _CONSTEXPR20_CONTAINER void _Take_contents(_Vector_val& _Right) noexcept {
        this->_Swap_proxy_and_iterators(_Right);
        _Myfirst = _Right._Myfirst;
        _Mylast  = _Right._Mylast;
        _Myend   = _Right._Myend;

        _Right._Myfirst = nullptr;
        _Right._Mylast  = nullptr;
        _Right._Myend   = nullptr;
    }

    pointer _Myfirst; // pointer to beginning of array
    pointer _Mylast; // pointer to current end of sequence
    pointer _Myend; // pointer to end of array
};
```
Since the elements are placed in sequential memory address, `_Mylast - _Myfirst` means "currently used size".
```cpp
_NODISCARD _CONSTEXPR20_CONTAINER size_type size() const noexcept {
    auto& _My_data = _Mypair._Myval2;
    return static_cast<size_type>(_My_data._Mylast - _My_data._Myfirst);
}
```
Similarly, `_Myend - _Myfirst` means "currently avaiable size".
```cpp
_NODISCARD _CONSTEXPR20_CONTAINER size_type capacity() const noexcept {
    auto& _My_data = _Mypair._Myval2;
    return static_cast<size_type>(_My_data._Myend - _My_data._Myfirst);
}
```
As a result, `_Mylast != _My_data._Myend` is true when `_Mylast < _Myend` is true. That is why reallocation not happens. Get back to `emplace_back()` code. According to those upper reasons, now we need to focus on `_Emplace_reallocated()` function.
```cpp
template <class... _Valty>
_CONSTEXPR20_CONTAINER pointer _Emplace_reallocate(const pointer _Whereptr, _Valty&&... _Val) {
    // reallocate and insert by perfectly forwarding _Val at _Whereptr
    _Alty& _Al        = _Getal();
    auto& _My_data    = _Mypair._Myval2;
    pointer& _Myfirst = _My_data._Myfirst;
    pointer& _Mylast  = _My_data._Mylast;

    _STL_INTERNAL_CHECK(_Mylast == _My_data._Myend); // check that we have no unused capacity

    const auto _Whereoff = static_cast<size_type>(_Whereptr - _Myfirst);
    const auto _Oldsize  = static_cast<size_type>(_Mylast - _Myfirst);

    if (_Oldsize == max_size()) {
        _Xlength();
    }

    const size_type _Newsize     = _Oldsize + 1;
    const size_type _Newcapacity = _Calculate_growth(_Newsize);

    const pointer _Newvec           = _Al.allocate(_Newcapacity);
    const pointer _Constructed_last = _Newvec + _Whereoff + 1;
    pointer _Constructed_first      = _Constructed_last;

    _TRY_BEGIN
    _Alty_traits::construct(_Al, _Unfancy(_Newvec + _Whereoff), _STD forward<_Valty>(_Val)...);
    _Constructed_first = _Newvec + _Whereoff;

    if (_Whereptr == _Mylast) { // at back, provide strong guarantee
        _Umove_if_noexcept(_Myfirst, _Mylast, _Newvec);
    } else { // provide basic guarantee
        _Umove(_Myfirst, _Whereptr, _Newvec);
        _Constructed_first = _Newvec;
        _Umove(_Whereptr, _Mylast, _Newvec + _Whereoff + 1);
    }
    _CATCH_ALL
    _Destroy(_Constructed_first, _Constructed_last);
    _Al.deallocate(_Newvec, _Newcapacity);
    _RERAISE;
    _CATCH_END

    _Change_array(_Newvec, _Newsize, _Newcapacity);
    return _Newvec + _Whereoff;
}
```
As you can see the codes, deallocation and reallocation happen. The variable `_Newcapacity` determines the size of memory will be reallocated. Let us check the function `_Calculate_growth()`.
```cpp
_NODISCARD _CONSTEXPR20_CONTAINER size_type max_size() const noexcept {
    return (_STD min)(
        static_cast<size_type>((numeric_limits<difference_type>::max)()), _Alty_traits::max_size(_Getal()));
}
...
_CONSTEXPR20_CONTAINER size_type _Calculate_growth(const size_type _Newsize) const {
    // given _Oldcapacity and _Newsize, calculate geometric growth
    const size_type _Oldcapacity = capacity();
    const auto _Max              = max_size();

    if (_Oldcapacity > _Max - _Oldcapacity / 2) {
        return _Max; // geometric growth would overflow
    }

    const size_type _Geometric = _Oldcapacity + _Oldcapacity / 2;

    if (_Geometric < _Newsize) {
        return _Newsize; // geometric growth would be insufficient
    }

    return _Geometric; // geometric growth is sufficient
}
```
There are three return statement in the function.

- First, when the current available size is bigger than 2/3 times of maximum size.

For instance, a maximum value of `int` type is `+2,147,483,647` and 2/3 times of value is `+1,431,655,764.666... ≒ +1,431,655,765`. Let us put them in the expression. `if (1431655765 > 2147483647 - 1431655765 / 2)` will be false, but how about if `_Oldcapacity = +1,431,655,766` ? `if (1431655766 > 2147483647 - 1431655766 / 2)` will be true. In this case, new size will be forced as the maximum size.

- Second, when the current available size is less than `2`.

For instance, when the `_Oldcapacity` is in `{0, 1}` the expression `const size_type _Geometric = _Oldcapacity + _Oldcapacity / 2;` will be the same with `_Oldcapacity`. In this case, new size will be forced as `_Newsize`, which is passed by `_Oldsize + 1` in `_Emplace_reallocate()`.

`_Oldcapacity` | Calculation
--- | ---
0 | 0 + 0 / 2 = 0
1 | 1 + 1 / 2 = 1

- Third, other cases of the current available size.

The `_Geometric` will have 3/2 times of `_Oldcapacity`. That is why the 3/2 times of growth happens in MSVC. And now you understand why new size has to be set by maximum value when the `_Oldcapacity` is bigger than 2/3 times of maximum size.

The `resize()` has a similar flow. Let us find out.
```cpp
_CONSTEXPR20_CONTAINER void resize(_CRT_GUARDOVERFLOW const size_type _Newsize) {
    // trim or append value-initialized elements, provide strong guarantee
    _Resize(_Newsize, _Value_init_tag{});
}
```
```cpp
template <class _Ty2>
_CONSTEXPR20_CONTAINER void _Resize(const size_type _Newsize, const _Ty2& _Val) {
    // trim or append elements, provide strong guarantee
    auto& _My_data      = _Mypair._Myval2;
    pointer& _Myfirst   = _My_data._Myfirst;
    pointer& _Mylast    = _My_data._Mylast;
    const auto _Oldsize = static_cast<size_type>(_Mylast - _Myfirst);
    if (_Newsize < _Oldsize) { // trim
        const pointer _Newlast = _Myfirst + _Newsize;
        _Orphan_range(_Newlast, _Mylast);
        _Destroy(_Newlast, _Mylast);
        _Mylast = _Newlast;
        return;
    }

    if (_Newsize > _Oldsize) { // append
        const auto _Oldcapacity = static_cast<size_type>(_My_data._Myend - _Myfirst);
        if (_Newsize > _Oldcapacity) { // reallocate
            _Resize_reallocate(_Newsize, _Val);
            return;
        }

        const pointer _Oldlast = _Mylast;
        _Mylast                = _Ufill(_Oldlast, _Newsize - _Oldsize, _Val);
        _Orphan_range(_Oldlast, _Oldlast);
    }

    // if _Newsize == _Oldsize, do nothing; avoid invalidating iterators
}
```
The `resize()` can trim or append available memory. Trimming happens when you call `resize()` with smaller value than current available size. Appending happens when you call `resize()` with greater value than current available size. We go to `_Resize_reallocate()`.
```cpp
template <class _Ty2>
_CONSTEXPR20_CONTAINER void _Resize_reallocate(const size_type _Newsize, const _Ty2& _Val) {
    if (_Newsize > max_size()) {
        _Xlength();
    }

    auto& _My_data    = _Mypair._Myval2;
    pointer& _Myfirst = _My_data._Myfirst;
    pointer& _Mylast  = _My_data._Mylast;

    const auto _Oldsize          = static_cast<size_type>(_Mylast - _Myfirst);
    const size_type _Newcapacity = _Calculate_growth(_Newsize);

    const pointer _Newvec         = _Getal().allocate(_Newcapacity);
    const pointer _Appended_first = _Newvec + _Oldsize;
    pointer _Appended_last        = _Appended_first;

    _TRY_BEGIN
    _Appended_last = _Ufill(_Appended_first, _Newsize - _Oldsize, _Val);
    _Umove_if_noexcept(_Myfirst, _Mylast, _Newvec);
    _CATCH_ALL
    _Destroy(_Appended_first, _Appended_last);
    _Getal().deallocate(_Newvec, _Newcapacity);
    _RERAISE;
    _CATCH_END

    _Change_array(_Newvec, _Newsize, _Newcapacity);
}
```
Oh, Hi. We meet again. It is him, the `_Calculate_growth()`. Now we know the `resize()` has a similar logic.

### *`Growth in GCC`*

First of all, let us find `push_back()` in GCC. Suppose we use the example written at MSVC part. You can find the code at [reference #3]
```cpp
// [23.2.4.3] modifiers
/**
*  @brief  Add data to the end of the %vector.
*  @param  __x  Data to be added.
*
*  This is a typical stack operation.  The function creates an
*  element at the end of the %vector and assigns the given data
*  to it.  Due to the nature of a %vector this operation can be
*  done in constant time if the %vector has preallocated space
*  available.
*/
void
push_back(const value_type& __x)
{
    if (this->_M_impl._M_finish != this->_M_impl._M_end_of_storage)
    {
        _GLIBCXX_ASAN_ANNOTATE_GROW(1);
        _Alloc_traits::construct(this->_M_impl, this->_M_impl._M_finish,
                        __x);
        ++this->_M_impl._M_finish;
        _GLIBCXX_ASAN_ANNOTATE_GREW(1);
    }
    else
        _M_realloc_insert(end(), __x);
}
```
Here are two cases. First, when an available memory exists. Second, otherwise.
```cpp
struct _Vector_impl_data
{
    pointer _M_start;
    pointer _M_finish;
    pointer _M_end_of_storage;

    _Vector_impl_data() _GLIBCXX_NOEXCEPT
    : _M_start(), _M_finish(), _M_end_of_storage()
    { }

#if __cplusplus >= 201103L
    _Vector_impl_data(_Vector_impl_data&& __x) noexcept
    : _M_start(__x._M_start), _M_finish(__x._M_finish),
        _M_end_of_storage(__x._M_end_of_storage)
    { __x._M_start = __x._M_finish = __x._M_end_of_storage = pointer(); }
#endif

    void
    _M_copy_data(_Vector_impl_data const& __x) _GLIBCXX_NOEXCEPT
    {
        _M_start = __x._M_start;
        _M_finish = __x._M_finish;
        _M_end_of_storage = __x._M_end_of_storage;
    }

    void
    _M_swap_data(_Vector_impl_data& __x) _GLIBCXX_NOEXCEPT
    {
        // Do not use std::swap(_M_start, __x._M_start), etc as it loses
        // information used by TBAA.
        _Vector_impl_data __tmp;
        __tmp._M_copy_data(*this);
        _M_copy_data(__x);
        __x._M_copy_data(__tmp);
    }
};
...
_Vector_impl _M_impl;
```
`std::vector` in GCC has internal indicators like `std::vector` in MSVC. So we should focus on `_M_realloc_insert()` function.
```cpp
#if __cplusplus >= 201103L
    template<typename _Tp, typename _Alloc>
    template<typename... _Args>
    void
    vector<_Tp, _Alloc>::
    _M_realloc_insert(iterator __position, _Args&&... __args)
#else
    template<typename _Tp, typename _Alloc>
    void
    vector<_Tp, _Alloc>::
    _M_realloc_insert(iterator __position, const _Tp& __x)
#endif
{
    const size_type __len =
        _M_check_len(size_type(1), "vector::_M_realloc_insert");
    pointer __old_start = this->_M_impl._M_start;
    pointer __old_finish = this->_M_impl._M_finish;
    const size_type __elems_before = __position - begin();
    pointer __new_start(this->_M_allocate(__len));
    pointer __new_finish(__new_start);
    __try
    {
        // The order of the three operations is dictated by the C++11
        // case, where the moves could alter a new element belonging
        // to the existing vector.  This is an issue only for callers
        // taking the element by lvalue ref (see last bullet of C++11
        // [res.on.arguments]).
        _Alloc_traits::construct(this->_M_impl,
                    __new_start + __elems_before,
#if __cplusplus >= 201103L
                    std::forward<_Args>(__args)...);
#else
                    __x);
#endif
        __new_finish = pointer();

#if __cplusplus >= 201103L
        if _GLIBCXX17_CONSTEXPR (_S_use_relocate())
        {
            __new_finish = _S_relocate(__old_start, __position.base(),
                        __new_start, _M_get_Tp_allocator());

            ++__new_finish;

            __new_finish = _S_relocate(__position.base(), __old_finish,
                        __new_finish, _M_get_Tp_allocator());
        }
        else
#endif
        {
            __new_finish
        = std::__uninitialized_move_if_noexcept_a
        (__old_start, __position.base(),
            __new_start, _M_get_Tp_allocator());

            ++__new_finish;

            __new_finish
        = std::__uninitialized_move_if_noexcept_a
        (__position.base(), __old_finish,
            __new_finish, _M_get_Tp_allocator());
        }
    }
        __catch(...)
    {
        if (!__new_finish)
        _Alloc_traits::destroy(this->_M_impl,
                    __new_start + __elems_before);
        else
        std::_Destroy(__new_start, __new_finish, _M_get_Tp_allocator());
        _M_deallocate(__new_start, __len);
        __throw_exception_again;
    }
#if __cplusplus >= 201103L
        if _GLIBCXX17_CONSTEXPR (!_S_use_relocate())
#endif
    std::_Destroy(__old_start, __old_finish, _M_get_Tp_allocator());
        _GLIBCXX_ASAN_ANNOTATE_REINIT;
        _M_deallocate(__old_start,
            this->_M_impl._M_end_of_storage - __old_start);
        this->_M_impl._M_start = __new_start;
        this->_M_impl._M_finish = __new_finish;
        this->_M_impl._M_end_of_storage = __new_start + __len;
}
```
Hoo, it is too long. We do not have to look into whole code, but the variable `__len`. The variable is used for reallocation. And it is set by `_M_check_len()`.
```cpp
// Called by _M_fill_insert, _M_insert_aux etc.
size_type
_M_check_len(size_type __n, const char* __s) const
{
    if (max_size() - size() < __n)
        __throw_length_error(__N(__s));

    const size_type __len = size() + (std::max)(size(), __n);
    return (__len < size() || __len > max_size()) ? max_size() : __len;
}
```
The code throw an error when current size is the same with maximum size because the function was called as `_M_check_len(size_type(1), ...)`. Otherwise, new size will be set by 2 times of current size. Except for when current size is `0`.

Current size | Calculation
--- | ---
0 | 1 = 0 + max(0, 1)
1 | 2 = 1 + max(1, 1)
2 | 4 = 2 + max(2, 1)

And, returns maximum size when underflow or overflow happens. Otherwise, returns new size calculated as 2 times of current size.

Next, check the `resize()` in GCC.
```cpp
/**
*  @brief  Resizes the %vector to the specified number of elements.
*  @param  __new_size  Number of elements the %vector should contain.
*
*  This function will %resize the %vector to the specified
*  number of elements.  If the number is smaller than the
*  %vector's current size the %vector is truncated, otherwise
*  default constructed elements are appended.
*/
void
resize(size_type __new_size)
{
    if (__new_size > size())
        _M_default_append(__new_size - size());
    else if (__new_size < size())
        _M_erase_at_end(this->_M_impl._M_start + __new_size);
}
```
We can see the `resize()` in GCC also do trimming and appending. (Interestingly, nothing happens when `__new_size` is equal to current size.) So, we should focus on `_M_default_append()` function.
```cpp
template<typename _Tp, typename _Alloc>
void
vector<_Tp, _Alloc>::
_M_default_append(size_type __n)
{
    if (__n != 0)
    {
        const size_type __size = size();
        size_type __navail = size_type(this->_M_impl._M_end_of_storage
                        - this->_M_impl._M_finish);

        if (__size > max_size() || __navail > max_size() - __size)
            __builtin_unreachable();

        if (__navail >= __n)
        {
            _GLIBCXX_ASAN_ANNOTATE_GROW(__n);
            this->_M_impl._M_finish =
                std::__uninitialized_default_n_a(this->_M_impl._M_finish,
                            __n, _M_get_Tp_allocator());
            _GLIBCXX_ASAN_ANNOTATE_GREW(__n);
        }
        else
        {
            const size_type __len =
                _M_check_len(__n, "vector::_M_default_append");
            pointer __new_start(this->_M_allocate(__len));
            if _GLIBCXX17_CONSTEXPR (_S_use_relocate())
            {
            __try
            {
                std::__uninitialized_default_n_a(__new_start + __size,
                    __n, _M_get_Tp_allocator());
            }
            __catch(...)
            {
                _M_deallocate(__new_start, __len);
                __throw_exception_again;
            }
            _S_relocate(this->_M_impl._M_start, this->_M_impl._M_finish,
                    __new_start, _M_get_Tp_allocator());
        }
        else
        {
            pointer __destroy_from = pointer();
            __try
            {
                std::__uninitialized_default_n_a(__new_start + __size,
                    __n, _M_get_Tp_allocator());
                __destroy_from = __new_start + __size;
                std::__uninitialized_move_if_noexcept_a(
                    this->_M_impl._M_start, this->_M_impl._M_finish,
                    __new_start, _M_get_Tp_allocator());
            }
            __catch(...)
            {
                if (__destroy_from)
                    std::_Destroy(__destroy_from, __destroy_from + __n,
                        _M_get_Tp_allocator());
                _M_deallocate(__new_start, __len);
                __throw_exception_again;
            }
            std::_Destroy(this->_M_impl._M_start, this->_M_impl._M_finish,
                _M_get_Tp_allocator());
        }
            _GLIBCXX_ASAN_ANNOTATE_REINIT;
            _M_deallocate(this->_M_impl._M_start,
                this->_M_impl._M_end_of_storage
                - this->_M_impl._M_start);
            this->_M_impl._M_start = __new_start;
            this->_M_impl._M_finish = __new_start + __size + __n;
            this->_M_impl._M_end_of_storage = __new_start + __len;
        }
    }
}
```
It is long one, too. What is `__navail` ? It seems meaning of `Number of AVAILable memory`, not the `Not AVAILable memory`. So, we can see the memory is reused when `if (__navail >= __n)` is true. Otherwise, reallocation happens. Oh, Hi. We meet `_M_check_len()` again. Then, new size will be 2 times of current size.

### *`Wrap-up`*

Common
- Try to recycle memory as possible as can. (e.g. reuse available memory in `push_back()` logic.)
- Care about underflow and overflow.
- Have internal indicators for `{First, Current, End}`
    - Currently allocated size = `End - First`
    - Currently used size = `Current - First`
    - Currently available size = `End - Current`

MSVC
- Growth happens with 3/2 times of amount. (in normal case)

GCC
- Growth happens with 2 times of amount. (in normal case)