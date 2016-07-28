---
layout: post
title: "Generalized Wrapper with Universal Reference Overloads"
description: ""
category: 
tags: ['c++']
---
{% include JB/setup %}

This is a follow up on Niebler's article on [Overloading with Universal References](http://ericniebler.com/2013/08/07/universal-references-and-the-copy-constructo/).

The simple `wrapper` implementation can be extended to account for the following desirable use case:

```cpp
wrapper<const char*> w1("abc"); 
wrapper<std::string> w2(w1);       // wrap a string which is constructed from raw char* in w1
wrapper<const std::string> w3(w1); // wrap a string which is copy-constructed from that in w2
```

In order to make this work, we need an explicit copy constructor which accounts for different compatible instances of the `wrapper` template:

```cpp
template<typename T>
struct wrapper
{
    T value;
    
    // Primary constructor
    template<typename U>
    wrapper( U && u )
      : value( std::forward<U>(u) ) {}

    // Copy constructor accepting other instances of the template
    template<typename U>
    wrapper( wrapper<U> const & w )  // Suspicious overload!
      : value( w.value ) {}
};
```

This naive approach fails for exactly the same reason explained in Niebler's article.
Against our intention, the primary constructor resolves to be a better match 
for an lvalue non-const reference arguement, even if that arguments is a `wrapper<U>`.

We can use a technique similar to Niebler's, this time with a slightly more advanced disabler:

```cpp
template<typename T>
struct wrapper
{
    T value;
    
    template<typename U, typename = disable_if_same_template<T,U>>
    wrapper( U && u )
      : value( std::forward<U>(u) ) {}

    template<typename U>
    wrapper( wrapper<U> const & w ) 
      : value( w.value ) {}
};
```

`disable_if_same_template` has two parts:


```cpp


```

