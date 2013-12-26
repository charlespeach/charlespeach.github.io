---
layout: post
title:  "Code blocks, procs and internal DSL's in Ruby"
date:   2013-12-26 17:45:00
categories: ruby
---

## Intro

A week ago I watched the first part of the 2 day Owning Rails videos. I found that the material was a no-nonsense approach to teaching the core fundamentals of ruby and rails and I highly recommend it.

One of the things I found interesting, and I guess I didnt quite grasp until explained to me was the passing of blocks. This is my first 'official' blog post ever so bear with me.

See almost everything in ruby is an object.. Except our friend Mr Block. Blocks in ruby are the little un-assignable bits of logic you add to your ruby files.

## Lets get into it
We can pass code to be executed within any method we define in ruby, take this simple example:

``` ruby
def do_something
  yield if block_given?
end

do_something { puts 'hello!' }
#=> prints 'hello!' to STDOUT.
```

We can implicitly pass a block to our `do_something` method because it yields the block. What if we wanted to keep the block that was passed to our `do_something` method and use it? Well with our current `do_something` method, we cant! Ampersand (`&`) to the resque!

``` ruby
def do_something(&block)
  block.call if block
end

do_something { puts 'hello!'}
#=> prints 'hello!' to STDOUT.
```

Ok, so whats going on here? Well this special keyword `&` in our method definition tells ruby that any code blocks passed in should be wrapped in a `Proc`. Once inside the method we can use the `call` method on our new proc object to execute the code.

So thats cool but what can we do with new found ability to pass blocks around like any other typical variable? Take a look at this very familiar DSL all rails developers use when deifing their routes in Rails:

``` ruby
RailsAppName::Application.routes.draw do
  match '/users' => 'users#index'
end
```

Lets try create a simple replication of this DSL in plain ruby:

``` ruby
class Router
  def initialize
    @routes = {}
  end

  def match(route)
    @routes.update route
  end

  def routes(&block)
    instance_eval(&block)
    yield
    puts @routes
  end
end
```

So whats happening inside the `routes` method?

1. `routes` takes in a block, which gets converted to a proc using the `&` keyword.
2. It is then converted back into a block when being passed to instance_eval.
3. The 'match' method gets called within this instance instead of the root context.

Now lets try using it!

``` ruby
Router.new.routes do
  match '/users' => 'users#index'
  match '/login' => 'sessions#new'
end
```

And we get a nice hash back of routes.

`{"/users"=>"users#index", "/login"=>"sessions#new"}`

## Wrap up

We have learnt that when you explicitly pass a block to a method that uses the `&` keyword, the ruby interpreter converts it into a block. Once turned into a proc we can pass it around as we please.

We have also learnt that we can execute code within the context of the methods class using `instance_eval` and passing a block to it using the `&` keyword, which converts the proc back into a block.
