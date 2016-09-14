---
layout: post
---

When I was eleven, I thought that it would be cool if I could invent a program,
that could write another program. So, now we have many languages, which can
create new parts of code at runtime.

Although, it isn't kinda artificial intelligence. It's a metaprogramming.
Metaprogramming in Ruby is just an ability to write a code that can operate
syntax constructions to create new code. That's it.

Ruby has many build-in methods to manipulate with program constructions and
it isn't a piece of cake to learn it all at once. It would be better to start
with the widely speared metamethods.

Lets discuss class_eval and instance_eval - basic parts of Ruby's meta.

### class_eval

A cool tool to open a class and to add new methods or  evaluate code in a context
of the current class. In other words, it changes self to the class. Use it like
this:

{% highlight ruby %}

  class MyClass
    # a class guts
  end

  MyClass.class_eval do
    def my_method
      puts "I'm inside!"
    end
  end

  obj = MyClass.new
  obj.my_method #=> "I'm inside!"

{% endhighlight %}

You may say, that it's kinda strange. Yes, it is. Just remember it.

### instance_eval

There is an another Kernel method which we can use: instance_eval. It opens
object and evaluates the code inside it. Access to instance variables,
methods - all of this stuff.

{% highlight ruby %}

  class MyClass
    # a class guts
  end

  obj = MyClass.new
  obj.instance_eval do
    def my_method
      puts "I'm inside!"
    end
  end

  obj.my_method #=> "I'm inside!"

{% endhighlight %}

Yeah :) Although, I must notice that you can use `my_method` only in the `obj`.


Of course, there are many metamethods in Ruby, but these are the most useful.
You can create new features via these methods. For instance, you can transform
this:

{% highlight ruby %}

  class MyClass
    def initialize(name, title, amount_of_kittens)
      @name   = name
      @title  = title
      @amount_of_kittens = amount_of_kittens
    end
  end

{% endhighlight %}

to this:

{% highlight ruby %}

  class MyClass
    init :name, :title, :amount_of_kittens
  end

{% endhighlight %}

via

{% highlight ruby %}

  class Class
    def init(\*args)
      params = args.join(', ')
      body   = args.map { |arg| "@#{arg}=#{arg}" }.join('; ')
      self.class_eval "def initialize(#{params}); #{body}; end;"
    end
  end

  class MyClass
    init :name, :title
  end

{% endhighlight %}

As you can see, it makes your code more concise. Use this poser careful, because
it can makes your ruby application less obvious and more confusing. With great
power, comes great responsibility.
