---
---

## Dictionaries

Lists are useful when you need to access elements by their position in a
sequence. In contrast, *dictionaries* make it easy to find values based on unique
identifiers called *keys*.

A dictionary is constructed by surrounding a comma-separated sequence of `key:value` pairs in curly brackets. Individual values are accessed using square brackets, as for indexing into lists. In a `dict`, however the keys are the indices and the values cannot be accessed by their integer position.


~~~python
animals = {'Snowy':'dog', 'Garfield':'cat', 'Bugs':'rabbit'}
~~~
{:.text-document title="worksheet.py"}



~~~python
>>> animals['Bugs']
'rabbit'

~~~
{:.output}



===

To add a single new element to the dictionary, define a new `key:value` pair by assigning a value to a novel key in the dictionary.


~~~python
animals['Lassie'] = 'dog'
~~~
{:.text-document title="worksheet.py"}



~~~python
>>> animals
{'Garfield': 'cat', 'Lassie': 'dog', 'Bugs': 'rabbit', 'Snowy': 'dog'}

~~~
{:.output}



===

The keys of a dictionary are unique. Assigning a value to an existing key would
overwrite its previously associated value. As you can also see from the example
above, the order in which Python returns dictionary elements is arbitrary.

===

## Exercise 3

Based on what we have learned so far about lists and dictionaries, think up a data structure suitable for an address book. Using what you come up with, store the contact information (i.e. the name and email address) of three or four (hypothetical) persons as a variable `addr`.

===

## Dictionary methods

The `update()` method allows you to extend a dictionary with multiple new `key:value` pairs, or simultaneously overwrite existing ones.


~~~python
animals.update(
  Tweety='bird',
  Bob='sponge',
  )
~~~
{:.text-document title="worksheet.py"}



===

Note a couple "pythonic" style elements of the above:

1. Leave no space around the `=` when using keyword arguments.
1. Trailing null arguments are syntactically correct, even advantageous.
1. White space between `(` and `)` is ignored.

===

An alternative method of calling the `update()` method reveals the full
flexibility of dictionaries. The keys can be any object, and different types of keys
can coexist in a single dictionary.


~~~python
animals.update({
  3.14:'pie',
  u'\U0001F98A':'Mr. Fox',
  })
~~~
{:.text-document title="worksheet.py"}

