---
layout: post
title: "A Primer on Ruby C Extensions"
date: 2012-01-10 14:23
comments: true
categories: ruby c
---

**NOTE:** This is a repost of an older blog post of mine from another [Blog](http://www.pmamediagroup.com/2010/07/making-superfast-things-in-ruby-using-c-extensions/)

I will address one of the primary uses for a C extension in Ruby, speed. Due to it’s very nature, Ruby is slow (as compared to compiled languages like C). It gets the job done, but sometimes it takes it’s sweet time doing it. Sometimes it is necessary to speed things up a bit, and here enter C extensions. There are several methods of implementing extensions, from the generic C extension, to ruby-inline. In this particular article I will focus on the generic C extension.

In this example, I am going to use a fairly inefficient piece of Ruby code I created a while ago for Project Euler (Problem 10) for finding the sum of all primes under 2000000:

```ruby
class Integer
 
  def prime?
    return true if self == 2
    return false if (self & 1) == 0
    square = Math.sqrt(self).round + 1
    i = 3
    while i <= square
      i+= 2
      return false if (self % i) == 0
    end
    true
  end
 
end
 
numbers = (2..2000000).to_a
 
numbers = numbers.select { |n| n.prime? }
puts numbers.inject { |result, element| result = element + result }
```

**UPDATE**: Changed `i` on line 7 from `1` to `3`, as otherwise it will cause `3` and `5` to return `false`.

At the time that I wrote this, I was relatively unaware of more efficient ways of resolving prime numbers (such as a euler sieve), however the code still ran under the allotted 2 minute window (52 seconds) so I went with it. Now to speed it up. To write a C extension you need, at a bare minimum two things:

an extconf.rb file – this file is used by ruby to generate the Makefile that is used to compile the extension
the source file for the extension (in this case primed.c)
Here is a look at these two files for my new version of problem 10:

*primed.c*

```c
#include "ruby.h"
#include <stdlib.h>
#include <math.h>
 
VALUE Primed;
 
VALUE method_prime(VALUE obj, VALUE args)
{
  register uint64_t n;
  n = NUM2INT(obj);
  if (n == 2)
    return Qtrue;
  if ((n & 1) == 0)
    return Qfalse;
 
  register uint64_t sqrt_n = ((uint64_t)sqrt(n)) + 1;
  register uint64_t i=3;
  for (i; i<= sqrt_n; i+=2) {
    if (n % i == 0)
      return Qfalse;
  }
  return Qtrue;
}
 
void Init_primed() {
  Primed = rb_define_module("Primed");
  rb_define_method(Primed, "prime?", method_prime, -2);
}
```

*extconf.rb*

```ruby
# Loads mkmf which is used to make makefiles for Ruby extensions
require 'mkmf'
 
# Give it a name
extension_name = 'primed'
 
# The destination
dir_config(extension_name)
 
# Do the work
create_makefile(extension_name)
```

First let me explain primed.c. The objective of this extension is to determine whether or not a number is prime, so that an integer can call `x.prime?` and return `true` or `false`. It is essentially identical to the method used in the pure ruby script above. One of the first thing you may notice is this line:

```c
VALUE Primed
```

`VALUE` is a data type defined by Ruby that represents the Ruby object in memory. It is basically a struct that contains the data related to the object. In this case, the object will represent the `Primed` module in ruby, so it will contain data about the instance methods, variables, etc. for that module. All Ruby objects are represented in C by `VALUE`, regardless of their type within the Ruby VM, anything else will likely result in a segfault.

Next we define the actual method to calculate whether the value is prime. Note that because we need to return a Ruby object, we set the return type as `VALUE` as well. `QTrue` and `QFalse` are directly representative of true and false in Ruby, and also return correctly within C (`QTrue` will evaluate as `true`, `QFalse` will evaluate as `false`).

Finally we see the `Init_primed method`. Every time a class or module is instantiated within the Ruby VM it calles `Init_name`. It is here we actually instantiate the `Primed` module and bind the `method_prime` function to the Ruby method `prime?`. Both functions used are pretty self explanatory as to what they do, except for the last argument used in `ruby_define_method` which is essentially the arity or number of arguments to expect in the Ruby method. In this case, `-2` actually causes Ruby to send back `self` as the first argument to the `method_prime` function, and an array of any other arguments as the second.

Now we have all of our code. The last thing to put in place is extconf.rb:

```ruby
# Loads mkmf which is used to make makefiles for Ruby extensions
require 'mkmf'
 
# Give it a name
extension_name = 'primed'
 
# The destination
dir_config(extension_name)
 
# Do the work
create_makefile(extension_name)
```

Pretty simple right? Now when you call ruby extconf.rb it will generate a Makefile that you can use to build the extension. And the final result? Using the C extension the code runs in just under 3 seconds. Still not really efficient, but it demonstrates the point. When Ruby’s speed is the bottle neck, using C is a viable and easy option.

[Go to Part 2 - FFI](/blog/2012/01/16/a-primer-on-c-extensions-part-2-ffi/)