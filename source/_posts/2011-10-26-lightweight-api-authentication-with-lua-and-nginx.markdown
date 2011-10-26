---
layout: post
title: "Lightweight API Authentication With Lua and NGiNX"
date: 2011-10-26 10:18
comments: true
categories: [NGiNX, Lua]
---

I recently was looking for a way to easily handle API authentication with as little overhead as possible. So first a little background. At [Scan](http://scan.me) we are following a service oriented structure, where each service defines and maintains its own API endpoints. The "API" is in fact nothing more than a glorified proxy, that sends back the request to the appropriate service which then handles the logic. This is all handled through NGiNX. More on the pros and cons of that structure in a future post. That being said, my goals were simple:

> * Little to no long term code maintinence. I want to be able to forget it is even there.
> * Fast. After all, it is authenticating every API request.
> * I wanted it to behave almost like a Ruby on Rails *before_filter* call. It should authenticate the key before the API endpoints are actually even hit.

Enter the [lua-nginx-module](https://github.com/chaoslawful/lua-nginx-module). Lua is an extremely light weight fast scripting language made popular by its ability to be embedded in almost anything, [even in other scripting languages](https://github.com/glejeune/ruby-lua). In this case, the Lua runtime can be compiled directly into NGiNX. The lua-nginx-module conveniently adds an NGiNX configuration option called [access_by_lua](http://wiki.nginx.org/HttpLuaModule#access_by_lua) which seems to fit all of the above requirements nicely:

> * Little or no maintinence - the entire authentication script run by NGiNX is a total of less than 70 lines of code (with comments and logging removed), and depending on requirements could be done in less.
> * Fast - Lua is on average [five times faster than ruby](http://shootout.alioth.debian.org/u32/benchmark.php?test=all&lang=yarv&lang2=lua), and is embedded in the actual webserver. It also runs concurrently. Each NGiNX worker gets it's own Lua runtime.
> * Prevents access before it hits the first endpoint - The access_by_lua acts in the same way as a simple basic authentication directive would. It will stop the call from ever continuing to the API endpoints if authentication fails.

The best thing is that it is flexible. How the lua authenticates is entirely up to the developer. It can read from a file, call a back end service, or even read directly from a database. It is entirely up to you.