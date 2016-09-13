---
layout: post
---

In my opinion, Ruby is the most elegant and readable language these days.
I really like it, but sometimes it can blow your brain by some magic.

So, Rails makes it worse. Sometimes expressions are pretty obscure and
misunderstandable to new Rails-developers.

How to learn Rails and also understand it? The answer is just learn Ruby better.
In this article I'll show you the most Rails syntax tricks.

### Magic Rails keywords

First and foremost, it is not keywords, it is just class methods.
And it looks like this:

{% highlight ruby %}

  class MyClass
    validate :name

    def self.validate(what, options = {})
      # method logic goes here
    end
  end

{% endhighlight %}

- - -

### Hash without curly brackets

Another strange thing in Ruby/Rails, that looks concise, but confuses. When you
pass a hash as a last argument to a method, you can omit curly brackets.

{% highlight ruby %}

  def my_method(str, hash = {})
    # method logic goes here
  end

  my_method("My method in use", my: "method", in: "use")

{% endhighlight %}

- - -

### Lambda vs Blocks

Ruby is flexible and elegant language. It has many killer features, and two of
these is blocks and lambdas. There are some difference between it:
  * Lambda works like a method
  * Blocks works like a part of a method
