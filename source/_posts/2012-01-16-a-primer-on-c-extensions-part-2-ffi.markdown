---
layout: post
title: "A Primer on Ruby C Extensions Part 2 - FFI"
date: 2012-01-16 10:44
comments: true
categories: c ruby ffi
---

In [my previous post](/blog/2012/01/10/a-primer-on-ruby-c-extensions/) I covered building a simple C extension written in Ruby. There are times however that you may need to call functions defined in an existing native library. Fortunately there is a tool precisely for this job. Enter FFI, or Foreign Function Interface. As defined by [Wikipedia](http://en.wikipedia.org/wiki/Foreign_function_interface) *"A foreign function interface (or FFI) is a mechanism by which a program written in one programming language can call routines or make use of services written in another"*. Thanks to the team working on [Ruby FFI](https://github.com/ffi/ffi) we have a relatively easy set of tools to work with.

First let say I ther is already a native library available that has an already defined method for determining if a number is prime. Lets call this *libprime.so*. Tying into this library is actually quite simple using Ruby FFI. The first step obviously is installing it. `gem install ffi` should do the trick. Now we need to take a look at the code defined in *libprime.so*.

```c
#include <stdlib.h>
#include <stdint.h>
#include <math.h>

int prime(register int n)
{
  
  if (n == 2)
    return 1;
  if ((n & 1) == 0)
    return 0;

  register uint64_t sqrt_n = ((uint64_t)sqrt(n)) + 1;
  register uint64_t i=3;
  for (i; i<= sqrt_n; i+=2) {
    if (n % i == 0)
      return 0;
  }
  return 1;
}
```
As you can see the method `prime` is made available. So our next step is to tie into the library using FFI.

```ruby
require 'rubygems'
require 'ffi'
module Primed
  extend FFI::Library
  ffi_lib "libprime.so"
  attach_function :prime, [:int], :int
  
  def prime?
    return false if prime(self) == 0
    true
  end
  
end
```

Looks simple enough right? Let me quickly explain what this is doing. Our first task is obviously to make FFI available to us. We call `extend FFI::Library` and that gives us all the tools that we need to work with. Second, we need to define the library to tie into, in this case *libprime.so*. Third, we call `attach_function`. This takes 3 arguments, the name of the library's method to call, an array of arguments that the method will take, and the data type the method returns. Finally, we wrap the method to our liking and we're done. For more information check out the [FFI Wiki](https://github.com/ffi/ffi/wiki) over at Github.