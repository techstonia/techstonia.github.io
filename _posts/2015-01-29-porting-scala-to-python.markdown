---
layout:   post
title:    "Lessons from porting Scala to Python"
date:     2015-01-29 10:00:00
tags:     python, scala
subtitle:  "Lately I ported a substantial chunk of Scala into Python. This is a post about some of the pitfalls I encountered."
author: techstonia
permalink: porting-scala-to-python.html
---

Lately I had to port 16 thousand lines of Scala code into Python. The task seemed odd at first as usually the Python code is re-written in Scala and not the other way round. Anyway it's done now and I managed to pick up some Scala skills and also observed the difficulties of translating between those two languages. Before I start: I just want to mention that Scala is a very cool language and I hope to work with Scala again in the near future.

## 1. Key-value pairs in Python dictionaries and Scala maps have different order

Python doesn't preserve the order in which the elements are added to the dictionary. I think it's best to illustrate this with an example:
{% highlight python %}
>>> # let's make a dictionary
>>> d = {'key_a': 'value_a', 'key_b': 'value_b', 'key_c': 'value_c'}
>>>
>>> # and iterate over the key/value pairs
>>> for item in d.items():
...   print item
... 
('key_a', 'value_a')
('key_c', 'value_c')
('key_b', 'value_b')
{% endhighlight %}

Boom - all messed up! The same snippet in Scala:
{% highlight scala %}
scala> // let's make a map
scala> val m = Map("key_b" -> "value_b", 
     |             "key_c" -> "value_c", 
     |             "key_a" -> "value_a")
scala> 
scala> // and iterate over key/value pairs
scala> m.map(item => println(item))

(key_b,value_b)
(key_c,value_c)
(key_a,value_a)
{% endhighlight %}
So while Python sorts the dictionary elements in a non-obvious way, Scala preserves the order of elements as they were added to the map. Unfortunately I had to learn this the hard way - some parts of the code assumed the 'correct' order of elements. Fortunately it's easy to fix the problem with [OrderedDict](https://docs.python.org/2/library/collections.html#collections.OrderedDict).

## 2. Be careful when checking whether variable has a numerical value or not

For example this simple code snippet checks whether `x` is `NaN` or not

{% highlight scala %}
scala> val x = Double.NaN
scala>
scala> if(x.isNaN) {
            println("x has no value")
        } else {
            println("x has a value")
        }

x has no value
{% endhighlight %}

The equivalent in Python is:
{% highlight python %}
>>> x = None
>>> if not x:
...   print "x has no value"
... else:
...   print "x has a value"
... 
x has no value
{% endhighlight %}

So far so good. But sometimes one wants to take advantage of [NumPy](http://www.numpy.org/) to perform some calculations in a more elegant way. Thus it makes more sense to use NumPy's `NaN` value as well instead of None. I'm not saying that it's a mistake of language design or whatever, but it pays to be extra careful when writing something like this:  
{% highlight python %}
>>> import numpy as np
>>>
>>> x = np.NaN
>>> if not x:
...   print "x has no value"
... else:
...   print "x has a value"
... 
x has a value
{% endhighlight %}
Surely this was not what I meant!
A similar 'anomaly' might occur if the `x` equals to `0` and thus already has a value:
{% highlight python %}
>>> x = 0
>>> if not x:
...   print "x has no value"
... else:
...   print "x has a value"
... 
x has no value
{% endhighlight %}
It's very easy to overlook stuff like this when being on autopilot.

## 3. Porting classes with multiple constructors
Let's say we have a following Scala code
{% highlight scala %}
class Person(val firstName: String, val lastName: String) {
   
  def this(firstName: String) {
    this(firstName, "");
  }
   
  def this(firstName: String, lastName: String) {
    this(firstName, lastName);
  }
}
{% endhighlight %}
How to you port this into Python? This is actually a major pain in the ass since Python doesn't support multiple constructors. The only reasonable solution I was able to come up with was:
{% highlight python %}
class Person:
  def __init__(self, first_name, last_name):
    self.first_name = first_name
    self.last_name = last_name

  @staticmethod
  def create(*args):
    if len(args) == 1:
      return Person(args[0], None)
    elif len(args) == 2:
      return Person(*args)
    else:
      raise Exception("Wrong number of arguments: %s" % len(args))
{% endhighlight %}
And you use it like this:
{% highlight python %}
shrek = Person.create("Shrek")
hugh = Person.create("Hugh", "Jass")
{% endhighlight %}
Of course there might be a case where one constructor accepts a string and the other an integer or whatever. In this case it's probably the best just to check the type of a variable. Something like this: 
{% highlight python %}
>>> x = 5
>>> type(x) == int
True
>>> type(x) == str
False
{% endhighlight %}

## 4. Port first, refactor later
The solutions in the last paragraph are far from ideal, but at least we're not changing the logic of the code while porting it. Just don't refactor the code before having a great overview of what all of it is doing. Unless, of course, you want to have some unpleasant what-the-f-is-going-on debugging marathons. This is obviously not Scala-to-Python specific and applies to any kind of code porting.