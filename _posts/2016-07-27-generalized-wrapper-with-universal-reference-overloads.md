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

{% highlight cpp %}
wrapper<const char*> w1("abc"); 
wrapper<std::string> w2(w1);  // wrap a string which is constructed from raw char* in w1
// or perhaps more relatable
wrapper<const std::string> w3(w1); // wrap a string which is copy-constructed from that in w2
{% endhighlight %}

In order to make this work, we need an explicit copy constructor which accounts for different compatible instances of the `wrapper` template.
The naive approach fails:

{% highlight cpp %}
template<typename T>
struct wrapper
{
    T value;
    template<typename U>
    wrapper( U && u )
      : value( std::forward<U>(u) ) {}

	// The explicit copy constructor for accepting other instances of the template
    template<typename U>
    wrapper( wrapper<U> const & w )
      : value( w.value ) {}
};
{% endhighlight %}

This fails for exactly the same reasons outlined in Niebler's article. So we need a SFINAE tactic, with a slightly more advanced disabler.



